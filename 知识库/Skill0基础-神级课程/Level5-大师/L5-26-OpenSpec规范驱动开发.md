---
tags:
  - 教程
  - Skill
  - 大师课
  - Level5
  - 大师
  - 规范驱动开发
  - SDD
created: 2026-05-06
updated: 2026-05-06
course_number: L5-26
prerequisites:
  - "[[L4-课程总览|Level 4 毕业]]"
  - "[[L5-25-Superpowers超能力框架|L5-25 Superpowers]]"
next_course: "[[L6-课程总览|Level 6：宗师]]"
---

# L5-26：OpenSpec 规范驱动开发 —— 让AI先写规格再写代码

> 🎯 **学什么**：掌握 OpenSpec 的 Propose→Apply→Archive 三阶段工作流，理解"规范驱动开发"如何让 AI 代码产出从"碰运气"变成"可预期"。
> 💡 **难易度**：⭐⭐⭐⭐⭐ | ⏱️ **预计时间**：55 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：你有没有遇到过让AI写个功能，它第一版完全不是你想要的？不是它不会写，而是你们没有"先对齐再动手"。OpenSpec 就是解决这个问题的——在AI写代码之前，先让它写一份"需求规格书"，你审核通过后再动手。

OpenSpec 是 Fission-AI（YC 孵化）推出的规范驱动开发（SDD/Spec-Driven Development/规范驱动开发：在编写任何代码之前，先写一份人类可读的规格文档——定义做什么、怎么做、如何验收——然后AI严格按照规格实施）框架。支持 30+ AI 编码工具，GitHub 27k+ Star。

学完这节课你将能：
- 理解规范驱动开发（SDD）的核心理念和与"自由式开发"的差异
- 掌握 Propose→Apply→Archive 三阶段工作流
- 理解 OpenSpec 11 个命令的用途和适用场景
- 能在自己的项目中使用 OpenSpec 管理 AI 开发流程

***

## 2. 问题：AI 编程的"猜谜游戏"

### 2.1 AI 自由编程的三大失败模式

> 💡 **白话小课堂**：没有规范的AI编程就像让一个没见过原型的工人看图造房子——每块砖都可能放错位置。不是工人技术不好，是图纸太模糊。

```
失败模式 1：方向偏离
  你说："加个用户登录功能"
  AI做："加了个OAuth+JWT+双因素认证+社交登录的完整系统"
  你内心："我只要一个简单的邮箱密码登录..."

失败模式 2：范围蔓延（Scope Creep：在开发过程中不断偷偷添加未授权的功能——"我觉得你可能还需要这个所以就一起做了"）
  AI 在添加登录时，"顺便"重构了用户表、改了路由、移除了旧代码
  结果：提交了 47 个文件，其中 38 个和登录无关

失败模式 3：验收困难
  你做完了看代码：功能好像能跑，但是否完全按需求做的？不知道
  "看起来没问题" ≠ "确实没问题"
```

### 2.2 OpenSpec 的答案：先写规格再写代码

> 🔑 **核心秘诀**：OpenSpec 的核心理念可以用一句话概括——"在AI碰键盘之前，先和它签一份合同"。合同就是 Spec（规格/Specification：定义软件"应该做什么"的文档——包括功能需求、验收场景、技术方案、任务拆解。是代码的"蓝图"）。

```
传统AI编程：
  用户需求 → AI直接写代码 → 产出（碰运气）

OpenSpec 规范驱动：
  用户需求 → Propose（写规格）→ 人类审核 → Apply（按规格写代码）→ Archive（归档规格）
  
  核心变化：在"写代码"前面加了一个"写规格"的步骤
```

***

## 3. 三阶段工作流：Propose → Apply → Archive

> 💡 **白话小课堂**：三个阶段就像"签合同 → 施工 → 验收存档"——每一步产生清晰的文档产物，每一步都不可跳过。

### 3.1 Phase 1：Propose（提案）—— 签合同

**命令**：`/opsx:propose [change-name]`

这一步 AI 生成四个制品（Artifact/制品：开发过程中产生的文档或代码产出物——OpenSpec 的四个核心制品是 proposal.md、specs/、design.md、tasks.md）：

```
openspec/changes/add-user-login/
├── proposal.md       # "为什么做" —— 动机、目标、范围
│   ## 动机：当前用户无法创建个人账户...
│   ## 目标：实现邮箱+密码注册和登录
│   ## 范围：仅邮箱密码，不含OAuth/社交登录
│   ## 非目标：密码重置（后续change）、双因素认证
│
├── specs/            # "做什么" —— 核心需求 + 验收场景
│   └── auth.md
│       ## Requirement: 用户注册
│       ### Scenario: 新用户用有效邮箱注册
│       - WHEN 用户输入有效邮箱和密码
│       - THEN 创建新账户并发送验证邮件
│       ### Scenario: 重复邮箱注册
│       - WHEN 用户输入已注册的邮箱
│       - THEN 显示"该邮箱已注册"提示
│
├── design.md         # "怎么做" —— 技术架构与决策
│   ## 技术选择：bcrypt 哈希密码
│   ## 数据库：新增 users 表
│   ## 路由：POST /api/auth/register, POST /api/auth/login
│   ## 无状态 JWT：access_token (15min) + refresh_token (7d)
│
└── tasks.md          # "分几步做" —— 实施任务清单
    ## 1. 数据库
    - [ ] 1.1 创建 users 表迁移文件
    - [ ] 1.2 添加 email 唯一索引
    ## 2. 后端
    - [ ] 2.1 实现 POST /api/auth/register
    - [ ] 2.2 实现 POST /api/auth/login
    - [ ] 2.3 实现 JWT 签发和验证中间件
    ## 3. 测试
    - [ ] 3.1 注册成功测试
    - [ ] 3.2 重复邮箱测试
    - [ ] 3.3 登录成功/失败测试
    ## 4. 前端
    - [ ] 4.1 注册页面组件
    - [ ] 4.2 登录页面组件
```

> ⚠️ **避坑指南**：Propose 之后，**你一定要审核这四个文件**！这是你唯一能在AI写代码前纠正方向的机会。如果 proposal.md 的方向就偏了，后面全白做。审核标准：proposal.md 的"非目标"是否清楚？specs/ 的验收场景是否具体可测？

### 3.2 Phase 2：Apply（实施）—— 施工

**命令**：`/opsx:apply [change-name]`

AI 读取 tasks.md，逐项执行，完成后标记 `[x]`：

```
AI 的执行过程：
  ✓ 1.1 创建 users 表迁移文件
  ✓ 1.2 添加 email 唯一索引
  ✓ 2.1 实现 POST /api/auth/register
  ...
  ✓ 4.2 登录页面组件
  
  所有任务完成！
```

> 🔑 **核心秘诀**：Apply 阶段 AI **只做 tasks.md 里写了的事**。如果实施过程中发现 spec 需要调整——回到 Propose 更新规格，而不是在 Apply 阶段偷偷改。

### 3.3 Phase 3：Archive（归档）—— 验收存档

**命令**：`/opsx:archive [change-name]`

```
归档做的事情：
1. 将 change 中的 delta spec 合并到主 spec（openspec/specs/）
2. 将 change 目录移动到 archive/（保留历史记录）
3. 更新 CHANGELOG（可选）

归档后：
openspec/
├── specs/          # 主规格（始终反映当前系统的"真相"）
│   └── auth.md     # 包含了刚刚实现的功能的规格
└── changes/
    └── archive/
        └── 2026-05-06-add-user-login/  # 历史记录
```

> 💡 **白话小课堂**：Archive 就像"工程验收后把图纸归档"——主 specs/ 永远反映系统的当前状态，archive/ 保存了每一次变更的历史。三个月后你想知道"登录功能当时为什么这样设计"，翻 archive/ 就找到了。

***

## 4. 完整命令体系（11 个命令）

> 💡 **白话小课堂**：Core Profile 给4个核心命令（够80%的场景），Custom Profile 解锁全部11个（像从自动挡换成手动挡——更精细但更复杂）。

### 4.1 Core Profile（默认，4 个命令）

| 命令 | 用途 | 何时用 |
|------|------|--------|
| `/opsx:propose` | 一步创建 change + 生成所有规划制品 | 日常最常用，80% 的场景 |
| `/opsx:explore` | 只读探索模式，不生成文件 | "我有个想法但还没想清楚" |
| `/opsx:apply` | 按 tasks.md 逐项实施 | Propose 审核通过后 |
| `/opsx:archive` | 合并 spec + 归档 change | Apply 完成且验收通过后 |

### 4.2 Custom Profile（11 个命令，精细控制）

| 命令 | 用途 | 对应 Core 的哪个 |
|------|------|-----------------|
| `/opsx:explore` | 探索调查（只读模式） | 同 Core |
| `/opsx:new` | 创建 change 脚手架（仅目录结构，不含内容） | Propose 的第一步 |
| `/opsx:continue` | 逐步生成下一个制品（一个一个来） | Propose 的分步版 |
| `/opsx:ff` | 快速生成所有规划制品（不创建 change） | Propose 的去脚手版 |
| `/opsx:propose` | 一步提案（new + ff） | 同 Core |
| `/opsx:apply` | 任务实施 | 同 Core |
| `/opsx:verify` | 验证实现 vs 规格一致性 | Archive 前建议先跑 |
| `/opsx:sync` | 同步 delta specs 到主 spec（不归档） | 只想更新 spec 不想归档 |
| `/opsx:archive` | 完成并归档 | 同 Core |
| `/opsx:bulk-archive` | 批量归档多个 change（含冲突检测） | 多个 change 同时完成时 |
| `/opsx:onboard` | 15 分钟交互式教程 | 新人第一次用时 |

> 🔑 **核心秘诀**：Core Profile 的 4 个命令已经够用。Custom Profile 的 11 个命令是为"需要精细控制每个步骤"的场景准备的——比如你想先生成 proposal.md 审核，再单独生成 design.md，而不是一次生成全部。

***

## 5. 核心创新：为什么 OpenSpec 与众不同

### 5.1 Delta-based Specs（增量式规格）

> 💡 **白话小课堂**：传统方式是为整个系统写一份大而全的规格文档——写完300页，项目已经变了。OpenSpec 只写"这次要改什么"——像Git的diff，只描述变更的部分。

```
传统规格（Big Design Up Front）：
  为整个系统写一份完整的规格文档
  结果：写300页，3天后项目变了，规格文档作废

OpenSpec Delta Spec（增量规格/Delta Spec：只描述本次变更中"新增了什么""修改了什么""删除了什么"——像Git diff，而不是整个文件的快照）：
  只写这次 change 新增、修改、删除的需求
  结果：5页规格，精准且不过时
```

**Delta Spec 格式**：
```markdown
## ADDED Requirements
### Requirement: 用户登录
用户应能使用邮箱和密码登录系统。

#### Scenario: 成功登录
- WHEN 用户输入正确的邮箱和密码
- THEN 返回 JWT access token 和 refresh token

## MODIFIED Requirements
### Requirement: 用户会话管理
原：会话存储在服务端 Session 中
改：会话使用客户端 JWT（无状态）

## REMOVED Requirements
### Requirement: 旧版 Cookie-based 认证
已替换为 JWT-based 认证，旧方式移除。
```

### 5.2 Brownfield-first（存量项目优先）

> 💡 **白话小课堂**：大多数AI开发工具假设你从零开始（Greenfield/绿地项目：从空白开始的全新项目——没有历史代码、没有技术债、没有约定）。OpenSpec 假设你有一个已经跑了3年的老项目（Brownfield/棕地项目：已有一定规模和历史的存量项目——有技术债、有约定、有历史包袱）——这才是真实世界。

OpenSpec 的自动化检测：
- `openspec init` 自动检测项目中已有的 AI 工具（Claude Code、Cursor、Windsurf 等）
- 自动适配不同的目录结构和配置格式
- 不会覆盖已有的 CLAUDE.md 或 AGENTS.md

### 5.3 Actions, Not Phases（行动，而非阶段）

```
传统（阶段锁定/Phase Gate）：
  PLANNING ────► IMPLEMENTING ────► DONE
       │               │
       │ "不能回头了"    │
       └───────────────┘

OpenSpec（流式行动/Continuous Actions）：
  proposal ⇄ specs ⇄ design ⇄ tasks ⇄ implement
  （随时可以回到任何上一步修改）
```

> 🔑 **核心秘诀**：OpenSpec 不做"阶段锁定"——你在 Apply 时发现 design.md 需要改？回去改，然后继续 Apply。规范和实现是互相校准的关系，不是单向流水线。

### 5.4 动态指令生成

> 💡 **白话小课堂**：OpenSpec 的 Skill 文件不含静态模板。它通过 CLI 命令动态生成指令——这意味着 OpenSpec 升级后所有 Skill 自动获得新能力，不需要重新安装 Skill 文件。

```bash
# AI 在 Skill 中会调用的 CLI 命令
openspec status --change add-login --json     # 查询当前进度
openspec instructions proposal --json         # 获取"如何写 proposal"的指令
openspec templates --schema spec-driven --json # 获取可用模板
openspec schemas --json                       # 获取可用 schema
```

> 这比静态模板优越的地方：当你升级 OpenSpec CLI 到新版本，所有 Skill 的指令自动更新，不需要手动重新安装 Skills。

***

## 6. 安装与初始化

```bash
# 全局安装 CLI
npm install -g @fission-ai/openspec@latest

# 验证安装
openspec --version

# 在项目中初始化
cd my-project
openspec init

# 初始化做了什么？
# 1. 检测你用的 AI 工具（Claude Code/Cursor/Windsurf...）
# 2. 创建 openspec/ 目录结构
# 3. 生成对应工具的 Skill 文件
# 4. 更新（或创建）AGENTS.md / CLAUDE.md
```

**初始化后的项目结构**：
```
my-project/
├── openspec/
│   ├── specs/          # 主规格（系统的"当前真相"）
│   ├── changes/        # 进行中的变更
│   │   └── archive/    # 已完成的变更历史
│   └── AGENTS.md       # AI 代理配置
├── CLAUDE.md           # 项目配置（可能被更新）
└── ...你的代码...
```

***

## 7. 结构拆解：OpenSpec 的 Skill 设计

### 7.1 Skill 文件结构

OpenSpec 的每个 Skill 遵循 Agent Skills 标准格式：

```yaml
---
name: openspec-propose
description: 当用户想要创建新的变更提案时使用——包括新功能、重构、Bug修复
license: MIT
compatibility: 需要 openspec CLI
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.2.0"  # 记录生成此Skill的CLI版本
---
```

> 🔑 **核心秘诀**：注意 `metadata.generatedBy` 字段——这是 OpenSpec 的"自动更新"机制。当 CLI 升级后，它会检测这个版本号，如果差异过大，提示你重新生成 Skill 文件。

### 7.2 双重交付模式

| 模式 | 安装内容 | 适用场景 |
|------|---------|---------|
| **Skills only** | 仅 SKILL.md 文件 | AI 自己管理 Skill 调用逻辑 |
| **Commands only** | 工具特定命令文件（如 `.claude/commands/opsx-propose.md`） | 用户手动用 `/opsx:xxx` 触发 |
| **Both**（默认） | Skills + Commands | 最灵活——AI 能自动调用，用户也能手动触发 |

***

## 8. 手写模仿：为你的项目写第一个 OpenSpec Change

请完成以下练习：

**任务**：为你的 Obsidian Vault 写一个 OpenSpec Change

**场景**：你想给 Vault 添加一个"每日随机回顾"功能——每天打开 Vault 时随机展示一条旧笔记帮你复习。

**要求**：
1. 写出 `proposal.md` 的内容（包含动机、目标、范围、非目标）
2. 写出至少 3 个验收场景（参考 specs/ 格式）
3. 写出 tasks.md 的任务清单（至少 5 个任务，按实施顺序排列）

**提示**：
- 范围要窄——先做最简单版本（随机展示一条旧笔记）
- 非目标要明确——"不含 AI 摘要""不含自动标签"
- 验收场景要具体可测——"WHEN...THEN..."

***

## 9. 常见错误

| 错误 | 为什么错 | 正确做法 |
|------|---------|---------|
| Propose 后不审核直接 Apply | 规格偏了，代码全偏 | 审核四个制品，确认 proposal 方向对、specs 验收场景具体 |
| 在 Apply 阶段偷偷改需求 | 破坏"规格-实现"的一致性 | 需要改需求？回 Propose 更新 spec，再次 Apply |
| 一个 Change 包含太多功能 | tasks.md 膨胀到 50+ 任务，实施不可控 | 一个 Change 一个逻辑工作单元，tasks 控制在 15 个以内 |
| Archive 前不跑 Verify | 实现和 spec 可能不一致 | Archive 前先 `/opsx:verify`，确认代码符合规格 |
| 把 specs/ 当成"写完不看"的文档 | specs/ 是系统的"真相来源"，代码不符合 spec 就是 bug | specs/ 必须保持更新，Archive 时要合并 delta |

***

## 10. 掌握检验

请回答以下 5 道题（答案在末尾）：

**Q1**：OpenSpec 三阶段工作流的正确顺序是？
- A) Apply → Propose → Archive
- B) Propose → Archive → Apply
- C) Propose → Apply → Archive
- D) Archive → Propose → Apply

**Q2**：关于 Delta Spec，以下哪项是正确的？
- A) 每次变更都要重写整个系统的完整规格
- B) 只描述本次变更中"新增/修改/删除"的需求
- C) Delta Spec 是 OpenSpec 的可选高级功能
- D) Delta Spec 是一种 Git 分支策略

**Q3**：OpenSpec 称自己是 "Brownfield-first"（棕地项目优先），这意味着什么？
- A) 仅支持在旧项目中运行
- B) 优先支持已有一定规模的存量项目，而非仅支持从零开始的新项目
- C) 需要特定的棕色主题 IDE
- D) 只能在 Linux 上运行

**Q4**：`metadata.generatedBy` 字段的作用是什么？
- A) 记录谁写了这个 Skill
- B) 记录生成此 Skill 的 CLI 版本，用于智能更新检测
- C) 是 YAML 的必填格式字段
- D) 记录用户的 Claude Code 版本

**Q5**：判断正误——"在 Apply 阶段，如果 AI 发现原 spec 不太合理，应该直接按更合理的方式实现，然后在 Archive 阶段更新 spec 即可。"

***

## 11. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**C** —— Propose（提案）→ Apply（实施）→ Archive（归档）。先签合同，再施工，最后验收存档。

**Q2**：**B** —— Delta Spec 只描述变更部分（ADDED/MODIFIED/REMOVED），而不是重写整个系统规格。这是 OpenSpec 能保持规格"轻盈"的核心设计。

**Q3**：**B** —— Brownfield-first 指 OpenSpec 优先考虑在已有存量项目中工作的场景。大多数真实项目都是 brownfield——有历史代码、有技术债、有约定。OpenSpec 的初始化会自动检测并适配现有项目结构。

**Q4**：**B** —— `generatedBy` 记录生成此 Skill 文件的 CLI 版本号。当 CLI 升级后（如从 1.2.0 升到 1.3.0），系统能检测到版本差异并提示用户重新生成 Skill 文件以获取新功能。

**Q5**：**错误** —— 如果 spec 不合理，应该回到 Propose 阶段更新 spec，然后重新 Apply。在 Apply 阶段偷偷改实现会让 specs/ 中的规格与代码不一致，破坏 spec 作为"系统真相来源"的信任。流程是：发现 spec 问题 → 更新 spec（Propose）→ 重新实施（Apply）→ 确认一致性（Verify）。

（4/5 通过）

</details>

***

## 12. 延伸阅读

- [[L5-25-Superpowers超能力框架|L5-25 Superpowers 超能力框架]] — 另一个方法论框架，互补使用
- [[L5-09-Skill创建器|L5-09 Skill创建器]] — 元技能设计
- [[L6-01-创业门诊室|L6-01 gstack 创业门诊室]] — 产品需求定义的方法论
- [[L6-03-工程架构评审|L6-03 gstack 工程架构评审]] — 与 OpenSpec design.md 的设计互补
- [[知识库/来源/openspec]] — 项目完整档案
- [[知识库/实体/OpenSpec项目]] — 项目背景信息

***

> 💡 **记住这句话**：OpenSpec 的哲学不是"让AI猜你想要什么"，而是"让AI先写清楚它要做什么，你点头了再动手"。代码的可靠性，取决于规格的清晰度。
