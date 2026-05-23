---
tags: [教程, Skill, 大师课, Level1, tavily, 工具型]
created: 2026-05-04
updated: 2026-05-04
course_number: L1-16
prerequisites: ["[[../../Level0-先修课/L0-02-环境搭建与工具链|L0-02 环境搭建]]"]
next_course: "[[L1-17-AgentBrowser浏览器自动化]]"
source_skill: "[[../../../原始资料/Skills/Tavily Search - SKILL.md]]"
---

# L1-16：Tavily 联网搜索 —— 给 AI 装上搜索引擎

> 🎯 **学什么**：这是工具型 Skill 的进阶形态——一个 Skill 封装了 5 个工具（搜索/提取/爬取/映射/研究），每个工具都有独立的脚本回退。
> 💡 **难易度**：⭐⭐ | ⏱️ **预计时间**：60 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**： 你问 AI「最近有什么大新闻？」——默认的 AI 可能不知道（因为它的知识有截止日期）。

`tavily-search` 做什么？**给 AI 提供联网能力。** 但它不止是搜索——它是一整套 5 个工具的互联网信息获取套件。

你真正要学的是：**「多工具型 Skill」的设计模式**——一个 SKILL.md 如何优雅地管理 5 个不同的能力入口。

### 能力地图

| 这个 Skill 能做什么 | 你学到什么                                |
| ------------- | ------------------------------------ |
| AI 优化网页搜索     | 多工具 Skill 的架构设计                      |
| 从 URL 提取干净内容  | `scripts/` 目录的多脚本组织                  |
| 爬取整个网站        | OpenClaw 插件配置 (openclaw.plugin.json) |
| 发现网站所有 URL    | 环境变量管理 (`TAVILY_API_KEY`)            |
| 深度研究生成报告      | 原生工具 + 脚本回退的双轨设计                     |

***

> [!quote]- 📖 原Skill精要
> 以下是该课程对应的原始 Skill 文件（SKILL.md）中的说明。
>
> **技能名称**：Tavily Search
> **核心功能**：通过 Tavily API 提供 AI 优化的网络搜索、内容提取、网站爬取、网站映射和深度研究。5 个工具覆盖从快速搜索到深度报告的全场景。
> **使用场景**：联网搜索、网页内容提取、网站结构分析、多源研究报告
> **安装方式**：`npx clawhub@latest install tavily-search`
> **必需配置**：`TAVILY_API_KEY` 环境变量
> **来源**：framix-team/openclaw-tavily

***

## 2. 源码精读

### 2.1 YAML 配置（逐行拆解）

> 💡 **白话小课堂**：跟 L1-01 只看一个脚本不同，这个 Skill 的头部多了一个关键字段 `requires: { env: ["TAVILY_API_KEY"] }`——它在告诉系统"没有这个 API（API/应用程序接口：让不同软件之间互相通信的"插座"，调用它就能使用外部服务功能） Key 就别加载我"。这是环境变量依赖声明，比 L1-01 的 `anyBins` 更进了一步。

```yaml
---
name: tavily-search
description: Web search, extraction, crawling, mapping, and deep research 
via Tavily API. Five tools for finding information, extracting content, 
exploring websites, and generating research reports.
metadata:
  openclaw:
    emoji: "🔍"
    requires:
      env:
        - TAVILY_API_KEY
    primaryEnv: TAVILY_API_KEY
---
```

**设计决策分析**：


| 元素 | 为什么这样写 | 如果写错会怎样 |
|------|-------------|---------------|
| `name: tavily-search` | 不带前缀，因为是独立生态（非某个仓库的子 Skill） | — |
| `description` 列举 5 个能力 | 告诉 AI 这个 Skill 不是"只能搜索"——它能做的事多得多 | 如果只写"web search"，AI 在需要提取网页时不会匹配这个 Skill |
| `requires.env: [TAVILY_API_KEY]` | **关键创新**——声明环境变量依赖，系统在加载前检查 | 不声明→系统加载了但调用 API 时报错，浪费上下文 |
| `emoji: "🔍"` | OpenClaw 特有的 UI 增强，纯装饰 | 不写也不影响功能 |

### 2.2 正文结构（多工具索引型）

```markdown
# Tavily Search                      ← 板块 1：简介 + 使用原则

## Default web search                 ← 板块 2：默认行为声明

## When to use                        ← 板块 3：5 工具速查表

## Native tools (preferred)           ← 板块 4：原生工具优先方案

## Script fallback                    ← 板块 5：5 个脚本的完整文档
  ├── Search
  ├── Extract content from URLs
  ├── Crawl a website
  ├── Map a website
  └── Research a topic

## Setup                              ← 板块 6：API Key 配置
## Links                              ← 板块 7：参考链接
```

**这种结构叫「多工具索引」模式**：

```
[简介+原则] → [工具速查] → [原生工具(优先)] → [脚本回退(每个工具一页)] → [配置] → [参考]
```

与 L1-01「脚本说明书」模式的区别：
- L1-01 只有 1 个脚本 → 结构：脚本在哪→参数表→示例
- Tavily 有 5 个工具 → 结构：速查表→原生/脚本双轨→每个工具独立文档

### 2.3 原生工具 + 脚本回退的双轨设计

> 🔑 **核心秘诀**：Tavily Skill 最精妙的设计——**同一功能有两条路径**。如果 OpenClaw 装了 `openclaw-tavily` 插件，直接用原生工具（零 token 开销）；如果没装，回退到 `scripts/` 下的 Node.js 脚本。

```
用户说 "搜索XX"
    │
    ├─→ 有 openclaw-tavily 插件？→ tavily_search 原生工具（推荐）
    │
    └─→ 没有插件？→ node scripts/search.mjs "XX"（回退）
```

**为什么要设计两条路径？**

| 路径 | 优点 | 代价 |
|------|------|------|
| 原生工具 | 零 SKILL.md 正文 token 消耗 | 需要额外安装插件 |
| 脚本回退 | 不需要插件，纯脚本就能跑 | 每次加载 SKILL.md 都会加载脚本文档 |

**这个设计体现了什么？**「渐进增强」——功能越来越多，但基础用法永远可用。

### 2.4 openclaw.plugin.json —— 插件配置的完整范例

```json
{
  "id": "openclaw-tavily",
  "kind": "tools",
  "configSchema": {
    "properties": {
      "apiKey": { "type": "string" },
      "searchDepth": { "enum": ["ultra-fast", "fast", "basic", "advanced"] },
      "maxResults": { "type": "number" },
      "includeAnswer": { "oneOf": [{ "type": "boolean" }, ...] },
      "includeRawContent": { "oneOf": [{ "type": "boolean" }, ...] },
      "timeoutSeconds": { "type": "number" },
      "cacheTtlMinutes": { "type": "number" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "Tavily API Key", "sensitive": true },
    "searchDepth": { "help": "ultra-fast and fast are cheapest; advanced is slowest but most thorough." }
  }
}
```

**每个字段的意义**：

| 字段 | 意义 |
|------|------|
| `configSchema.properties` | 声明所有可配置项，系统会用这个 Schema 验证配置 |
| `enum` 约束 | `searchDepth` 只允许 4 个值，防止拼写错误 |
| `uiHints` | 给 UI 的显示提示——`sensitive: true` 表示密码字段要遮蔽 |
| `oneOf` | `includeAnswer` 既可以是 boolean 也可以是 "basic"/"advanced" 字符串 |

***

## 3. 结构拆解：多工具型 Skill 的通用模板

从这个 Skill 中提取多工具型模板：

````markdown
---
name: [skill-name]
description: [一句话总结 + 列举所有工具]，Use when user asks to "[触发词]".
metadata:
  requires:
    env:
      - [API_KEY_1]
      - [API_KEY_2]
---

# [Skill 名称]

[一句话简介]。

## When to use

- **`[tool_1]`** — [功能描述和使用场景]
- **`[tool_2]`** — [功能描述和使用场景]
- **`[tool_3]`** — [功能描述和使用场景]

## Native tools (preferred)

If the `[plugin-name]` plugin is installed, use these tools directly:

| Tool | Description |
|------|-------------|
| `[tool_1]` | [描述] |
| `[tool_2]` | [描述] |

## Script fallback

### Tool 1

```bash
# {baseDir} = Skill 安装目录（跟所有工具型 Skill 一样）
node {baseDir}/scripts/[tool_1].mjs "input" [options]
```

Options:
- `--option <value>`: [说明] (default: [默认值])

### Tool 2

```bash
node {baseDir}/scripts/[tool_2].mjs "url1" "url2"
```

## Setup

Get an API key at [url].

Set `[API_KEY]` in your environment, or configure via the plugin:
````

### 对比：单工具 vs 多工具的模板差异

| 维度 | 单工具（L1-01 模板） | 多工具（本模板） |
|------|-------------------|-----------------|
| description | 写功能+卖点 | 列举所有工具名 |
| 正文结构 | 脚本在哪→参数表→示例 | 速查表→原生/脚本双轨→每个工具独立文档 |
| 脚本数量 | 1 个 | 5 个（每个工具一个脚本） |
| 配置方式 | EXTEND.md 偏好系统 | ConfigSchema + env 变量 |
| 适用场景 | 单一功能（压缩、转换） | 多功能套件（搜索、提取、爬取） |

***

## 4. 手写模仿

**不看源码，凭理解写一个「多平台社交媒体 API」Skill 的 SKILL.md 框架。**

要求：
- name: `social-media-toolkit`
- 3 个工具：发布帖子、获取分析数据、管理草稿
- API Key: `SOCIAL_API_KEY`
- 每个工具有独立的脚本回退

<details>
<summary>点击查看参考写法</summary>

````markdown
---
name: social-media-toolkit
description: Multi-platform social media management — post, analyze, and manage drafts across platforms. Use when user asks to "post", "schedule", "social media", "publish", "analytics", "drafts".
metadata:
  requires:
    env:
      - SOCIAL_API_KEY
---

# Social Media Toolkit

Manage social media across platforms — post content, fetch analytics, and manage drafts.

## When to use

- **`social_post`** — Create and publish posts to Twitter, LinkedIn, Facebook
- **`social_analytics`** — Get engagement metrics, follower growth, top posts
- **`social_drafts`** — Create, list, update, and delete draft posts

## Native tools (preferred)

If the `social-plugin` is installed, use these tools directly:

| Tool | Description |
|------|-------------|
| `social_post` | Publish to one or multiple platforms simultaneously |
| `social_analytics` | Fetch analytics with date range and platform filters |
| `social_drafts` | CRUD operations on draft posts |

## Script fallback

### Post to social media

```bash
node {baseDir}/scripts/post.mjs "content" --platforms twitter,linkedin
node {baseDir}/scripts/post.mjs "content" --platforms all --schedule "2026-05-10 09:00"
```

### Get analytics

```bash
node {baseDir}/scripts/analytics.mjs --from 2026-04-01 --to 2026-05-01
node {baseDir}/scripts/analytics.mjs --platform twitter --metric engagement
```

### Manage drafts

```bash
node {baseDir}/scripts/drafts.mjs list
node {baseDir}/scripts/drafts.mjs create "Draft content here" --platform twitter
node {baseDir}/scripts/drafts.mjs delete draft_123
```

## Setup

Get an API key at [social-api.example.com].

```bash
export SOCIAL_API_KEY="sk-..."
```
````

</details>

***

## 5. 深度思考

### 5.1 为什么 Tavily 要分成 5 个工具而不是一个大工具？

四个原因：
1. **AI 精确匹配**：AI 匹配 Skill 时只看 description。如果只有一个大工具叫 "search"，用户说"帮我提取这个网页的内容"，AI 不会匹配。
2. **脚本复用**：`crawl.mjs` 和 `map.mjs` 可以共享相同的爬取逻辑但输出不同格式。
3. **Token 成本**：如果用户只需要搜索，只加载 search 脚本的文档就够了，不需要加载 extract/crawl/map/research 的。
4. **独立迭代**：提取功能可以单独升级而不影响搜索功能。

**教训**：如果一套功能的使用场景差异很大，拆成多个小工具比做成一个大工具更好。

### 5.2 为什么脚本名用 `.mjs` 后缀？

`.mjs` = ES Module JavaScript（ES = ECMAScript，JavaScript 的国际标准名称；JavaScript 是浏览器和 Node.js（Node.js：让 JavaScript 脱离浏览器在服务器上运行的环境）中运行最广泛的编程语言）。相比 `.js`：
- `.mjs` 明确声明使用 ES Module 语法（`import`/`export`）
- Node.js 不需要 `package.json` 里的 `"type": "module"` 就能识别
- 跨平台兼容性更好——不依赖项目级配置

### 5.3 环境变量 vs 配置文件：Tavily 为什么选环境变量？

| 方式 | 优点 | 缺点 |
|------|------|------|
| 环境变量 | 安全性最高，不入库，CI/CD 友好 | 新用户需要手动设置 |
| 配置文件 | 可视化编辑，方便切换 | API Key 可能被误提交到 Git |

Tavily 选了环境变量（`TAVILY_API_KEY`），但也提供了插件配置作为备选方案——又是一个「双轨」设计。

***

## 6. 实战案例：用 Tavily 十分钟完成竞品调研

> 🕵️ **实战案例**：你是一个 SaaS 创业者，需要快速了解竞品「Notion」的最新动态和产品结构。
>
> **用 Tavily Skill 的 4 步调研流程**：
>
> **Step 1 — 搜索最新动态**
> ```bash
> tavily_search "Notion AI features 2026" --topic news --time-range month
> ```
> → 获得 5 篇最新的 Notion AI 功能报道
>
> **Step 2 — 提取关键文章**
> ```bash
> tavily_extract "https://techcrunch.com/notion-ai-2026" --format markdown
> ```
> → 全文提取为干净的 Markdown，方便 AI 分析
>
> **Step 3 — 映射官网结构**
> ```bash
> tavily_map "https://notion.so" --depth 2 --instructions "Find product and pricing pages"
> ```
> → 发现 Notion 网站有 /product、/pricing、/templates、/integrations 等关键页面
>
> **Step 4 — 深度研究生成报告**
> ```bash
> tavily_research "Compare Notion's AI strategy with competitors ClickUp and Coda in 2026"
> ```
> → 自动生成一份包含引用来源的对比研究报告
>
> **全流程耗时**：约 10 分钟（含 AI 分析时间）
> **传统方式**：至少 2-3 小时的搜索+阅读+整理
>
> > 🔑 **启示**：多工具 Skill 的价值不在于"每个工具强"，而在于"工具之间无缝衔接"——搜索→提取→映射→研究形成一条完整的信息获取链。

***

## 7. 常见错误

| ❌ 错误 | ✅ 正确 |
|---------|--------|
| 忘记设置 `TAVILY_API_KEY` 环境变量 | 先在终端 `echo $TAVILY_API_KEY` 确认已设置 |
| 把 5 个工具的实现逻辑写在 SKILL.md 正文里 | 正文只写「怎么调用」，实现放 `scripts/` |
| `searchDepth` 始终用 advanced（慢且贵） | 快速搜索用 ultra-fast，深度研究才用 advanced |
| 不写 `includes`/`excludes` 域名过滤 | 搜索时可以指定只搜索某个域名，提高精度 |
| 忘记 `cacheTtlMinutes` 导致重复搜索浪费额度 | Tavily 免费版有月度额度，合理使用缓存 |

***

## 8. 掌握检验

**Q1**：Tavily Search 的 SKILL.md 中，`requires.env: [TAVILY_API_KEY]` 这个声明的作用是什么？它与在文档里写「需要设置 API Key」有什么区别？

**Q2**：假设你需要给 Tavily 增加第 6 个工具「tavily_summarize」（批量 URL 摘要），除了写脚本 `scripts/summarize.mjs` 之外，SKILL.md 的哪些部分需要修改？

**Q3**：以下哪个选项正确描述了 Tavily Skill 中「原生工具」和「脚本回退」的关系？
- A) 两者互相排斥，只能选一个
- B) 原生工具优先，不可用时回退到脚本
- C) 脚本优先，原生工具是备用方案
- D) 两者必须同时可用才能工作

**Q4**：`openclaw.plugin.json` 中的 `uiHints` 字段是干什么用的？如果去掉 `"sensitive": true` 标记会发生什么？

**Q5**：Tavily 为什么要设计 5 个独立工具而不是 1 个全能工具？给出你理解的两个以上原因。

**Q6**（开放性）：如果你要用 Tavily Skill 调研「2026 年 AI 编程助手市场」，请设计一个 3-4 步的调研流程，每一步说明用什么工具、为什么用这个工具。

**Q7**：多工具型 Skill 模板和单工具型模板在 `description` 字段的写法上有什么关键区别？为什么？

***

## 9. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：`requires.env: [TAVILY_API_KEY]` 是系统级别的依赖声明——系统在加载 Skill 前会检查环境变量是否存在，不存在则不会加载该 Skill。单纯在文档里写「需要设置 API Key」只是给人看的提示，系统不会自动检查。声明式依赖让 Skill 加载失败的原因能被系统自动诊断，而不是在执行时才报错。

**Q2**：需要修改 4 处：1) YAML 的 `description` 中添加 "summarization"；2) 「When to use」速查表中新增 `tavily_summarize` 条目；3) 「Native tools」表中（如果有原生工具）新增对应条目；4) 「Script fallback」中新增「Summarize URLs」板块，写出脚本调用方式和选项。此外也可以在「Setup」或「Links」中补充相关文档链接。

**Q3**：**B** — 原生工具优先，不可用时回退到脚本。这是一种「渐进增强」设计：有插件时享受零 token 开销的原生体验，没插件时脚本照样能跑。

**Q4**：`uiHints` 是给图形界面（如 OpenClaw 的设置面板）的显示提示。`"sensitive": true` 告诉 UI「这是密码字段，输入时要遮蔽显示（显示为 ****）」。去掉后 API Key 会在 UI 中明文显示，有安全风险。

**Q5**：1) AI 精确匹配——AI 根据 description 中的关键词匹配工具，细粒度工具匹配更准；2) 脚本复用——crawl 和 map 可共享爬取逻辑但输出不同；3) Token 成本——用哪个加载哪个的文档，不浪费上下文；4) 独立迭代——每个工具可以独立升级，互不影响。

**Q6**（参考流程）：Step 1 — `tavily_search "AI coding assistants market 2026" --topic news` 获取最新市场报道；Step 2 — `tavily_extract <关键报道URL>` 提取全文深入分析；Step 3 — `tavily_map "https://cursor.com" --depth 2` 了解头部产品的网站结构；Step 4 — `tavily_research "Compare GitHub Copilot, Cursor, and Windsurf in 2026: features, pricing, market position"` 生成综合对比报告。

**Q7**：单工具型 description 写「功能 + 卖点」（如 "automatic tool selection"），多工具型 description 要**列举所有工具**（如 "Five tools for finding information, extracting content, exploring websites, and generating research reports"）。因为单工具只需要告诉 AI 这一个工具比别的强在哪，多工具需要让 AI 知道这个 Skill 能覆盖哪些不同的子任务，以便正确匹配。

（6/7 通过）

</details>

***

## 10. 延伸阅读

- [[L1-17-AgentBrowser浏览器自动化|下一课：Agent Browser 浏览器自动化]] — CLI 驱动型 Skill 的另一种设计
- [[L1-01-智能图片压缩|L1-01 图片压缩]] — 回顾单工具型 Skill 的模板
- [[../../Level5-大师/L5-课程总览|Level 5 大师课]] — browser-use：更高级的浏览器自动化 Skill
- [Tavily API 官方文档](https://docs.tavily.com)
- [openclaw-tavily 源码](https://github.com/framix-team/openclaw-tavily)
