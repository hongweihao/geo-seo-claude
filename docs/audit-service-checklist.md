# GEO 审计服务 - 实现 Checklist

> **用途:** 给自建 GEO 审计 API 服务用的精简清单。每条 check 都可机械实现,失败时按 `Issue + Spec + Plan` 三段输出。
> **输入:** 用户提交 domain
> **输出:** 给客户 Agent 执行的报告(JSON / Markdown)

---

## 1. 服务整体流程

```
POST /audit { "domain": "example.com" }
    │
    ▼
[1] 抓取主页 + robots.txt + sitemap.xml + llms.txt
[2] 抓取 3-5 个内页(sitemap 中前 5 个)
[3] 逐项跑下面的 check
[4] 每项失败 → 生成一个 Issue(含 Spec + Plan)
[5] 汇总输出
    │
    ▼
{
  "domain": "...",
  "overall_score": 0-100,
  "category_scores": { ... },
  "issues": [
    { "issue": {...}, "spec": {...}, "plan": [...] },
    ...
  ]
}
```

---

## 2. Checklist(35 项,按类目分组)

每项格式:**`ID | 类目 | 检测方法 | 触发条件 | 严重度 | 权重`**

### 2.1 AI 爬虫可达性(8 项,总权重 15%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `CRAWL-001` | `GET /robots.txt` 解析 | GPTBot `Disallow: /` 或 `Disallow: 任何路径` 命中关键页 | Critical | 3 |
| `CRAWL-002` | 同上 | ClaudeBot 被阻止 | Critical | 2 |
| `CRAWL-003` | 同上 | PerplexityBot 被阻止 | Critical | 2 |
| `CRAWL-004` | 同上 | Google-Extended 被阻止 | High | 1 |
| `CRAWL-005` | 同上 | OAI-SearchBot 或 ChatGPT-User 被阻止 | High | 2 |
| `CRAWL-006` | 同上 | Bingbot 被阻止 | High | 1 |
| `CRAWL-007` | `GET /robots.txt` | robots.txt 返回 404 | Medium | 1 |
| `CRAWL-008` | `GET /robots.txt` | 文件中未引用 `Sitemap:` | Medium | 1 |

### 2.2 服务端渲染(3 项,总权重 12%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `SSR-001` | `curl` 抓主页,统计 `<body>` 文字字数 | 词数 < 200 且存在 `<div id="root\|app\|__next">` | Critical | 6 |
| `SSR-002` | 检测 JSON-LD `<script type="application/ld+json">` | 主 HTML 中无 JSON-LD,但 JS 加载后有 | High | 3 |
| `SSR-003` | 比较 raw HTML vs `WebFetch` 渲染后 markdown | 主内容只出现在渲染版 | Critical | 3 |

### 2.3 llms.txt(1 项,总权重 3%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `LLMS-001` | `GET /llms.txt` | 返回 404,或返回 200 但缺 `# 标题` / `> 描述` / `## 章节` / `- [链接]` | Medium | 3 |

### 2.4 结构化数据(6 项,总权重 10%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `SCHEMA-001` | 解析主页所有 `<script type="application/ld+json">` | 无任何 JSON-LD | Critical | 3 |
| `SCHEMA-002` | 同上 | 缺 `@type=Organization`(或 LocalBusiness / Person) | Critical | 2 |
| `SCHEMA-003` | 同上 | Organization 缺 `sameAs` 数组,或数组少于 3 个权威平台链接 | High | 2 |
| `SCHEMA-004` | 同上 | 内页是 Article/BlogPosting 但缺 `author` + `datePublished` | High | 1 |
| `SCHEMA-005` | 同上 | JSON 解析失败(语法错误) | High | 1 |
| `SCHEMA-006` | 比较 `curl` 主页 vs `WebFetch` 主页 | JSON-LD 仅在 JS 注入后出现 | Medium | 1 |

### 2.5 页面基础信号(5 项,总权重 8%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `META-001` | 解析 `<title>` | 缺失或长度 < 10 字符 / > 70 字符 | High | 2 |
| `META-002` | 解析 `<meta name="description">` | 缺失或长度 < 50 / > 160 字符 | High | 2 |
| `META-003` | 解析 `<link rel="canonical">` | 缺失或指向不同 origin | High | 2 |
| `META-004` | 数 `<h1>` 数量 | 0 个或 ≥ 2 个 | Medium | 1 |
| `META-005` | H1-H6 层级 | 有跳级(H1 后直接 H3) | Low | 1 |

### 2.6 内容引用度(5 项,总权重 15%)

每项对**所有 H2/H3 段落**逐段跑,失败比例 > 50% 才触发。

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `CITE-001` | 每段词数 | < 50 或 > 300 词的段落占比 > 50% | High | 4 |
| `CITE-002` | 段首 60 词正则:`\b(?:is\|are\|means\|refers)\b` 或 `\d+%` 或 `\$\d+` | 命中段落占比 < 30% | High | 4 |
| `CITE-003` | 代词密度:`\b(it\|they\|this\|that)\b` ÷ 总词数 | > 6% 的段落占比 > 40% | Medium | 3 |
| `CITE-004` | 统计密度:`\d+%`、`$\d+`、`年份` 命中数 | 每 500 词 < 1 个数据点 | Medium | 2 |
| `CITE-005` | 问句式标题(以 `?` 结尾) | 占比 < 10% | Low | 2 |

### 2.7 品牌实体(2 项,总权重 8%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `BRAND-001` | `GET https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch={brand}` | 无结果,或 top result.title 不含品牌名 | High | 4 |
| `BRAND-002` | `GET https://www.wikidata.org/w/api.php?action=wbsearchentities&search={brand}` | 无实体 | High | 4 |

> **品牌名提取规则:** 从主页 `<title>` 或 Organization schema 的 `name` 取。

### 2.8 安全 / 性能(5 项,总权重 10%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `SEC-001` | URL scheme | 非 HTTPS | Critical | 3 |
| `SEC-002` | HTTPS 证书 | 过期 / 无效 | Critical | 2 |
| `SEC-003` | HTTP 响应头 | 缺 `Strict-Transport-Security` | Medium | 1 |
| `SEC-004` | HTTP 响应头 | 缺 `X-Content-Type-Options: nosniff` | Low | 1 |
| `PERF-001` | TTFB:`curl -w '%{time_starttransfer}'` | > 1.5 秒 | High | 3 |

### 2.9 移动适配(2 项,总权重 5%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `MOB-001` | 解析 `<meta name="viewport">` | 缺失或不含 `width=device-width` | High | 3 |
| `MOB-002` | 主 HTML grep `width="`, `style="width:` | 含 `width=\d{4}px` 等绝对像素超过 viewport | Low | 2 |

### 2.10 内部链接(3 项,总权重 4%)

| ID | 检测 | 触发条件 | 严重度 | 权重 |
|----|------|---------|-------|------|
| `LINK-001` | 主页内链数 | < 10 个 | Medium | 1 |
| `LINK-002` | sitemap.xml 中 URL 数 | < 5 个 | Medium | 2 |
| `LINK-003` | 主页所有 `<a>` 文本 | 含 "click here" / "read more" 占比 > 30% | Low | 1 |

---

## 3. 总分计算

```
overall_score = 100 - Σ(failed_check.weight × severity_multiplier)

severity_multiplier:
  Critical = 1.0
  High     = 0.7
  Medium   = 0.4
  Low      = 0.2

总权重之和 = 100(35 项,权重已经标好)
```

分数下限 0,上限 100。

---

## 4. 输出格式:每个 Issue 的三段结构

每个失败的 check 产出一个对象:

```json
{
  "issue": {
    "id": "CRAWL-001",
    "category": "crawlability",
    "severity": "critical",
    "title": "GPTBot is blocked in robots.txt",
    "evidence": {
      "source": "https://example.com/robots.txt",
      "matched_line": "User-agent: GPTBot\nDisallow: /",
      "checked_at": "2026-05-09T10:30:00Z"
    },
    "impact": "Content is invisible to ChatGPT search. Affects 300M+ weekly users."
  },
  "spec": {
    "goal": "Allow GPTBot to crawl all public content",
    "acceptance_criteria": [
      "robots.txt MUST contain a User-agent: GPTBot block",
      "Under that block, either omit Disallow OR use Disallow: with empty value OR Allow: /",
      "Verify: curl -s https://example.com/robots.txt | grep -A2 'User-agent: GPTBot' returns no blocking Disallow"
    ],
    "non_goals": [
      "Not changing access for non-AI crawlers",
      "Not changing private paths protection (e.g., /admin)"
    ],
    "constraints": [
      "Must not break existing Googlebot rules",
      "Changes must deploy without cache invalidation issues"
    ]
  },
  "plan": [
    {
      "step": 1,
      "action": "Locate robots.txt source",
      "details": "Check /public/robots.txt or framework config (next.config.js / nuxt.config.js / etc.)",
      "owner": "developer",
      "estimate_minutes": 15
    },
    {
      "step": 2,
      "action": "Replace blocking rule with allow",
      "details": "Change 'User-agent: GPTBot\\nDisallow: /' to 'User-agent: GPTBot\\nAllow: /'",
      "owner": "developer",
      "estimate_minutes": 5
    },
    {
      "step": 3,
      "action": "Deploy to production",
      "details": "Standard deploy pipeline",
      "owner": "developer",
      "estimate_minutes": 10
    },
    {
      "step": 4,
      "action": "Verify fix",
      "details": "curl -s https://example.com/robots.txt | grep -A2 GPTBot",
      "owner": "developer",
      "estimate_minutes": 5
    }
  ]
}
```

---

## 5. 完整审计响应 schema

```json
{
  "domain": "example.com",
  "audit_id": "AUDIT-2026-05-09-001",
  "audited_at": "2026-05-09T10:30:00Z",
  "overall_score": 47,
  "category_scores": {
    "crawlability": 12,
    "ssr": 18,
    "llmstxt": 0,
    "schema": 30,
    "meta": 70,
    "citability": 45,
    "brand": 0,
    "security": 80,
    "mobile": 90,
    "links": 60
  },
  "summary": {
    "total_checks": 35,
    "passed": 18,
    "failed": 17,
    "critical_count": 3,
    "high_count": 8,
    "medium_count": 4,
    "low_count": 2
  },
  "issues": [
    { "issue": {...}, "spec": {...}, "plan": [...] }
    // 按严重度排序,Critical 在前
  ]
}
```

---

## 6. 每条 check 的 Spec/Plan 模板速查

| Check ID | Spec 模板要点 | Plan 步骤数 | 工时估计 |
|----------|--------------|-----------|---------|
| CRAWL-001~006 | "在 robots.txt 中允许 {bot} 访问" | 3-4 | 30-45 分钟 |
| CRAWL-007 | "创建 robots.txt 含基础 Allow + Sitemap" | 3 | 30 分钟 |
| CRAWL-008 | "在 robots.txt 末尾追加 `Sitemap:` 行" | 2 | 15 分钟 |
| SSR-001 | "上 SSR 框架(Next.js / Nuxt / Astro)" | 5-8 | 2-4 周 |
| SSR-002 | "把 JSON-LD 写进 HTML 模板,不依赖 JS 注入" | 3 | 1-2 天 |
| LLMS-001 | "在域名根目录添加 llms.txt(标题+描述+章节+链接)" | 4 | 2-4 小时 |
| SCHEMA-001 | "在主页添加最小 Organization JSON-LD" | 3 | 1 天 |
| SCHEMA-002 | "添加业务类型对应的 schema(LocalBusiness/Product/Article)" | 4 | 2-3 天 |
| SCHEMA-003 | "扩充 sameAs 数组,覆盖 Wikipedia/Wikidata/LinkedIn/YouTube/Twitter" | 3 | 2 小时 |
| SCHEMA-004 | "Article schema 加 author + datePublished + dateModified" | 3 | 半天 |
| META-001 | "为每页设置 10-70 字符的独特 title" | 3 | 1-2 天 |
| META-002 | "为每页设置 50-160 字符的 description" | 3 | 1-2 天 |
| META-003 | "添加自指向的 canonical 链接" | 2 | 半天 |
| META-004 | "确保每页恰好 1 个 H1" | 3 | 半天 |
| META-005 | "修复标题层级,去除跳级" | 2 | 1 天 |
| CITE-001 | "把超长段落拆成 50-200 词的独立段" | 5+ | 按页数估,1 页 ~1 小时 |
| CITE-002 | "重写段首,在前 60 词内给出直接答案" | 3 | 按页数估,1 页 ~30 分钟 |
| CITE-003 | "替换段落开头的代词,改用主语名词" | 3 | 按页数估 |
| CITE-004 | "为每个核心论点加 1 个统计或来源" | 4 | 按页数估,1 页 ~30 分钟 |
| CITE-005 | "把 30% 以上的 H2/H3 改成问句式" | 3 | 半天 |
| BRAND-001 | "建立 Wikipedia 页面(需满足 notability)" | 8 | 2-3 个月 |
| BRAND-002 | "建立 Wikidata 实体(无 notability 要求)" | 5 | 1-2 天 |
| SEC-001 | "强制 HTTPS + 301 重定向" | 4 | 1 天 |
| SEC-002 | "更新 / 续期 SSL 证书" | 3 | 半天 |
| SEC-003 | "在响应头添加 HSTS" | 2 | 1 小时 |
| SEC-004 | "添加 X-Content-Type-Options: nosniff" | 2 | 1 小时 |
| PERF-001 | "降低 TTFB(缓存 / CDN / DB 优化)" | 5+ | 1-2 周 |
| MOB-001 | "添加 viewport meta tag" | 2 | 30 分钟 |
| MOB-002 | "修复绝对像素宽度,改用响应式单位" | 3-5 | 1-3 天 |
| LINK-001 | "增加主页内链至 10+ 个核心页" | 3 | 半天 |
| LINK-002 | "生成完整 sitemap.xml" | 3 | 半天 |
| LINK-003 | "把模糊锚文本改成描述性文本" | 3 | 1-2 天 |

---

## 7. 实现要点

### 7.1 取证缓存
按 `domain` + 24h TTL 缓存这些数据(避免重复抓取):
- 主页 HTML(`curl` 原始版本)
- robots.txt
- sitemap.xml
- llms.txt
- Wikipedia / Wikidata API 响应

### 7.2 并发抓取
一次审计内,这些请求可以并行:
```
parallel:
  - GET https://{domain}/
  - GET https://{domain}/robots.txt
  - GET https://{domain}/sitemap.xml
  - GET https://{domain}/llms.txt
  - GET https://en.wikipedia.org/w/api.php?...
  - GET https://www.wikidata.org/w/api.php?...
```

### 7.3 失败容错
- 抓取超时(30s)→ 该 check 标记 `skipped`,不影响总分
- DNS 失败 / 5xx → 直接返回 `audit_failed`
- 不要因为单点检测失败让整个审计崩溃

### 7.4 输出排序
issues 数组按以下顺序排序,方便客户 Agent 优先处理:
1. severity:Critical > High > Medium > Low
2. 同 severity 内按 weight 降序
3. 同 weight 内按 ID 字母序

### 7.5 给客户 Agent 用的两种格式
- **JSON**:程序化消费(默认)
- **Markdown**:人类可读(可选 `?format=markdown`)

---

## 8. 最小可用版本(MVP)建议

如果想先跑起来,实现这 12 项就能覆盖 70% 价值:

```
CRAWL-001  GPTBot 检测
CRAWL-002  ClaudeBot 检测
CRAWL-003  PerplexityBot 检测
SSR-001    SSR 内容检测
LLMS-001   llms.txt 验证
SCHEMA-001 JSON-LD 存在性
SCHEMA-002 Organization schema
SCHEMA-003 sameAs 数组
META-001   title
META-002   description
META-003   canonical
SEC-001    HTTPS
```

这 12 项的总权重约 50%。后续按业务反馈逐步补全其余 23 项。

---

*GEO 审计服务实现 Checklist — 文档结束。*
