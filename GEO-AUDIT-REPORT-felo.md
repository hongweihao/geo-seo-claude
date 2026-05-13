# GEO 审计报告：Felo.ai

**审计日期：** 2026-05-13
**审计 URL：** https://felo.ai/
**业务类型：** SaaS（多语种 AI 搜索与创作平台）
**运营公司：** Felo Inc.（日本）
**分析页面：** 首页、/enterprise、/blog（10+ 篇文章样本）、/blog/felo-ai-slide-generator-21-tools/、official.felo.ai、robots.txt、sitemap.xml、llms.txt
**子代理：** 5 个并行专项代理（AI 可见性、平台优化、技术 GEO、内容 E-E-A-T、Schema）

---

## 执行摘要

**综合 GEO 得分：48/100（较弱 Poor）**

Felo.ai 存在一个核心悖论：它本身是一款 AI 搜索产品，但它自己的网站对竞争性 AI 引擎（ChatGPT、Claude、Perplexity、Gemini）的可被引用性却处于"较弱"区间。其多语种产品策略（19 个 hreflang）、稳定的博客节奏（2026 年 4–5 月持续更新）、HSTS preload 等技术基础令人鼓舞；但**首页 308 重定向到被 robots.txt 间接屏蔽的 `/search` 路径、缺失 sitemap.xml 与 llms.txt、博客全部署名"Felo Search Tips Buddy"虚构角色、无 `/about` 页面、Organization schema 仅有 2 个 sameAs**——这些缺陷叠加，使 AI 引擎在抓取、解析、信任、引用这四个维度上都受到了严重削弱。

### 三大致命发现
1. 首页 `https://felo.ai/` 308 跳转至 `/search`，而 `/*search?q=` 等路径在 robots.txt 中被 Disallow；canonical 与 hreflang 全部指向被屏蔽路径。这一冲突导致 Felo 在 AI 引擎面前**几乎没有可索引的首页**。
2. 首页 SSR 文本仅 ~210 字符（仅 1 个 H1、0 个 H2/H3）。AI 抓取器（不执行 JS）看到的 Felo 是空壳；其全部产品价值藏在 JS 渲染的对话 UI 后。
3. 全站无 sitemap.xml、无 llms.txt、博客全部使用虚构署名"Felo Search Tips Buddy"、无 `/about`、无团队、无地址、无联系邮箱——E-E-A-T 信号崩溃。

### 分数明细

| 类别 | 得分 | 权重 | 加权分 |
|---|---|---|---|
| AI 可被引用性（Citability） | 42/100 | 25% | 10.5 |
| 品牌权威性（Brand Authority） | 58/100 | 20% | 11.6 |
| 内容 E-E-A-T | 28/100 | 20% | 5.6 |
| 技术 GEO | 58/100 | 15% | 8.7 |
| Schema 结构化数据 | 58/100 | 10% | 5.8 |
| 平台优化 | 58/100 | 10% | 5.8 |
| **综合 GEO 得分** | | | **48.0/100** |

---

## Critical 级问题（立即修复）

### C1. 首页重定向 → robots.txt Disallow 路径
- **现象：** `https://felo.ai/` HTTP 308 → `/search`；同时 robots.txt 含 `Disallow: /*search?q=`、各语言 `/zh-Hans/search/`、`/ja/search/` 等。canonical 与 19 个 hreflang 全部指向 `/search` 系列。
- **影响：** Googlebot、Bingbot、GPTBot、ClaudeBot、PerplexityBot 都被告知"别抓 canonical 指向的页面"。Felo 等于**没有面向爬虫的着陆 URL**。
- **修复：** 在 `https://felo.ai/`（或 `/en`）放置真实的 SSR 营销落地页（600–1000 字可抓取文案 + Organization JSON-LD + FAQ）。更新 canonical/hreflang 指向 locale 根，而非 `/search`。

### C2. 首页 SSR 文本仅 210 字符（AI 抓取器看到空壳）
- **现象：** Next.js SPA 外壳；可见正文仅 "Felo · Upgrade · Search, understand, and create with AI · All docs. Together. Evolving · © 2026 Felo Inc."；67 个 `<script>`，1 个 H1，0 个 H2/H3。
- **影响：** 不执行 JS 的 AI 爬虫（GPTBot、ClaudeBot、PerplexityBot、CCBot）几乎抓不到任何可引用的实质内容。
- **修复：** 在 SSR 层注入"What is Felo（40–60 词定义段）+ 5×Q&A FAQ + 与 Perplexity/ChatGPT 的对比表 + 3 个关键数据（如"支持 20+ 语言"）"。

### C3. 无英文维基百科词条
- **现象：** en.wikipedia.org 仅有 "Felo" 消歧义页（与足球运动员、棒球运动员、工具厂商等混杂），且只链中文维基；不存在 "Felo Inc." 或 "Felo (search engine)" 词条。
- **影响：** AI 模型（尤其 ChatGPT、Gemini）严重依赖维基百科做实体确认。Felo 无法与同名干扰项消歧。
- **修复：** 基于已有的 Yahoo Finance 发布稿、Product Hunt 多次发布、Stackmatix/Communeify 评测等二手来源，撰写并提交 "Felo (search engine)" 英文维基词条，并创建 Wikidata 条目。

### C4. 无 `/about`、无团队、无地址、无联系邮箱
- **现象：** `/about` 返回 404；主域 felo.ai 与 official.felo.ai 都未公开创始人、团队、办公地址、客服邮箱。
- **影响：** 触发 Google 质量评估指南中的"低质量"信号；AI 知识图谱无法建立实体；与对手 Perplexity（创始人公开、媒体覆盖丰富）的信任差距巨大。
- **修复：** 在 felo.ai 主域上线 `/about`，包含创始人姓名+照片、团队、日本注册地址、客服邮箱、成立年份，并配 Organization JSON-LD（含 `founder`、`address`、`sameAs`）。

### C5. 无 sitemap.xml + 无 llms.txt
- **现象：** `/sitemap.xml`、`/sitemap_index.xml` 均 404；robots.txt 未声明 Sitemap；`/llms.txt`、`/llms-full.txt` 均 404。
- **影响：** AI 爬虫只能靠链接发现内容；博客、工具、企业页、Agent、LLM Playground 等关键页可能被忽略。一个名为"AI 搜索"的产品却不发布 llms.txt，象征意义同样负面。
- **修复：** 立即发布多语种 sitemap.xml（带 hreflang alternate）+ llms.txt（手写策划约 30 条核心 URL，按 Product / Tools / Blog / Docs 分组）+ 在 robots.txt 末尾加 `Sitemap:` 指令。

---

## High 级问题（一周内修复）

### H1. 博客全部署名为虚构角色 "Felo Search Tips Buddy"
所有 10+ 篇博客均挂同一虚构署名，无照片、无简介、无 LinkedIn、无 Person schema。AI 模型对 YMYL 边缘话题（AI 工具推荐影响购买决策）的作者身份核验权重很高——虚构署名等同于"无作者"，估算单项扣分 15–20 点。
**修复：** 引入 2–4 位真实有名的编辑/研究员；为每位作者写 100 字简介 + 头像 + LinkedIn + Person JSON-LD（`jobTitle`、`worksFor`、`sameAs`、`knowsAbout`）。

### H2. 缺失 FAQPage Schema（/enterprise 与多篇博客均有可见 FAQ）
- `/enterprise` 有 5 个可见 Q&A，无 FAQPage schema。
- `/blog/felo-ai-slide-generator-21-tools/` 有 6 个可见 Q&A，无 FAQPage schema。
**影响：** Perplexity、ChatGPT 与 Google AI Overviews 高度依赖 FAQ 类结构化数据做问答式引用——这是 GEO 损失最大的单项可结构化资产。
**修复：** 立即上线 FAQPage JSON-LD（模板见附录）。

### H3. 内容存在强 AI 生成痕迹，零外链引用
抽样三篇博客（AI Tweet Generator、GPT Image 2 Prompts、DeepSeek V4）：均无外链引用、无产品截图、无第一手测试笔记；存在套话开头、模糊化（"tends to"、"may"）、论点重复 4 次以上、模板化推销口吻——AI 内容检测特征明显。
**修复：** 每周至少把 1 篇推销文重构为原创研究文（如"我们用 50 个真实研究查询测了 DeepSeek V4 vs GPT-5.4 的结果"），强制全员加入原始来源链接、Felo UI 截图、最后更新日期。

### H4. Organization Schema 的 sameAs 仅 2 个平台
现有 sameAs 只链 X 与 YouTube；缺 LinkedIn、Crunchbase、Wikipedia、Wikidata、GitHub、Facebook。这是 AI 实体绑定的最强单一信号。
**修复：** 扩展到 7+ 平台，并把 logo 升级到 600×600 PNG（当前 32×32 icon.svg 不达 Google 富结果标准）；补 `founder`、`foundingDate`、`contactPoint`。

### H5. robots.txt 未对主流 AI 爬虫做显式 Allow
当前只对 Twitterbot、facebookexternalhit 单独 Allow。GPTBot、ClaudeBot、PerplexityBot、OAI-SearchBot、Google-Extended、Applebot-Extended 都依赖通配符规则。
**修复：** 对上述爬虫做显式 `Allow: /` 声明，并按业务策略加入 `Content-Signal: search=yes, ai-retrieval=yes, ai-train=yes|no`（参考 IETF aipref 草案）。

---

## Medium 级问题（一个月内修复）

### M1. `/compare-ai-engines` 被 robots.txt 屏蔽
对比类查询是 Perplexity、ChatGPT 引用量最高的 AI 搜索词类型之一。屏蔽 `/compare-ai-engines` 等于自废武功。
**修复：** 解除屏蔽（或迁移至 `/compare/`）并加 FAQPage + 对比表 schema。

### M2. WebApplication 应升级为 SoftwareApplication
现 schema 用 `WebApplication`，可加深为 `SoftwareApplication`，补 `aggregateRating`、`softwareVersion`、`screenshot`、`downloadUrl`、`offers`（多个 tier）、`applicationSubCategory: "AI Search Engine"`。这是 AI 模型对比 SaaS 工具时的核心字段。

### M3. 关键词同类相食（cannibalization）
2026-05-08 同一天发布 3 篇近重复主题（AI tweet / social media post / social media writer）。
**修复：** 合并为一篇规范的支柱文，其余 301 重定向。

### M4. CSP 不完整 + 缺 Permissions-Policy
当前 CSP 仅 `frame-ancestors`，无 `default-src`/`script-src`；无 `Permissions-Policy` 头。
**修复：** 补齐 CSP（至少 script-src、style-src、img-src）与 Permissions-Policy。

### M5. INP/CLS 风险
67 个 `<script>` + Twitter 像素 + Clarity 等第三方脚本叠加；logo `<img>` 无宽高，CLS 风险。
**修复：** 延迟/移除非关键第三方脚本；为所有图片补 width/height；为预加载字体加 `font-display: swap`。

### M6. WebSite Schema 缺 SearchAction
无法启用 Google 站内搜索框富结果。
**修复：** 加 `potentialAction: { @type: SearchAction, target: "https://felo.ai/search?q={search_term_string}" }`。

### M7. BlogPosting 缺 dateModified / publisher / wordCount
影响新鲜度信号与发布者识别。
**修复：** 为每篇博客补全。

---

## Low 级问题（择机优化）

- L1. BreadcrumbList 全站缺失。
- L2. `keywords: []` 在 BlogPosting 中为空数组。
- L3. `speakable` 未声明（影响语音助手就绪度）。
- L4. HTML 缓存 `private, no-cache` 对营销页可考虑放宽以降低 TTFB。
- L5. 隐私政策与服务条款位于 `account.felo.ai` 子域，主域未链。
- L6. 无 IndexNow key 文件、无 Bing 验证 `msvalidate.01` 元标签。

---

## 类别深度分析

### AI 可被引用性 42/100
首页基本不可引用："Search, understand, and create with AI" / "All docs. Together. Evolving" 都是品牌诗，不是事实陈述。博客可引用度略好（Slide Generator 文中 "21 specialized slide generators—from PDFs to YouTube links to raw ideas" 评 72/100），但缺 FAQ、缺对比表、缺统计数据、缺自包含定义段。

### 品牌权威性 58/100
**强：** LinkedIn 公司页活跃（Felo 株式会社，CFO Yao Li 公开），YouTube 有第三方评测，Product Hunt 多次发布（Felo、Felo Translator、Felo Visual Workspace 2025-11），行业站点 Stackmatix/Communeify/Yahoo Finance 有覆盖，HN 有零星讨论。
**弱：** 无英文维基词条、Reddit 几乎无自然讨论、Felo 用 Reddit（Agent Search）多于 Reddit 谈 Felo，月访问量约 27 万远落后于 Perplexity 的 ~2.4 亿。

### 内容 E-E-A-T 28/100（全审计最低分）
| 维度 | 得分 | 说明 |
|---|---|---|
| Experience 经验 | 6/25 | 无第一手测试、无 Felo UI 截图、无客户案例 |
| Expertise 专业性 | 5/25 | 零真实作者、零资质、技术深度仅"复述规格表" |
| Authoritativeness 权威 | 8/25 | 无 About、无团队、无媒体引用、无奖项 |
| Trustworthiness 可信度 | 9/25 | 隐私/条款不在主域、无联系方式、无 SOC2/ISO、无更新时间戳、无利益冲突声明 |

抽样三篇博客均呈现强 AI 生成痕迹。

### 技术 GEO 58/100
**强：** HSTS preload (2 年 + includeSubDomains)，HTTP/2 + HTTP/3，HTML lang+translate=no，OG/Twitter 完整，19 hreflang + x-default，移动端响应式，SSR 框架（Next.js + Docusaurus）。
**弱：** 首页 308 → `/search`（被间接屏蔽）、SSR 文本贫瘠、无 sitemap、无 llms.txt、`/compare-ai-engines` 被屏蔽、CSP 不完整、67 脚本 INP 风险。

### Schema 结构化数据 58/100
（**注：** 初次 WebFetch 误报"无 schema"。原始 HTML 检查发现首页有 SSR 的 `@graph`：Organization、WebApplication、WebSite、WebPage 都已存在；博客有 Docusaurus 注入的 BlogPosting + Person。）
**缺口：** FAQPage（关键缺失）、扩展 sameAs、SoftwareApplication 升级、真实 Person、BreadcrumbList、SearchAction、speakable、dateModified。

### 平台优化 58/100
| 平台 | 得分 | 最强 | 最弱 |
|---|---|---|---|
| Google AI Overviews | 62 | 博客标题结构契合 How-to 模板 | 缺 FAQPage / Article schema |
| Google Gemini | 60 | 多语种 + 主题集群 | Organization sameAs 不足 |
| ChatGPT Web Search | 58 | 内容新鲜（2026-05） | 实体图弱、维基缺失 |
| Perplexity | 55 | 多语种内容 + 发布日期 | 无原创研究、Reddit 痕迹弱 |
| Bing Copilot | 55 | 企业版内容贴合工作场景 | 无 sitemap、无 IndexNow |

---

## 快速可执行的 5 项 Quick Wins（本周内）

1. **加 sitemap.xml + llms.txt + robots.txt Sitemap 行。** 工程量小，立即修复 5 大平台中的 4 项发现路径。
2. **/enterprise 加 FAQPage JSON-LD（5 个 Q&A 已有可见内容）。** 立等可取的 AI 引用资产。
3. **Organization sameAs 从 2 个扩展到 7+（LinkedIn / Crunchbase / Wikidata / Facebook / GitHub），logo 换 600×600 PNG。** 一次性修复 ChatGPT/Gemini/Bing Copilot 三大实体识别短板。
4. **首页注入 SSR 营销内容块**（"What is Felo" 定义段 + 5 个 FAQ + 与 Perplexity/ChatGPT 对比表 + 3 个关键数字）。
5. **robots.txt 加显式 AI 爬虫 Allow 行 + Content-Signal 指令。**

---

## 30 天行动计划

### 第 1 周：止血关键基础设施
- [ ] 取消 `/` → `/search` 308 重定向；在 `/` 放置 SSR 营销落地页
- [ ] 上线 `/sitemap.xml`（多语种 + hreflang alternate）
- [ ] 上线 `/llms.txt`（约 30 条 URL）
- [ ] robots.txt 加 `Sitemap:` + 对 GPTBot/ClaudeBot/PerplexityBot/OAI-SearchBot/Google-Extended 显式 Allow
- [ ] 解除 `/compare-ai-engines` 屏蔽（或迁移到 `/compare/`）

### 第 2 周：Schema 与实体绑定
- [ ] 扩展 Organization sameAs 到 7+ 平台；升级 logo 至 600×600
- [ ] `/enterprise` 与博客带 FAQ 的文章加 FAQPage JSON-LD
- [ ] WebApplication 升级为 SoftwareApplication（含 aggregateRating、screenshot、softwareVersion、多 Offer）
- [ ] BlogPosting 补 dateModified / publisher / wordCount
- [ ] WebSite 加 SearchAction、BreadcrumbList 加到 /enterprise /blog /tools

### 第 3 周：信任与作者重建
- [ ] 主域上线 `/about`（创始人、团队、地址、邮箱、年份）+ Organization 配 founder/address/contactPoint
- [ ] 用真实有名作者（2–4 人）替换 "Felo Search Tips Buddy"；为每位作者写 Person schema
- [ ] 在博客全站加"最后更新日期"与"How we test"编辑标准页
- [ ] 把隐私/条款副本镜像到主域 felo.ai/privacy 与 /terms

### 第 4 周：内容资产与品牌实体
- [ ] 撰写并提交英文 Wikipedia 词条 "Felo (search engine)" + Wikidata 条目
- [ ] 发布 1 篇原创研究文（如"Felo 跨 8 语种 vs Perplexity 跨语种检索：50 查询基准"）
- [ ] 合并 2026-05-08 的 3 篇重复博客为 1 篇支柱文 + 301
- [ ] 在 felo.ai 上线"Felo vs Perplexity vs ChatGPT Search"对比页（FAQPage + 对比表 schema）
- [ ] 注册 Bing Webmaster + 提交 IndexNow key

---

## 附录 A：分析的页面

| URL | 标题 | 发现的关键 GEO 问题 |
|---|---|---|
| https://felo.ai/ | Felo — Free Multilingual AI Search & Creation Platform | 308 → /search、SSR 文本 210 字符、canonical 指向被屏蔽路径 |
| https://felo.ai/enterprise | Enterprise Pro - Felo | FAQ 区无 FAQPage schema、无定价 schema |
| https://felo.ai/blog | Felo Search Blog | 虚构作者、无 Article schema 列表页、无 RSS |
| https://felo.ai/blog/felo-ai-slide-generator-21-tools/ | Felo AI Slide Generator: 21 Tools | BlogPosting 缺 dateModified/publisher、Person 是虚构、6 Q&A 无 FAQPage |
| https://felo.ai/robots.txt | — | 无 AI 爬虫显式 Allow、无 Sitemap、屏蔽 /compare-ai-engines |
| https://felo.ai/sitemap.xml | — | 404 |
| https://felo.ai/sitemap_index.xml | — | 404 |
| https://felo.ai/llms.txt | — | 404 |
| https://felo.ai/about | — | 404 |
| https://official.felo.ai/ | Felo – AI検索・リアルタイム翻訳 ｜ Felo 株式会社 | 无地址、无团队、无 Organization schema |

---

## 附录 B：可直接复用的 JSON-LD 模板（节选）

详见 geo-schema 子代理输出。三项最高优先级：
1. 扩展版 Organization（含 7+ sameAs、founder、address、contactPoint）
2. SoftwareApplication（带 aggregateRating、多 Offer、featureList、screenshot）
3. /enterprise 的 FAQPage（5 个 Q&A）

均已在 schema 子代理报告中提供模板，按 `[REPLACE: ...]` 占位填入真实值即可。

---

## 一句话总结

> **Felo 拥有 Next.js + SSR 框架、HTTPS preload、多语种 hreflang 的"豪车底盘"，却把首页变成被自家 robots 屏蔽的空壳、用虚构作者发布无引用的 AI 推销文、跳过了 sitemap 与 llms.txt 这类 30 分钟就能上线的资产。综合 48/100 不是因为缺能力，而是因为没有把基础卫生工作做完。第 1 周的 P0 修复就能把综合分推到 60+。**
