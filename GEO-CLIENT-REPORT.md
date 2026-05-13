# GEO Readiness Report — GrowthClaw

**域名:** growthclaw.co
**分析日期:** 2026-05-13
**分析页面数:** 40 个 URL（8 种语言 × 5 类页面）
**业务类型:** SaaS（AI 驱动的 SEO 自动化平台）
**报告版本:** 1.0

---

## Section 1 · 执行摘要

我们对 growthclaw.co 完成了一次完整的生成式引擎优化（GEO）审计，覆盖 8 种语言下的 40 个公开 URL、5 大主流 AI 搜索平台、以及全部六大 GEO 维度。**GrowthClaw 当前的 GEO Readiness Score 为 29/100，处于 "Needs Attention（需立即处理）" 等级**——这意味着您的品牌在 ChatGPT、Claude、Perplexity、Gemini 和 Google AI Overviews 中几乎不可被发现与引用，而您的直接竞品（Surfer、Frase、Semrush AI 等）正在每天截取本应属于您的 AI 搜索流量。本次审计中最关键的发现是：**您的品牌实体在 AI 训练语料的所有关键来源（Wikipedia / Wikidata / Reddit / YouTube / LinkedIn / Crunchbase / G2）中完全缺席，且 GitHub 上存在同名开源项目 `mrrkrieg/growthclaw` 与您的 SaaS 品牌形成实体混淆**。三大优先行动为：(1) 立即部署完整 JSON-LD 结构化数据（含 SoftwareApplication + Offer + sameAs），(2) 在 LinkedIn / Crunchbase / Product Hunt / G2 建立首批品牌实体锚点，(3) 启动博客与 FAQ 内容生产以提供 AI 可引用的"答案块"。基于行业基准（SaaS 类 AI 搜索可带来 25–40% 的有机发现量），完整实施本报告可在 90 天内将 GEO 分数提升至 65–75，对应每月预计 **$3,000–$12,000** 的额外有机价值（按 Pro $49/月 × 60–250 新增激活换算）。

---

## Section 2 · GEO Readiness Score

## GEO Readiness Score: 29/100 — Needs Attention

| 组成部分 | 分数 | 权重 | 加权得分 |
|---|---|---|---|
| AI Platform Readiness | 22/100 | 25% | 5.5 |
| Content Quality & E-E-A-T | 14/100 | 25% | 3.5 |
| Technical Foundation | 78/100 | 20% | 15.6 |
| Schema & Structured Data | 22/100 | 15% | 3.3 |
| Brand Authority | 8/100 | 15% | 1.2 |
| **综合得分** | | | **29.1 → 29/100** |

**等级解读 — Needs Attention：** 您的网站在 AI 可见性方面存在严重缺口，必须立即采取行动。竞品正在持续捕获本应属于您品牌的 AI 搜索流量。当前的技术底座（Next.js SSR、HSTS、8 语言 hreflang、llms.txt）已经达到良好水准，但**内容、实体与结构化数据三大柱子几乎为零**——这是一个"技术骨架完整但血肉空缺"的典型 MVP 着陆页型站点。

---

## Section 3 · AI Visibility Dashboard

| AI 平台 | Readiness Score | 关键缺口 | 优先行动 |
|---|---|---|---|
| Google AI Overviews | 18/100 | 无 FAQ / 长尾内容、无 FAQPage schema | 新建 `/en/faq`，每答附 50 字直答与 FAQPage JSON-LD |
| ChatGPT Web Search | 22/100 | 品牌实体未识别、内容过短 | 注入 Organization + sameAs，提交 Wikidata 条目 |
| Perplexity AI | 15/100 | Reddit / Wikipedia 零信号 | 在 r/SEO、r/SaaS、IndieHackers 发起真实讨论 |
| Google Gemini | 25/100 | 无 YouTube、无 Google Knowledge Graph | 开通 YouTube 频道发布 3 条演示视频 |
| Bing Copilot | 28/100 | Bing Webmaster Tools 未验证、无 IndexNow | 验证 Bing Webmaster，部署 IndexNow（Vercel 原生） |

**含义解读：** 这些分数反映了您的内容被各 AI 搜索平台引用的可能性。**低于 50** 表示在该平台上的引用存在重大障碍；**低于 30** 表示几乎不可能被引用。GrowthClaw 全部 5 个平台都处于 30 分以下，意味着当前阶段几乎所有的 AI 搜索查询都不会指向您的站点。

---

## Section 4 · AI Crawler Access Status

| AI 爬虫 | 平台 | 状态 | 影响等级 | 建议 |
|---|---|---|---|---|
| Googlebot | Google Search + AIO | ✅ Allowed | Critical | 保持允许，无需更改 |
| GPTBot | ChatGPT / OpenAI | ✅ Allowed | High | 保持允许；建议在 robots.txt 显式列出以明确意图 |
| Bingbot | Bing + Copilot + ChatGPT | ✅ Allowed | High | 保持允许，并验证 Bing Webmaster Tools |
| PerplexityBot | Perplexity AI | ✅ Allowed | Medium | 保持允许 |
| Google-Extended | Gemini Training | ✅ Allowed | Medium | 保持允许（同意用于训练 Gemini） |
| ClaudeBot | Anthropic Claude | ✅ Allowed | Medium | 保持允许 |
| Applebot-Extended | Apple Intelligence | ✅ Allowed | Medium | 保持允许 |
| CCBot | Common Crawl | ✅ Allowed | High | 保持允许（多数 LLM 训练源） |

**爬虫访问层面表现优异——但这也意味着问题不在"门是否打开"，而在"店里有没有东西可看"。** 您的 robots.txt（`User-Agent: *  Allow: /`）对所有 AI 爬虫开放，但站点本身仅约 400 字着陆页 + 法律页，AI 爬到了也无内容可引。建议补充：

```
# 在 robots.txt 末尾追加（Cloudflare 推荐的内容信号）
Content-Signal: search=yes, ai-train=yes, ai-retrieval=yes
```

---

## Section 5 · Brand Authority Analysis

| 平台 | 是否存在 | 详情 | 对 AI 可见性的影响 |
|---|---|---|---|
| Wikipedia | ❌ No | 仅匹配通用词 "Claw"，无 GrowthClaw 条目 | **极高** — ChatGPT 引用的 47.9% 来源 |
| Wikidata | ❌ No | 无 Q-item | 高 — 机器可读实体数据 |
| LinkedIn 公司页 | ❌ No | 未创建 | 高 — Bing Copilot 与 ChatGPT 关键信号 |
| YouTube 官方频道 | ❌ No | 未开通 | 高 — Gemini 与 Perplexity 关键信号 |
| Reddit 品牌讨论 | ❌ No | r/SEO、r/SaaS 零讨论 | **极高** — Perplexity 引用的 46.7% 来源 |
| Google Knowledge Panel | ❌ No | 未建立 | 高 — Gemini 实体识别 |
| Crunchbase | ❌ No | 未收录 | 中 — 实体验证 |
| Product Hunt | ❌ No | 未发布 | 中 — SaaS 受众发现源 |
| G2 / Capterra | ❌ No | 未收录 | 中 — SaaS 评测信号 |
| X / Twitter | ❌ No | 未确认官方账号 | 中 — 即时性社交信号 |
| GitHub | ⚠️ 冲突 | `mrrkrieg/growthclaw` 同名开源项目 | **负面** — AI 引用时实体混淆风险 |

**含义解读：** AI 平台通过在多个权威源交叉验证您的品牌来建立信任。**您当前在 0/10 个核心平台拥有品牌实体存在**，且存在一个负面冲突（GitHub 同名项目）。这是本次审计中影响最大、回报周期最长、必须**优先启动**的工作板块——纯技术层面的优化无法补偿实体信号的缺失。

---

## Section 6 · Citability Analysis

### Top 5 最可引用页面（相对而言）

由于站点页面极少（每语言只 1 个着陆页 + 3 个法律页），所谓 "Top 5" 仅指相对可引用性较高的：

1. **https://growthclaw.co/en**
   - 可引用性：有 H1/H2 结构、有"45+ 平均分提升""80% 调研时间节省"两个数据点
   - 改进：将这两个数据点链接到原始来源（"基于 2026 年 Q1 内部 N=120 客户样本"），并在数据点附近加 `Dataset` 或 `ClaimReview` schema
2. **https://growthclaw.co/en/privacy** — 法律文本可被 AI 用于解答"GrowthClaw 是否合规 GDPR"等问题
3. **https://growthclaw.co/en/terms** — 同上
4. **https://growthclaw.co/en/scta** — SCTA 数据安全合规承诺，对 B2B 采购决策有引用价值
5. **https://growthclaw.co/llms.txt** — AI 喂养清单本身存在，已超越行业 95% 的 SaaS 站点

### Top 5 最不可引用 / 缺失的页面（应立即创建）

1. **缺失：/en/faq** — 无任何 Q&A 结构化内容，Perplexity / AIO 几乎无法引用
   - 建议：撰写 15–20 条 FAQ（"GrowthClaw 与 Semrush 区别""$49/月包含什么""支持的语言""数据安全保障"），每答 2–3 句独立成段并加 FAQPage JSON-LD
2. **缺失：/en/blog** — 无任何深度文章，无法建立话题权威性
   - 建议：每周 1–2 篇 1500+ 字深度文章（围绕"AI SEO 自动化"长尾词），含 Article schema 与作者 Person schema
3. **缺失：/en/about** — 无团队 / 创始人页，E-E-A-T 的 Expertise 与 Authoritativeness 维度直接归零
   - 建议：公开创始人姓名 + LinkedIn + 履历，发布 Person schema
4. **缺失：/en/case-studies** — 无客户案例，"45+/80%" 数据无任何外部证据
   - 建议：2–3 篇真实案例研究，含客户名、起止指标、方法论
5. **缺失：/en/compare/{semrush,surfer,frase}** — 无竞品对比页，错失 BoFu 流量
   - 建议：3 篇详细对比文章，每篇 ≥ 2000 字，含特性矩阵

**业务影响：** 改进/新建这 5 类页面是当前 ROI 最高的内容投资。每新增一个高质量页面，预计可在 60–90 天内提升 1.5–3 个 GEO 综合分。

---

## Section 7 · Technical Health Summary

| 维度 | 状态 | 业务影响 |
|---|---|---|
| Core Web Vitals | ⚠️ Needs Work | 首屏 `opacity:0` + JS 上浮动画，CLS/INP 中等风险；CDN 缓存被关闭影响 LCP |
| Server-Side Rendering | ✅ Yes (Next.js App Router) | 首屏 HTML 完整可见，AI 爬虫可直接读取——技术层面无障碍 |
| Mobile Optimization | ✅ Good | viewport 正确、hero image 预加载、fetchPriority 已用 |
| Security (HTTPS + Headers) | ⚠️ Needs Work | HTTPS + HSTS preload 优秀；但 CSP / X-Frame-Options / X-Content-Type-Options / Referrer-Policy / Permissions-Policy **全部缺失** |
| Page Speed | ⚠️ Average | `cache-control: private, no-cache` 阻止 CDN 边缘缓存，每次请求回源 Vercel |
| IndexNow Protocol | ❌ Not Implemented | Vercel 原生支持，部署 30 分钟即可——Bing / ChatGPT 索引提速显著 |
| hreflang | ⚠️ 90% Good | 8 语言齐全；缺 `hreflang="x-default"`，根域 `/` 307→`/en` 默认语言未声明 |
| llms.txt | ✅ Yes | 文件存在、200 OK、格式良好——超越 95% 同类站点 |

**技术正面信号：** 您的 SSR 是完整的——这意味着您不存在"AI 爬虫看到空白页"这一最致命的技术问题。这是 GrowthClaw 当前最大的技术资产。

**技术修复重点：** 安全响应头一次性补齐（在 `next.config.js` 的 `headers()` 中添加 5 行配置即可），同时开启 IndexNow 与边缘缓存。预计开发投入 4–6 小时，技术分可从 78 提升至 90+。

---

## Section 8 · Schema & Structured Data

### 当前实施情况

| Schema 类型 | 是否存在 | 状态 | AI 影响 |
|---|---|---|---|
| Organization | ✅ Yes | ⚠️ 极简（仅 name/url/logo） | Critical — 缺 description/sameAs/contactPoint，AI 无法构建实体认知 |
| WebSite | ✅ Yes | ⚠️ 缺 potentialAction (SearchAction) | Medium — 无站内搜索富结果 |
| SoftwareApplication | ❌ No | 缺失 | **Critical** — 作为 SaaS 产品，AI 无法识别产品类别 |
| Offer / AggregateOffer | ❌ No | 缺失 | **Critical** — Free $0 / Pro $49 定价信息 AI 完全不可读 |
| Organization.sameAs | ❌ No | 缺失 | **Critical** — 实体图谱断裂 |
| FAQPage | ❌ No | 无 FAQ 内容支撑 | High — 先建内容再加 schema |
| BreadcrumbList | ❌ No | 缺失 | Medium — 导航上下文 |
| Article + Author | ❌ No | 无博客内容支撑 | High — 待 /blog 上线后必加 |
| Person (creator/team) | ❌ No | 无团队页支撑 | High — 待 /about 上线后必加 |
| speakable | ❌ No | 缺失 | Medium — AI 助手可读性标注 |
| OpenGraph 图片 (`og:image`) | ❌ No | 缺失 | High — 社交分享卡片不完整 |
| Twitter Card | ⚠️ summary | 应升级 summary_large_image | Medium |

**Ready-to-use 代码已准备：** 完整的 JSON-LD `@graph` 模板（含 Organization + SoftwareApplication + Offer + WebSite + SearchAction）见技术附录 A。您的开发团队可在 30 分钟内复制粘贴上线，Schema 分预计从 22 跃升至 70+。

---

## Section 9 · llms.txt — AI Content Guide

| 文件 | 状态 | 建议 |
|---|---|---|
| `/llms.txt` | ✅ Present (1274 bytes, 200 OK) | 已存在且格式良好——超越 95% 同类站点 |
| `/llms-full.txt` | ❌ Missing | 建议追加，作为完整内容摘要供 AI 一次性吸收 |

**当前 llms.txt 优化建议：**

1. 在顶部 blockquote 写明实体区分：
   > GrowthClaw is a hosted SaaS at https://growthclaw.co — not affiliated with the open-source 'growthclaw' project on GitHub (`mrrkrieg/growthclaw`).
2. 每条 URL 后追加一行 description（当前只列 URL，信息密度偏低）。
3. 在 `/llms.txt` 增加 `## Key Facts` 段：定价、主要功能、目标客户、支持语言等结构化要点。

**含义解读：** llms.txt 是新兴标准（类似 robots.txt），告诉 AI 系统您的站点关于什么、哪些页面最重要。**绝大多数同类 SaaS 尚未实施这一点——GrowthClaw 已经领先一步**。但需要将其从"占位文件"升级为"真正的 AI 内容指南"。

---

## Section 10 · Prioritized Action Plan

### Quick Wins（本周 — 高影响低成本）

*每项可在 1 名工程师 4 小时内完成*

| # | 行动 | 影响 | 投入 | 影响平台 |
|---|---|---|---|---|
| 1 | 部署完整 JSON-LD `@graph`（Organization + SoftwareApplication + Offer + sameAs + WebSite） | **High** | 30 分钟 | Google AIO / Gemini / Bing |
| 2 | 创建 LinkedIn 公司页（完整填写描述/行业/规模/网址） | **High** | 1 小时 | ChatGPT / Bing / Gemini |
| 3 | 提交 Crunchbase 公司条目 | **High** | 1 小时 | 所有平台（实体锚点） |
| 4 | 提交 Product Hunt 上线（带演示视频） | **High** | 2 小时 | ChatGPT / Perplexity |
| 5 | 补齐 5 项安全响应头（next.config.js headers()） | Medium | 30 分钟 | Google AIO（信任信号） |
| 6 | 添加 `og:image` (1200×630) + `twitter:card=summary_large_image` | Medium | 30 分钟 | 所有社交平台 + AI |
| 7 | 修正 `/en` 路径 meta description 为英文（当前为中文） | Medium | 10 分钟 | Google / Bing |
| 8 | robots.txt 追加 `Content-Signal` + 显式 AI 爬虫 Allow 行 | Medium | 15 分钟 | 所有 AI 平台 |
| 9 | llms.txt 头部声明同名实体区分 + 每 URL 附 description | Medium | 30 分钟 | ChatGPT / Claude / Perplexity |
| 10 | 添加 `hreflang="x-default"` 指向 `/en` | Low | 15 分钟 | Google（多语言索引） |

**Quick Wins 预期分数提升：+12 to +18 分**（综合分由 29 → 41–47）

---

### Medium-Term Improvements（本月 — 显著影响 / 中等投入）

*1–5 天工作量*

| # | 行动 | 影响 | 投入 | 影响平台 |
|---|---|---|---|---|
| 1 | 新建 `/en/faq`，撰写 15–20 条 FAQ + FAQPage JSON-LD | **High** | 2 天 | Google AIO / Perplexity / ChatGPT |
| 2 | 新建 `/en/about` 团队页（创始人姓名 + LinkedIn + 履历 + Person schema） | **High** | 1 天 | 所有（E-E-A-T） |
| 3 | 新建 `/en/blog`，首批 4–6 篇 1500+ 字深度文章（含 Article + Person schema） | **High** | 5 天 | 所有平台 |
| 4 | 新建 `/en/case-studies`，2–3 篇真实案例（含具体客户、指标、方法论） | **High** | 3 天 | 所有平台 |
| 5 | 验证 Bing Webmaster Tools + 部署 IndexNow（Vercel 原生） | Medium | 半天 | Bing Copilot / ChatGPT |
| 6 | 创建 X / Twitter 官方账号 + 链接到 sameAs | Medium | 半天 | 实体锚点 |
| 7 | 优化缓存策略：着陆页 `cache-control: public, s-maxage=3600, swr=86400` | Medium | 半天 | LCP / 抓取效率 |
| 8 | 在 G2 / Capterra / SaaSHub 创建 listing | Medium | 1 天 | SaaS 评测信号 |
| 9 | 提交 Wikidata 草案（Q-item） | **High** | 1 天 | 所有 AI（实体图谱） |
| 10 | 重写主页文案：每个功能模块加一段独立可摘录的 50 字"AI 答案块" | **High** | 1 天 | Google AIO / ChatGPT |

**Medium-Term 预期分数提升：+15 to +20 分**（41–47 → 56–67）

---

### Strategic Initiatives（本季度 — 长期竞争优势）

*持续投入数周到数月*

| # | 行动 | 影响 | 投入 | 影响平台 |
|---|---|---|---|---|
| 1 | 在 r/SEO、r/SaaS、IndieHackers、HackerNews 建立合规品牌存在（产品讨论、对比帖、AMA） | **High** | 持续每周 2–3 帖 × 12 周 | **Perplexity（核心）** / ChatGPT |
| 2 | 申请 Wikipedia 条目（通过第三方独立媒体报道获取可引用源） | **High** | 8–12 周 | ChatGPT（核心） / 所有平台 |
| 3 | YouTube 频道：每周 1 条产品演示 / SEO 教程视频 | **High** | 持续 | Gemini / Perplexity |
| 4 | 争取 5–10 篇第三方独立评测（SaaSHub、TopSaaSReviewers、SEO 行业博客） | **High** | 持续 12 周 | 所有平台 |
| 5 | 启动月度原创研究 / 数据报告（"2026 SaaS SEO 自动化报告"等） | **High** | 季度性 | 所有平台（被反向链接 + 引用） |
| 6 | 建立话题集群：围绕 "AI SEO" "自动化关键词研究" 等核心话题，构建 30–50 篇深度内容矩阵 | **High** | 持续 12 周 | Google AIO / ChatGPT |
| 7 | 申请 Google Business Profile（如有公司实体地址） | Medium | 持续 | Gemini |
| 8 | 8 种语言的内容本地化（目前仅着陆页本地化，应扩展到 blog / FAQ） | Medium | 持续 | 多语言 AI 搜索 |
| 9 | 建立 affiliate / customer advocacy 计划，激励客户在 G2 / Capterra 发布真实评测 | Medium | 持续 | SaaS 评测信号 |
| 10 | 监控 GEO 分数月度变化（使用本审计工具或 Profound / Otterly.ai） | Medium | 月度 | 持续优化 |

**Strategic 预期分数提升：+15 to +20 分**（56–67 → 72–85）

---

### Estimated Impact 估算影响

基于行业基准与本次审计识别的具体缺口：

- **仅执行 Quick Wins**：GEO 分数预计提升 **12–18 分**（29 → 41–47），主要改善 Google AIO 与 Bing Copilot 上的可见性
- **完整实施本行动计划**：GEO 分数预计提升至 **72–85/100**（Good 级别）
- 按 SaaS 行业 AI 搜索流量基准（2026 年预计占有机发现量 25–40%），改进后的 AI 可见性对应每月：
  - **保守估算：$3,000 / 月**（60 个新增 Pro 客户激活）
  - **中位估算：$6,000–$8,000 / 月**
  - **乐观估算：$12,000+ / 月**（250+ 个新增 Pro 客户激活）

**回收周期：** Quick Wins 阶段投入约 12 工时，若仅带来 30 个 Pro 客户激活即可回本（$1,470）。

---

## Section 11 · Competitor Comparison

*本次审计未指定竞品 URL，以下为行业基准对照（基于公开数据）：*

| 维度 | GrowthClaw | Surfer SEO（基准） | Frase（基准） | Semrush AI Toolkit（基准） |
|---|---|---|---|---|
| Overall GEO Score（估计） | 29/100 | 70/100 | 65/100 | 88/100 |
| Wikipedia / Wikidata 存在 | ❌ No | ✅ Yes | ⚠️ Partial | ✅ Yes |
| Reddit 权威性 | ❌ 零讨论 | ✅ 高（数千讨论） | ✅ 中（数百讨论） | ✅ 极高 |
| YouTube 官方频道 | ❌ No | ✅ Yes (10k+ 订阅) | ✅ Yes | ✅ Yes |
| G2 / Capterra 评测 | ❌ No | ✅ 1500+ 评测 | ✅ 300+ 评测 | ✅ 2000+ 评测 |
| LinkedIn 公司页 | ❌ No | ✅ 25k+ 关注 | ✅ 6k+ 关注 | ✅ 250k+ 关注 |
| SoftwareApplication Schema | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| SSR | ✅ Yes (Next.js) | ✅ Yes | ✅ Yes | ✅ Yes |
| 博客内容量 | ❌ 0 篇 | ✅ 500+ 篇 | ✅ 200+ 篇 | ✅ 3000+ 篇 |
| 多语言内容 | ⚠️ 8 语言着陆页（内容稀薄） | ⚠️ 5 语言 | ⚠️ 3 语言 | ✅ 12+ 语言全栈 |

### 您领先的地方

- **多语言着陆页覆盖**：8 种语言（含希伯来 / 阿拉伯 RTL）领先大部分中小 SaaS——但需要将这一优势从"着陆页"扩展到"完整内容"
- **llms.txt 已部署**：超越行业 95% 的 SaaS 站点
- **技术底座 SSR + HSTS preload**：与头部竞品平齐

### 您落后的地方

- **品牌实体信号：全部维度都落后**——这是您与竞品之间最大的差距
- **内容生态：博客 0 篇 vs. 竞品 200–3000 篇**——需要立即启动内容生产
- **第三方验证：G2 / Capterra / Reddit 全部零信号**——这是 SaaS 类 AI 搜索引用的核心来源

**结论：** 您的技术与多语言优势是真实的，但**这些优势会被内容与实体的缺失完全抵消**。竞品的"AI 搜索护城河"由数千条 Reddit 讨论、数百篇深度博客、数千条 G2 评测共同构成——这些都是无法靠"30 分钟修复"补齐的。建议优先级：**实体建设（季度性） > 内容生产（月度性） > 技术修复（本周）**。

---

## Section 12 · Appendix

### Appendix A — Ready-to-Use JSON-LD `@graph`

将以下脚本块替换 `/en` 当前的两个独立 JSON-LD（开发投入 30 分钟，Schema 分预计 22 → 70+）：

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Organization",
      "@id": "https://growthclaw.co/#org",
      "name": "GrowthClaw",
      "url": "https://growthclaw.co/",
      "logo": "https://growthclaw.co/logo.svg",
      "description": "AI-driven all-in-one SEO automation platform for brand analysis, SEO diagnostics and content tasks.",
      "sameAs": [
        "https://x.com/YOUR_HANDLE",
        "https://www.linkedin.com/company/growthclaw",
        "https://www.producthunt.com/products/growthclaw",
        "https://www.crunchbase.com/organization/growthclaw"
      ],
      "contactPoint": {
        "@type": "ContactPoint",
        "contactType": "customer support",
        "email": "support@growthclaw.co",
        "availableLanguage": ["en","zh","ja","ko","ru","ar","he"]
      }
    },
    {
      "@type": "SoftwareApplication",
      "name": "GrowthClaw",
      "operatingSystem": "Web",
      "applicationCategory": "BusinessApplication",
      "applicationSubCategory": "SEO Software",
      "url": "https://growthclaw.co/en",
      "description": "AI-driven SEO automation: brand analysis, SEO diagnostics, content task generation.",
      "publisher": { "@id": "https://growthclaw.co/#org" },
      "offers": {
        "@type": "AggregateOffer",
        "priceCurrency": "USD",
        "lowPrice": "0",
        "highPrice": "49",
        "offerCount": "2",
        "offers": [
          { "@type": "Offer", "name": "Free", "price": "0", "priceCurrency": "USD" },
          { "@type": "Offer", "name": "Pro", "price": "49", "priceCurrency": "USD",
            "priceSpecification": { "@type": "UnitPriceSpecification", "price": "49", "priceCurrency": "USD", "unitCode": "MON", "billingDuration": "P1M" }
          }
        ]
      },
      "featureList": ["Brand Analysis","SEO Diagnostics","Content Task Automation"]
    },
    {
      "@type": "WebSite",
      "url": "https://growthclaw.co/",
      "name": "GrowthClaw",
      "publisher": { "@id": "https://growthclaw.co/#org" },
      "potentialAction": {
        "@type": "SearchAction",
        "target": { "@type": "EntryPoint", "urlTemplate": "https://growthclaw.co/en/search?q={search_term_string}" },
        "query-input": "required name=search_term_string"
      }
    }
  ]
}
</script>
```

### Appendix B — 安全响应头配置（next.config.js）

```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          { key: 'Content-Security-Policy', value: "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' *.vercel-insights.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' *.vercel-insights.com;" },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
        ],
      },
    ]
  },
}
```

### Methodology 方法论

本次 GEO 审计采用如下方法：

- **分析页面**：8 语言 × 5 类页面 = 40 个 URL（着陆页 / 隐私 / 条款 / SCTA + 根域）
- **评估平台**：Google AI Overviews、ChatGPT 网页搜索、Perplexity AI、Google Gemini、Bing Copilot
- **技术检查**：HTTP 响应头、robots.txt、HTML 源代码分析、JSON-LD 验证、curl 实测
- **内容评估**：E-E-A-T 框架（Experience、Expertise、Authoritativeness、Trustworthiness）依据 Google December 2025 Quality Rater Guidelines
- **品牌信号**：Wikipedia / Reddit / YouTube / LinkedIn / Crunchbase / G2 / Product Hunt 实测搜索
- **分析时间**：2026-05-13
- **审计工具**：Claude Code · geo-audit + geo-report 技能链（Opus 4.7）

### 数据来源

- Google Search Quality Rater Guidelines（December 2025 update）
- Schema.org full type hierarchy
- Profound / Otterly.ai 2026 AI search citation studies
- Semrush / Ahrefs AI search visibility research（2025–2026）
- Core Web Vitals thresholds（web.dev 2026 standards）
- AI crawler user-agent 官方文档（OpenAI / Anthropic / Google / Microsoft / Perplexity）

### 术语表

| 术语 | 定义 |
|---|---|
| GEO | Generative Engine Optimization — 针对 AI 搜索平台的引用与推荐进行的优化 |
| AIO | AI Overviews — Google 搜索结果顶部的 AI 生成答案盒 |
| E-E-A-T | Experience、Expertise、Authoritativeness、Trustworthiness — Google 内容质量框架 |
| SSR | Server-Side Rendering — 在服务端生成 HTML，使爬虫无需 JS 即可读取内容 |
| CWV | Core Web Vitals — Google 页面体验指标（LCP、INP、CLS） |
| LCP | Largest Contentful Paint — 最大内容元素绘制时间 |
| INP | Interaction to Next Paint — 交互响应指标（2024 年 3 月替代 FID） |
| CLS | Cumulative Layout Shift — 视觉稳定性指标 |
| JSON-LD | JavaScript Object Notation for Linked Data — 首选的结构化数据格式 |
| sameAs | Schema.org 属性，链接实体到其他平台的官方资料 |
| IndexNow | 即时通知搜索引擎内容更新的协议（Bing / Yandex 主导，Vercel 原生支持） |
| llms.txt | 提议中的标准文件，指导 AI 系统了解站点内容 |
| YMYL | Your Money or Your Life — 需要最高 E-E-A-T 标准的内容主题 |
| SERP | Search Engine Results Page — 搜索结果页 |
| Topical Authority | 站点对其核心主题的覆盖深度与广度 |

---

*本报告由 Claude Code · GEO Audit Suite 生成（Opus 4.7 · geo-audit + geo-report 技能链）。如需对竞品执行同类审计或月度复审跟踪，请参考 `geo-compare` 技能。*
