# `geo-seo-claude` 业务流程深度调研报告

> **报告范围:** 仓库 `geo-seo-claude` 的产品定位、架构、业务流程、评分实现、商业化链路、工程取舍
> **报告日期:** 2026-05-09
> **分析方法:** 静态代码与提示词审阅(14 个 Skill / 5 个 Agent / 6 个 Python 脚本 / 6 个 JSON-LD 模板)

---

## 0. TL;DR(给经理看的 8 行)

1. 这是一个**面向 GEO(Generative Engine Optimization)代理服务商的 Claude Code Skill 套件**,把"网站 AI 可见度审计 → 客户报告 → 销售报价 → 月度续约证据"这条代理公司的完整业务链装进 14 个 `/geo` 斜杠命令。
2. 架构分三层:**主调度器(`geo/SKILL.md`)→ 14 个 Skill + 5 个 Agent → 6 个 Python 工具脚本**,以 Markdown 提示词为主、Python 为辅。
3. 核心交付物是一个 **0–100 的 GEO Score**,由 6 个加权类目合成(Citability 25% / Brand 20% / E-E-A-T 20% / Technical 15% / Schema 10% / Platform 10%)。
4. 商业化由三个 Skill 串成销售漏斗:`/geo prospect`(CRM)→ `/geo proposal`(自动按分推荐 €2.5K / €5K / €9.5K 三档报价)→ `/geo compare`(月度 delta 报告做续约证据)。
5. 评分是 **LLM-as-judge** 模式——提示词把开放分析拆成 rubric,Claude 用 `WebFetch` 取证后按规则打子分,再加权。**没有传统意义上的"评分算法"**。
6. 客户数据持久化在 `~/.geo-prospects/`(JSON + Markdown 归档),还附带 Flask 网页 UI 与 rich CLI dashboard。
7. 主要工程化弱点:rubric 子分上限不一致、Wikipedia / Reddit 检测靠 WebFetch 不调 API、缺少缓存、同站审计有 ±5 分波动。
8. 商业模型显式与 Skool 社区联动——工具开源,**销售 playbook 收费**;€2.5K–€9.5K/月的报价定式直接写进了 `/geo proposal` 的代码逻辑。

---

## 1. 项目定位与商业价值

### 1.1 产品口号

`README.md:6` 一句话定位:**"GEO-first, SEO-supported. Optimize websites for AI-powered search engines while maintaining traditional SEO foundations."**

它刻意区别于传统 SEO 工具(Ahrefs / Semrush 等):后者优化 Google 排名,这套工具优化"被 AI 模型引用的概率"。

### 1.2 市场叙事(写进了提示词)

`geo/SKILL.md:46-59` 把以下数据点直接埋进了主 Skill 提示词,所有生成的报告/提案会自动带上同样的市场说辞:

| 市场指标 | 数值 |
|----------|------|
| GEO 服务市场(2025) | $850M–$886M |
| 预测市场(2031) | $7.3B(34% CAGR) |
| AI 引用流量增长 | +527% YoY |
| AI 流量转化率 vs 传统自然流量 | 4.4× |
| Gartner 预测 2028 年传统搜索流量 | -50% |
| 投资 GEO 的营销人员比例 | 仅 23% |
| 品牌提及 vs 反向链接(对 AI 影响) | 3× 强相关 |

这些数据点既是**用户教育素材**,也是**销售提案的修辞武器**——`/geo proposal` 生成的提案直接复用这些数据(`skills/geo-proposal/SKILL.md:118-127`)。

### 1.3 显式的商业化定位

`README.md:241-253` 直接说明:工具免费,但变现知识在 **Skool 付费社区**(AI Workshop Community)里。GEO 代理公司收费区间 **$2K–$12K/月**。这意味着此仓库是社区的"工具样品",而非纯开源生产力工具。

---

## 2. 整体架构

### 2.1 三层 + 持久化

```
┌─────────────────────────────────────────────────────────────┐
│ 入口层:geo/SKILL.md(主调度器,定义所有 /geo <cmd> 命令)    │
└──────────────────────┬──────────────────────────────────────┘
                       │ 命令路由
        ┌──────────────┴──────────────┐
        ▼                              ▼
┌────────────────────┐        ┌────────────────────┐
│ Skills(14 个)     │        │ Agents(5 个,并行)│
│ skills/geo-*/      │  ◀───▶ │ agents/geo-*.md    │
│ 单一职责能力包      │        │ 并行 worker         │
└──────────┬─────────┘        └──────────┬─────────┘
           │ Bash 调用                    │
           ▼                              ▼
┌─────────────────────────────────────────────────────────────┐
│ 工具层: scripts/*.py(打分/抓取/PDF/CRM)                     │
│        + schema/*.json(6 套 JSON-LD 模板)                  │
└─────────────────────────────────────────────────────────────┘
                       │ 写入
                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 持久化:~/.geo-prospects/                                    │
│   ├── prospects.json     (CRM 主数据,JSON 单文件)           │
│   ├── audits/            (审计快照,Markdown)                │
│   ├── proposals/         (生成的提案,Markdown)              │
│   └── reports/           (月度 delta 报告,Markdown)         │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Skill / Agent / Script 数量与体量

| 类别 | 数量 | 总行数 | 备注 |
|------|------|--------|------|
| 主 Skill(`geo/SKILL.md`) | 1 | 234 | 命令路由 + 市场叙事 |
| 子 Skill(`skills/*/SKILL.md`) | 14 | ~4928 | 单一职责能力包 |
| 子 Agent(`agents/*.md`) | 5 | ~1400 | 并行执行体 |
| Python 脚本(`scripts/*.py`) | 6 | 2670 | 含 PDF 生成、Flask Web UI、CLI dashboard |
| JSON-LD 模板(`schema/*.json`) | 6 | — | 按业务类型选用 |

**关键观察:** Markdown 提示词总量是 Python 代码量的约 2.4 倍。**业务规则主要写在提示词里,Python 只承担确定性计算**(打分细节、HTML 抓取、PDF 渲染、CRM 持久化)。

### 2.3 安装与隔离

`install.sh` 把整个工具装进独立 venv:

```
~/.claude/skills/geo/.venv/    ← 独立 Python 环境
~/.claude/skills/geo-*/        ← 14 个子 Skill
~/.claude/agents/geo-*.md      ← 5 个 Agent
```

Skill / Agent 中的 `python3` 引用都被替换成 venv 内的解释器路径(`install.sh:21-26`),保证不污染系统 Python。

**关键设计:** CRM 数据放在 `~/.geo-prospects/` 而非 `.claude/skills/` 之下(`docs/architecture.md:67-74`)——这是为了让 `uninstall.sh` 不会清除客户数据。卸载时需要手动删除该目录。

---

## 3. 核心业务流程:三条主线

### 主线 A:技术分析流(13 个分析命令)

以 `/geo audit <url>` 为旗舰,代表整个产品的技术能力。流程严格三阶段(`skills/geo-audit/SKILL.md:25-148`):

#### Phase 1 — Discovery(串行,~30 秒)

1. `WebFetch` 抓主页
2. 业务类型识别(SaaS / Local / E-com / Publisher / Agency / Hybrid)
3. 抓 `sitemap.xml`,最多取 50 页;无 sitemap 则爬主页内链 2 层

业务类型识别用以下信号(`skills/geo-audit/SKILL.md:43-52`):

| 业务类型 | 关键信号 |
|---------|----------|
| SaaS | Pricing / Sign up / Free trial / app.domain / API docs |
| Local | 物理地址、电话、Google Maps embed、LocalBusiness schema |
| E-commerce | 购物车、Add to cart、价格元素、Product schema |
| Publisher | 博客 nav 重、Article schema、作者页、RSS |
| Agency | 案例集、Our Work、客户 logo、团队页 |

业务类型决定后续每个 Skill 的**评分权重微调**与**推荐 schema 选择**——例如 SaaS 加重 Comparison Table 权重,Local 加重 NAP 一致性。

#### Phase 2 — Parallel Analysis(并行,~2 分钟)

5 个 subagent 同时启动,各自调用对应的 sub-skill:

| Agent | 调用的 Skill | 输出维度 |
|-------|-------------|---------|
| `geo-ai-visibility` | citability + crawlers + llmstxt + brand-mentions | 引用度 / 爬虫可达 / llms.txt / 品牌提及 |
| `geo-platform-analysis` | platform-optimizer | 5 个 AI 平台分(AIO / ChatGPT / Perplexity / Gemini / Copilot) |
| `geo-technical` | technical | 8 类技术 SEO(crawl / index / SSR / CWV / security / mobile / headers / sitemaps) |
| `geo-content` | content | E-E-A-T 4 维 + 内容结构 + 主题权威 |
| `geo-schema` | schema | JSON-LD / Microdata / RDFa 检测 + 验证 + 生成 |

#### Phase 3 — Synthesis(串行)

合成 6 类目加权总分 → 生成 `GEO-AUDIT-REPORT.md`。

#### 评分公式

```
GEO_Score = 0.25·Citability + 0.20·Brand + 0.20·EEAT
          + 0.15·Technical + 0.10·Schema + 0.10·Platform
```

权重显著向 **AI Citability + Brand Authority** 倾斜(45%),这是产品定位"GEO-first"的代码体现——传统 SEO 工具会把 Technical 拉到 30%+。

#### 分数解读表(`skills/geo-audit/SKILL.md:140-148`)

| 分数 | 评级 | 含义 |
|------|------|------|
| 90–100 | Excellent | 顶级 GEO 优化,极有可能被 AI 引用 |
| 75–89 | Good | 强基础,有提升空间 |
| 60–74 | Fair | 中等存在,显著优化机会 |
| 40–59 | Poor | 信号弱,AI 难以引用 |
| 0–39 | Critical | 几乎对 AI 不可见 |

这个分段表是**整个商业化链路的锚点**——分数直接驱动 `/geo proposal` 的报价档位选择(见 §5.2)。

---

### 主线 B:商业化流(销售漏斗三件套)

这是把"工具"变成"代理生意"的关键。三个 Skill 串成完整销售漏斗:

```
              ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
新发现的域名 → │ /geo prospect    │ →  │ /geo proposal    │ →  │ /geo compare     │
              │ (CRM 管线)       │    │ (按分自动报价)    │    │ (月度续约证据)    │
              └──────────────────┘    └──────────────────┘    └──────────────────┘
              lead → qualified         Basic €2.5K            baseline vs current
              → proposal               Standard €5K           ▲/▼/── delta 表
              → won / lost             Premium €9.5K          6 个月轨迹
```

#### B.1 `/geo prospect` — CRM-lite

`skills/geo-prospect/SKILL.md` 完整定义了一个轻量 CRM:

- **数据模型**:`~/.geo-prospects/prospects.json`,每条记录含 `id / company / domain / status / geo_score / monthly_value / contract_months / notes[]`
- **管线状态**:`lead → qualified → proposal → won / lost`
- **命令面**:`new / list / show / audit / note / status / won / lost / pipeline`
- **额外 UI**:除了 CLI,仓库还附带 `scripts/crm_dashboard.py`(rich 框架的终端 dashboard)和 `scripts/webapp/app.py`(Flask + HTMX 的 Web UI,端口 5050)

#### B.2 `/geo proposal` — 按分自动报价

**核心规则**(`skills/geo-proposal/SKILL.md:339-345`):

| GEO Score | 推荐档位 | 月费 | 逻辑 |
|-----------|---------|------|------|
| 0–40 | **Premium** | €9,500 | 关键问题,需密集介入 |
| 41–60 | **Standard** | €5,000 | 显著缺口,月度优化 |
| 61–75 | **Basic** | €2,500 | 基础扎实,维护监控 |
| 76+ | Basic 或季度回访 | — | 仅作健康检查 |

**关键观察:得分越低,推荐报价越贵**。这是把"诊断结果"直接变成"销售杠杆"的产品设计——审计报告的痛点描述越严重,客户对 Premium 档的接受度越高。

提案模板(`skills/geo-proposal/SKILL.md:80-322`)包含:执行摘要、市场机会、当前定位对比表、6 类目得分明细、关键问题(红色图标)、3 档套餐对比、ROI 投影、6 个月路线图、合同条款。**是一份完整的商业提案**,不是技术报告。

#### B.3 `/geo compare` — 月度 delta

设计意图(`skills/geo-compare/SKILL.md:18-21`):**"The single most powerful retention tool for a GEO agency: show clients exactly what improved since they started working with you. Every point gained on the GEO score is proof of value."**

实现机制:

1. 用 `Glob` 匹配 `~/.geo-prospects/audits/<domain>-*.md`,按日期排序
2. 取最旧 vs 最新两份,parse 6 类目分数
3. 计算 delta,标注 ▲(改善)/▼(下降)/── (无变化)
4. 输出 `~/.geo-prospects/reports/<domain>-monthly-<YYYY-MM>.md`

报告核心要素:

- **进度条**:文本艺术 `[▓▓▓▓░░░░] 32→44` 直观对比
- **6 类目 delta 表**:Before/After/Change/Trend
- **5 平台 delta 表**:每个 AI 平台的提升
- **14 个爬虫状态变化表**:哪些从 Block 改为 Allow
- **行动项进度表**:`✅ Done / 🔄 In Progress / ❌ Not started / 📋 Planned`
- **6 个月轨迹表**:已经发生的月份填实数,未来填 `—`

这套报告设计的目标是**让客户每个月看到分数往上涨**——一个 32→44 的提升就是续约的最强证据。

---

### 主线 C:交付物生产流

| 产物 | 命令 | 输出位置 | 用途 |
|------|------|---------|------|
| 全审计(技术) | `/geo audit` | `./GEO-AUDIT-REPORT.md` | 内部参考 |
| 客户报告(MD) | `/geo report` | `./GEO-CLIENT-REPORT.md` | 交付 |
| 客户报告(PDF) | `/geo report-pdf` | `./GEO-REPORT.pdf` | 终极交付 |
| 销售提案 | `/geo proposal` | `~/.geo-prospects/proposals/<domain>-proposal-<date>.md` | 销售 |
| 月度报告 | `/geo compare` | `~/.geo-prospects/reports/<domain>-monthly-<YYYY-MM>.md` | 续约 |
| llms.txt | `/geo llmstxt` | `./llms.txt` | 直接部署 |
| JSON-LD | `/geo schema` | 多种,根据业务类型 | 网站修复 |
| 引用度报告 | `/geo citability` | `./GEO-CITABILITY-SCORE.md` | 内部 |
| 爬虫报告 | `/geo crawlers` | `./GEO-CRAWLER-ACCESS.md` | 内部 |

PDF 由 `scripts/generate_pdf_report.py` 用 ReportLab 生成,共 931 行,含分数仪表盘、平台横向条形图、爬虫表(色彩编码 Allow/Block)、关键发现按严重度归类、行动计划三档(Quick Wins / Medium-Term / Strategic)。

---

## 4. 评分体系深度分析

### 4.1 总体设计:LLM-as-Judge,不是确定性算法

整个评分系统的本质是**一份精心结构化的 LLM 工作说明书**:

```
[Agent 提示词]   ← 定义 7 步执行流水线(获取顺序、聚合方式)
     +
[Skill 提示词]   ← 定义 N 项 rubric(每项满分、判定锚点)
     +
[Python 脚本]    ← 处理"必须确定性"的部分(HTML 抓取、字数统计、JSON-LD 解析)
     +
[Claude 自身]    ← 把"主观判断"塞进 rubric 框架内,输出子分加和
```

为什么这样设计?——因为 GEO 信号(Wikipedia 是否存在、Reddit 是否有高质量讨论、内容是否"可引用")**根本无法被纯规则化检测**,但完全主观又会失去可重复性。**用 rubric 约束 LLM 的判断空间**是这套系统的核心创新。

### 4.2 6 大评分类目实现矩阵

| 类目 | Skill | 满分维度数 | 取证手段 | LLM 判断比例 |
|------|-------|----------|---------|-------------|
| AI Citability | `geo-citability` | 5 维(answer / containment / structural / statistical / uniqueness) | `scripts/citability_scorer.py`(可选)+ LLM | 高 |
| Brand Authority | `geo-brand-mentions` | 11 个平台加权(YouTube 0.737 最强) | `scripts/brand_scanner.py` | 中 |
| Content E-E-A-T | `geo-content` | 4 维 ×25 分(Experience / Expertise / Authoritativeness / Trustworthiness) | LLM 主观 + 信号清单 | 极高 |
| Technical | `geo-technical` | 8 类目(crawl / index / SSR / CWV / security / mobile / headers / sitemaps) | `curl` + WebFetch + 头检测 | 中 |
| Schema | `geo-schema` | JSON-LD 检测 + Schema.org 验证 + 业务类型必备清单 | `scripts/fetch_page.py`(WebFetch 会丢 head) | 低 |
| Platform Optimization | `geo-platform-optimizer` | 5 平台 ×10 项 checklist | LLM + WebFetch(robots.txt) | 高 |

### 4.3 案例:ChatGPT Web Search 评分(完整推演)

以 `electron-srl.com`(`examples/` 中已有的样例)为例,完整走一次 ChatGPT 平台分计算。

#### 子类 A:Entity Recognition(满分 35)

`agents/geo-platform-analysis.md:47-51` + `skills/geo-platform-optimizer/SKILL.md:91-99`

| 信号 | 验证方法 | 标尺 | 假设结果 |
|------|---------|------|---------|
| Wikipedia 页面存在? | `WebFetch en.wikipedia.org/wiki/Electron_Srl` | 完整 20 / stub 10 / 无 0 | 0 |
| Wikidata 实体存在? | `WebFetch wikidata.org/...` | 完整 5+ 属性 10 / 基础 5 / 无 0 | 0 |
| 第三方权威背书? | LLM 主观 | 上限 5 | 0 |
| 主页 Organization schema 含 sameAs? | 解析 JSON-LD | 完整 sameAs 10 / 部分 3-5 / 无 0 | 3 |
| **小计** | | | **3 / 35** |

#### 子类 B:Content Preferences(满分 40)

| 信号 | 验证方法 | 标尺 | 假设结果 |
|------|---------|------|---------|
| 字数 ≥ 2000? | WebFetch 后 split count | 满分 10 / adequate 5 / 0 | 0(主页 ~800 字) |
| Q&A / 5W1H 结构? | H2/H3 句式正则 | 满分 15 / 部分 7 / 0 | 0 |
| 作者署名 + 资质? | 找 author byline + author 页 | 满分 10 / 仅名字 5 / 0 | 0 |
| 发布/修改日期可见? | DOM `<time>` 或 schema | 满分 5 / 单一 3 / 0 | 0 |
| **小计** | | | **0 / 40** |

#### 子类 C:Crawler Access(满分 25)

| 爬虫 | 验证方法 | 假设结果 |
|------|---------|---------|
| OAI-SearchBot | `Grep "OAI-SearchBot"` in robots.txt | 默认放行 |
| ChatGPT-User | `Grep "ChatGPT-User"` | 默认放行 |
| GPTBot | `Grep "GPTBot"` | 默认放行 |
| **小计** | | **25 / 25** |

#### ChatGPT 平台分汇总

```
ChatGPT Score = 3 (Entity) + 0 (Content) + 25 (Crawler) = 28 / 100
状态:Critical(< 40)
```

**关键观察:** 这个站靠"默认不写 robots.txt"拿到 25 分,Entity 和 Content 几乎归零。`/geo proposal` 看到这个总分会自动推荐 €9.5K Premium 档(`skills/geo-proposal/SKILL.md:341`)——审计的"严重诊断"直接转化为"高单价销售机会"。

### 4.4 Action Items 生成机制

打完分还不够,Step 7 强制要求每平台输出 2-3 条具体行动项(`agents/geo-platform-analysis.md:171-173`)。提示词里有反空话约束:

> "Be specific in action items. Instead of 'add schema markup,' say 'add Organization schema with sameAs linking to your Wikipedia article and LinkedIn company page.'"

实际生成示例(基于上述评分):

> 1. **[CRITICAL]** Create a Wikipedia draft for Electron Srl. Notability angle: 30+ years educational equipment manufacturing, EU-wide distribution. Effort: Medium.
> 2. **[HIGH]** Register Wikidata entity with properties: instance of (Q4830453=business), official website, founding date, headquarters location. Effort: Low (~30 min).
> 3. **[HIGH]** Expand homepage Organization schema with `sameAs` array linking to LinkedIn, YouTube, Wikipedia.

LLM 会从 Skill checklist 中**挑当前最缺的项**,翻译成命令式句子。这是评分→行动→提案的关键转化层。

---

## 5. 数据持久化与状态管理

### 5.1 JSON 单文件作为唯一事实源

`~/.geo-prospects/prospects.json` 是 CRM 的唯一可信数据源。结构:

```json
{
  "id": "PRO-001",
  "company": "Electron Srl",
  "domain": "electron-srl.com",
  "status": "qualified",                      ← 管线状态
  "geo_score": 32,                            ← 最近一次审计分
  "audit_date": "2026-03-12",
  "audit_file": "~/.geo-prospects/audits/...",
  "proposal_file": "~/.geo-prospects/proposals/...",
  "monthly_value": 0,                         ← 已签约月费
  "contract_start": null,
  "contract_months": 0,
  "notes": [{"date": "...", "text": "..."}],
  "created_at": "...",
  "updated_at": "..."
}
```

每个命令(`new / status / won / lost / note`)都是对这份 JSON 的原子读改写。Markdown 文件(audits / proposals / reports)是**只追加的归档物**,以 `<domain>-<date>.md` 命名,便于 `geo-compare` 用 Glob 自动配对。

### 5.2 多前端访问同一份数据

仓库提供三个 UI 访问 CRM:

| 入口 | 实现 | 适合场景 |
|------|------|---------|
| Claude Code 斜杠命令 | `skills/geo-prospect/SKILL.md` | 边做审计边记录 |
| 终端 dashboard | `scripts/crm_dashboard.py`(`rich` 库) | 早会 stand-up |
| Web UI | `scripts/webapp/app.py`(Flask + HTMX,端口 5050) | 多人协作、给客户演示 |

三者都读同一份 `prospects.json`,这是经典的**单一存储多接口**模式。

---

## 6. 关键设计模式与亮点

### 6.1 提示词即架构

主调度器 `geo/SKILL.md` 用一张表(`:64-87`)把每个命令路由到 Skill / Agent,本质是**Markdown 表格作为路由表**。Claude 读懂表后,把命令视作正确的 sub-skill 调用。

这种设计的优势:

- **零代码可改业务规则**:改提示词即可调整评分权重或行动项标准
- **可读性极强**:产品经理能直接审阅打分逻辑
- **不需要构建系统**:无 build step,装好即用

代价:

- **分数有 ±3-5 浮动**:同一站连续审计两次会出现差异
- **难以单元测试**:LLM 输出无法 100% 一致
- **Token 消耗大**:5 个并行 agent 各读一份长 skill,context 占用高

### 6.2 业务类型贯穿全链路

业务类型识别(SaaS / Local / E-com / Publisher / Agency)在 Phase 1 完成后,会传递给所有下游 Skill,影响:

- **schema 模板选择**:`schema/` 下 6 套 JSON-LD 按类型选用
- **评分权重微调**:SaaS 加重 Comparison Table,Local 加重 NAP
- **提案话术调整**:`/geo proposal` 在 ROI 投影部分按行业引用不同的转化率
- **平台优先级**:E-com 优先 Gemini(因为接 Google Merchant Center),SaaS 优先 ChatGPT(技术决策者用得多)

### 6.3 输出契约的规范化

每个 Skill / Agent 末尾都有固定的 markdown 模板(例 `agents/geo-platform-analysis.md:175-287`),包含:

- 顶部一行 `**Platform Readiness Average: [X]/100**` ← 主审计正则提取的锚
- 标准化表格列名 ← `geo-compare` 用相同列名做 diff
- 严重度图标(🔴 Critical / ⚠️ High / 等) ← `/geo report-pdf` 按图标着色

这套契约让上下游松耦合——**任何 Skill 单独运行时输出可读,组合运行时可被机器解析**。

### 6.4 显式的"反 LLM 偷懒"约束

提示词中多处出现禁令:

- `agents/geo-platform-analysis.md:292`:"Be specific. Instead of 'add schema markup,' say ..."
- `agents/geo-platform-analysis.md:294`:"If you cannot verify a signal, note it as 'unverifiable from external analysis' rather than assuming absence."
- `skills/geo-citability/SKILL.md`:用具体的 high-citability / low-citability 示例段对比训练 LLM

这是**用提示词工程对抗 LLM 趋同性回答**,提升输出的可执行性。

---

## 7. 工程化弱点与改进建议

### 7.1 评分系统弱点

| 问题 | 位置 | 影响 | 建议 |
|------|------|------|------|
| Agent 平台分(35/40/25)与 Skill rubric 子分(20+10+10+15+...) **总额不一致** | `agents/geo-platform-analysis.md:65` vs `skills/geo-platform-optimizer/SKILL.md:91` | LLM 需要缩放,引入 ±3 分波动 | 重写 rubric 让两份文件总额对齐 |
| Wikipedia / Reddit / Wikidata 检测靠 WebFetch | `geo-brand-mentions`, `geo-platform-analysis` | 小品牌容易漏判;搜索结果页可能有反爬 | 接入 Wikipedia / Wikidata 官方 API |
| 同站重复审计无缓存 | 整个 audit 链路 | 每次都重新抓 robots.txt / schema,慢且分数会浮动 | 在 `~/.geo-prospects/cache/<domain>/` 缓存抓取结果,设 TTL |
| Citability 5 维总和不显式 = 100 | `skills/geo-citability/SKILL.md:25-80` | 子项加起来是 100% 还是 30%×5 不清晰 | 改为加权和并显式声明权重表 |
| `/geo compare` 用正则从 markdown 抽数 | `skills/geo-compare/SKILL.md:262-272` | 报告格式微调就抽不到 | 让审计同时输出一份 `audit.json` 作为机器可读源 |

### 7.2 业务流程弱点

| 问题 | 位置 | 影响 | 建议 |
|------|------|------|------|
| `prospects.json` 无并发保护 | CLI / dashboard / webapp 三个写入者 | 同时操作可能丢数据 | 加文件锁或迁移到 SQLite |
| 月度报告依赖 `audits/` 命名约定 | `<domain>-<date>.md` | 手动改名会破坏对比 | 把 audit 元数据写进 `prospects.json` |
| 没有审计版本号 | 所有 skill | rubric 调整后历史分不可比 | 在 audit 文件里记录 `geo-skill-version` |
| 没有失败降级路径 | WebFetch 超时直接报错 | 50 页爬到一半失败,前面分数白算 | 实现 partial-result 落盘 |

### 7.3 商业流程弱点

| 问题 | 影响 | 建议 |
|------|------|------|
| 报价档位币种硬编码为 € | 美国/英国市场用户需要手改 | 加 `--currency` 参数 |
| 提案模板里"Your Agency Name"是 placeholder | 用户每次手填 | 在 `~/.geo-prospects/config.json` 存代理商资料 |
| 没有签约后自动生成 onboarding 任务 | 销售→交付有断层 | 加 `/geo onboard <domain>` 生成 30 天行动板 |
| 没有 churn 信号(连续 N 个月分数停滞) | 续约风险被动发现 | 月报里加"风险标记":连续 2 个月 delta ≤ 2 时提示 |

---

## 8. 与传统 SEO 工具的对比

| 维度 | 传统 SEO 工具(Ahrefs / Semrush) | `geo-seo-claude` |
|------|---------------------------------|-----------------|
| 优化目标 | Google 排名 | AI 平台引用率 |
| 主要数据源 | 自家爬虫指数 + 反向链接库 | 实时 WebFetch + LLM 推断 |
| 评分基础 | 反向链接 + 关键词排名 | Citability + Brand mentions + E-E-A-T |
| 用户主体 | 内部 SEO 团队 | GEO 代理服务商 |
| 交付物 | Dashboard 截图 | Markdown / PDF 提案 / 月报 |
| 商业模型 | SaaS 订阅($99–$999/月) | 工具开源 + 代理商月度合同(€2.5K–€9.5K) |
| 数据库 | 自家 PB 级 index | 客户的 JSON 单文件 |
| 启动成本 | 工具订阅费 | 0(工具)+ Skool 社区(可选) |

`geo-seo-claude` 不是 Ahrefs 的替代品,而是**代理人工服务的工具化包装**——它提供的是"诊断方法论 + 销售话术 + 客户管理"三件套。

---

## 9. 总结评价

### 9.1 这是一个什么样的产品

一份**"AI 时代 SEO 代理商的开箱即用工具包"**:

- 技术侧:5 路并行 subagent 给网站打 GEO 分,产出技术报告
- 销售侧:JSON CRM + 按分自动报价 + 三档定价模型
- 留存侧:月度 delta 报告把审计差值变成续约证据
- 教学侧:把市场叙事数据写进提示词,所有产出自带销售话术

### 9.2 核心创新点

1. **rubric-driven LLM scoring**:用结构化提示词把开放评估变成可重复的子分加和
2. **业务流程即 Skill 编排**:整条销售漏斗用 Markdown + JSON 实现,无需后端
3. **业务类型贯穿评分**:同一审计在 SaaS / Local / E-com 上输出不同优先级
4. **审计→提案→月报闭环**:输出契约让相邻阶段松耦合,数据自动流转

### 9.3 适合谁

- ✅ 准备做 GEO 代理生意的独立咨询师 / 小团队
- ✅ 想给客户演示 AI 搜索可见度的市场顾问
- ✅ 已有 SEO 业务、想加 GEO 上层服务的代理公司
- ❌ 需要工业级监控的大型营销团队(没有定时审计、没有告警)
- ❌ 关心严格可重复性的科研用途(LLM 打分有波动)
- ❌ 大规模 SaaS 部署(没有 multi-tenant 设计)

### 9.4 一句话定性

> **它是一份产品化的 GEO 服务方法论,而非软件意义上的产品。Markdown 是它的"代码",Claude 是它的"运行时",`~/.geo-prospects/prospects.json` 是它的"业务账本"。**

---

## 附录 A:核心命令速查

| 命令 | 用途 | 关键输入 | 输出 |
|------|------|---------|------|
| `/geo audit <url>` | 全审计(5 agent 并行) | URL | `GEO-AUDIT-REPORT.md` |
| `/geo quick <url>` | 60 秒快照 | URL | 终端摘要 |
| `/geo citability <url>` | 引用度评分 | URL | `GEO-CITABILITY-SCORE.md` |
| `/geo crawlers <url>` | 14+ AI 爬虫可达性 | URL | `GEO-CRAWLER-ACCESS.md` |
| `/geo llmstxt <url>` | llms.txt 分析或生成 | URL | `llms.txt` |
| `/geo brands <url>` | 品牌提及扫描 | URL | `GEO-BRAND-MENTIONS.md` |
| `/geo platforms <url>` | 5 平台分别打分 | URL | `GEO-PLATFORM-OPTIMIZATION.md` |
| `/geo schema <url>` | JSON-LD 检测+生成 | URL | `GEO-SCHEMA-REPORT.md` + JSON-LD |
| `/geo technical <url>` | 8 类技术 SEO | URL | `GEO-TECHNICAL-AUDIT.md` |
| `/geo content <url>` | E-E-A-T 评分 | URL | `GEO-CONTENT-ANALYSIS.md` |
| `/geo report <url>` | 客户演示版报告 | URL | `GEO-CLIENT-REPORT.md` |
| `/geo report-pdf` | PDF 版客户报告 | 已运行 audit | `GEO-REPORT.pdf` |
| `/geo prospect <cmd>` | CRM 管理 | 子命令 | `~/.geo-prospects/prospects.json` |
| `/geo proposal <domain>` | 自动生成报价 | 域名 | `~/.geo-prospects/proposals/...` |
| `/geo compare <domain>` | 月度 delta 报告 | 域名 | `~/.geo-prospects/reports/...` |
| `/geo update` | 拉取上游最新版 | — | 替换 `~/.claude/skills/geo*` |

## 附录 B:文件索引

| 文件 | 行数 | 角色 |
|------|------|------|
| `geo/SKILL.md` | 234 | 主调度器 |
| `skills/geo-audit/SKILL.md` | 337 | 全审计编排 |
| `skills/geo-citability/SKILL.md` | 319 | 引用度评分(5 维) |
| `skills/geo-crawlers/SKILL.md` | 377 | 14+ AI 爬虫识别库 |
| `skills/geo-llmstxt/SKILL.md` | 432 | llms.txt 标准 |
| `skills/geo-brand-mentions/SKILL.md` | 480 | 11 平台品牌提及扫描 |
| `skills/geo-platform-optimizer/SKILL.md` | 275 | 5 平台 rubric |
| `skills/geo-schema/SKILL.md` | 370 | Schema.org 检测+生成 |
| `skills/geo-technical/SKILL.md` | 448 | 8 类技术 SEO |
| `skills/geo-content/SKILL.md` | 345 | E-E-A-T(4×25) |
| `skills/geo-report/SKILL.md` | 399 | 客户报告聚合 |
| `skills/geo-report-pdf/SKILL.md` | 164 | PDF 模板 |
| `skills/geo-prospect/SKILL.md` | 193 | CRM-lite |
| `skills/geo-proposal/SKILL.md` | 345 | 自动报价 |
| `skills/geo-compare/SKILL.md` | 307 | 月度 delta |
| `skills/geo-update/SKILL.md` | 137 | 自更新 |
| `agents/geo-ai-visibility.md` | ~300 | 并行 worker |
| `agents/geo-platform-analysis.md` | 296 | 并行 worker |
| `agents/geo-technical.md` | ~280 | 并行 worker |
| `agents/geo-content.md` | ~290 | 并行 worker |
| `agents/geo-schema.md` | ~280 | 并行 worker |
| `scripts/citability_scorer.py` | 343 | 引用度打分引擎 |
| `scripts/brand_scanner.py` | 276 | 品牌提及扫描 |
| `scripts/llmstxt_generator.py` | 294 | llms.txt 生成 |
| `scripts/fetch_page.py` | 490 | 页面抓取(含 SSR 检测) |
| `scripts/generate_pdf_report.py` | 931 | ReportLab PDF 生成 |
| `scripts/crm_dashboard.py` | 336 | rich CLI dashboard |
| `scripts/webapp/app.py` | — | Flask + HTMX Web UI |

---

*调研报告结束。*
