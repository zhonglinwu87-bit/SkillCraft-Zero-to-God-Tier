---
tags: [教程, Skill, 大师课, Level3, marketing, Schema]
created: 2026-05-03
updated: 2026-05-05
course_number: L3-12
prerequisites: ["[[L3-11-AI时代SEO]]", "[[L3-10-SEO审计]]"]
next_course: "[[L3-13-新用户引导优化]]"
---

# L3-12：结构化数据标记 —— Schema + JSON-LD + @graph 完整方法论

> 🎯 **学什么**：学会用 JSON-LD 给网页打上"机器标签"，让搜索引擎精确理解你的内容（是文章还是产品？评分多少？价格多少？），在搜索结果里展示星级、价格、FAQ 等富媒体卡片。核心洞察：Schema 是 SEO 里"投入产出比最高"的操作——不改产品、不改内容、不改排名，只加了代码标签，点击率通常提升 20-40%。在 10 个蓝色链接中，有星星和价格的那条最显眼。
> 💡 **难易度**：⭐⭐ | ⏱️ **预计时间**：40 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：这个 Skill 就像一个**给网页贴"普通话标签"的翻译器**——你有一仓库的物品但没有标签，外人只能乱猜"这里可能有产品？这个数字可能是价格？"Schema 就是给每个物品贴上标准标签：这个叫"产品"，价格 ¥158，评分 4.7 星，有 328 条评价。贴标之后谷歌和 AI 都能精确理解，搜索结果直接显示星星和价格——不贴标的竞争对手只有一行蓝字。

> **本课的独特定位**——数据标注型 Skill：

| 特征 | L3-10（SEO 审计） | L3-12（本课） |
|------|-----------------|-------------|
| 操作对象 | 整站 | **每个页面的代码** |
| 输出 | 诊断报告 | **JSON-LD 代码块** |
| 可见效果 | 排名提升（慢） | **富媒体摘要（快——几周内见效）** |
| 对 AI SEO 价值 | 基础设施 | **直接提升 AI 可提取性 30-40%** |

> 🔑 **核心秘诀**：Schema（Schema：数据结构化标记）的四条最高原则中，"Accuracy First"（准确优先）是王——标记不存在的内容（页面上没评分但 Schema 标了 4.8）是 Google 最大的禁忌，会导致手动处罚——你整个网站的 Schema（Schema：数据结构化标记） 都可能被 Google 忽略。

> ⚠️ **避坑指南**：JSON-LD 中一个逗号放错位置，或者 @type 拼成 @Type——整个 Schema 块无效，但 HTML 页面完全正常。你看着页面没有任何问题，但 Google 看到的是"没有 Schema（Schema：数据结构化标记）"。

---

## 2. Schema（Schema：数据结构化标记）的核心价值

### 2.1 Schema 是什么？

```
Schema = 网页的"机器翻译层"

没有 Schema（Schema：数据结构化标记）：
  搜索引擎看到一堆 HTML 标签和文字
  → 猜测："这里可能有个产品？这个数字 ¥158 可能是价格？
           这个 4.7 可能是评分？有 328 可能是评价数？"
  → 猜测可能错——把面包屑导航的"首页 > 产品"当成产品名

有 Schema（Schema：数据结构化标记）：
  搜索引擎精确知道：
  → @type: Product
  → name: "手冲咖啡壶 温控版"
  → offers.price: 399
  → offers.priceCurrency: CNY
  → aggregateRating.ratingValue: 4.7
  → aggregateRating.reviewCount: 328
  → 可以在搜索结果中直接展示：⭐⭐⭐⭐⭐ 4.7 (328) · ¥399
```

### 2.2 为什么 JSON-LD 是 Google 推荐的格式？

```
三种 Schema（Schema：数据结构化标记） 格式对比：

Microdata（嵌入式）：
  <div itemscope itemtype="https://schema.org/Product">
    <span itemprop="name">手冲咖啡壶</span>
  </div>
  → Schema 属性嵌入 HTML 标签中
  → 问题：污染 HTML 结构，维护困难（改 Schema 要动 HTML）

RDFa（属性扩展）：
  <div vocab="https://schema.org/" typeof="Product">
    <span property="name">手冲咖啡壶</span>
  </div>
  → 类似 Microdata，使用不同属性名
  → 问题：同样污染 HTML 结构

JSON-LD（独立 JSON 块）✅ Google 推荐：
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "手冲咖啡壶"
  }
  </script>
  → 独立的 <script> 代码块，不嵌入 HTML 结构
  → 优势：修改 Schema 不需要改动页面 HTML
  → 优势：可以放在 <head> 或 </body> 前
  → 优势：对动态网站友好——Schema 逻辑独立管理
```

---

## 3. 10 种常用 Schema（Schema：数据结构化标记）类型详解

### 3.1 每种类型的关键属性

```
1. Organization（组织/公司）
   用途：告诉搜索引擎"这个网站是谁的"
   关键属性：name, url, logo, sameAs（社交媒体链接）
   页面：首页 / 关于页
   
2. WebSite（网站）
   用途：网站搜索功能 + 站点名称
   关键属性：name, url, potentialAction（搜索动作）
   页面：首页

3. Article（文章）
   用途：博客文章 / 新闻 / 深度内容
   关键属性：headline, author, datePublished, dateModified, image
   页面：博客文章页
   特殊：author 最好是 Person 类型（含 name+url）

4. Product（产品）⭐ 电商最重要的 Schema（Schema：数据结构化标记）
   用途：产品信息
   关键属性：name, image, description, offers(price/priceCurrency/availability),
            aggregateRating(ratingValue/reviewCount), brand
   页面：产品详情页
   注意：aggregateRating 必须在页面上有真实对应的评分展示！

5. SoftwareApplication（软件应用）
   用途：App / SaaS（SaaS：软件即服务）产品
   关键属性：name, operatingSystem, applicationCategory, offers
   页面：App 下载页 / SaaS 首页

6. FAQPage（常见问答）⭐ 投资回报率最高
   用途：FAQ 内容
   关键属性：mainEntity（数组，每项是 Question + Answer）
   页面：FAQ 页 / 任何有 Q&A 内容的页面
   效果：搜索结果展开显示问答 → 占 2-3 倍高度 → 视觉碾压普通蓝色链接
   为什么 ROI（ROI：投资回报率）最高：条件低（只要有 Q&A 内容即可）→ 效果显著（CTR +15-30%）

7. HowTo（操作指南）
   用途：步骤式教程
   关键属性：name, step（数组，每项 HowToStep）
   页面：教程/指南页
   效果：搜索结果展示步骤预览

8. BreadcrumbList（面包屑导航）
   用途：告诉搜索引擎页面在站点结构中的位置
   关键属性：itemListElement（数组，按层级排列）
   页面：所有内页（首页不需要）
   效果：搜索结果展示层级路径而非纯 URL

9. LocalBusiness（本地商家）
   用途：实体店铺信息
   关键属性：name, address, telephone, openingHours, geo
   页面：联系页 / 首页

10. Event（活动）
    用途：线上线下活动
    关键属性：name, startDate, endDate, location, offers
    页面：活动页
```

### 3.2 FAQPage 为什么是投入产出比最高的 Schema（Schema：数据结构化标记）？

```
FAQPage 的效果：
  → 搜索结果中展开显示 Q&A
  → 占据正常结果 2-3 倍的高度
  → 视觉上碾压周围的普通蓝色链接
  → 同等排名下 CTR（CTR：点击率）高 15-30%

为什么 ROI（ROI：投资回报率）最高？
  → 条件低：只要你的页面确实有 Q&A 内容
  → 不需要评分数据（不像 Product Schema 需要真实评价）
  → 不需要特定页面类型（任何有 Q&A 内容的页面都可以加）
  → 实施简单：一个 JSON-LD（JSON-LD：结构化数据格式）块 + 页面上真有的 Q&A
  
示例 JSON-LD：
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    "mainEntity": [{
      "@type": "Question",
      "name": "手冲咖啡水温多少度最合适？",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "88°C-92°C 是最佳水温区间。深烘焙豆用 85°C-88°C，浅烘焙豆 92°C-96°C。"
      }
    }, {
      "@type": "Question",
      "name": "手冲壶为什么比普通水壶贵？",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "手冲壶的壶嘴设计让水流可以精准控制——细而稳定的水流是均匀萃取的关键。普通水壶倒水时容易忽大忽小，导致咖啡粉部分过萃部分欠萃。"
      }
    }]
  }
  </script>
```

---

## 4. @graph —— 一个页面多种 Schema（Schema：数据结构化标记）

### 4.1 为什么需要 @graph？

现实中很少有页面只需要一种 Schema（Schema：数据结构化标记）——一个博客文章页可能需要 4 种：

```
场景：一篇博客文章页面

需要的 Schema（Schema：数据结构化标记）：
  1. Article（文章本身）
  2. BreadcrumbList（导航路径："首页 > 博客 > 咖啡 > 手冲指南"）
  3. Organization（发布者组织信息）
  4. WebSite（所属网站信息）

没有 @graph 的做法：
  → 写 4 个独立的 <script type="application/ld+json"> 块
  → 维护复杂，容易遗漏
  → 每个块都要重复 @context

有 @graph 的做法：
  → 一个 <script> 块包含 @graph 数组
  → 所有 Schema（Schema：数据结构化标记） 打包在一个数据包中
```

### 4.2 @graph 示例

```json
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Article",
      "headline": "手冲咖啡终极指南（2026版）",
      "author": {
        "@type": "Person",
        "name": "李明"
      },
      "datePublished": "2026-01-15",
      "dateModified": "2026-03-20"
    },
    {
      "@type": "BreadcrumbList",
      "itemListElement": [
        {
          "@type": "ListItem",
          "position": 1,
          "name": "首页",
          "item": "https://example.com"
        },
        {
          "@type": "ListItem",
          "position": 2,
          "name": "博客",
          "item": "https://example.com/blog"
        },
        {
          "@type": "ListItem",
          "position": 3,
          "name": "手冲咖啡指南",
          "item": "https://example.com/blog/pour-over-guide"
        }
      ]
    },
    {
      "@type": "Organization",
      "name": "咖啡大师",
      "url": "https://example.com",
      "logo": "https://example.com/logo.png"
    },
    {
      "@type": "WebSite",
      "name": "咖啡大师",
      "url": "https://example.com"
    }
  ]
}
</script>
```

---

## 5. Schema（Schema：数据结构化标记）的四条最高原则

```
原则 1: Accuracy First（准确优先）
  → Schema 必须准确反映页面上真实存在的内容
  → 不得标记不存在的内容（最大的禁忌）
  → 违反后果：整个网站的 Schema（Schema：数据结构化标记） 可能被 Google 忽略（手动处罚）
  → 这跟"写得越多越好"正相反——宁缺毋滥
  示例：页面上没有用户评价但 Schema 标了 ratingValue: "4.9"
       → 判定为"垃圾结构化数据"
       → 该页面 + 整个域名的 Schema（Schema：数据结构化标记） 全部失效

原则 2: Completeness（完整性）
  → 填写 Google 要求的"必需属性"
  → 对每种 Schema（Schema：数据结构化标记）类型，Google 文档列出了必需和推荐属性
  → 缺少必需属性 = Schema 可能不会生成富媒体摘要

原则 3: Specificity（具体性）
  → 使用最具体的 Schema（Schema：数据结构化标记）类型
  → 例如：不要用 Product，而是尽量找到更具体的子类型
  → @type: "Book" 而非 @type: "Product"（如果你卖的是书）

原则 4: Freshness（时效性）
  → 内容变了 → Schema 也要更新
  → 特别是 dateModified、price、availability
  → 注意：不实时的价格/库存信息 = 用户体验差 + Google 可能处罚
```

---

## 6. 结构拆解：数据标注型 Skill 模板

```markdown
## 数据标注型 Skill 模板

### 核心特征
→ 不改页面内容/视觉，只加机器可读的"标签层"
→ 核心资产 = Schema（Schema：数据结构化标记）类型目录 + JSON-LD（JSON-LD：结构化数据格式）格式规范 + 验证工具链
→ 关键设计 = 类型选择 + 属性填写 + 准确性检查
→ 输出 = 可直接嵌入 HTML 的JSON-LD（JSON-LD：结构化数据格式）代码块

### 通用结构

## Schema Catalog（类型目录）
Type 1: [名称] — 用途 — 必需属性 — 推荐属性 — 示例
Type 2: [名称] — ...
...
Type N: [名称] — ...

## Format Spec（格式规范）
- 格式：JSON-LD（Google 推荐）
- @graph 多类型组合方法
- 常见错误：[缺失逗号、@type 拼错、嵌套错误]

## Validation Chain（验证链）
1. Rich Results Test（富媒体结果测试工具）（检查能否生成富媒体摘要）
2. Schema.org Validator（检查 Schema 语法）
3. Google Search Console（谷歌搜索控制台）（检查索引状态和错误报告）

## Core Principles（核心原则）
1. Accuracy First（不标记不存在的内容）
2. Completeness（必需属性不能缺）
3. Specificity（用最具体的类型）
4. Freshness（内容变了 Schema 要同步更新）
```

**与 L3-11（AI SEO）的关系**：Schema 是 AI SEO 三大支柱中 Structure（让内容可提取）的技术实现层。有 Schema（Schema：数据结构化标记）的内容 AI 可见度高 30-40%。L3-11 告诉你"为什么切成块"，L3-12 告诉你"怎么打标签"。

---

## 7. 电商案例：设计师耳饰独立站 120 款产品的 JSON-LD（JSON-LD：结构化数据格式） 结构化数据标记

> 🛒 **实战案例**：独立站「耳语 EYU」设计师耳饰品牌（Shopify 建站，120 款 SKU，均价 ¥158-¥398），运营 18 个月没有任何 Schema（Schema：数据结构化标记）标记。Google 搜索"法式复古耳环""珍珠耳钉设计师""18K 镀金耳饰"时，品牌出现在搜索结果第 1 页，但只有纯蓝色文字链接——没有星级、没有价格、没有库存状态。CTR（搜索结果点击率）仅 2.3%（行业有 Schema（Schema：数据结构化标记）的同类品牌 CTR（CTR：点击率）约 5-6%）。竞品「素觉」的搜索结果展示"⭐⭐⭐⭐⭐ 4.8 (432) · ¥168 · 有货"——同样的排名位置，点击率是 5.8%。
>
> **用 Schema Markup Skill 实施 5 类 Schema（使用 @graph 一个 <script> 块内嵌所有类型）**：
>
> | Schema（Schema：数据结构化标记）类型 | 应用页面数 | 关键属性 | 预期搜索结果效果 |
> |------------|:--:|------|------|
> | **Product**（含 offers + aggregateRating + brand） | 120 个产品页 | name, image, description, offers(price/priceCurrency/availability), aggregateRating(ratingValue/reviewCount), brand | 搜索结果展示 ⭐4.7 (328) · ¥158 · 有货 |
> | **Organization + WebSite** | 首页 | name, url, logo, sameAs（品牌社交媒体链接） | 品牌 Knowledge Panel 信息补全 |
> | **BreadcrumbList** | 分类页+产品页 | itemListElement（首页 > 耳饰 > 珍珠 > 产品名） | 搜索结果展示面包屑导航路径 |
> | **Article** | 品牌故事页+搭配指南 8 篇 | headline, author(Person), datePublished, dateModified, image | 内容页被正确归类为文章 |
> | **FAQPage** | 2 个FAQ（FAQ：常见问题）页（材质保养+退换政策） | mainEntity（数组，每项 Question + acceptedAnswer） | 搜索结果展开显示 Q&A，占据 2-3 倍高度 |

> **关键 Product Schema 代码（JSON-LD）**：
> ```json
> {
>   "@context": "https://schema.org",
>   "@type": "Product",
>   "name": "法式复古珍珠耳环 - 耳语原创设计",
>   "image": "https://eyu.com/images/er-fr-pearl-01.jpg",
>   "description": "18K 镀金 + 8-9mm 天然淡水珍珠，法式复古不对称设计",
>   "sku": "EYU-PE-2026-001",
>   "offers": {
>     "@type": "Offer",
>     "price": "158",
>     "priceCurrency": "CNY",
>     "availability": "https://schema.org/InStock",
>     "priceValidUntil": "2026-06-30"
>   },
>   "aggregateRating": {
>     "@type": "AggregateRating",
>     "ratingValue": "4.7",
>     "reviewCount": "328",
>     "bestRating": "5"
>   },
>   "brand": { "@type": "Brand", "name": "耳语 EYU" }
> }
> ```
>
> **效果**：
> - Google 搜索结果 CTR：2.3% → 4.9%（仅加 Schema（Schema：数据结构化标记），不改标题/描述/排名/产品），30 天数据
> - FAQPage Schema 效果最显著——"耳语 珍珠怎么保养"查询的搜索结果展示 3 条展开的 Q&A，占据正常结果 3 倍高度，CTR 达 8.2%
> - Product Schema（Schema：数据结构化标记）的星级+价格展示使品牌在 10 条蓝色链接中视觉上碾压无 Schema（Schema：数据结构化标记）的竞品
> - 有机流量增长：月均 12,000 → 19,500 UV（+62.5%），核心驱动是 CTR 提升而非排名提升
> - **Accuracy First 警告**：aggregateRating 的 ratingValue 和 reviewCount 必须与页面上真实展示的评价数据一致——品牌在 Schema 部署前已积攒了 328 条真实评价，不存在标记虚假评分风险

> 🔑 **启示**：Schema 是 SEO 里投资回报比最高的操作——不改产品、不改内容、不改排名，只在 HTML 中加了JSON-LD（JSON-LD：结构化数据格式）代码块，点击率从 2.3% 翻到 4.9%。原因很简单：在 10 个蓝色链接中，有星星和价格的那条最显眼。FAQPage 的视觉碾压效应（展开占 3 倍高度）更是把 CTR 拉到了 8.2%。但必须牢记 Accuracy First——只有页面上真的展示评分数据才能标记 aggregateRating，否则整个网站的 Schema（Schema：数据结构化标记） 会被 Google 拉黑。

---

## 8. 掌握检验

**Q1**：Schema（Schema：数据结构化标记）标记的核心原则中，"Accuracy First"的最重要含义是什么？
- A) Schema 必须包含尽可能多的属性
- B) Schema 必须准确反映页面上真实存在的内容，不得标记不存在的内容
- C) Schema（Schema：数据结构化标记）的内容要写得比页面内容更吸引人
- D) Schema 要每周更新一次

**Q2**：为什么 JSON-LD 是 Google 推荐的 Schema（Schema：数据结构化标记） 格式？与其他格式相比核心优势是什么？

**Q3**：以下哪个 Schema（Schema：数据结构化标记）类型目前在搜索结果中获得富媒体摘要的"投资回报率最高"？
- A) Organization
- B) FAQPage
- C) WebSite
- D) BreadcrumbList

**Q4**：@graph 解决了什么问题？为什么现实中很少有页面只需要一种 Schema（Schema：数据结构化标记）？

**Q5**：一家设计师耳饰品牌的独立网站有 120 款产品，没有任何 Schema。请列出至少 4 种应该实施的 Schema（Schema：数据结构化标记）类型，并为 Product Schema 写出关键属性。

**Q6**：Schema（Schema：数据结构化标记）标记对 AI SEO 有什么价值？课程提到的"有 Schema（Schema：数据结构化标记）的内容 AI 可见度高 30-40%"是什么意思？

**Q7**：一位开发者在产品页 Schema 中标记了 ratingValue: "4.9"，但页面上没有任何用户评价。请预测 Google 会如何处理，以及对整个网站 SEO 的影响。

---

## 9. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** —— Schema 必须准确反映页面上真实存在的内容。标记不存在的内容是"垃圾结构化数据"，会导致整个网站的 Schema（Schema：数据结构化标记） 被忽略。

**Q2**：参考答案——JSON-LD 核心优势：(1) 分离性——独立 `<script>` 块不嵌入 HTML；(2) Google 官方推荐；(3) 灵活放置；(4) 不污染 HTML 结构。对动态网站尤其友好——Schema 逻辑独立管理。

**Q3**：**B** —— FAQPage 是目前投资回报率最高的 Schema（Schema：数据结构化标记）。条件低（只要有 Q&A 内容）+ 效果显著（搜索结果展开显示、占据 2-3 倍高度、CTR 高 15-30%）。

**Q4**：参考答案——@graph 允许一个 JSON-LD（JSON-LD：结构化数据格式）块包含多种 Schema（Schema：数据结构化标记）。现实中很少有页面只需要一种——博客页需要 Article + BreadcrumbList + Organization + WebSite。没有 @graph 就需要多个独立 JSON-LD（JSON-LD：结构化数据格式）块。

**Q5**：参考答案——应实施：(1) 首页→Organization + WebSite；(2) 产品页→Product（含 offers、aggregateRating、brand）；(3) 分类页→BreadcrumbList + ItemList；(4) 品牌故事页→Article；(5)FAQ（FAQ：常见问题）页→FAQPage。

**Q6**：参考答案——Schema 为 AI 提供"机器可精确理解的语义层"。有 Schema（Schema：数据结构化标记）的内容 AI 可直接提取结构化信息回答问题，不需要在 HTML 文字中乱猜。30-40% 提升指被 AI 系统识别、提取、引用的概率高出 30-40%。

**Q7**：参考答案——Google 检测到页面上没有用户评价但 Schema 标了评分→判定为"垃圾结构化数据"。处理：该页面失去所有富媒体摘要资格，严重或系统性的情况整个网站 Schema 被忽略。SEO 影响：有星级的搜索结果全部变回普通蓝链接，点击率断崖式下跌。手动处罚很难撤销——恢复周期数周到数月。

**评分**：答对 5/7 = 通过。

</details>

---

## 9. 延伸阅读 (continued)

- [[L3-13-新用户引导优化|下一课：新用户引导优化]] — Onboarding CRO
- [[L3-11-AI时代SEO|上一课：AI 时代 SEO]] — Schema 是 AI SEO 的关键实现层
- [[L3-10-SEO审计|L3-10：SEO 审计]] — Layer 1 和 3 需要检查 Schema
