---
tags:
  - 教程
  - Skill
  - 大师课
  - Level0
  - 基础
  - CLAUDE.md
  - 行为规范
created: 2026-05-06
updated: 2026-05-06
course_number: L0-03
prerequisites:
  - "[[L0-01-Skill的本质与工作原理]]"
  - "[[L0-02-环境搭建与工具链]]"
next_course: "[[L1-课程总览|Level 1：新手村]]"
---

# L0-03：给AI立规矩 —— CLAUDE.md 项目配置与Karpathy行为规范

> 🎯 **学什么**：理解 CLAUDE.md 的配置层级，掌握 Karpathy 四大行为原则，能写出项目级 AI 行为约束。
> 💡 **难易度**：⭐ | ⏱️ **预计时间**：35 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：前两课讲了"Skill是什么"和"环境怎么搭"。这一课讲第三件事——**怎么给AI定规矩**。就像入职新员工要签员工手册一样，你想让AI在你的项目里怎么写代码，得先告诉它。

学完这节课你将能：
- 理解 CLAUDE.md、SKILL.md、System Prompt 三层配置的区别
- 安装并使用 Karpathy 行为规范 Skill
- 理解四大原则的设计逻辑
- 为自己的项目定制 CLAUDE.md

***

## 2. 三层配置：AI 的"规矩体系"

> 💡 **白话小课堂**：给AI定规矩就像给员工定规矩——有公司级别的（System Prompt/系统提示词）、有部门级别的（CLAUDE.md/项目配置）、有具体任务的操作手册（SKILL.md/Skill文件）。三层各有分工，各管各的事。

```
三层配置体系：

┌─────────────────────────────────────┐
│  System Prompt（会话级·全局生效）      │
│  "你是一个有10年经验的前端工程师...      │
│   永远用中文回复..."                  │
│  所有对话都受它影响，只有1个           │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  CLAUDE.md（项目级·项目内生效）        │
│  "这个项目用 TypeScript 严格模式...    │
│   不要引入不必要的抽象层...             │
│   修改前先读 README.md..."            │
│  放在项目根目录，该项目内所有对话都读它  │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  SKILL.md（任务级·触发时加载）          │
│  "当用户需要翻译文档时...              │
│   流程：识别语言→逐段翻译→保持格式"     │
│  只有匹配到触发词时才加载，可以有N个    │
└─────────────────────────────────────┘
```

| 配置 | 层级 | 数量 | 加载时机 | 典型内容 |
|------|------|------|---------|---------|
| System Prompt | 会话 | 1个 | 始终生效 | 角色设定、全局规则 |
| **CLAUDE.md** | **项目** | **1个/项目** | **项目内始终生效** | **编码规范、行为约束、项目约定** |
| SKILL.md | 任务 | N个 | 触发时加载 | 特定任务的执行流程 |

> 🔑 **核心秘诀**：CLAUDE.md（项目级 AI 配置文件：放在项目根目录的 Markdown 文件，Claude Code 启动时自动读取，定义该项目中 AI 的编码行为规范）是"部门守则"——不太宽（不像 System Prompt 管所有对话），也不太窄（不像 Skill 只管一个任务），刚好管一个项目的范围。

***

## 3. Karpathy Skill：一个文件改变 AI 编码质量

### 3.1 起源故事

> 💡 **白话小课堂**：2026年1月，AI界大佬 Andrej Karpathy（前特斯拉AI负责人、OpenAI创始成员）发了一条长帖吐槽AI编程的问题，800万人看了。开发者 Forrest Chang 把他的吐槽变成了一个可安装的 Skill——结果一周内冲上 GitHub 榜首，10万+程序员"抄作业"。

Karpathy 指出了 LLM（大语言模型：Large Language Model，能理解和生成人类语言的大规模AI模型，Claude/GPT等都是LLM）编程的三大系统性失败：

1. **静默假设**（Silent Assumption：AI在没有明确信息的情况下自行脑补，不主动提问就直接执行——"我猜你想要这样"）——模型做出错误假设，不验证直接执行
2. **过度工程化**（Over-engineering：把简单问题复杂化——100行能解决的事写成1000行，堆砌不必要的抽象和"灵活性"）——明明100行能解决，偏要写成1000行
3. **越界修改**（Scope Creep/范围蔓延：AI在完成任务时"顺手"改动了与任务无关的代码、注释或格式——"我看这个不顺眼一起改了"）——改动理解不充分的甚至无关的代码

### 3.2 安装 Karpathy Skill

```bash
# Claude Code 中安装
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install andrej-karpathy-skills@karpathy-skills

# 或直接下载 CLAUDE.md 放到项目中
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

### 3.3 安装为项目级配置

如果是已有项目，追加到现有 CLAUDE.md：

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md >> CLAUDE.md
```

***

## 4. 四大原则深度解读

> 🔑 **核心秘诀**：这四条规则本质上不是"教你写代码"，而是"管住AI别乱来"——像一个严格的代码审查员，在AI每次动手前拉一下手刹。

### 原则 1：编码前思考（Think Before Coding）

**原文（Think Before Coding：在写代码之前，先停下来想清楚——这是所有原则的基石）**：
```
Don't assume. Don't hide confusion. Surface tradeoffs.
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
```

**中文解读**：
- **不假设**：不确定就问，别自己猜
- **不隐藏困惑**：遇到不懂的地方，说出来，不要假装懂了
- **呈现权衡**：有多个方案时，列出来让用户选，别自己默默决定
- **质疑需求**：如果有更简单的做法，主动提出来

> ⚠️ **避坑指南**：AI最大的问题不是"不会写"，而是"以为自己会写"。这条原则强制AI在动手前先确认自己真的理解了需求。

### 原则 2：简约至上（Simplicity First）

**原文（Simplicity First/简约至上：用最少的代码解决最多的问题——不加任何用户没要求的功能）**：
```
Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.
```

**中文解读**：
- **不加未被要求的功能**：用户说加个按钮，就别自动加一套主题系统
- **不为单次使用的代码建抽象**：别搞"以后可能用到"的过度设计
- **不加没要求的"灵活性"和"可配置性"**：YAGNI（You Ain't Gonna Need It/你不会需要它：极限编程核心原则——在你真正需要某个功能之前，不要实现它）
- **不为不可能的场景写错误处理**：别为"万一数据库炸了""万一地球停转"写代码
- **200行能缩到50行就重写**

> 💡 **白话小课堂**：这条原则就是"代码极简主义"——写得越少，bug越少，维护越简单。

### 原则 3：精准修改（Surgical Changes）

**原文（Surgical Changes/精准修改：像外科手术一样精确——只碰必须碰的代码，不"顺手优化"任何无关内容）**：
```
Touch only what you must. Clean up only your own mess.
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.
```

**中文解读**：
- **不要"顺便优化"旁边的代码**：这是越界修改的第一大来源
- **没坏的东西别重构**：如果它工作正常，别碰它
- **匹配现有风格**：哪怕你觉得现有风格不好，也要保持一致
- **看到死代码？提一嘴，别直接删**：死代码（Dead Code/死代码：不会被执行到的代码——可能是有意保留的、有历史原因的，删除前需要确认）可能有你不知道的用途

> ⚠️ **避坑指南**：这是AI最常见的"好心办坏事"——它看到代码就忍不住要"优化"。这条规则强制它管住手。

### 原则 4：目标驱动执行（Goal-Driven Execution）

**原文（Goal-Driven Execution/目标驱动执行：把模糊指令转化为可验证目标，用"测试通过"而不是"我觉得好了"来验收）**：
```
Define success criteria. Loop until verified.
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"
```

**中文解读**：
- **把模糊指令变成可验证目标**：
  - "加个验证" → "先写无效输入的测试，再让测试通过"
  - "修这个bug" → "先写一个能复现bug的测试，再让它通过"
  - "重构X" → "确保重构前后测试都通过"
- **多步骤任务要列计划**：每步写清楚验证方式
- **强成功标准让AI独立循环**：模糊标准（"让它能工作"）需要不断问你

> 💡 **白话小课堂**：这是 Karpathy 最重要的洞察——"LLM极其擅长循环直到达成目标……不要告诉它做什么，给它成功标准然后看它跑。"

***

## 5. 源码精读：CLAUDE.md 完整文件

> 💡 **白话小课堂**：下面是这个 Skill 的核心文件——只有65行的 CLAUDE.md。注意看它的结构：先从最宏观的原则开始，然后每一条都给了具体的"做什么"和"不做什么"。

````markdown
# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

Tradeoff: These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

These guidelines are working if: fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
````

***

## 6. 结构拆解：为什么这样设计？

> 💡 **白话小课堂**：拆解这个文件的设计，你会发现它遵循了"先立原则、再给方法论、最后给验收标准"的三段式结构。

### 6.1 文件结构分析

```
CLAUDE.md
├── 标题（1行）           → 身份声明
├── 一句话说明（1行）       → 这是干什么的
├── 权衡声明（1行）         → 坦诚说明代价（偏谨慎、牺牲速度）
├── 原则1：编码前思考（11行）→ 解决问题：AI"想当然"
├── 原则2：简约至上（10行）  → 解决问题：AI"过度设计"
├── 原则3：精准修改（14行）  → 解决问题：AI"越界修改"
└── 原则4：目标驱动（12行）  → 解决问题：AI"说不清楚做完了没"
```

### 6.2 设计亮点

| 设计决策 | 为什么好 |
|---------|---------|
| **不含"you are"角色设定** | 这是行为约束，不是角色扮演——更通用、可叠加 |
| **每条原则有正面指令+反面禁止** | 正面告诉AI做什么，反面防止AI越界 |
| **Tradeoff 坦诚声明** | 承认代价（偏谨慎），让用户知情选择 |
| **结尾有"检验标准"** | "这些规则生效的标志是……"——给了可观测的成功指标 |
| **65⾏极简长度** | 不超过AI的注意力范围，每条原则都在"一眼能看完"的长度 |

> 🔑 **核心秘诀**：好的 CLAUDE.md 不是"越多越好"——是"每条都精准命中AI最常犯的错"。

***

## 7. 手写模仿：定制你的 CLAUDE.md

> 💡 **白话小课堂**：Karpathy 的规范是"通用版"。你需要给它加上"你的项目的味道"——就像在通用菜谱上加上你家的独门调料。

在项目根目录创建或追加 CLAUDE.md，参考这个模板：

````markdown
# CLAUDE.md

<!-- 第一部分：引入通用行为规范（可选） -->
<!-- 以下内容来自 Karpathy 行为规范 -->

## 核心行为准则

### 编码前思考
- 不假设。不确定就问。
- 有多种理解时，列出选项让我选。
- 有更简单的方案，主动提出来。

### 简约至上
- 只用最少代码解决当前问题。
- 不添加未请求的功能、抽象、"灵活性"。
- 如果写了200行但50行能搞定，重写。

### 精准修改
- 只改必须改的代码。不"顺手优化"其他东西。
- 匹配项目现有的代码风格。
- 发现无关的死代码，提一嘴但别删。

### 目标驱动
- 把模糊任务转为可验证目标（"先写测试→再让它通过"）。
- 多步骤任务列出计划，每步写清验证方法。

---

<!-- 第二部分：项目特定规范 -->
## 本项目约定

- 语言：TypeScript 严格模式，不用 any
- 格式化：使用项目配置的 Prettier，不要自己改格式
- 修改任何文件前，先读对应的 README.md
- 组件命名：PascalCase，文件名与组件名一致
- API 调用：统一用 lib/api.ts 中的封装，不要直接 fetch

---

<!-- 第三部分：项目文件索引 -->
## 关键文件
- README.md：项目说明
- ARCHITECTURE.md：架构设计
- lib/api.ts：API 封装层
- components/：React 组件
````

> ⚠️ **避坑指南**：项目特定规范不要写得太长！Karpathy 规范才65行，你的项目规范也控制在100行以内。太长的 CLAUDE.md 反而会让 AI "视而不见"。

***

## 8. 变体练习

请完成以下三个练习：

**练习 1：为你的 Obsidian Vault 写一个 CLAUDE.md**
- 包含：笔记命名规范、Wiki-link 格式要求、YAML frontmatter 约定
- 长度控制在 80 行以内

**练习 2：将 Karpathy 原则翻译成"设计师版"**
- 原版给程序员，你把它改编给用 AI 做设计的场景
- 例如："简约至上"→ 设计方案不要过度装饰

**练习 3：追加一条"第5原则"**
- Karpathy 列了4条，你觉得还缺什么？
- 写一条你认为AI最需要的第5条行为规范，要有正面指令+反面禁止

***

## 9. 常见错误

| 错误 | 为什么错 | 正确做法 |
|------|---------|---------|
| 把 CLAUDE.md 写成 500 行 | AI 注意力分散，反而忽略关键约束 | 控制在 150 行以内，只写最重要的 |
| 只复制 Karpathy 规范，不加项目约定 | Karpathy 规范是通用版，没有项目信息 | 追加项目特定的编码规范、文件索引 |
| 在 CLAUDE.md 里写 Skill 触发词 | 混淆了 CLAUDE.md（项目级）和 SKILL.md（任务级） | CLAUDE.md 写行为约束，SKILL.md 写任务流程 |
| 不写 Tradeoff 声明 | AI 不知道"谨慎"是故意的设计选择 | 坦诚说明：偏谨慎 → 更多确认，但更少出错 |
| 以为装了就万事大吉 | Skill 需要 AI 有上下文才能生效 | 验证方式：给AI一个编码任务，观察它是否先确认需求再动手 |

***

## 10. 掌握检验

请回答以下 5 道题（答案在末尾）：

**Q1**：CLAUDE.md、SKILL.md、System Prompt 三者中，哪个是"项目级配置"？
- A) CLAUDE.md
- B) SKILL.md
- C) System Prompt
- D) 三者都是

**Q2**：Karpathy 四大原则中，"Surgical Changes"解决的核心问题是什么？
- A) AI 写的代码太多
- B) AI 擅自修改与任务无关的代码
- C) AI 不写测试
- D) AI 代码格式不规范

**Q3**：关于 CLAUDE.md 的长度，以下哪项是正确的？
- A) 越长越好，覆盖所有可能情况
- B) 控制在150行以内，只写最重要的约束
- C) 必须刚好65行
- D) 不需要控制长度，AI 能处理任意长度

**Q4**：判断以下做法是否正确——"我的 CLAUDE.md 里写了详细的翻译 Skill 指令，包括翻译流程、格式要求、术语表。"

**Q5**：Karpathy 规范的开头写了 "Tradeoff: These guidelines bias toward caution over speed." 这一行的设计目的是什么？

***

## 11. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**A** — CLAUDE.md 是项目级配置（放在项目根目录）。System Prompt 是会话级，SKILL.md 是任务级。

**Q2**：**B** — "Surgical Changes"（精准修改）解决的是AI越界修改的问题——只改必须改的代码，不"顺手优化"无关内容。

**Q3**：**B** — 控制在150行以内。超过这个长度，AI的注意力会分散，反而忽略关键约束。Karpathy 原版只有65行。

**Q4**：**错误** — 翻译 Skill 的指令应该放在 SKILL.md（任务级配置），不应该放在 CLAUDE.md（项目级行为约束）。CLAUDE.md 写的是"怎么做"，不是"做什么"。

**Q5**：**坦诚声明 + 管理预期**。告诉用户：这些规则会让 AI 更谨慎（更多确认对话），代码产出的"速度"会变慢，但"质量"会提高。用户知道这是有意为之的设计选择，而不是 AI 出了 bug。同时给了用户退出路径——"如果只是简单任务，用你自己的判断"。

（4/5 通过）

</details>

***

## 12. 延伸阅读

- [[L0-01-Skill的本质与工作原理|上一课：Skill的本质与工作原理]] — 理解三层加载系统
- [[L0-02-环境搭建与工具链|上一课：环境搭建与工具链]] — Skill 安装操作
- [[L1-课程总览|下一站：Level 1 新手村]] — 开始系统学习工具型 Skill
- [[知识库/概念/CLAUDE.md配置]] — 概念深度解读
- [[知识库/来源/andrej-karpathy-skills]] — 项目完整档案

***

> 💡 **记住这句话**：CLAUDE.md 不是 AI 的"使用说明书"，是 AI 的"员工手册"。写得越精准，AI 越少犯低级错误。给 AI 立好规矩，它才是你最好的搭档。
