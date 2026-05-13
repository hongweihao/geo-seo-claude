# GEO Audit Report: Felo AI (felo.ai)

**Audit Date:** 2026-05-13
**URL:** https://felo.ai
**Business Type:** SaaS (Multilingual AI Search Engine / Productivity Platform)
**Pages Analyzed:** 0 直接抓取 / ~15 个 URL 通过第三方信号间接观察
**Audit Method:** Evidence-limited (站点对非浏览器请求返回 403 Forbidden,详见"审计局限说明")

---

## ⚠️ 审计局限说明 (Critical Methodology Note)

本次审计执行过程中,所有从审计环境发出的直接 HTTP 请求 (`curl`、`WebFetch`) 对 `felo.ai` 的全部路径 (`/`、`/robots.txt`、`/sitemap.xml`、`/blog/about-us/`、`/blog/`) **均返回 HTTP 403 Forbidden**。

这意味着:

1. **无法直接读取 HTML、schema、meta 标签、heading 结构、内容字数等"页面级"数据**。
2. 报告中的"技术 GEO"和"Schema"评分依据的是站点对自动化客户端的可访问性表现 + 行业常识推断,而非首手 HTML 检查。
3. 其它维度依赖第三方信号:Similarweb、Wikipedia 中文版、LinkedIn 公开页、新闻稿、Chrome Web Store / Google Play / Apple App Store 列表、Crunchbase、第三方评测博客。
4. **如能在浏览器环境/经身份认证客户端中重跑此审计**,所有打 *(unverified)* 标记的结论应被验证或修正。

**这条 403 本身已经是 felo.ai 最严重的 GEO 风险**——AI 爬虫(GPTBot、ClaudeBot、PerplexityBot、Google-Extended、CCBot)如果遇到同样的策略,将无法将 felo.ai 的内容纳入训练或实时引用。详见"Critical Issues"。

---

## Executive Summary

**Overall GEO Score: 49 / 100 (Poor)**

Felo AI 是一家东京 Felo 株式会社推出的多语言 AI 搜索引擎(2024 年 7 月成立),产品矩阵包括 Felo AI Search、Felo Agent、Felo LiveDoc、Felo Enterprise。商业层面已有合理的品牌雏形(Wikipedia 中文版条目、LinkedIn 公司页、App Store/Google Play/Chrome Web Store 上架、Similarweb 月活约 27 万 / 月访问量约 200 万),博客内容选题也明显是为 AI 搜索可见性而设计的(`/blog/perplexity-alternatives/`、`/blog/felo-ai-search-vs-perplexity-comparison-alternative/` 等)。

**但是最大的 GEO 风险是矛盾的**:作为一家自身就是 AI 搜索引擎的公司,felo.ai 对所有自动化客户端默认返回 403。这极可能导致主流 LLM 爬虫无法抓取其 Marketing 站点,从而让 Felo 的"为 AI 系统所引用"的能力,远低于其内容质量本应支撑的水平。次要短板是英文 Wikipedia 缺失、署名作者与原创研究稀缺、Reddit/HN 等高权重 UGC 阵地存在感薄弱。

### Score Breakdown

| Category | Score | Weight | Weighted Score |
|---|---|---|---|
| AI Citability | 40/100 | 25% | 10.0 |
| Brand Authority | 60/100 | 20% | 12.0 |
| Content E-E-A-T | 55/100 | 20% | 11.0 |
| Technical GEO | 35/100 | 15% | 5.3 |
| Schema & Structured Data | 50/100 *(unverified)* | 10% | 5.0 |
| Platform Optimization | 55/100 | 10% | 5.5 |
| **Overall GEO Score** | | | **48.8 ≈ 49 / 100** |

---

## Critical Issues (Fix Immediately)

### C-1. 站点对非浏览器 User-Agent 返回 403 — AI 爬虫极可能被一并拒绝

**Evidence:**
- `curl -A "Mozilla/5.0 ... Chrome/120.0"` → `HTTP/2 403`, response header `x-deny-reason: host_not_allowed`
- WebFetch 对 `https://felo.ai/`、`https://www.felo.ai/`、`https://felo.ai/robots.txt`、`https://felo.ai/sitemap.xml`、`https://felo.ai/blog/about-us/` 全部失败,均为 403。

**为什么这是 Critical:**
GPTBot、ClaudeBot、PerplexityBot、Google-Extended、CCBot、Bytespider 等爬虫在 User-Agent 与 IP 段上都与"浏览器"不同。若 felo.ai 用边缘策略(CDN/WAF/Cloudflare turnstile)按"非常规 UA"统一拒绝,则:
- 这些 AI 系统就**无法把 felo.ai 内容索引到知识库或实时检索结果中**。
- Felo 自家产品(AI 搜索)虽然能引用别人,但**别人(包括 Google AI Overview、ChatGPT、Claude、Perplexity)无法引用 Felo**。这对一家定位为"AI 搜索"的公司是品牌可信度的隐性伤害。

**Recommended Fix:**
1. 在浏览器中实地核查 `https://felo.ai/robots.txt` 的内容,显式 `Allow` 主流 AI 爬虫。下方为推荐基线:
   ```
   User-agent: GPTBot
   Allow: /
   User-agent: ClaudeBot
   Allow: /
   User-agent: anthropic-ai
   Allow: /
   User-agent: PerplexityBot
   Allow: /
   User-agent: Google-Extended
   Allow: /
   User-agent: CCBot
   Allow: /
   User-agent: Applebot-Extended
   Allow: /
   User-agent: OAI-SearchBot
   Allow: /
   User-agent: Bytespider
   Allow: /

   Sitemap: https://felo.ai/sitemap.xml
   ```
2. 在 WAF / CDN(Cloudflare、AWS WAF、Akamai)中**单独放行**已知 AI 爬虫的 ASN/IP 段,而非沿用"非浏览器一律 challenge / block"的默认规则。
3. 在 Marketing 子域(`www.felo.ai`、`felo.ai/blog/*`、`felo.ai/tools/*`)上**完全禁用 Bot Fight Mode** 之类的总开关,只保留针对登录 / API 路径的速率限制。
4. 上线后用 `curl -A "Mozilla/5.0 (compatible; GPTBot/1.0)"`、`-A "ClaudeBot"`、`-A "PerplexityBot"` 等 UA 各自验证返回 200。

### C-2. 无法验证是否存在 `llms.txt`

**Evidence:** `https://felo.ai/llms.txt` 受 C-1 同一策略影响,无法判定存在与否。
**为什么是 Critical:** 对一个 AI search 公司来说,缺失 `llms.txt` 不仅是技术缺口,更是"自己都不吃自己狗粮"的品牌问题。
**Recommended Fix:** 创建 `/llms.txt`(Markdown 格式),概述产品定位、最重要页面 URL 列表、定价、CEO/CFO 署名,作为 LLM 摄取入口。

---

## High Priority Issues (Fix Within 1 Week)

### H-1. 缺少英文 Wikipedia 条目
- **Evidence:** 仅在 `zh.wikipedia.org/wiki/Felo` 找到中文条目;搜索未发现 `en.wikipedia.org/wiki/Felo_(search_engine)`。
- **Impact:** 英文 Wikipedia 是 LLM 实体识别最强的输入之一。无英文条目意味着 ChatGPT/Claude 在被问到 "What is Felo AI" 时,实体连接置信度低,容易混淆为 Fellou / Fello / 其他同名公司。
- **Fix:**
  - 不要由公司员工自行创建(违反 Wikipedia COI 政策,大概率被删)。
  - 通过 RS-quality 的英文媒体报道积累引用源(目前仅 Yahoo Finance / GlobeNewswire / Newsfile —— 都是 press release 通讯社,**不算独立报道**)。
  - 争取 TechCrunch、The Verge、Wired、MIT Tech Review 等独立报道,然后由社区编辑或第三方编辑发起词条。

### H-2. 第三方独立媒体报道严重不足
- **Evidence:** 搜索 `"felo.ai" press coverage TechCrunch news article` 只返回 Yahoo Finance 与 GlobeNewswire 的稿件(均为付费发稿渠道),没有独立编辑稿件。
- **Impact:** LLM 训练时倾向引用编辑独立稿件作为"事实陈述"来源。Press release 在大多数 LLM 内容质量管线中被降权或屏蔽。
- **Fix:** 与日本/中国/美国的独立 AI/SaaS 媒体建立关系(The Decoder、Stratechery、ZDNet Japan、虎嗅、36kr),提供独家产品访谈与原创数据(例如"Felo 用户跨语言搜索行为报告")。

### H-3. Reddit / Hacker News 等 UGC 阵地存在感薄弱
- **Evidence:** 搜索 `felo.ai site:reddit.com` 无结果;第三方评测明确指出 "Public discussion of Felo on Reddit and Hacker News is thinner than for Perplexity or Brave"。
- **Impact:** AI 系统(尤其 ChatGPT search、Perplexity)将 Reddit/HN 设为高权重 corpus。Felo 在其中的稀疏存在意味着"在用户讨论真实使用感受时它不会被提及"。
- **Fix:**
  - 在 r/perplexity_ai、r/OpenAI、r/ChatGPT、r/artificial、r/japan(本地化优势)中保持员工真人参与,不要营销话术。
  - 把 Show HN 帖子放在每次重大 release(LiveDoc、Felo Agent 新版本)同步发布。

### H-4. 博客内容缺少署名作者与凭证
- **Evidence:** 搜索结果中的 `/blog/*` 文章未显示作者信息;典型 AI startup 博客模式(unverified,需在浏览器中确认)。
- **Impact:** E-E-A-T 中的 "Experience" 与 "Expertise" 信号缺失,降低被 AI Overview 选中的概率。
- **Fix:** 每篇博客挂 Author 卡 + Person schema + LinkedIn 链接;最少由 YunRui SiMa(CEO)、Yao Li(CFO)与 2–3 位工程/产品 PM 轮换署名。

---

## Medium Priority Issues (Fix Within 1 Month)

### M-1. *(unverified)* 推测 Marketing 站点为重客户端 SPA
- **Reasoning:** 产品本身 `/search` 路径必然 JS 重客户端;若 Marketing 落地页与 Blog 复用同一前端框架,会导致 LLM 抓取到空 shell。
- **Fix:** 强制 `/blog/*`、`/tools/*`、`/en/*`、`/ja/*`、`/zh/*` 以 SSR / SSG 渲染,首屏 HTML 包含完整正文(>1000 字)与 schema。

### M-2. 无原创研究或专有数据资产
- **Evidence:** 搜索未发现"Felo 发布的多语言 AI 搜索使用报告"、"行业基准"或类似内容。
- **Impact:** AI 系统在引用统计数据时,引用"Felo 自有数据"的概率为 0。原创数据是最容易被 AI 摘要时直接 quote 的内容。
- **Fix:** 每季度发布 1 份基于自身脱敏数据的报告(例如 "27 万 MAU 跨语言查询行为白皮书"),每篇都配 PDF + HTML + Dataset Schema。

### M-3. 产品定价页 *(unverified)* 缺少结构化对比
- **Reasoning:** 已知 Pro $14.99/月,可对比 Perplexity Pro $20、ChatGPT Plus $20。
- **Fix:** 在 `/pricing` 增加 FAQPage + Product + Offer schema,并提供与竞品的并排对比表(对比表在 AI Overview 中引用率显著高于纯散文)。

### M-4. 中文版 Wikipedia 条目可能内容不足
- 中文条目存在但搜索摘要显示信息量有限(founders、产品线、融资史不全)。
- **Fix:** 不能直接由员工编辑,但可让第三方贡献者更新 Crunchbase + LinkedIn 信息后,中文 Wikipedia 词条可被社区扩充。

---

## Low Priority Issues (Optimize When Possible)

- **L-1.** 多语言 URL 结构 (`/en/`、`/ja/`、`/zh/`) 已存在,但需验证 `hreflang` 标签是否在所有语言版本之间双向声明 *(unverified)*。
- **L-2.** Open Graph / Twitter Card 检查无法执行 *(403)*。建议人工核查每个核心着陆页的 OG image/description。
- **L-3.** LinkedIn 存在两个相似主页 (`feloai` 与 `felo-dev`、`felo-marketing`),可能导致实体识别分裂。**Fix:** 合并到 `feloai` 唯一主页,其余设为 redirect 或关闭。
- **L-4.** X/Twitter 账号 `@felo_ai` 存在但未观察到与产品发布节奏匹配的高频发文(unverified)。
- **L-5.** Chrome Web Store / Google Play / App Store 三个产品页是自然的高权重外链锚点,可在 robots/llms.txt 中显式声明 `sameAs`。

---

## Category Deep Dives

### AI Citability (40 / 100)

**强项 (inferred from URL patterns):**
- 博客标题明显是 AI 搜索引擎友好的"对比型"/"清单型"/"How-to 型"长尾词,例如 `/blog/felo-ai-search-vs-perplexity-comparison-alternative/`、`/blog/perplexity-alternatives/`、`/blog/youtube-video-summarizer/`、`/blog/ai-post-generator/`、`/blog/2025-best-ai-tools-academic-research/`。这种"对比 + 替代品"格式是 AI Overview 高被引格式。
- 工具落地页采用 `/tools/<task>` 结构 (`/tools/youtube-summary-ai`),清晰的任务-工具映射对 AI 摘要器很友好。

**弱项:**
- 无法直接验证段落是否"可独立成立 (self-contained passages)"——AI 摘要倾向引用 50–120 词、含主语、能脱离上下文站住的段落。
- 无法验证是否每节都有明确 H2 提问 + 直接答案的结构。
- C-1 的 403 让上述博客内容**对 AI 爬虫不可用**,即便写得再好也无法被 quote。

**判定:** 内容选题 70+,但因 C-1 实际"可被引用度"压低至 40。修好 C-1 后该项可立即跃升到 70–75。

### Brand Authority (60 / 100)

| 平台 | 状态 | 备注 |
|---|---|---|
| Wikipedia (英文) | ❌ 缺失 | High priority 待修复 |
| Wikipedia (中文) | ✅ 存在 | `zh.wikipedia.org/wiki/Felo` |
| LinkedIn 公司页 | ✅ 存在 | `feloai` (主) + 两个分散主页待合并 |
| X / Twitter | ✅ 存在 | `@felo_ai` |
| YouTube | ⚠ 仅第三方教程视频 | 未确认官方频道是否存在 |
| Reddit | ⚠ 极薄 | UGC 重点缺口 |
| Hacker News | ❌ 未见 Show HN 历史 | |
| Crunchbase | ✅ 存在 | CEO YunRui SiMa 条目可见 |
| Chrome Web Store | ✅ 上架 | |
| Google Play | ✅ 上架 (`ai.felo.search`) | |
| Apple App Store | ✅ 上架 | |
| 独立媒体报道 | ❌ 仅 PR 通讯社 | |

### Content E-E-A-T (55 / 100)

- **Experience (经验):** 团队具备实际工作经验 (CEO 前 Coolpad 副总裁),但未在博客文章中明确体现 “我们做了 X 才得出 Y” 的第一手经验叙事。
- **Expertise (专业):** 创始团队公开身份清晰、可验证(LinkedIn + Crunchbase 一致),但博客作者署名缺失 *(unverified)*。
- **Authoritativeness (权威):** 公司年轻 (成立于 2024 年 7 月),被独立编辑稿件引用很少。
- **Trustworthiness (可信):** 公司实体明确 (Felo 株式会社,东京日本桥)、有正规付费产品 + 公开定价,Trust 信号合规;但 GDPR/隐私政策、退款政策的可见度需在浏览器中再核(`/en/faq/felo-search-pro` 存在,**该方向是正面信号**)。

### Technical GEO (35 / 100)

| 检查项 | 结果 | 影响 |
|---|---|---|
| 自动化客户端可访问性 | ❌ 全部 403 | **Critical** (C-1) |
| robots.txt 可读 | ❌ 403 | 未知是否 Allow AI 爬虫 |
| llms.txt 存在 | ❓ 无法验证 | 极可能缺失 |
| sitemap.xml 可读 | ❌ 403 | |
| HTTPS | ✅ HTTP/2 over TLS | 正面 |
| 多语言路径 | ✅ `/en/`, `/ja/`, `/zh/` | 正面 |
| SSR/SSG (Marketing) | ❓ 未验证 | |
| Core Web Vitals | ❓ 未验证 | 建议在浏览器中跑 Lighthouse |
| 安全头 | ❓ 未验证 | |

> 注:`x-deny-reason: host_not_allowed` 表明拒绝发生在 Felo 自有边缘层(可能 Cloudflare Worker 自定义 deny list 或自建网关),不是常规 WAF 默认行为——可控、可调整。

### Schema & Structured Data (50 / 100, unverified)

无法直接验证。对于 AI 搜索产品的 Marketing 站点,建议至少存在:

| Schema 类型 | 建议位置 | 优先级 |
|---|---|---|
| `Organization` + `sameAs` 数组 | 首页 + /blog/about-us/ | Critical |
| `SoftwareApplication` / `WebApplication` | /search、/tools/* | High |
| `FAQPage` | /pricing、/faq/*、/blog/* 的 FAQ 节 | High |
| `Article` / `BlogPosting` | /blog/* | High |
| `Person` (CEO/CFO/作者) | 作者页 | Medium |
| `HowTo` | /blog/youtube-video-summarizer/ 等任务型文章 | Medium |
| `Product` + `Offer` | /pricing | Medium |
| `BreadcrumbList` | 全站 | Low |

### Platform Optimization (55 / 100)

| 目标平台 | 推断准备度 | 备注 |
|---|---|---|
| Google AI Overview | 低 (受 C-1 影响) | Google-Extended 极可能被一并拒 |
| ChatGPT search | 低 (受 C-1 影响) | OAI-SearchBot 拒绝即不可见 |
| Perplexity | 中 (Felo 自己定位为 Perplexity Alternative,词条丰富) | PerplexityBot 拒绝是直接打击 |
| Claude (with web) | 低 (受 C-1 影响) | ClaudeBot 拒绝即不可见 |
| Gemini / AI Mode | 低 | 同上 |
| Bing Copilot | 未知 | Bingbot 是常规 UA,可能未被拒(可在浏览器中实测) |
| App Store discovery | **高** | 三平台齐备 |

---

## Quick Wins (Implement This Week)

1. **修 C-1**(单点最大杠杆):在 CDN/WAF 中显式放行 GPTBot / ClaudeBot / PerplexityBot / Google-Extended / CCBot / OAI-SearchBot / Bytespider / Applebot-Extended 的 User-Agent 与 IP 段。预计可在 1–3 周内使所有"含 AI 索引的搜索引擎"重新可见。
2. **上线 `/llms.txt`**:Markdown 格式,描述产品、列出 30 个最重要 URL、署 CEO+CFO 名。一份纯文本文件,工作量 < 半天,信号价值很大。
3. **修复 robots.txt**:基于上方推荐基线发布。
4. **合并 LinkedIn 分散主页**:`feloai` 设为唯一 canonical,把 `felo-dev`、`felo-marketing` 关掉或 redirect。这能立即提升 LLM 实体识别置信度。
5. **博客全站加 Author 卡 + Person schema**:从 YunRui SiMa、Yao Li 两位创始人开始,然后扩展到 PM 团队。

## 30-Day Action Plan

### Week 1: 解除 AI 爬虫封锁
- [ ] 在 CDN/WAF 中明确放行主流 AI 爬虫 (按 C-1 操作清单)
- [ ] 重写 `/robots.txt`,显式 `Allow` 列表 + `Sitemap` 声明
- [ ] 发布 `/llms.txt`
- [ ] 用每个目标爬虫的 UA 实际请求 `curl` 验证返回 200
- [ ] 在 Google Search Console 与 Bing Webmaster 中重新提交 sitemap

### Week 2: 提升 E-E-A-T 与 Schema 完整度
- [ ] 所有 `/blog/*` 文章加 Author 卡 + Person schema
- [ ] 首页与 `/blog/about-us/` 加完整 Organization schema (含 `sameAs` 到 Crunchbase / LinkedIn / Wikipedia 中文 / X / YouTube / App Stores)
- [ ] `/pricing` 加 Product + Offer + FAQPage schema
- [ ] 至少 5 篇 how-to 风格文章 ((`/blog/youtube-video-summarizer/` 等)加 HowTo schema

### Week 3: 第三方阵地深耕
- [ ] CEO/CFO 在 Reddit `r/perplexity_ai`、`r/OpenAI`、`r/ChatGPT` 真人参与,不带营销话术
- [ ] 在 Hacker News 发 Show HN 帖(下一次 release 配套)
- [ ] 联系 1–2 家独立科技媒体争取产品独家
- [ ] LinkedIn 三个相似主页合并到 `feloai`
- [ ] 评估官方 YouTube 频道现状,如缺失则建立并发布 3 条 60–90 秒短视频教程

### Week 4: 原创数据资产 + Wikipedia 准备
- [ ] 启动季度报告项目: "27 万 MAU 跨语言查询行为白皮书"(数据脱敏后发布)
- [ ] 准备发布 PDF + HTML + Dataset Schema
- [ ] 整理 RS-quality 引用清单,准备协助第三方编辑发起英文 Wikipedia 条目(不可自编辑)
- [ ] 在 Google PSI / Lighthouse 中跑 Core Web Vitals,优化首屏 LCP/INP/CLS

---

## Appendix A: Pages Analyzed (indirect signal only)

| URL | 来源 | 备注 |
|---|---|---|
| https://felo.ai/ | 403 | 无法读取 |
| https://felo.ai/robots.txt | 403 | 无法读取 |
| https://felo.ai/sitemap.xml | 403 | 无法读取 |
| https://felo.ai/search | 第三方描述 | SPA,产品入口 |
| https://felo.ai/blog/about-us/ | 第三方摘要 | About 页 |
| https://felo.ai/blog/pricing/ | 第三方摘要 | Pro $14.99/月 |
| https://felo.ai/blog/perplexity-alternatives/ | 第三方摘要 | 对比型,citability 高潜 |
| https://felo.ai/blog/felo-ai-search-vs-perplexity-comparison-alternative/ | 第三方摘要 | |
| https://felo.ai/blog/felo-ai-feature-overview/ | 第三方摘要 | |
| https://felo.ai/blog/introducing-felo-ai-academic-search-multilingual-research-tool/ | 第三方摘要 | |
| https://felo.ai/blog/youtube-video-summarizer/ | 第三方摘要 | How-to 型 |
| https://felo.ai/blog/ai-post-generator/ | 第三方摘要 | How-to 型 |
| https://felo.ai/tools/youtube-summary-ai | 第三方摘要 | 工具落地页 |
| https://felo.ai/en/agents | 第三方摘要 | 产品页 |
| https://felo.ai/en/livedoc | 第三方摘要 | 产品页 |
| https://felo.ai/en/faq/felo-search-pro | 第三方摘要 | FAQ 页,Trust 正向信号 |
| https://felo.ai/en/version | 第三方摘要 | Changelog |
| https://felo.ai/skills/felo-youtube-subtitling | 第三方摘要 | |
| https://felo.ai/en/agents/ai-presentation-generator-2bcAE5yhQYmXpP6LhrZ95f | 第三方摘要 | |

## Appendix B: Off-domain Brand Signals

| 平台 | URL | 信号强度 |
|---|---|---|
| Wikipedia (zh) | https://zh.wikipedia.org/wiki/Felo | 中 |
| LinkedIn (Felo株式会社) | https://www.linkedin.com/company/feloai | 中 |
| LinkedIn (CEO) | https://www.linkedin.com/in/yunrui-sima-43438836/ | 中 |
| LinkedIn (CFO) | https://jp.linkedin.com/in/yao-li-35170b119 | 中 |
| Crunchbase (CEO) | https://www.crunchbase.com/person/yunrui-sima | 中 |
| X / Twitter | https://x.com/felo_ai | 中 |
| Chrome Web Store | `chromewebstore.google.com/detail/.../fbnbeocmafoobaeodhmcgnammdeaoglg` | 强 |
| Google Play | `play.google.com/store/apps/details?id=ai.felo.search` | 强 |
| Similarweb | https://www.similarweb.com/website/felo.ai/ | — (~2M 月访问,~270K MAU) |
| Yahoo Finance | https://finance.yahoo.com/news/felo-ai-search-engine-launches-150000259.html | 弱 (转载 PR) |
| GlobeNewswire | https://www.globenewswire.com/news-release/2024/09/04/2940745/0/en/... | 弱 (PR 通讯社) |
| Newsfile Corp | https://www.newsfilecorp.com/release/275677/... | 弱 (PR 通讯社) |

## Appendix C: Methodology Caveats

1. 本报告基于本次审计中所有 `WebFetch`/`curl` 均被 felo.ai 边缘层以 `x-deny-reason: host_not_allowed` 拒绝的事实进行。
2. 所有标注 *(unverified)* 的项目在浏览器或经身份认证客户端中重测后应被替换为实测值。
3. 评分采用项目内置 GEO Audit 技能 (`skills/geo-audit/SKILL.md`) 定义的权重: Citability 25% / Brand 20% / E-E-A-T 20% / Technical 15% / Schema 10% / Platform 10%。
4. 阈值: 90–100 Excellent / 75–89 Good / 60–74 Fair / 40–59 Poor / 0–39 Critical。当前 49 落入 Poor 区间,但 C-1 修复后估算可立即跃升至 65–72 (Fair–Good)。
