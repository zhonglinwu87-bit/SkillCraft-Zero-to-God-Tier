---
tags: [教程, Skill, 大师课, Level1, agent-browser, 工具型, CLI]
created: 2026-05-04
updated: 2026-05-04
course_number: L1-17
prerequisites: ["[[../../Level0-先修课/L0-02-环境搭建与工具链|L0-02 环境搭建]]"]
source_skill: "[[../../../原始资料/Skills/Agent Browser - SKILL.md]]"
---

# L1-17：Agent Browser —— 让 AI 像人一样操作浏览器

> 🎯 **学什么**：这是「纯 CLI 型 Skill」的典范——不需要 scripts/ 目录，不需要 node 脚本，一个全局命令行工具就是全部。但它用 50+ 个子命令覆盖了浏览器自动化的所有场景。
> 💡 **难易度**：⭐⭐ | ⏱️ **预计时间**：60 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**： 你跟 AI 说「帮我看看这个网页里有什么」，AI 根本看不见网页。Agent Browser 就是给 AI 装了一双「眼睛」和一只「手」——眼睛能看到网页内容（snapshot、screenshot），手能操作网页（click、type、fill form）。它通过 CDP（CDP/Chrome DevTools Protocol：Chrome 开发者工具协议，让外部程序能操控浏览器）协议控制真实的 Chrome 浏览器。

`agent-browser` 做什么？**让 AI 通过命令行控制一个真实的 Chrome 浏览器。** 打开网页、点击按钮、填写表单、截图、提取数据——全部通过一行命令完成。

你真正要学的是：**「命令驱动型 Skill」的设计模式**——没有脚本、没有 API Key、纯命令行工具如何被封装成 Skill。

### 能力地图

| 这个 Skill 能做什么 | 你学到什么 |
|-------------------|-----------|
| 打开网页、导航、截图 | CLI 型 Skill 的 SKILL.md 写法 |
| 填写表单、点击按钮、滚动 | 50+ 子命令的分类和组织方式 |
| 提取文本/HTML/属性/数量 | `snapshot` + `@ref` 精准定位系统 |
| 运行 JavaScript 在页面中 | 常用模式（Pattern）的文档化方法 |
| 连接已有浏览器实例 | 内置 Skill 系统（`agent-browser skills`）|

***

> [!quote]- 📖 原Skill精要
> 以下是该课程对应的原始 Skill 文件说明。
>
> **技能名称**：Agent Browser
> **核心功能**：专为 AI Agent 设计的快速 headless 浏览器自动化 CLI。支持导航、交互、数据提取、截图、PDF、JS 执行等 50+ 命令。
> **使用场景**：网页浏览、数据抓取、表单自动填写、网页截图、可访问性分析
> **安装方式**：`npm install -g agent-browser`（npm/Node 包管理器：JavaScript 世界最大的"应用商店"，`-g` 表示全局安装——装一次，所有项目都能用）
> **版本**：0.26.0 | **来源**：TheSethRose/agent-browser

***

## 2. 源码精读

> 📖 本节逐块阅读 `Agent Browser - SKILL.md`（共 177 行），配合作者的设计意图分析，帮你理解每一行"为什么这样写"。

### 2.1 YAML 头部——最小化配置

```yaml
---
name: agent-browser
description: Headless browser automation CLI for AI agents. Visit pages, 
click elements, fill forms, take screenshots, extract data, and run 
JavaScript — all from the command line. Use when user asks to "browse", 
"screenshot", "scrape", "fill form", "automate browser", or "open webpage".
---
```

**逐行解读**：

| 行 | 内容 | 设计意图 |
|----|------|---------|
| 1 | `name: agent-browser` | 对应 npm 包名，AI 可以通过 name 找到正确的 CLI 工具 |
| 2-4 | `description: ...` | **这一段是整个 Skill 的"触发开关"**——包含了 8 个动词（browse, screenshot, scrape, fill form, automate browser, open webpage），覆盖了用户可能说出的几乎所有浏览器操作表述 |
| 缺失 | 没有 `requires` | 本地 CLI 工具不需要 API Key —— 这是它最本质的优势 |

> [!tip] 对比思考
> Tavily 的 YAML 头有 `requires.env: [TAVILY_API_KEY]`，因为它是 API 服务。Agent Browser 是本地工具——`npm install -g` 后直接能用。要不要 `requires` 不是审美选择，是被"工具的性质"决定的。

### 2.2 定位声明 + 安装——前两段决定 Skill 的第一印象

源文件正文前两段：

```markdown
# Agent Browser

Fast, headless browser automation CLI built for AI agents. 
Navigate, interact, extract, and capture — without Playwright overhead.

## When to use

- **Visit + interact** — Open pages, click buttons, fill forms, navigate
- **Extract data** — Get text, HTML, attributes, counts, styles from pages
- **Take screenshots** — Full page or viewport captures
- **Accessibility snapshots** — Get the page's accessibility tree with 
  @ref handles for precise element targeting
- **Run JavaScript** — Execute arbitrary JS in page context
- **Download + Upload** — Handle files through the browser

## Installation

```bash
npm install -g agent-browser
```

Verify:
```bash
agent-browser --version
```
```

**逐段分析**：

**① 一句话定位**（第 3-4 行）：
> "Fast, headless browser automation CLI built for AI agents."

三个关键信息压缩在一句话里：**快**（Fast）、**无头**（headless）、**为 AI Agent 打造**（built for AI agents）。最后半句 "without Playwright overhead" 直接回应了开发者最常见的疑问——"为什么不用 Playwright？"

**② When to use**（第 8-14 行）：
6 个场景，每个用 `**关键词** — 解释` 格式。这里有个不易察觉的精妙设计：每个场景的**关键词本身就是触发词**。AI 读到用户说 "extract data" 时能直接匹配到第 2 个场景。

**③ Installation**（第 18-26 行）：
CLI 型 Skill 的标配——告诉 AI 怎么装、怎么验证。如果用户说 "agent-browser 命令找不到"，AI 可以凭这两行给出准确的安装指令。

### 2.3 Navigation + Interaction ——「移动」和「操作」

源文件 Core Commands 的前两个类别：

**Navigation（4 个命令）**：

```bash
agent-browser open <url>          # Navigate to URL
agent-browser back                # Go back
agent-browser forward             # Go forward
agent-browser reload              # Reload page
```

**Interaction（12 个命令）**：

```bash
agent-browser click <sel>         # Click element (CSS selector or @ref)
agent-browser dblclick <sel>      # Double-click element
agent-browser type <sel> <text>   # Type into element
agent-browser fill <sel> <text>   # Clear and fill
agent-browser press <key>         # Press key (Enter, Tab, Control+a)
agent-browser hover <sel>         # Hover element
agent-browser focus <sel>         # Focus element
agent-browser check <sel>         # Check checkbox
agent-browser uncheck <sel>       # Uncheck checkbox
agent-browser select <sel> <val>  # Select dropdown option
agent-browser scroll <dir> [px]   # Scroll up/down/left/right
agent-browser scrollintoview <sel> # Scroll element into view
agent-browser wait <sel|ms>       # Wait for element or time
```

**深入解读**：

**① `type` vs `fill` 的区别**——一个容易被忽略但非常重要的设计：
- `type <sel> <text>` — 逐字符输入，触发每个按键事件（适合搜索框、自动补全）
- `fill <sel> <text>` — 先清空再一次性填入（适合表单字段，更快）

这说明作者深入理解了 Web 自动化的真实痛点——不是所有"输入"都一样。

**② `<sel>` 参数的双重语义**：
命令注释中写 "CSS selector or @ref"，意味着同一个参数位置可以传两种完全不同的东西。这种设计的巧妙之处在于：AI 可以先 `snapshot` 拿 @ref，如果 @ref 不行再 fallback 到 CSS 选择器。

**③ `wait` 的两用设计**：
```bash
agent-browser wait 2000         # 等 2 秒（毫秒）
agent-browser wait ".loaded"    # 等某个元素出现
```
既可以传数字（固定等待），也可以传选择器（条件等待）。一个命令覆盖两种完全不同的等待策略。

### 2.4 Extraction + State Checks + Find Elements ——「看到什么」+「还在吗」+「在哪」

**Extraction（10 个命令）**——数据提取的全部手段：

```bash
agent-browser get text <sel>       # Get element text
agent-browser get html <sel>       # Get element HTML
agent-browser get value <sel>      # Get input value
agent-browser get attr <name> <sel> # Get attribute
agent-browser get title             # Get page title
agent-browser get url               # Get current URL
agent-browser get count <sel>       # Count matching elements
agent-browser get box <sel>         # Get bounding box
agent-browser get styles <sel>      # Get computed styles
agent-browser get cdp-url           # Get CDP WebSocket URL
```

**State Checks（3 个命令）**——判断元素状态：

```bash
agent-browser is visible <sel>     # Check if element is visible
agent-browser is enabled <sel>     # Check if element is enabled
agent-browser is checked <sel>     # Check if checkbox is checked
```

**Find Elements（10 个命令）**——10 种定位方式：

```bash
agent-browser find role <role> <action> [text]       # By ARIA role
agent-browser find text <text> <action>              # By text content
agent-browser find label <label> <action>            # By associated label
agent-browser find placeholder <text> <action>       # By placeholder
agent-browser find alt <text> <action>               # By alt text
agent-browser find title <text> <action>             # By title attribute
agent-browser find testid <id> <action>              # By data-testid
agent-browser find first <sel> <action>              # First match
agent-browser find last <sel> <action>               # Last match
agent-browser find nth <n> <sel> <action>            # Nth match
```

> [!note] 关键设计洞察
> **这三组命令合在一起，构成了浏览器的"感知系统"**：
> - Extraction = AI 的"眼睛"——看到页面内容
> - State Checks = AI 的"触觉"——确认元素是否可交互
> - Find Elements = AI 的"搜索能力"——10 种方式找到目标元素
>
> 为什么 Find Elements 要提供 10 种定位方式？因为网页上定位同一个按钮，有时只能用文本（`find text "Submit"`），有时只能用 placeholder（`find placeholder "Enter email"`），有时只能用 testid（`find testid "login-btn"`）。这 10 种方式覆盖了所有可能的定位场景，AI 不需要学 CSS 选择器就能找到元素。

### 2.5 Capture + Advanced ——「存下来」+「更高级的事」

**Capture（3 个命令）**：

```bash
agent-browser screenshot [path]    # Take screenshot
agent-browser pdf <path>           # Save as PDF
agent-browser snapshot             # Accessibility tree with @ref handles
```

这三个命令中，**`snapshot` 是最重要的**——它不是简单的截图，而是输出页面的无障碍树（accessibility tree），并给每个可交互元素分配一个 `@ref` 编号。后续所有交互命令都可以用 `@ref=N` 来精确定位。

**Advanced（7 个命令）**：

```bash
agent-browser eval <js>            # Run JavaScript in page
agent-browser drag <src> <dst>     # Drag and drop
agent-browser upload <sel> <files> # Upload files
agent-browser download <sel> <path> # Download file via click
agent-browser keyboard type <text>  # Type with real keystrokes
agent-browser keyboard inserttext <text> # Insert without key events
agent-browser connect <port|url>    # Connect to existing browser via CDP
agent-browser close [--all]         # Close browser
```

**深入解读**：

**① `eval`——万能后门**：当所有内置命令都不够用时，`eval` 允许 AI 直接执行 JavaScript。这是 CLI 工具设计中常见的"逃生舱"（escape hatch）——不限制高级用户的可能性。

**② `keyboard type` vs `type`**：源文件中有两个"输入"命令。
- `type <sel> <text>` — 聚焦元素后输入（高层抽象）
- `keyboard type <text>` — 模拟真实键盘逐键按下（底层模拟）

对 AI 来说，前者是首选（简单直接）；后者用于无法用选择器定位的场景（如聚焦在 Canvas 里）。

**③ `connect`——连接已有浏览器**：这是高手功能。允许 Agent Browser 接管一个已经在运行的 Chrome 实例（通过 CDP 协议）。这意味着 AI 可以在你**正在浏览的网页**上操作——不是打开新窗口，而是直接在你眼前的页面上点击、填写。

### 2.6 @ref 系统——这个 Skill 最聪明的设计

```bash
agent-browser snapshot
# 输出：
# └─ document
#    ├─ banner
#    │  ├─ heading "Welcome"          @ref=1
#    │  └─ button "Get Started"       @ref=2
#    ├─ main
#    │  ├─ textbox "Email"            @ref=3
#    │  └─ textbox "Password"         @ref=4
#    └─ contentinfo
#       └─ link "About"               @ref=5

agent-browser get text "@ref=1"   # "Welcome"
agent-browser click "@ref=2"      # 点击按钮
agent-browser fill "@ref=3" "me@example.com"  # 填写邮箱
```

**@ref 解决了什么问题？**

CSS 选择器（如 `div.container > form > input:nth-child(2)`）有三个致命问题：
1. **脆弱**——网站改版，选择器就失效
2. **难写**——需要理解 DOM 结构
3. **不语义化**——AI 不知道 `input:nth-child(2)` 是密码框还是邮箱框

@ref 系统用**无障碍树**代替 DOM 树：
- **稳定性**：无障碍树比 DOM 结构更稳定（改 CSS 不影响无障碍结构）
- **语义化**：`textbox "Email"` 比 `input[type="email"]` 更直观
- **AI 友好**：AI 读 `@ref=3` 比读 CSS 选择器更容易理解和操作

> [!tip] 工作流闭环
> ```
> snapshot → 分析 @ref → click/fill/get text @ref=N → 完成任务
> ```
> 这是一个**封闭的认知循环**：先看（snapshot）、再定位（@ref）、最后操作。AI 不需要理解网页的 DOM 结构——只需要看懂无障碍树。

### 2.7 Common Patterns ——4 个模式如何覆盖 80% 场景

源文件提供了 4 个模式，我们逐行解读：

**Pattern 1：Navigate + Extract（浏览 + 提取）**

```bash
agent-browser open "https://example.com"
agent-browser snapshot
agent-browser get text "h1"
agent-browser close
```

> 最基础的模式：打开 → 看结构 → 取数据 → 关闭。适用于 90% 的简单抓取任务。

**Pattern 2：Form Fill（表单填写）**

```bash
agent-browser open "https://example.com/form"
agent-browser fill "input[name='email']" "user@example.com"
agent-browser fill "input[name='password']" "secret"
agent-browser click "button[type='submit']"
agent-browser wait 2000
agent-browser get text ".success-message"
agent-browser close
```

> 这个模式揭示了自动化表单的 5 个步骤：**打开 → 填写 → 提交 → 等待 → 验证**。注意 `wait 2000` 的位置——在 `click` 之后、`get text` 之前。这个顺序不能错，否则提取的是旧页面的内容。

**Pattern 3：Screenshot Capture（截图）**

```bash
agent-browser open "https://example.com"
agent-browser wait 1000
agent-browser screenshot output.png
agent-browser close
```

> 截图前 `wait 1000` 是一个实用经验——给页面渲染留时间，避免截到白屏。

**Pattern 4：Data Extraction with @ref（精确定位提取）**

```bash
agent-browser open "https://example.com/products"
agent-browser snapshot
agent-browser get text "@ref1"
agent-browser close
```

> 展示了 @ref 系统的标准用法：snapshot → @ref → 操作。适用于页面结构复杂、CSS 选择器难以编写的场景。

### 2.8 Built-in Skills + Notes ——自举系统和底层约束

**Built-in Skills**：

```bash
agent-browser skills list              # List available skills
agent-browser skills get core --full   # Full command reference and patterns
agent-browser skills get <name>        # Load specialized skill
agent-browser skills path [name]       # Print skill directory path
```

这是一个被大多数人忽略的精妙设计：**Agent Browser 自己就内置了一套"Skill 系统"**。执行 `skills get core --full` 会返回一份详尽的命令参考和模式指南——这实际上是一个"二级 SKILL.md"，专门给 AI 在需要时查阅更深层的帮助。

**Notes——底层的 5 个关键约束**：

```markdown
- Agent Browser uses a **persistent browser profile** 
  — cookies and sessions survive between commands
- The `snapshot` command returns an **accessibility tree** 
  with `@ref` handles — use these for precise targeting
- For complex workflows, chain multiple commands together; 
  each command runs in the same session
- Use `connect` to attach to an already-running Chrome instance 
  instead of launching a new one
- Version: 0.26.0 | Source: https://github.com/TheSethRose/agent-browser
```

这 5 条注释回答了使用中最关键的 3 个问题：
1. **"两次命令之间状态会丢失吗？"** → 不会，persistent browser profile 保证 Cookies 和 Session 跨命令保持
2. **"怎么精准操作页面？"** → 用 `snapshot` 拿 @ref
3. **"能在我现有的浏览器上操作吗？"** → 能，用 `connect`

### 2.9 命令分类手册模式的通用公式

总结这个 177 行源文件的骨架：

```
[定位声明]  →  一句话告诉 AI 这是什么
[能力清单]  →  6 个场景覆盖所有触发条件
[安装步骤]  →  给 AI 装上工具的指令
[命令手册]  →  7 个类别 × 50+ 命令（按认知模型分组）
[内置帮助]  →  工具自己的 Skill 系统（二级文档）
[常用模式]  →  4 个 Pattern 覆盖 80% 场景
[关键约束]  →  5 条 Notes 回答最深层的使用疑问
```

与之前学过的模式对比：

| 模式 | 代表 Skill | 命令组织 | 核心复杂度 |
|------|-----------|---------|-----------|
| 脚本说明书 | L1-01 图片压缩 | 1 脚本 + 参数表 | 参数组合 |
| 多工具索引 | L1-16 Tavily | 5 工具分别说明 | 工具选择 |
| **命令分类手册** | **L1-17 Agent Browser** | **7 类别 + 4 模式** | **命令发现和组合** |

***

## 3. 结构拆解：CLI 型 Skill 的通用模板

```markdown
---
name: [cli-tool-name]
description: [一句话总结 + 列举关键能力动词]，Use when user asks to 
"[动词1]", "[动词2]", "[动词3]".
---

# [工具名称]

## When to use

- **[场景1]** — [描述]
- **[场景2]** — [描述]

## Installation

```bash
[安装命令]
```

## Core Commands

### [类别1]
```bash
command1 <arg>     # 说明
command2 <arg>     # 说明
```

### [类别2]
```bash
command3 <arg>     # 说明
```

## Common Patterns

### Pattern N: [场景名称]
```bash
command1
command2
```

## Notes

- [关键注意事项1]
- [关键注意事项2]
```

### 什么时候用这个模板？

| 判断条件 | 是→用 |
|---------|-------|
| 工具是全局 CLI 命令？ | ✅ |
| 不需要额外脚本来包装？ | ✅ |
| 命令数 > 20？ | ✅ |
| 有多种使用模式（不只一种）？ | ✅ |

***

## 4. 手写模仿

**不看源码，凭理解写一个「Git 操作」CLI 型 Skill 的 SKILL.md。**

要求：
- name: `git-helper`
- 覆盖的场景：提交、分支管理、查看历史、解决冲突
- 至少 3 个常用 Pattern
- 分为 4 个命令类别

<details>
<summary>点击查看参考写法</summary>

```markdown
---
name: git-helper
description: Git command-line helper for AI agents. Stage, commit, branch, merge, and resolve conflicts — all from the command line. Use when user asks to "commit", "branch", "merge", "resolve conflict", "git log", "git status".
---

# Git Helper

Safe Git operations wrapped for AI agents. Never force-pushes to main or deletes remote branches without confirmation.

## When to use

- **Stage + Commit** — Add files, write commits, amend
- **Branch Operations** — Create, switch, list, delete branches
- **History Inspection** — Log, diff, blame, show
- **Conflict Resolution** — Status, diff, merge tools

## Installation

```bash
# Git must be installed: https://git-scm.com
git --version
```

## Core Commands

### Stage + Commit
```bash
git add <files...>              # Stage files
git commit -m "message"         # Commit staged
git commit --amend -m "new"     # Amend last commit
```

### Branch Operations
```bash
git branch                      # List branches
git checkout -b <name>          # Create and switch
git merge <branch>              # Merge branch
```

### History Inspection
```bash
git log --oneline -n 10         # Recent commits
git diff HEAD~1                 # Last change
git blame <file>                # Who wrote what
```

### Conflict Resolution
```bash
git status                      # See conflicted files
git diff --name-only --diff-filter=U  # List conflicts
```

## Common Patterns

### Pattern 1: Feature Branch Workflow
```bash
git checkout -b feature/new-thing
# ... make changes ...
git add .
git commit -m "feat: add new thing"
git checkout main
git merge feature/new-thing
```

### Pattern 2: Check What Changed
```bash
git status
git diff --stat
git log --oneline -n 5
```

### Pattern 3: Fix Last Commit
```bash
git add forgotten-file.txt
git commit --amend -m "corrected message"
```

## Safety Rules

- NEVER `git push --force` to main/master
- NEVER `git branch -D` without confirmation
- Always `git status` before committing
```
</details>

***

## 5. 深度思考

### 5.1 Agent Browser vs Playwright MCP：为什么选 CLI 而不是 JS API？

| 维度 | Agent Browser (CLI) | Playwright MCP (Tool) |
|------|-------------------|----------------------|
| 安装 | `npm install -g` | 需要 MCP Server 配置 |
| 调用方式 | AI 直接输命令 | AI 调用 Tool 函数 |
| 调试 | 可以在终端手动试 | 只能通过 AI 调用 |
| Token 消耗 | 中等（命令文本） | 低（工具调用） |
| 灵活性 | 高（命令组合自由）| 受限于 Tool 定义 |

Agent Browser 选择了 CLI 路线，这意味着一件事：**它既可以被 AI 用，也可以被人直接用。** 这是 CLI 型 Skill 的独特优势——不绑定任何 AI 平台。

### 5.2 内置 Skill 系统：为什么一个 CLI 工具要自带 Skill？

```bash
agent-browser skills list
agent-browser skills get core --full
agent-browser skills get electron
agent-browser skills get slack
```

Agent Browser 自己就内置了「Skill 系统」——这是 **Skill 的 Skill**。`skills get core` 返回的不是枯燥的 flag 列表，而是精心编写的指南，包含：
- 常用模式（带可复制的示例）
- ref/selector 使用指南
- 专门场景（Electron 应用、Slack、探索性测试）

**启示**：好的工具不只是提供功能，还提供「使用智慧」。Skill 不是功能的说明书，是使用功能的**方法论**。

### 5.3 为什么命令名用自然语言而不是缩写？

```bash
agent-browser get text "h1"       # ✅ 自然语言
agent-browser g t "h1"            # ❌ 如果写成缩写
```

Agent Browser 选择了「宁可长一点，但要一眼看懂」的设计。对人来可能有点啰嗦，但对 AI 来说——自然语言命令就是最好的 API。AI 不需要记 `g` 是 `get` 的缩写，写 `get text` 自然就知道在干嘛。

这是 CLI 设计中的一个趋势：**为 AI 优化命令命名，而不是为打字速度优化。**

***

## 6. 实战案例：用 Agent Browser 自动签到领积分

> 🤖 **实战案例**：某开发者社区每天签到可以领积分，但经常忘记。用 Agent Browser 写一个自动化脚本，让 AI 每天帮忙签到。
>
> **4 步签到流程**：
>
> ```bash
> # Step 1：打开签到页面
> agent-browser open "https://dev-community.example.com/login"
>
> # Step 2：登录
> agent-browser fill "input[name='email']" "$DEV_EMAIL"
> agent-browser fill "input[name='password']" "$DEV_PASSWORD"
> agent-browser click "button[type='submit']"
> agent-browser wait 2000
>
> # Step 3：找到签到按钮
> agent-browser snapshot
> # 从输出中找到签到按钮的 @ref
> agent-browser click "@ref=15"
>
> # Step 4：验证签到成功
> agent-browser get text ".checkin-success-message"
> # 输出："签到成功！+10 积分，已连续签到 42 天"
>
> agent-browser close
> ```
>
> **进一步优化**：可以把这些命令保存成 `.sh` 脚本，配合 cron 定时执行——全程不需要打开浏览器。
>
> > 🔑 **启示**：Agent Browser 不只是帮 AI 操作网页，更是把网页操作变成了可脚本化的自动化流程。任何「每天要在网页上做的事情」都可以用它自动化。

***

## 7. 常见错误

| ❌ 错误 | ✅ 正确 |
|---------|--------|
| 用 CSS 选择器而不先用 snapshot | 先 `snapshot` 获取 @ref，再用 @ref 定位——更稳定 |
| 命令之间不加 `wait` | 页面加载需要时间——在 click/fill 后加 `wait 1000-2000` |
| 忘记 `close` 命令 | 每次用完要关闭浏览器，否则残留进程越来越多 |
| 把 50+ 命令全列在 SKILL.md 里不加分类 | 按功能分 7 个类别，AI 才能快速定位 |
| description 只写 "browser automation" | 加上具体动词：browse, screenshot, scrape, fill form |

***

## 8. 掌握检验

**Q1**：Agent Browser 的 YAML 头部为什么没有 `requires` 字段？与 Tavily Search 的 `requires.env: [TAVILY_API_KEY]` 相比，这说明两种 Skill 在设计上有什么本质区别？

**Q2**：Agent Browser 的 50+ 个子命令被分成 7 个类别。如果让你给一个「文件管理」CLI 工具设计命令分类，你会分哪几个类别？（至少 4 个）

**Q3**：以下哪项最准确地描述了 `@ref` 系统的核心优势？
- A) 比 CSS 选择器写起来更短
- B) 基于无障碍树，更稳定、更语义化、对 AI 更友好
- C) 比 XPath 运行更快
- D) 不需要先加载页面就能使用

**Q4**：CLI 型 Skill 的正文结构被称为「命令分类手册」模式。请写出这个模式的 6 个板块（按顺序），并说明为什么「Installation」板块是 CLI 型 Skill 特有的。

**Q5**：Agent Browser 内置了自己的「Skill 系统」。这种「工具的 Skill」和我们学的「Agent Skill」（SKILL.md）有什么区别和联系？为什么说这是「Skill 的 Skill」？

**Q6**（开放性）：某个你经常访问的网站需要每天做同样的事情（比如查看订单状态）。请用 Agent Browser 的命令写一个 3-5 步的自动化流程，说明每一步在做什么。

**Q7**：命令行参数设计上有两种哲学：缩写优先（curl、git）和自然语言优先（Agent Browser）。对于 AI Agent 使用的 CLI 工具，为什么自然语言优先是更好的选择？

**Q8**：综合对比：L1-01（图片压缩）是「脚本说明书」模式，L1-16（Tavily）是「多工具索引」模式，L1-17（Agent Browser）是「命令分类手册」模式。这三种模式各适用于什么类型的 Skill？

***

## 9. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：Agent Browser 不需要 `requires` 字段因为它是一个本地 CLI 工具——`npm install -g` 后就能直接使用，不依赖外部 API 或特殊权限。Tavily 需要 `requires.env` 因为它依赖外部 API Key 才能工作。这揭示了两种 Skill 的本质区别：本地工具型（自给自足）vs API 依赖型（需要外部凭证）。

**Q2**（参考分类）：1) 导航——cd, ls, pwd（定位和查看）；2) 操作——cp, mv, rm, mkdir（增删改）；3) 查看——cat, head, tail, less（读取内容）；4) 查找——find, grep, which（搜索定位）；5) 权限——chmod, chown（权限管理）。分类原则：每个类别对应一个用户意图（我要去哪、我要干嘛、我想看什么）。

**Q3**：**B** — @ref 基于无障碍树（accessibility tree），比 DOM 树更稳定（网站改版通常不会改变无障碍结构），更语义化（textbox "Email" 比 `input[type="email"]` 直观），且对 AI 更友好（AI 擅长理解语义化标签）。

**Q4**：6 个板块按顺序：能力清单 → 安装步骤 → 命令手册(按类别) → 内置帮助 → 常用模式 → 注意事项。「Installation」板块是 CLI 型 Skill 特有的，因为脚本型 Skill 的安装由系统托管（npx skills add），而 CLI 工具需要用户手动 `npm install -g`。把这个步骤写在 Skill 里，AI 可以在用户说「我不会装」时给出清晰的安装指引。

**Q5**：「工具的 Skill」是 Agent Browser 内置的帮助系统（`agent-browser skills get core`），作用域限定在该 CLI 工具内；「Agent Skill」（SKILL.md）是跨工具的 Agent 能力扩展协议。联系在于它们共享相同哲学——提供方法论而非功能清单。说它是「Skill 的 Skill」因为它用 Skill 的思维（分层文档、按场景组织、带示例）来组织自己的帮助文档，做到了自描述。

**Q6**（参考流程）：1) `agent-browser open "https://shop.example.com/orders"` 打开订单页面；2) `agent-browser fill "input[name='email']" "$EMAIL"` 登录；3) `agent-browser wait 3000` 等加载；4) `agent-browser snapshot` 获取订单列表的 @ref；5) `agent-browser get text "@ref=12"` 提取第一条订单的状态。如果状态为「已发货」就继续下一步（如截图保存）。

**Q7**：AI 不关心命令长度——AI 生成文本的成本与长度几乎无关。但 AI 非常依赖语义——`get text` 对 AI 来说语义清晰，可以直接推断用途；`g t` 则没有语义，AI 需要额外记忆映射关系。为 AI 设计的 CLI 应该优化「理解成本」而非「打字成本」。

**Q8**：

| 模式 | 适用类型 | 脚本数 | API Key | 命令数 |
|------|---------|--------|---------|--------|
| 脚本说明书 | 单功能工具（压缩、转换、格式化） | 1 个 | 可选 | < 10 参数 |
| 多工具索引 | API 服务套件（搜索、提取、爬取） | 多个 | 需要 | 每工具几个参数 |
| 命令分类手册 | CLI 工具包装（浏览器、Git、数据库） | 0 个 | 不需要 | > 20 命令 |

评分：答对 7/8 = 通过，少于 7 道需重读对应章节。

</details>

***

## 10. 延伸阅读

- [[L1-16-Tavily联网搜索|上一课：Tavily 联网搜索]] — 对比「多工具索引」模式
- [[L1-01-智能图片压缩|L1-01 图片压缩]] — 对比「脚本说明书」模式
- [[../../Level5-大师/L5-课程总览|Level 5：browser-use]] — Playwright 级高级浏览器自动化
- [Agent Browser 官方文档](https://agent-browser.dev)
- [[Skill写作教程-从零开始#模式选择决策|Skill 写作：如何选择正确的模式]]
