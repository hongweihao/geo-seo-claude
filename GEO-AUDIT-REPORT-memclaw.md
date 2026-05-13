# GEO 审计报告：MemClaw (memclaw.me)

**审计日期：** 2026-05-13
**站点 URL：** https://memclaw.me
**业务类型：** SaaS（OpenClaw 持久化项目记忆产品，由 Felo Inc. 提供）
**分析页面数：** 13（首页 + 6 个核心页 + 2 个语言版本 + 博客索引 + 2 篇博客 + sitemap + llms.txt）
**审计工具：** geo-audit（5 个并行子代理）

---

## 执行摘要

**总体 GEO 分数：57/100（评级：Poor / 接近 Fair 下限）**

MemClaw 在**技术 GEO 基础设施**和**llms.txt 实施**上表现优异（行业领先级别），并且已经部署了相当完整的 Schema.org 结构化数据（Organization、SoftwareApplication、FAQPage、HowTo、BlogPosting 等）。SSR 渲染、多语言架构（18 种语言、127 条 URL）、安全头部基础配置均合格。

但是**两项致命短板**严重拉低总分：
1. **品牌权威性（18/100）** — 维基百科、Reddit、HN、ProductHunt 均无任何出现；GitHub 上还存在 5+ 个同名仓库导致 AI 实体识别歧义。
2. **内容 E-E-A-T（41/100）** — 7 篇博客无一署名，且全部使用同一发布日期 2026-05-11，缺少 About/Team/Privacy/Terms 页面，安全页只有"声明"无 SOC2/GDPR 证据。

此外发现一个**关键 Bug**：所有 BlogPosting 的 `image` 字段是损坏的 token（`https://memclaw.me01KQA8RZX7Z6G3JS6CXC3AT7GC`），导致 Google 富结果失效。

### 分类得分细分

| 类别 | 分数 | 权重 | 加权分 |
|---|---|---|---|
| AI 可引用性（Citability） | 78/100 | 25% | 19.5 |
| 品牌权威性（Brand Authority） | 18/100 | 20% | 3.6 |
| 内容 E-E-A-T | 41/100 | 20% | 8.2 |
| 技术 GEO | 86/100 | 15% | 12.9 |
| Schema 结构化数据 | 68/100 | 10% | 6.8 |
| 平台优化（AIO/ChatGPT/Perplexity/Gemini/Bing） | 60/100 | 10% | 6.0 |
| **总分** | | | **57.0/100** |

---

## 严重问题（必须立即修复）

### C1. BlogPosting 的 `image` 字段全部损坏
- **影响范围：** 所有 `/blog/posts/*` 页面
- **证据：** Schema 中 `image` 值为 `"https://memclaw.me01KQA8RZX7Z6G3JS6CXC3AT7GC"`（缺少 `/`，疑似 CMS token 未渲染）
- **后果：** Google 富结果失效；AI 抓取器获取错误的封面图；BlogPosting schema 整体被判为无效
- **修复：** 立即检查博客模板中的 image 资产引用逻辑

### C2. 品牌实体识别危机
- **证据：** Wikipedia / Reddit / HackerNews / ProductHunt 上 MemClaw 与 Felo Inc. 均零搜索结果；GitHub 上存在至少 5 个同名 "memclaw" 仓库（emanuilo/memclaw、GIglss/memclaw、yoloshii/ClawMem、memovai/mimiclaw 等）
- **后果：** ChatGPT / Claude / Perplexity / Gemini 无法将 "MemClaw" 唯一映射到 Felo 产品
- **修复：** 创建 Wikidata 条目；申请 Wikipedia 草稿；ProductHunt + HackerNews Show HN 发布

### C3. 关键信任页面缺失
- **缺失：** /about、/privacy、/terms、/contact 全部不可见或未在 footer 中链接
- **后果：** 对于"存储用户记忆数据"的 SaaS 来说是企业采购红线；触发 GDPR 合规风险
- **修复：** 1 周内补齐 4 个法律 / 信任页面，并加入全站 footer

---

## 高优先级问题（1 周内修复）

### H1. 博客全部无作者署名 + 全部同一发布日（2026-05-11）
- 7 篇博客无一篇有 byline，且打包同日发布 → 强烈的 AI 批量生成嫌疑信号
- 修复：补加作者 Person schema（`@type: Person` + `worksFor` + `sameAs`）；回填或追加"Updated"日期

### H2. 安全页面只有断言无证据
- 缺少 SOC2 / GDPR / ISO27001 / 加密算法 / 数据驻留 / DSAR 流程
- 修复：升级 /security 加入加密算法（AES-256/TLS 1.3）、云提供商、安全路线图

### H3. robots.txt 缺少显式 AI 爬虫白名单
- 现状：`User-agent: *  Allow: /`（隐式允许，但无显式信号）
- 修复：
  ```
  User-agent: GPTBot
  Allow: /
  User-agent: ClaudeBot
  Allow: /
  User-agent: PerplexityBot
  Allow: /
  User-agent: Google-Extended
  Allow: /
  User-agent: OAI-SearchBot
  Allow: /
  User-agent: Applebot-Extended
  Allow: /
  ```

### H4. 根路径 302/307 重定向链
- `/` → 302 `/claw` → 307 `/en/claw`
- 应改为 **301** 以合并权重，或基于 Accept-Language 一步直达 `/en/claw`

### H5. Sitemap 缺少 `<lastmod>`
- 127 条 URL 均无 `<lastmod>` 标签，降低重抓取优先级
- 修复：为每条 URL 添加 ISO-8601 格式 lastmod

### H6. Organization Schema 缺少完整 sameAs
- 现状：仅作为字符串 publisher 引用，未独立部署
- 修复：在 `<head>` 全站注入完整 Organization schema（含 GitHub、Discord、LinkedIn、X、Crunchbase、Wikidata 等 sameAs）

---

## 中优先级问题（1 个月内修复）

| ID | 问题 | 建议 |
|---|---|---|
| M1 | HSTS max-age = 182 天，未达 1 年推荐值 | 升级为 `max-age=31536000; includeSubDomains; preload` |
| M2 | 缺少完整 CSP（仅 `frame-ancestors`） | 补充 `script-src`、`default-src` |
| M3 | 缺少 `Permissions-Policy` 和显式 `X-Frame-Options` | 添加 |
| M4 | 实证密度低 — 整页几乎无数字、对比表、基准数据 | 补一张 MemClaw vs Mem0 vs ChatGPT Memory vs Claude Projects 的对比表（含 token 成本、延迟） |
| M5 | 用例与证言全为匿名 | 替换为 2 个具名 mini case study + 量化结果 |
| M6 | 缺少 Bing Webmaster 验证（msvalidate.01） + IndexNow | 接入 |
| M7 | 缺少 YouTube 演示视频 | 上传 2-3 个 60s 安装 / 对比视频 |
| M8 | `WebSite` schema 缺少 `SearchAction` | 添加站内搜索 Action |
| M9 | `/blog` 索引页用 bare `WebSite` 而非 `Blog` + `ItemList` | 升级为 Blog 类型 + 文章 ItemList |
| M10 | `BlogPosting.publisher` 用 "MemClaw"，`/security Article` 用 "Felo Inc." | 统一为 `@id` 引用同一 Organization |
| M11 | llms-full.txt 在根路径返回 404（仅 `/claw/llms-full.txt` 存在） | 在根路径也发布一份 |
| M12 | Cache-Control: no-store 用于全部营销页 | 对非个性化页改为 `public, max-age=300, s-maxage=3600` |

---

## 低优先级问题（有余力时优化）

- L1: 缺少 `speakable` 属性（FAQ 答案 + 安全要点是理想候选）
- L2: 多数 schema 缺 `inLanguage` 标签
- L3: 首页 H1 可能渲染为带样式 `<div>` 而非语义 `<h1>`（需 view-source 验证）
- L4: 首页 emoji（🧠⚡🌐）夹杂段落中，污染机器抽取
- L5: 缺少 meta description 标签
- L6: HowTo schema 已被 Google 取消富结果（2023-09），但保留对 AI 仍有语义价值
- L7: bare `/claw`（无语言前缀）增加一跳 307，建议直接 301 到 `/en/claw`

---

## 分类深度分析

### AI 可引用性（78/100）

**亮点：**
- FAQ 内容是教科书级可引用：`"All data is encrypted in transit and at rest"`、`"The context window is what OpenClaw can actively process in the current conversation. OpenClaw memory is what can be recalled over time."` 这种短小、自包含、定义性的句子，AI 提取即用。
- 多个清晰的"痛点 → 一句话方案"结构，AI 适合做总结性引用。

**不足：**
- 统计密度极低 — 全站只有 `"5 个项目和 6 个客户"` 一个数字，没有基准数据、性能指标、对比数字。
- 首页 emoji 污染机器抽取。

### 品牌权威性（18/100）

| 平台 | 状态 |
|---|---|
| Wikipedia | 缺失 |
| Reddit | 零结果 |
| HackerNews | 零结果 |
| ProductHunt | 缺失 |
| GitHub | 存在但与 5+ 同名仓库冲突 |
| LinkedIn | 极少 |
| YouTube | 缺失 |
| 自有博客（felo.ai） | 较强（≥7 篇 MemClaw 文章） |

**最大单点提升：** 创建 Wikidata + Felo Inc. 维基百科条目；这单一动作可将 Brand 从 18 提升到 50+。

### 内容 E-E-A-T（41/100）

| 支柱 | 分数 |
|---|---|
| Experience | 45 |
| Expertise | 38 |
| Authoritativeness | 28 |
| Trustworthiness | 52 |

**AI 内容风险等级：中** — 文章本身行文质量尚可（具体工具名 / 代码片段 / 真实截图），但全部同日发布 + 全部无署名仍触发批量生成红旗。

### 技术 GEO（86/100）

- ✅ SSR 验证通过（H1、价值主张、FAQ 内容均在初始 HTML 中）— 这是最大的 GEO 风险点已规避
- ✅ HTTPS、HTTP/2、HTTP/3、Brotli、Cloudflare CDN
- ✅ 18 种语言 hreflang 配置正确，含 x-default
- ✅ Canonical 自指正确
- ⚠️ 重定向链是 302/307（应为 301）
- ⚠️ HSTS、CSP、Permissions-Policy 不完整

### Schema 结构化数据（68/100）

**已有：** WebPage、FAQPage、SoftwareApplication、Offer、BreadcrumbList、HowTo、Article、BlogPosting、WebSite

**关键缺失：** 独立 Organization（含 sameAs）、Person（作者）、WebSite.SearchAction、speakable

**关键 Bug：** BlogPosting.image 值损坏（C1）

### 平台优化（60/100）

| 平台 | 分数 |
|---|---|
| ChatGPT Search | 68（最强 — llms.txt 极佳） |
| Google AI Overviews | 65 |
| Google Gemini | 58 |
| Microsoft Copilot / Bing | 56 |
| Perplexity AI | 55（最弱 — Reddit/HN/社区无影响力） |

---

## Quick Wins（本周即可实施）

1. **修复 BlogPosting.image 损坏 Token** — 1 小时工作量，立即恢复 7+ 篇博客的 Google 富结果资格
2. **robots.txt 显式列出 AI 爬虫白名单**（GPTBot/ClaudeBot/PerplexityBot/Google-Extended/OAI-SearchBot/Applebot-Extended）— 10 分钟
3. **根路径 302/307 全部改为 301** — 30 分钟（Cloudflare 规则或源站配置）
4. **Sitemap 加 `<lastmod>` 字段** — 1 小时（取决于 CMS 是否原生支持）
5. **博客文章补加作者 byline + Person schema** — 半天（先挑 1 个真实作者即可）

---

## 30 天行动计划

### Week 1：止血修复（紧急）
- [ ] 修复 BlogPosting.image 损坏 Token（C1）
- [ ] 创建 /about、/privacy、/terms、/contact 4 个页面并接入 footer（C3）
- [ ] robots.txt 显式 AI 爬虫白名单（H3）
- [ ] 302/307 → 301 重定向（H4）
- [ ] Sitemap 加 `<lastmod>`（H5）

### Week 2：实体与权威建设
- [ ] 创建 Wikidata 条目（MemClaw 实例 + Felo Inc. 实例）
- [ ] 准备 Wikipedia 草稿（先 Felo Inc.，更易过审）
- [ ] ProductHunt + HackerNews Show HN 联合发布
- [ ] 部署全站 Organization schema（含完整 sameAs）（H6）

### Week 3：内容 E-E-A-T 升级
- [ ] 7 篇博客补加 Person 作者 + 修订发布日期分散化（H1）
- [ ] /security 页面从断言改证据：加密算法 + 云提供商 + 安全路线图（H2）
- [ ] 增加 1 张 MemClaw vs Mem0 vs ChatGPT Memory vs Claude Projects 对比表（含真实数字）（M4）
- [ ] 替换 2 个匿名证言为具名 mini case study（M5）

### Week 4：平台扩张
- [ ] Bing Webmaster 验证（msvalidate.01）+ IndexNow 接入（M6）
- [ ] 上传 2-3 个 YouTube 演示视频（M7）
- [ ] 在 r/LocalLLaMA / r/OpenClaw 等子版发布带基准数据的产品贴
- [ ] 升级 /blog schema：WebSite → Blog + ItemList，加 SearchAction（M8, M9）
- [ ] 安全头部全部强化：HSTS preload、完整 CSP、Permissions-Policy（M1, M2, M3）

---

## 附录：已分析页面

| URL | 标题 | 主要 GEO 问题 |
|---|---|---|
| https://memclaw.me/ | (302 → /claw) | 302 应为 301 |
| https://memclaw.me/en/claw | Give your OpenClaw a second brain | emoji 污染；缺 Organization schema |
| https://memclaw.me/en/claw/faq | FAQ | FAQPage schema ✓ 已实施 |
| https://memclaw.me/en/claw/openclaw-memory | OpenClaw memory 概念页 | 技术深度不足 |
| https://memclaw.me/en/claw/use-cases | Use Cases | 全匿名角色，缺案例研究 |
| https://memclaw.me/en/claw/security | Security | 无 SOC2/GDPR 等证据 |
| https://memclaw.me/en/claw/pricing | Pricing | 仅 Free 一档，缺 priceValidUntil/seller |
| https://memclaw.me/en/claw/install | Install | HowTo schema ✓ |
| https://memclaw.me/blog | Blog 索引 | 缺 ItemList；缺 SearchAction |
| https://memclaw.me/blog/posts/ai-context-bleed | AI Context Bleed | BlogPosting.image 损坏 |
| https://memclaw.me/llms.txt | (200 OK, 2985B) | 优秀，仅 llms-full.txt 链接需修正 |
| https://memclaw.me/robots.txt | (200 OK) | 缺 AI 爬虫显式白名单 |
| https://memclaw.me/claw/sitemap.xml | (127 URLs) | 缺 lastmod |

---

## 结论

MemClaw 已具备**优秀的技术 GEO 底盘**（SSR + 18 语言 + 完整 Schema 栈 + 高分 llms.txt），距离 75+ 的"Good"评级仅剩两步：

1. **修复一个 Bug + 补三个法律页**（1 周可完成）
2. **建立实体权威性**（Wikidata + Wikipedia + Reddit/HN + 作者署名）

完成这两步后，总分预计可从 **57 提升至 75-80**，进入主流 AI 搜索引擎的"高引用候选池"。

最大的风险不是技术，而是**品牌实体在 AI 训练数据 / 知识图谱中不存在**。这一点不解决，Schema 再完美也只能服务 AI 抓取，而无法服务 AI 推荐。
