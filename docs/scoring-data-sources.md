# 评分数据源详解:从原始字节到最终分数

> **目的:** 把 6 大评分类目中每一分的来源追溯到代码层 —— HTTP 请求、HTML DOM 元素、正则模式、第三方 API、LLM 判断。
> **配套文档:** `docs/business-workflow-analysis.md`(总体架构与商业流程)
> **报告日期:** 2026-05-09

---

## 数据源类型标记

| 标记 | 含义 |
|------|------|
| 🌐 | HTTP 请求(`requests.get` / `curl`) |
| 🔍 | HTML DOM 元素解析(BeautifulSoup) |
| 📜 | 正则模式匹配 |
| 🔑 | 第三方 API 调用 |
| 🤖 | LLM 主观判断(Claude 看了再决定) |
| ⚙️ | HTTP 响应头检查 |

---

## 0. 全局采集器:`scripts/fetch_page.py`

整个评分系统的"取证基座"。一次 `requests.get(url)` 返回的数据结构(`fetch_page.py:38-200`):

```python
{
  "status_code": 200,
  "redirect_chain": [{"url": "...", "status": 301}, ...],
  "headers": {完整 HTTP 响应头字典},
  "security_headers": {6 个安全头逐个检查},
  "meta_tags": {所有 <meta> 解析为 dict},
  "title": "...",
  "description": "...",
  "canonical": "<link rel=canonical>",
  "h1_tags": [...],
  "heading_structure": [{"level": 1-6, "text": "..."}],
  "word_count": int,
  "text_content": "...",
  "internal_links": [{"url", "text"}, ...],
  "external_links": [...],
  "images": [{"src", "alt", "width", "height", "loading"}],
  "structured_data": [所有 JSON-LD 块解析为 dict],
  "has_ssr_content": bool,
  "errors": [...]
}
```

**为什么不用 WebFetch?** WebFetch 会把 HTML 转 markdown 并丢失 `<head>` 内容,导致 JSON-LD / meta tag / canonical 全部丢失。所以 schema 检测必须用 `fetch_page.py` 抓原始 HTML。

---

## A. AI Citability(权重 25%)

**实现脚本:** `scripts/citability_scorer.py`(343 行)
**特点:** 整个仓库里**唯一完全机械化**的打分器 —— 没有 LLM 主观判断。

### A.1 输入数据采集流程(`citability_scorer.py:247-294`)

```
1. requests.get(url, User-Agent="Mozilla/5.0 ...", timeout=30)
2. BeautifulSoup 解析
3. decompose 掉 <script><style><nav><footer><header><aside><form>
4. 遍历 <h1><h2><h3><h4><p><ul><ol><table>
5. 按 H 标签切段,每段含 heading + 合并的段落文本
6. 过滤:段长 ≥ 20 词才进入打分
```

### A.2 子分 1:Answer Block Quality(满分 30)

| 检测项 | 数据源 | 实现 | 加分 |
|-------|-------|------|------|
| 定义模式 | 📜 5 个正则 | `\b\w+\s+is\s+(?:a\|an\|the)\s` 等(`L42-49`) | +15(任一命中) |
| 答案在前 60 词 | 📜 早期答案关键词 | `\b(?:is\|are\|was\|were\|means?\|refers?)\b\|\d+%\|\$[\d,]+\|\d+\s+(?:million\|billion\|thousand)`(`L57-66`) | +15 |
| 问句式标题 | 🔍 heading 文本 | `heading.endswith("?")` | +10 |
| 句长清晰度 | 📜 句子分割 | `5 <= len(s.split()) <= 25` 的句子比例 × 10 | 0-10 |
| 权威引用句式 | 📜 关键词 | `(?:according to\|research shows\|studies?\s+(?:show\|indicate\|suggest\|found)\|data\s+(?:shows\|indicates\|suggests))` | +10 |
| **封顶** | | | **30** |

### A.3 子分 2:Self-Containment(满分 25)

| 检测项 | 数据源 | 阈值 |
|-------|-------|------|
| 段落词数 | 📜 `len(text.split())` | 134-167 词 = **10**;100-200 = 7;80-250 = 4;<30 或 >400 = 0 |
| 代词密度 | 📜 `\b(?:it\|they\|them\|their\|this\|that\|these\|those\|he\|she\|his\|her)\b` 计数 ÷ 总词 | <2% = **8**;<4% = 5;<6% = 3 |
| 专有名词数 | 📜 `\b[A-Z][a-z]+(?:\s+[A-Z][a-z]+)*\b` 计数 | ≥3 = **7**;≥1 = 4 |

**实战含义:**
- 整段全用 "it / this / they" → 直接 0 分代词分
- 段落 134-167 词的"黄金长度"是最重要单条加分
- 无品牌名/地名/人名 → 失去 7 分

### A.4 子分 3:Structural Readability(满分 20)

| 检测项 | 数据源 | 评分 |
|-------|-------|------|
| 平均句长 | 📜 `word_count / sentence_count` | 10-20 词 = 8;8-25 = 5;否则 = 2 |
| 序列副词 | 📜 `(?:first\|second\|third\|finally\|additionally\|moreover\|furthermore)` | 命中 +4 |
| 编号项 | 📜 `(?:\d+[\.\)]\s\|\b(?:step\|tip\|point)\s+\d+)` | 命中 +4 |
| 段内换行 | 📜 `"\n" in text` | +4 |

### A.5 子分 4:Statistical Density(满分 15)

| 计数项 | 正则 | 每命中 | 单项上限 |
|-------|------|-------|---------|
| 百分比 | `\d+(?:\.\d+)?%` | +3 | 6 |
| 美元金额 | `\$[\d,]+(?:\.\d+)?(?:\s*(?:million\|billion\|M\|B\|K))?` | +3 | 5 |
| 数字 + 单位 | `\d+,?\d*\.?\d*\s+(?:users\|customers\|pages\|sites\|companies\|businesses\|people\|percent\|times\|x\b)` | +2 | 4 |
| 年份 | `\b20(?:2[3-6]\|1\d)\b`(2010-2026) | +2 | 2 |
| 命名机构 | `(?:according to\|per\|from\|by)\s+[A-Z]` 或 `Gartner\|Forrester\|McKinsey\|Harvard\|Stanford\|MIT\|Google\|Microsoft\|OpenAI\|Anthropic` 或 `\([A-Z][a-z]+(?:\s+\d{4})?\)` | +2 | — |

**实战:** "73% of marketers, $4.5M ARR, in 2025 according to Gartner" → 三条命中,~9 分。
"很多公司发现这个工具有效" → 0 分。

### A.6 子分 5:Uniqueness Signals(满分 10)

| 检测项 | 正则 | 加分 |
|-------|------|------|
| 原创研究 | `our (?:research\|study\|data\|analysis\|survey\|findings)\|we (?:found\|discovered\|analyzed\|surveyed\|measured)` | +5 |
| 案例标识 | `case study\|for example\|for instance\|in practice\|real-world\|hands-on` | +3 |
| 工具/产品提及 | `(?:using\|with\|via\|through)\s+[A-Z][a-z]+` | +2 |

### A.7 页面总分聚合(`citability_scorer.py:302-321`)

```
page_score = mean(每个段落的总分)
optimal_count = 词数在 134-167 区间的段落数
grade_dist = A/B/C/D/F 的分布(A:≥80, B:65-79, C:50-64, D:35-49, F:<35)
```

### A.8 评分客观度

**完全机械化,可重复。** 同一页面跑两次结果完全一致。

---

## B. Brand Authority(权重 20%)

**实现脚本:** `scripts/brand_scanner.py`(276 行)
**特点:** 脚本本身**不直接打分** —— 它输出取证 URL + 检查指令,真正的分数在 LLM 那一层。

### B.1 真正机械化的部分:Wikipedia / Wikidata API

**这是 Brand 类目唯一确定性的检测**(`brand_scanner.py:120-146`):

```python
# Wikipedia API
api_url = f"https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch={brand}"
response = requests.get(api_url, timeout=15)
search_results = response.json()["query"]["search"]
# 判定:top result.title.lower() 包含 brand_name.lower() → has_wikipedia_page

# Wikidata API
wikidata_url = f"https://www.wikidata.org/w/api.php?action=wbsearchentities&search={brand}&language=en"
response = requests.get(wikidata_url, timeout=15)
entities = response.json()["search"]
# 判定:有结果即 has_wikidata_entry,记录 Q-number
```

**Skill 文件强制要求**(`geo-brand-mentions/SKILL.md:277`):
> "ALWAYS run the Python API check first. If the API says a page exists, it exists — do not override this with a search result that fails to find it."

### B.2 其他平台:LLM 读搜索结果

| 平台 | 权重 | 数据源 | 取证手段 |
|------|------|--------|---------|
| YouTube | 25% | 🌐 `youtube.com/results?search_query=<brand>` 的 HTML | 🤖 LLM 读结果页,数频道/视频/订阅数 |
| Reddit | 25% | 🌐 `reddit.com/search/?q=<brand>` | 🤖 LLM 数 thread、subreddit |
| Wikipedia | 20% | 🔑 Wikipedia API + Wikidata API | 客观 + 🤖 LLM 判文章质量 |
| LinkedIn | 15% | 🌐 LinkedIn 搜索(常被登录墙阻拦) | 🤖 主观估计 |
| Other(7 个) | 15% | 🌐 Quora/SO/GitHub/Crunchbase/PH/G2/Trustpilot 搜索 | 🤖 LLM 读结果 |

### B.3 LLM 怎么把"看到的"转成 0-100

Skill 提供分段标尺(`geo-brand-mentions/SKILL.md:48-55`,以 YouTube 为例):

```
90-100:Active channel with 10K+ subscribers, brand in 20+ third-party videos
70-89: 1K+ subscribers, 10-19 third-party mentions
50-69: Channel exists, 5-9 mentions
30-49: Inactive, 1-4 mentions
10-29: 1-2 mentions only
0-9:   No presence
```

LLM 看搜索页 → 数大致数字 → 套这五档 → 给值。**这是 ±5-10 分浮动的来源。**

### B.4 评分客观度

**半主观。** Wikipedia/Wikidata 部分客观,其余靠 LLM 读取搜索结果页。

---

## C. Content Quality & E-E-A-T(权重 20%)

**实现:** **无专用 Python 脚本**。完全靠 Claude 用 `WebFetch` 抓页面后,对照 Skill 信号清单逐项打分。

### C.1 唯一可机械验证的部分

| 信号 | 数据源 | 评分 |
|------|--------|------|
| HTTPS | 🌐 URL scheme | Trust +2 |
| 页面词数 | 📜 `len(text.split())` | 决定页面类型门槛 |
| Privacy policy 链接 | 🔍 Grep `href` 包含 "privacy" | Trust +2 |
| Terms of service | 🔍 Grep `href` 包含 "terms" | Trust +1 |
| 联系信息 | 📜 `@\w+\.\w+` 邮箱、`\+?\d{3}` 电话、地址结构 | Trust 0-4 |
| schema 含 `author` `datePublished` `dateModified` | 🔍 JSON-LD 解析 | Experience + Freshness |

### C.2 几乎全靠 LLM 判断的部分

**Experience 25 分**(`geo-content/SKILL.md:50-66`)的 6 项:

| 信号 | 判断方式 |
|------|---------|
| First-person accounts("I tested...", "We implemented...") | 🤖 LLM 读全文找句式 |
| Original research / data | 🤖 极主观 |
| Case studies with specific results | 🤖 找有无数字 |
| Screenshots/photos as evidence | 🤖 无法可靠判断品牌实拍 vs stock |
| Specific examples from personal experience | 🤖 极主观 |
| Demonstrations of process | 🤖 主观 |

**Expertise / Authoritativeness 各 25 分** 同样几乎全主观:作者资质、媒体引用、行业奖项 —— 都是 LLM 读页面后判断。

### C.3 Topical Authority 修饰符(`geo-content/SKILL.md:246-253`)

```
sitemap.xml 中页面数 + 内链聚类强度:
  20+ pages + 强聚类  → +10
  10-20 pages + 部分聚类 → +5
  5-10 pages → 0
  <5 pages → -5(扣分)
```

页面数靠 🌐 抓 sitemap 数,聚类强度靠 🤖 LLM 看内链结构判断。

### C.4 评分客观度

**主观。** 同一页面在不同 Claude session 可能浮动 8-10 分。

---

## D. Technical Foundations(权重 15%)

**特点:** 大部分可机械化,几乎都能 `curl + grep` 验证。

### D.1 单次 fetch 的数据(同 §0,`fetch_page.py`)

### D.2 Crawlability(15 分)的精确算法

| 分项 | 数据点 | 阈值 |
|-----|--------|------|
| robots.txt 有效 + 含 Sitemap | 🌐 `GET /robots.txt` → 解析语法 + grep "Sitemap:" | 3 / 0 |
| AI 爬虫允许 | 🌐 robots.txt 中 grep GPTBot/PerplexityBot/ClaudeBot/Google-Extended | 全允许 5;关键阻止 1;Googlebot 阻止 0 |
| sitemap.xml 有效 | 🌐 `GET /sitemap.xml` → 校验 XML + `<lastmod>` 存在 | 3 / 0 |
| 爬深 ≤ 3 次点击 | 🔍 BFS 内链 | 2 / 0 |
| 无误用 noindex | 🔍 `<meta robots>` + ⚙️ `X-Robots-Tag` | 2 / 0 |

### D.3 Indexability(12 分)

| 分项 | 数据点 |
|-----|--------|
| Canonical 自指向且无冲突 | 🔍 `<link rel=canonical>` + ⚙️ HTTP `Link` header 比对 |
| 无 www/non-www 重复 | 🌐 测两侧重定向 |
| 无 HTTP/HTTPS 重复 | 🌐 同上 |
| Hreflang 合法 | 🔍 `<link rel="alternate" hreflang="...">` + ISO 639-1/3166-1 校验 |
| 无 index bloat | 🌐 sitemap 数量 vs 实际有价值页面 |

### D.4 Security(10 分)— **完全客观**

`fetch_page.py:74-89` 检查这 6 个 HTTP 响应头:

| Header | 分值 | 来源 |
|--------|------|------|
| HTTPS + 有效证书 | 4 | 🌐 URL scheme + 证书检查 |
| `Strict-Transport-Security` | 2 | ⚙️ |
| `X-Content-Type-Options: nosniff` | 1 | ⚙️ |
| `X-Frame-Options` | 1 | ⚙️ |
| `Referrer-Policy` | 1 | ⚙️ |
| `Content-Security-Policy` | 1 | ⚙️ |

**这是整个评分系统里最客观的部分。** 头存在/不存在,二值判定。

### D.5 Core Web Vitals(15 分)— **实际不可精确测量**

Skill 提示词坦白(`geo-technical/SKILL.md:242-247`):
> "When real user data is unavailable, **estimate from page characteristics**"

未接 PageSpeed Insights API 或 CrUX,只能用代理信号:

| 指标 | 代理信号 | 精度 |
|-----|---------|------|
| LCP(5 分) | 🔍 首屏最大 `<img>` 大小、是否 preload、🌐 TTFB | 估计 |
| INP(5 分) | 🔍 第三方 `<script>` 数量、async/defer、bundle 大小 | 估计 |
| CLS(5 分) | 🔍 `<img>` 是否 width/height、是否 font-display: swap | 估计 |

**这意味着 CWV 的 15 分是 Claude 看页面"特征"猜的**,不是真实数据。

### D.6 SSR(15 分)的精确检测算法(`fetch_page.py:121-194`)— **最严谨**

```python
# 1. 在 decompose 前找 SPA 框架根容器
js_app_roots = soup.find_all(id=re.compile(r"(app|root|__next|__nuxt)", re.I))

# 2. 测量根容器内的文字量
for root_el in js_app_roots:
    inner_text = root_el.get_text(strip=True)
    text_length = len(inner_text)

# 3. 判定:根容器文字 < 50 字符 AND 全页词数 < 200
if text_length < 50 and word_count < 200:
    has_ssr_content = False  # 0 分
```

**双重判断**避免误伤(WordPress + LiteSpeed Cache 等也用 `<div id="root">` 但有 SSR)。

| 评分 | 条件 |
|-----|------|
| 15 | 全部 SSR(主内容 8 + meta/schema 4 + 内链 3) |
| 10 | 主内容 SSR,部分元素 JS-only |
| 5 | 关键内容需 JS |
| 0 | 完全 CSR(根 div < 50 字符 AND 全页 < 200 词) |

### D.7 Page Speed(15 分)

| 检测 | 命令/方法 |
|-----|----------|
| TTFB | 🌐 `curl -o /dev/null -s -w '%{time_starttransfer}' [URL]` → 直接得秒数 |
| 页面总重 | 🌐 主页 HTML 抓取后 BeautifulSoup 找 `<img>` `<script>` `<link>` —— 不下载,靠 🤖 推断 |
| 压缩 | ⚙️ `Content-Encoding: gzip\|br` |
| 图片优化 | 🔍 `<img>` 的 src 扩展名(webp/avif)、loading="lazy"、width/height |
| 静态缓存 | ⚙️ 静态资源的 `Cache-Control` header |

### D.8 评分客观度

**客观。** 95% 可机械化,仅 CWV 因缺真实用户数据需估算。

---

## E. Structured Data(权重 10%)

### E.1 数据采集(`fetch_page.py:107-114`)

**关键约束:** 必须用 `fetch_page.py` 抓原始 HTML —— WebFetch 会丢 `<head>`,导致 JSON-LD 丢失。

```python
for script in soup.find_all("script", type="application/ld+json"):
    try:
        data = json.loads(script.string)
        result["structured_data"].append(data)
    except (json.JSONDecodeError, TypeError):
        result["errors"].append("Invalid JSON-LD detected")
```

得到 dict 列表,每个元素是一个 JSON-LD 块的解析结果。

### E.2 评分流程

对每个 JSON-LD 块:

| 检查 | 做法 | 加/扣分 |
|-----|------|--------|
| `@type` 合法? | 🤖 LLM 对比 Schema.org 类型列表 | 不合法扣分 |
| 必填属性齐全? | 📜 例 Organization 需 `name` `url` `logo` | 缺一项 -5 |
| `sameAs` 含几个权威链接? | 🔍 数 `sameAs` 数组里的 wikipedia/linkedin/youtube/wikidata 命中 | 0-15 |
| 业务类型对应 schema 存在? | 🤖 SaaS 需 SoftwareApplication;Local 需 LocalBusiness | 命中 +10/项 |
| schema 在 SSR 还是 JS 注入? | 🌐 比对 `curl` 结果 vs JS 渲染后结果 | JS 注入扣分 |
| URL 在 sameAs 中是否 200? | 🌐 逐个 `requests.head()` | 404 扣分 |

### E.3 业务类型对应必备 schema(影响评分)

| 业务类型 | 必备 schema | 扣分项 |
|---------|------------|-------|
| 通用 | `Organization` + `WebSite` + SearchAction | 缺 -15 |
| Local | `LocalBusiness` + `geo` + `openingHoursSpecification` | 缺 -10 |
| Publisher | `Article` + `Person`(author with sameAs) | 缺 -10 |
| E-commerce | `Product` + `Offer` + `AggregateRating` | 缺 -10 |
| SaaS | `SoftwareApplication` + `featureList` | 缺 -10 |

### E.4 评分客观度

**半客观。** JSON-LD 解析是机械的,但"业务类型推断"和"schema 完整度判断"靠 LLM。

---

## F. Platform Optimization(权重 10%)

5 个平台各打 0-100,最后求平均。详见 `agents/geo-platform-analysis.md`。

### F.1 每个平台**独有**的可机械检测信号

#### Google AI Overviews
| 信号 | 取证 |
|------|------|
| FAQ 区 5+ 问题 | 🔍 数页面 `<h2>`/`<h3>` 问句式标题 + 🔍 检测 `FAQPage` schema |
| 比较表格存在 | 🔍 `<table>` 数量 |
| 直接答案在标题后 | 🔍 H 标签后第一个 `<p>` 的前 60 词 |
| 排前 10 名 | 🤖 **无法直接测**,LLM 凭页面质量"推断" |

#### ChatGPT Web Search
| 信号 | 取证 |
|------|------|
| Wikipedia 存在 | 🔑 Wikipedia API(同 §B.1) |
| Wikidata 实体 | 🔑 Wikidata API |
| OAI-SearchBot / ChatGPT-User / GPTBot 在 robots.txt | 🌐 `/robots.txt` + grep |
| Bing 索引覆盖 | 🤖 **不能外部测**,LLM 估计 |

#### Perplexity
| 信号 | 取证 |
|------|------|
| PerplexityBot 在 robots.txt | 🌐 `/robots.txt` + grep |
| 内容更新 <6 月 | 🔍 schema `dateModified` 比对当前日期 |
| Reddit 活跃度 | 🌐 + 🤖 WebFetch reddit.com 搜索 + LLM 数 |

#### Google Gemini
| 信号 | 取证 |
|------|------|
| Knowledge Panel | 🤖 LLM 从 Google 搜索结果推断 |
| YouTube + chapters | 🌐 WebFetch YouTube 频道 + 抽样视频描述含 `0:00` 时间戳 |
| Schema 完整度 | 🔍 同 §E |

#### Bing Copilot — **最确定性**
| 信号 | 取证 | 分值 |
|------|------|------|
| **IndexNow 实现** | 🌐 `GET /<api-key>.txt` 或 `/.well-known/indexnow-key.txt` 返回 200 | +15 |
| **Bing WMT 验证** | 🔍 `<meta name="msvalidate.01">` | +5 |
| LCP < 2s | 🌐 TTFB + 🔍 资源分析 | 10 |
| Bingbot 在 robots.txt | 🌐 `/robots.txt` + grep | — |

### F.2 评分客观度

**半主观。** 客观信号(API、robots.txt、IndexNow)占约 30%,其余靠 LLM 读搜索结果。

---

## G. llms.txt 验证(影响 AI Visibility 子分)

**实现脚本:** `scripts/llmstxt_generator.py:30-127`

### G.1 完全机械化的验证算法

```python
1. GET https://{domain}/llms.txt
   if 404 → 直接 0 分
   if 200 → 继续

2. 解析 markdown:
   - 第一行以 "# " 开头?     → has_title (4 分)
   - 任意行以 "> " 开头?      → has_description (3 分)
   - "## " 标题数 → section_count
   - 正则 "- \[.+\]\(.+\)" 匹配 → link_count (link_count >= 5 → 5 分)

3. 完全合规:has_title + has_description + has_sections + has_links 全 True

4. 自动建议:
   - link_count < 5 → "增加到 10-20"
   - section_count < 2 → "增加 section"
   - 内容不含 "contact" → "加联系方式"
```

### G.2 评分客观度

**完全客观。** 100% 可重复。

---

## 总结:每类目客观度对比

| 类目 | 权重 | 🌐 HTTP | 🔍 DOM | 📜 正则 | 🔑 API | 🤖 LLM | ⚙️ Header | **客观度** |
|------|------|--------|--------|---------|--------|--------|----------|-----------|
| Citability | 25% | ★ | ★★★ | ★★★ | — | — | — | **★★★★★ 客观** |
| Brand Authority | 20% | ★★ | ★ | — | ★★ | ★★★ | — | ★★ 半主观 |
| E-E-A-T | 20% | ★ | ★★ | ★ | — | ★★★ | — | ★ **主观** |
| Technical | 15% | ★★★ | ★★★ | ★★ | — | ★ | ★★★ | **★★★★★ 客观** |
| Schema | 10% | ★★ | ★★★ | — | — | ★★ | — | ★★★ 半客观 |
| Platform Optimization | 10% | ★★ | ★★ | ★ | ★ | ★★★ | — | ★★ 半主观 |

### 加权总分的"客观度":

```
确定性可重复部分 = 25%·Citability + 15%·Technical + 50%·Schema(机械部分)
                = 25 + 15 + 5 = 约 45%

LLM 主观判断部分 = 100%·E-E-A-T + 70%·Brand + 70%·Platform + 50%·Schema(主观部分)
                = 20 + 14 + 7 + 5 = 约 46%

混合部分(API + LLM)= 30%·Brand + 30%·Platform = 6 + 3 = 约 9%
```

**结论:** GEO 总分大约**一半可重复,一半依赖 LLM 判断**。同一站点在不同 session 跑两次,总分差异约 ±3-8 分。

---

## 工程化改进路径

如果要把这套系统稳定化(用于产品级别交付):

| 当前实现 | 改进方向 |
|---------|---------|
| Brand 类目 70% 靠 LLM 读搜索页 | 接入 YouTube Data API、Reddit API、SerpAPI |
| CWV 估算 | 接入 PageSpeed Insights API / CrUX |
| Bing 索引覆盖靠猜 | 接入 Bing Webmaster Tools API |
| Google Knowledge Panel 靠猜 | 接入 Google Knowledge Graph Search API |
| 每次重新抓取 | 加缓存层 `~/.geo-prospects/cache/<domain>/`,TTL 7 天 |
| 同站审计 ±5 浮动 | E-E-A-T 类目用多次采样取均值 |

实现以上 6 项后,客观度可从约 50% 提升到约 85%。

---

## 附录:核心代码定位索引

| 评分逻辑 | 代码位置 |
|---------|---------|
| Citability 所有子分 | `scripts/citability_scorer.py:39-214` |
| Wikipedia API 检测 | `scripts/brand_scanner.py:120-132` |
| Wikidata API 检测 | `scripts/brand_scanner.py:135-146` |
| HTML/HTTP 完整解析 | `scripts/fetch_page.py:38-200` |
| 6 个安全头检查 | `scripts/fetch_page.py:74-89` |
| JSON-LD 提取 | `scripts/fetch_page.py:107-114` |
| SSR 检测算法 | `scripts/fetch_page.py:121-194` |
| llms.txt 验证 | `scripts/llmstxt_generator.py:30-127` |
| 平台评分 rubric | `skills/geo-platform-optimizer/SKILL.md:47-216` |
| E-E-A-T 4 维信号 | `skills/geo-content/SKILL.md:36-117` |
| 8 类技术 SEO checklist | `skills/geo-technical/SKILL.md:30-348` |
| Schema 业务类型映射 | `skills/geo-schema/SKILL.md:64-195` |
| Citability 5 维 rubric | `skills/geo-citability/SKILL.md:25-168` |
| 11 平台权重定义 | `skills/geo-brand-mentions/SKILL.md:32-208` |
| AIO/ChatGPT/Perplexity/Gemini/Copilot 各自 10 项 checklist | `skills/geo-platform-optimizer/SKILL.md:34-216` |

---

*评分数据源详解 — 报告结束。*
