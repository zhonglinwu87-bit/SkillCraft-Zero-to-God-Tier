---
tags: [教程, Skill, 大师课, Level5, Anthropic, 元技能]
created: 2026-05-03
updated: 2026-05-05
course_number: L5-09
prerequisites: ["[[L5-08-MCP工具构建器]]"]
next_course: "[[L5-10-Slack动图生成]]"
---

# L5-09：元技能——用AI创造AI技能 —— 屠龙刀中的屠龙刀

> 🎯 **学什么**：用 AI 来创造新的 AI 技能——这是所有技能的「母技能」。核心洞察三个：(1) 渐进式披露的三级加载系统——Level 1 常驻 100 词、Level 2 触发时加载 ≤ 500 行、Level 3 按需加载无限制；(2) 基线对比测试是唯一客观验证 Skill 有效性的方式——如果没有 baseline 对比，你写了 500 行 SKILL.md 可能跟没有 Skill 的效果几乎一样；(3) 「解释为什么」比「全大写命令 ALWAYS/NEVER」更有效——今天的 LLM 理解因果关系比盲目遵守命令更可靠。
> 💡 **难易度**：⭐⭐⭐⭐⭐ | ⏱️ **预计时间**：60 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：假设你是一家工厂的老板。前面的课程都是教你「操作机器生产产品」——怎么用注塑机做塑料杯、怎么用缝纫机做衣服。而这门课教你的不是操作机器——是**制造机器**。skill-creator 是一台「生产机器的机器」——用 AI 来创造 AI 技能。99% 的人只需要会开车（使用已有 Skills），但学会了造车（skill-creator），你的影响力从「自己做的事」变成了「帮助别人做的事」乘以「别人的人数」。

> **来源**：Anthropic 官方 Skills 仓库

---

## 2. 渐进式披露：三级加载系统

### 2.1 为什么要三级加载

```
如果所有 Skill 内容都常驻上下文：
  → 安装 10 个 Skill × 每个 500 行 → 5000 行常驻
  → 每次对话都加载，大部分 Skill 根本没被触发
  → 浪费大量上下文窗口和 token

三级加载解决这个问题：
  → 没触发的 Skill 只占 100 词（Level 1）
  → 触发了才加载正文（Level 2）
  → 需要时才加载资源（Level 3）
  → 就像一个好老师上课——先给标题，学生选了才进教室，进了教室才发教材
```

### 2.2 三级详解

```
Level 1：元数据（Metadata）——常驻上下文，始终存在
  内容：name + description
  大小：约 100 词
  作用：Skill 的「触发入口」——AI 通过 description 判断此时该不该触发这个 Skill
  关键设计：description 决定 Skill 的触发精确度

Level 2：SKILL.md 正文——Skill 被触发时才加载
  内容：Skill 的核心指令、工作流、规则、示例
  大小：建议 ≤ 500 行（接近限制时增加层级结构）
  作用：Skill 的「操作手册」——AI 如何执行这个 Skill
  关键设计：保持精简，超出 500 行考虑拆分到 Level 3

Level 3：打包资源——按需加载，无大小限制
  内容：scripts/（可执行脚本）、references/（参考资料）、assets/（模板文件）
  作用：Skill 的「工具箱」——需要时才用
  关键设计：脚本可直接执行无需加载到上下文
```

---

## 3. 八步开发循环

### 3.1 完整流程

```
Step 1：捕获意图（Capture Intent）
  → 明确 Skill 的触发条件
  → 明确期望输出格式
  → 输出：一句话描述「什么时候触发 + 输出什么」

Step 2：采访调研（Interview & Research）
  → 主动询问边缘情况、格式需求、成功标准
  → 不假设用户知道所有技术细节
  → 输出：需求清单

Step 3：编写 SKILL.md
  → name：Skill 标识符（如 competitive-intel）
  → description：触发描述（AI 用来匹配的判断标准）
  → body：核心指令和工作流
  → 输出：SKILL.md 文件

Step 4：编写测试用例
  → 2-3 个真实测试用例
  → 每个用例包含：输入 + 期望输出 + 评分标准
  → 输出：测试文件

Step 5：并行运行测试 ⚠️ 最容易被跳过
  → 每个用例跑两版：
    版本 A：带 Skill（with-skill）
    版本 B：基线（baseline，不带 Skill）
  → 关键：必须有基线对比！这是验证 Skill 是否有效的唯一方式

Step 6：评估与打分
  → 每个用例打分（1-10 分）
  → 聚合生成 benchmark.json
  → 对比 with-skill vs baseline 的分数差异

Step 7：启动评审查看器
  → 收集用户反馈
  → 反馈文件：feedback.json

Step 8：迭代改进
  → 根据 feedback.json 修改 SKILL.md
  → 重新测试 → 重新评估 → 循环
```

### 3.2 基线对比的重要性

```
如果没有基线对比：
  → 你写了 500 行 SKILL.md
  → 测试得分 85 分
  → 「看起来不错！」
  → 但 baseline（不带 Skill）可能也是 83 分
  → 你花了 500 行 token = 只提升了 2 分
  → Skill 几乎没有起到效果

基线对比告诉你：
  → Skill 真正贡献了多少提升
  → 如果 baseline 80、with-skill 82 → Skill 可能只告诉了 AI 它本就知道的事
  → 如果 baseline 50、with-skill 85 → Skill 提供了 AI 原本不知道的关键知识
```

---

## 4. 改进 Skill 的四个核心思维

### 4.1 四个思维

```
思维 1：从反馈中泛化（Generalize from Feedback）
  → 不要过度拟合到测试用例
  → Skill 要服务百万次调用，而非仅仅通过几个样例
  → 从具体案例中提取通用模式
  → 不是「为每个测试用例写一条规则」，是「找出为什么这些用例会失败」

思维 2：保持 Prompt 精简（Keep Prompts Lean）
  → 删除不产生价值的指令
  → 阅读运行记录（run logs）而非只看最终输出
  → 每个指令问自己：「删掉这句话，输出会变差吗？」

思维 3：解释为什么（Explain WHY）⭐ 最反直觉的洞察
  → 不要写「ALWAYS use X format」
  → 而要写「Use X format because downstream systems parse it automatically; 
    any other format will break the pipeline」
  → 今天的 LLM（Opus 4.7 级别）理解因果关系比盲目遵守命令更可靠
  → 理解「不遵守的后果」→ 比看到全大写命令更严格地执行

思维 4：寻找跨用例的重复工作（Cross-Case Reuse）
  → 如果多个测试用例都需要类似的辅助脚本
  → 打包到 scripts/ 中——一次写入，每次受益
  → 不需要在 SKILL.md 中重复描述相同的逻辑
```

### 4.2 Near-Miss 测试

```
near-miss 查询在触发描述优化中价值最高：

should-trigger（应该触发）：
  「帮我写一份母亲节促销文案」→ 明确提到促销文案 → 应该触发

should-not-trigger（不应该触发）：
  「今天天气怎么样」→ 完全不相关 → 不应该触发

near-miss should-not-trigger（最难的）：
  「帮我分析一下竞品的母亲节广告策略」
  → 提到了「母亲节」和「广告」（关键词重叠）
  → 但实际需要的是竞品分析 Skill 而不是促销文案 Skill
  → AI 必须能区分「促销文案」和「竞品分析」
  → near-miss 测试检测的是这种边界上的精确度
```

---

## 5. 结构拆解：元技能型 Skill 模板

```markdown
## 元技能型 Skill 模板

### 核心特征
→ 管理对象 = 其他 Skill 的创建和迭代
→ 核心原则 = 三级加载 + 基线对比 + 解释为什么
→ 关键设计 = 八步开发循环 + 四个改进思维 + near-miss 测试

### 通用结构

## Progressive Disclosure（三级加载系统）
- Level 1: name + description（~100 词，常驻上下文）
- Level 2: SKILL.md 正文（≤ 500 行，触发时加载）
- Level 3: scripts/references/assets（按需加载，无限制）

## Eight-Step Development Loop（八步开发循环）
意图 → 调研 → 编写 → 测试 → 并行运行 → 评估 → 评审 → 迭代
Step 5（并行运行/基线对比）最容易被跳过

## Four Improvement Mindsets（四个改进思维）
1. 从反馈中泛化（不要过度拟合测试用例）
2. 保持 Prompt 精简（删除不产生价值的指令）
3. 解释为什么（不比 ALWAYS/NEVER 全大写命令）
4. 寻找跨用例重复（打包到 scripts/）

## Near-Miss Testing（near-miss 测试）
- should-trigger / should-not-trigger / near-miss
- near-miss 检测触发描述的「边界精确度」
```

---

## 6. 电商案例：用 skill-creator 创建「自动比价器」Skill

某拼多多店铺运营团队（主营日用百货，SKU 数量 2000+）每天需要手动对比 20 个核心竞品的价格变化——打开竞品店铺→逐一截图→录入 Excel→计算价差→标注「需调价」SKU。单人耗时 45 分钟/天，且经常漏标。

用 skill-creator 创建一个「自动比价器」Skill，让 AI 自动完成竞品价格监控与调价建议。

**Step 1-2（捕获意图 + 采访）**：
- 触发条件：用户说「比价」「竞品价格」「调价建议」「看一下竞品」
- 期望输入：用户提供竞品店铺链接或商品名
- 期望输出：(1) 价格对比表（我方 vs 竞品 A vs 竞品 B）；(2) 标注「需调价」的 SKU（价差 >10%）；(3) 建议新价格（基于毛利底线和历史转化率）
- 约束：毛利底线不得低于 15%（低于则标注「触及底线，不建议降价」）；价格数据必须标注获取时间（避免用过时数据做调价决策）

**Step 3（编写 SKILL.md）**：
```markdown
name: price-watcher
description: Monitors competitor prices and generates repricing suggestions for e-commerce SKUs. 
  Use when user asks about "比价", "竞品价格", "调价", "价格监控".

## Rules
- Always output a comparison table: SKU | 我方售价 | 竞品A | 竞品B | 价差% | 建议
- Flag SKU with price difference >10% as "需调价"
- Never suggest price below cost × 1.15 (15% gross margin floor)
- Always include data fetch timestamp in output
- Suggest 3 levels: 立即调价(>15%差) / 关注(10-15%差) / 正常(<10%差)
```

**Step 5-6（并行测试 + 评估）**：
- Baseline（不带 Skill）：AI 做了一般性的竞品分析但没有定量比价，没有给出具体调价数字，没有毛利底线约束
- With-skill：输出完整的对比表 + 价差百分比 + 三档调价建议 + 数据时间戳
- 评分：baseline 45 → with-skill 85（提升 40 分）

**效果**：
- 每日比价耗时从 45 分钟（手动）→ 5 分钟（AI 生成 + 人工复核）
- 漏标率从约 15%（手动比价注意力疲劳）→ 0%（AI 全量对比）
- 一个月内因及时调价，20 个核心 SKU 的加购率平均提升 8.3%（价格竞争力恢复）

> 🔑 **启示**：电商运营的重复性劳动不是「做一次很复杂」，而是「每天做一样的很消耗」。自动比价器 Skill 不是取代「比价」这个动作——是压缩了每天 45 分钟中的 40 分钟，把人的时间留给「判断是否调价」的决策环节。Baseline 从 45→85 的 40 分提升说明：Skill 提供了 AI 本不具备的领域知识（价差阈值、毛利底线、三档分类标准）。

---

## 7. 掌握检验

**Q1**：渐进式披露的三级加载系统中，Level 2 的内容是什么？它的大小建议是多少？
- A) Level 2 是元数据 name + description，始终在上下文中，大小约 100 词
- B) Level 2 是 SKILL.md 正文，Skill 触发时加载，理想大小 < 500 行
- C) Level 2 是打包资源（scripts/references/assets），按需加载，无大小限制
- D) Level 2 是测试用例文件

**Q2**：为什么「不要只看 ALWAYS/NEVER 的全大写指令，要解释为什么」是核心洞察？
- A) 全大写指令占用更多 token
- B) 今天的 LLM 足够聪明，理解因果关系比盲目遵守命令更有效
- C) 全大写指令可能导致代码错误
- D) 小写指令渲染更快

**Q3**：在并行运行 with-skill 和 baseline 测试中，如果 baseline 输出 80 分，with-skill 输出 82 分——这意味着什么？你应该怎么做？

**Q4**：near-miss 查询在触发描述优化中为什么最有价值？请举一个电商运营 Skill 的 should-trigger 和 near-miss should-not-trigger 的例子。

**Q5**：以下哪项是对「从反馈中泛化」的正确理解？
- A) 把所有测试用例的反馈堆在一起
- B) Skill 不是只通过这几个测试用例——它要被使用百万次，要从具体案例中提取通用模式
- C) 每一个测试用例对应一条专门的规则
- D) 泛化就是把规则写得更模糊

**Q6**：「如果所有测试用例都写了类似的辅助脚本，就应该打包到 scripts/」——这个洞察反映了什么设计原则？

**Q7**：请简要描述 skill-creator 的 8 步开发循环，并标注哪一步被课程明确指出为「最容易被跳过」以及跳过的后果。

---

## 8. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** — Level 2 是 SKILL.md 正文，包含 Skill 的核心指令，在 Skill 触发时加载到上下文，理想大小 < 500 行。Level 1 是元数据（~100 词常驻），Level 3 是打包资源（无限制，按需加载）。

**Q2**：**B** — 今天的 LLM 拥有强大的因果推理能力。如果你解释「使用 X 格式是因为它能被下游系统解析，如果不用就会在流水线上卡住」，AI 会比看到「ALWAYS USE X FORMAT」时更严格地执行——因为它理解了不遵守的后果。

**Q3**：参考答案 — 这意味着 Skill 几乎没有起到提升效果（只增加了 2 分）。可能原因：SKILL.md 的内容与 baseline 已有的能力重叠（Skill 告诉 AI 的都是 AI 本来就知道的），或者 Skill 的指令不够具体。应该回到 SKILL.md 检查——是否加入了 AI 不知道的领域知识？然后重新测试。

**Q4**：参考答案 — near-miss 的价值在于它能检测触发描述的「精确度」。电商运营 Skill 的例子：should-trigger：「帮我写一份母亲节促销文案」；near-miss should-not-trigger：「帮我分析一下竞品的母亲节广告策略」（提到了「母亲节」和「广告」，关键词重叠，但实际需要的是竞品分析 Skill 而不是促销文案 Skill）。

**Q5**：**B** — 「从反馈中泛化」意味着 Skill 要服务百万次调用，不能仅仅通过几个测试样例。关键是提取通用模式，而不是为每个测试用例单独写一条规则。

**Q6**：参考答案 — 这反映了 DRY（Don't Repeat Yourself）原则在 Skill 开发中的应用。放在 scripts/ 中后：每一次 Skill 被触发时，这个脚本都可用，不需要在 SKILL.md 中占用宝贵的 token 重复描述。

**Q7**：参考答案 — 8 步开发循环：(1) 捕获意图 → (2) 采访与调研 → (3) 编写 SKILL.md → (4) 编写测试用例 → (5) 并行运行测试 → (6) 评分与聚合 → (7) 启动评审查看器 → (8) 迭代改进。最容易被跳过的是 Step 5（并行运行测试/基线对比）。跳过后果：无法客观验证 Skill 是否真的有效——可能写了 500 行 SKILL.md，但实际效果跟没有 Skill 的 baseline 几乎一样。

（5/7 通过）
</details>

---

## 9. 延伸阅读

- [[L5-08-MCP工具构建器|上一课：MCP工具构建器]]
- [[L5-10-Slack动图生成|下一课：Slack GIF动画生成]]
