---
tags: [教程, Skill, 大师课, Level5, Anthropic, API, 开发]
created: 2026-05-03
updated: 2026-05-05
course_number: L5-17
prerequisites: ["[[L5-16-Excel表格生成]]"]
next_course: "[[L5-18-浏览器自动化命令行]]"
---

# L5-17：Claude API最佳实践 —— 把最强AI集成到你自己的产品里

> 🎯 **学什么**：学会在自己的产品中调用 Claude API——从简单的"问一句话、得到回答"，到复杂的"AI 自己决定下一步做什么"的智能代理。包括提示缓存（省钱）、思考模式（变聪明）、流式输出（打字机效果）、工具调用（AI 能操作外部系统）。核心洞察：(1) 不是所有任务都需要 Agent——Agent 是一把双刃剑，灵活但成本高（10-20 次 API 调用 vs 1 次）、延迟大（30 秒 vs 2 秒）、行为难以预测——用四条件检查表判断是否值得；(2) 提示缓存的"静默失效"是最隐蔽的性能杀手——`datetime.now()`、未排序的 JSON、变化的工具集都会让缓存静默失效；(3) Opus 4.7 的 task_budget 是最被低估的功能——让 AI 拥有了时间管理的能力。
> 💡 **难易度**：⭐⭐⭐⭐⭐ | ⏱️ **预计时间**：60 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：你用的 Claude 是一个成品 App（像去餐厅吃饭）。Claude API 是"后厨"——你可以把 Claude 的能力接入你自己的产品（像开自己的餐厅，但后厨用米其林大厨）。这门课教你从"点一道菜"（单次调用）到"让大厨自己决定做什么菜"（Agent 模式）的全部技能。

> **来源**：Anthropic 官方 Skills 仓库

---

## 2. 四个 API 层次：从简单到 Agent

### 2.1 决策树

```
选择哪个 API 层次？按以下决策树判断：

0. 是否通过 Bedrock/Vertex/Foundry 部署？
   → Yes：Claude API（+ tool use）——Managed Agents 仅限第一方
   → No：继续

1. 单次 LLM 调用（分类、摘要、提取、问答）
   → 一次请求、一次响应
   → 最简单：Claude API 基础调用
   → 适合：>80% 的 AI 集成场景

2. 是否要 Anthropic 运行 Agent 循环并托管执行容器？
   → Yes：Managed Agents
   → 适合：想把 Agent 循环和基础设施交给 Anthropic 管理
   → 限制：仅限第一方（Anthropic 直接 API）

3. 工作流（多步骤、代码编排、自有工具）
   → Claude API + tool use
   → 你控制循环，你管理工具执行
   → 适合：有明确的步骤流程但需要 AI 参与决策

4. 开放式的 Agent（模型自主决策路径，自有工具，自托管计算）
   → Claude API agentic loop
   → 最大灵活性——AI 自主决定每一步
   → 适合：路径不确定的探索性任务
```

### 2.2 Agent 构建的四条件检查表

```
什么时候应该构建 Agent？四个条件必须全部满足：

✅ 复杂性（Complexity）
   → 任务需要多步骤，难以预先完全指定所有步骤
   → 反例：简单的文本分类——一次调用就够，不需要 Agent

✅ 价值（Value）
   → 结果值得更高的成本（10-20 次 API 调用 vs 1 次）和延迟（30 秒 vs 2 秒）
   → 反例：如果单次调用的延迟已经是用户体验的极限

✅ 可行性（Feasibility）
   → Claude 在这个任务类型上胜任（不涉及它不擅长的领域）
   → 反例：需要精确到小数点后 6 位的数学——LLM 天然方差大

✅ 容错（Fault Tolerance）
   → 错误可被捕获和恢复（测试/审查/回滚机制到位）
   → 反例：直接扣款、删除数据等不可逆操作

任何一条"否"→ 使用更简单的层次（单次调用或工作流）。
```

---

## 3. 提示缓存：省钱的核心机制

### 3.1 缓存的工作原理

```
提示缓存（Prompt Caching）：
  → 第一次请求（写入）：完整处理，结果缓存
  → 后续请求（命中）：缓存部分只需 10% 的成本
  → 就像学生上课记笔记——第一次记下来，以后翻笔记比重新听一遍快

前缀匹配原则：
  → 缓存匹配是从前往后逐个 token 比较的
  → 任何字节的变化都会使该变化点之后的所有内容缓存失效
  → 因此渲染顺序至关重要：tools → system → messages
```

### 3.2 缓存静默失效器

```
最常见的三种缓存静默失效：

静默失效 1：嵌入 datetime.now()
  → 在 system prompt 中嵌入了当前时间戳
  → 每次请求时间戳不同 → 整个 system prompt 缓存失效
  → 修复：把时间戳放在最后一个 user message 中

静默失效 2：未排序的 JSON
  → JSON 对象的键顺序不同导致哈希值不同
  → 内容完全相同但 {b:2, a:1} ≠ {a:1, b:2} 在哈希上
  → 修复：始终对 JSON 键排序（sort_keys=True）

静默失效 3：变化的工具集
  → 每次请求的工具定义不同（动态生成工具列表）
  → 工具定义部分的缓存失效
  → 修复：尽量使用固定的工具集，或者把不变的工具放前面
```

### 3.3 缓存最大化策略

```
渲染顺序：tools → system → messages

稳定内容（放前面）：
  → 冻结的系统提示（不变）
  → 确定性的工具列表（不变）
  → 静态的示例（不变）

变化内容（放最后）：
  → 用户问题（每次都不同）
  → 时间戳、当前状态

效果：
  → 前 90% 内容稳定 → 缓存命中
  → 只有最后 10% 变化 → 这部分重新处理
  → 节省约 90% 的缓存相关成本
```

---

## 4. 思考模式与 Effort 参数

### 4.1 Opus 4.7 的关键变化

```
Opus 4.7 的思考模式重大变化：

移除的功能：
  ✗ budget_tokens（使用会返回 400 错误）
  ✗ temperature、top_p、top_k

新增/保留的功能：
  ✓ thinking: {type: "adaptive"}——仅支持自适应思考
  ✓ effort 参数（low / medium / high / xhigh）

Effort 选择指南：
  low：简单任务（分类、提取）——低成本低延迟
  medium：一般问答——平衡
  high：默认值——智能敏感任务的最佳平衡
  xhigh：Opus 4.7 专属——coding 和 agentic 场景最佳

关键认知：
  → Effort 对 Opus 4.7 比任何之前的版本都更重要
  → 从旧版迁移时必须重新调优——不能沿用旧版本的经验值
```

### 4.2 task_budget：最被低估的功能

```
task_budget vs max_tokens 的本质区别：

max_tokens：
  → 硬性的每次响应上限
  → 模型不可见这个上限
  → 可能在回答一半时被截断
  → 像一个"硬天花板"——AI 不知道它，撞上了就停了

task_budget（Opus 4.7 agentic loop 专用）：
  → 告诉模型整个 agentic 循环有多少 token 可用
  → 模型会看到一个倒计时
  → 自我调节：简单步骤快速完成，需要深度思考的分配更多 token
  → 像"时间预算"——AI 知道还剩多少资源，会主动管理

这就像给员工一个"项目预算"而不是"每天工作必须恰好 8 小时"——
员工会自己判断什么值得多花时间、什么可以快速完成。
```

---

## 5. Compaction（上下文压缩）的正确处理

### 5.1 常见错误

```
Compaction 是 Agent 长时间运行时的上下文管理机制。

❌ 常见错误：只提取 text 字符串
  response.content.map(c => c.text).join('')

  后果：
    → Compaction blocks（特殊的 content block type）被丢弃
    → API 无法恢复压缩状态
    → 对话上下文静默丢失
    → 模型表现出"失忆"——突然忘记了前面说过的话

✅ 正确做法：保留完整的 content 数组
  // 不要提取 .text
  // 保留整个 content 数组（包含 compaction blocks）
  messages.push({role: "assistant", content: response.content})
```

---

## 6. 结构拆解：Claude API 开发 Skill 模板

```markdown
## Claude API 开发型 Skill 模板

### 核心特征
→ 管理对象 = Claude API 集成的应用开发
→ 核心原则 = 四层次决策 + 缓存最大化 + Agent 四条件
→ 关键设计 = 缓存机制 + 思考模式 + 流式输出 + 工具调用

### 通用结构

## Four API Levels（四个 API 层次）
1. 单次调用（>80% 场景）
2. Managed Agents（Anthropic 托管）
3. 工作流 + tool use（你控制循环）
4. Agentic loop（最大灵活性）

## Agent Four-Condition Checklist（Agent 四条件检查）
复杂性 × 价值 × 可行性 × 容错 → 全部满足才用 Agent

## Prompt Caching（提示缓存）
- 前缀匹配原则：tools → system → messages
- 三个静默失效器：datetime.now() / 未排序JSON / 变化工具集
- 缓存最大化：稳定内容前、变化内容后

## Opus 4.7 Migration（Opus 4.7 迁移关键）
- budget_tokens 移除 → 改用 effort 参数
- task_budget：给 AI 时间管理能力
- Compaction：保留 response.content，不提取 .text
```

---

## 7. 电商案例：天猫店铺客服 AI 机器人集成

某国货美妆品牌天猫旗舰店（日均咨询量 800+ 条，客服团队 6 人）计划用 Claude API 构建一个智能客服 Agent，负责 80% 的常见咨询（查订单状态、查物流、退换货政策说明、产品使用问答），人工客服只处理投诉和复杂售后。

**四条件检查**：
- ✅ 复杂性：常见咨询需要多步骤（识别意图→查订单→查物流→匹配 FAQ→生成回答），有 17 种高频意图需要路由
- ✅ 价值：6 个客服每天处理 800+ 条消息中约 640 条是重复性问题（占 80%），自动化后释放 4.8 人/天
- ✅ 可行性：Claude 在中文客服场景胜任——能理解「我的快递到哪了」「这个精华液孕妇能用吗」等非结构化口语
- ✅ 容错：退款和投诉类仅做意图识别 + 自动转接人工，Agent 不直接执行退款——指令存在人工「确认」按钮之后

→ 四个条件全部满足，适合构建 Agent。

**工具设计**：

```python
tools = [
    {
        "name": "order_lookup_by_phone",
        "description": "Look up customer's recent orders by phone number (last 4 digits). Returns order list with status.",
        "input_schema": {"phone_last4": "string", "limit": "number (default 5)"}
    },
    {
        "name": "logistics_track",
        "description": "Track shipment by order ID. Returns current status + last update time + estimated delivery.",
        "input_schema": {"order_id": "string"}
    },
    {
        "name": "faq_search",
        "description": "Search product FAQ database for usage, ingredients, contraindications.",
        "input_schema": {"query": "string", "category": "string (enum: usage|ingredients|safety|return)"}
    },
    {
        "name": "escalate_to_human",
        "description": "Escalate conversation to human agent. Returns ticket number.",
        "input_schema": {"reason": "string (enum: complaint|refund|complex_query|abuse)", 
                         "summary": "string (50-300 chars summarizing the issue)"},
        "annotations": {"destructiveHint": False}
    }
]
```

**缓存优化（tools → system → messages 顺序）**：
- System prompt 冻结在最前：「品牌话术规范（称用户'亲'而非'您'、每句回复必须用😊结尾）+ 退换货政策（7 天无理由/运费险自动理赔/过敏包退）+ 17 种 FAQ 路由规则」
- 工具定义（4 个固定工具）→ 放第二位
- 用户消息（每次不同的客户口语化提问）→ 放最后

**效果**：
- 上线首月：640 条/天的重复咨询中 560 条由 Agent 自动处理（自动处理率 87.5%）
- 客服团队工作量：从 6 人 × 8 小时 → 6 人 × 4 小时（释放 4 小时/人/天，用于主动回访和差评挽回）
- 缓存命中率 94%——system prompt 和工具定义始终命中，单次对话平均 token 消耗降低 52%
- CSAT（客户满意度）：Agent 处理的对话 4.3/5.0，与人工客服的 4.5/5.0 差距仅 0.2 分

> 🔑 **启示**：电商客服 Agent 不是替代人工——是让人工从「订单号发我」「快递到哪了」「这个能用吗」的重复劳动中解放出来，把精力放在投诉挽回和差评沟通上（这些是 AI 处理不好但真正影响店铺 DSR 评分的环节）。另外，`escalate_to_human` 工具的阈值设计（AI 置信度 < 80% 自动转人工）避免了 AI 「硬答」导致的客诉升级。

---

## 8. 掌握检验

**Q1**：Claude API 四个层次中，什么时候应该选择"Managed Agents"？
- A) 永远选 Managed Agents——它是最新的
- B) 需要一个简单的单次分类任务时
- C) 需要 Anthropic 托管 Agent 循环并执行工具代码，且不通过 Bedrock/Vertex/Foundry 部署时
- D) 需要自己控制 Agent 循环和工具执行时

**Q2**：Opus 4.7 的思考模式与之前版本相比，以下哪个变化是正确的？
- A) budget_tokens 改为可选参数
- B) budget_tokens 完全移除——使用会返回 400 错误；只支持自适应思考
- C) temperature 参数被增强
- D) 所有模型都移除了 thinking 参数

**Q3**：提示缓存的"静默失效器"有哪些常见例子？请列出 3 个并解释为什么每个都会导致缓存失效。

**Q4**：`task_budget` 和 `max_tokens` 有什么区别？为什么 task_budget 是"Opus 4.7 最被低估的功能"？

**Q5**：Agent 构建的四条件检查表是什么？如果四个条件中有一条是"否"，你应该怎么做？

**Q6**：Effort 参数在 Opus 4.7 中为什么比之前版本更重要？

**Q7**：提示缓存的前缀匹配原则是什么？为什么"把稳定内容放前面、不稳定内容放最后"能最大化缓存命中率？

**Q8**：Compaction 中为什么"必须保留 response.content 而不仅是 text"？

---

## 9. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**C** — Managed Agents 是 Anthropic 服务端托管的有状态 Agent。限制：仅限第一方，Bedrock/Vertex/Foundry 不可用。

**Q2**：**B** — Opus 4.7 仅支持自适应思考，`budget_tokens` 被完全移除。同时 `temperature`、`top_p`、`top_k` 也被移除。

**Q3**：参考答案 — (1) `datetime.now()`：时间戳每次都不同，整个 system prompt 缓存失效；(2) 未排序的 JSON：键顺序不同导致哈希不同；(3) 变化的工具集：动态生成工具列表，工具定义缓存失效。

**Q4**：参考答案 — `max_tokens` 是硬上限，模型不可见，可能在回答一半时截断。`task_budget` 告诉模型整个循环有多少 token，模型会自我调节。这就像给 AI "时间预算"而非"硬天花板"——让 AI 有了时间管理能力。

**Q5**：参考答案 — 四条件：复杂性、价值、可行性、容错。任何一条"否"→ 使用更简单的层次。因为 Agent 灵活但成本高、延迟大、行为难以预测。

**Q6**：参考答案 — Effort 对 Opus 4.7 比任何之前的 Opus 都更重要——迁移时必须重新调优。high 是默认值（智能敏感任务最佳平衡），xhigh 是 Opus 4.7 专属（coding 和 agentic 最佳）。

**Q7**：参考答案 — 前缀匹配：缓存从前往后逐 token 比较，任何变化使之后的内容全部失效。稳定内容（system prompt、工具定义）放最前面，变化内容（用户问题）放最后——只有最后的变化部分使缓存失效。

**Q8**：参考答案 — Compaction blocks 是特殊 content block type，被 API 用来恢复压缩状态。如果只提取 text 字符串，compaction blocks 被丢弃，API 无法恢复压缩状态——导致对话上下文静默丢失，模型"失忆"。正确做法是保留完整 content 数组。

（6/8 通过）
</details>

---

## 10. 延伸阅读

- [[L5-16-Excel表格生成|上一课：Excel表格魔法师]]
- [[L5-18-浏览器自动化命令行|下一课：命令行浏览器操控]]
