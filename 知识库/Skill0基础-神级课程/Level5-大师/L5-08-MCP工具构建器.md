---
tags: [教程, Skill, 大师课, Level5, Anthropic, MCP, API]
created: 2026-05-03
updated: 2026-05-05
course_number: L5-08
prerequisites: ["[[L5-07-Web应用自动化测试]]"]
next_course: "[[L5-09-Skill创建器]]"
---

# L5-08：MCP工具构建器 —— 给自己的AI助手装上更多超能力

> 🎯 **学什么**：创建高质量 MCP（MCP：模型上下文协议）服务器，让 AI 不仅能聊天，还能直接操作外部工具——查数据库、发邮件、调 API、搜文档。核心洞察：MCP 开发的成败 70% 在 Phase 1（规划）——工具粒度、命名规范、分页策略。跳过规划直接敲代码是最常见的失败原因。另一个关键：错误信息是区分「好 MCP」和「烂 MCP」的核心标志——烂错误信息让 AI 卡住（404→不知道下一步），好错误信息给 AI 指明出路。
> 💡 **难易度**：⭐⭐⭐⭐⭐ | ⏱️ **预计时间**：55 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：你的 AI 助手就像钢铁侠战甲里的贾维斯——它能陪你聊天、回答问题。但如果没有 MCP，贾维斯只能「说」不能「做」。有了 MCP，贾维斯可以帮你查数据库、发邮件、看日历——直接操控外部世界。MCP 就像 USB 接口标准——在 USB 出现之前，每个设备有自己的接口；MCP 出现后，所有 LLM 用同一套标准协议连接外部工具。

> **来源**：Anthropic 官方 Skills 仓库

---

## 2. 四阶段开发流程

### 2.1 Phase 1：深入研究与规划 ⚠️ 最容易被跳过

```
Phase 1 是 MCP 开发最关键但最被忽视的阶段。

核心决策（必须在 Phase 1 完成）：

决策 1：工具粒度（Granularity）
  → API 覆盖率策略：每个 API 端点对应一个工具
    优势：Agent 可自由组合操作，灵活性最高
    劣势：需要更多工具调用次数
    适合：API 功能多样且使用模式不可预测
  → 专门工作流工具策略：把常见多步骤打包成一个工具
    优势：特定任务更方便高效
    劣势：灵活性降低
    适合：某个多步骤操作被频繁使用
    如：「创建订单+扣库存+发邮件通知」三步总是一起做 → 封装为一个工作流工具

决策 2：命名规范（Naming Convention）
  → 统一前缀 + 一致的动词+名词结构
  ✓ order_search / order_cancel / order_detail
  ✗ lookup_order / cancel_order / detail_order
  ✗ findOrder / deleteOrder / showOrder
  → AI 通过名字就能猜到工具用途和归属

决策 3：分页策略（Pagination）
  → 一次返回多少条？5 条？50 条？500 条？
  → 默认建议：offset/limit 模式，默认返回 20 条
  → 考虑二次过滤：先模糊搜索、再精确查询

决策 4：技术栈
  → TypeScript（推荐）：高质量 SDK、兼容性好、AI 生成 TS 代码质量高
  → Python（备选）：FastMCP 框架 + Pydantic 模型 + @mcp.tool 装饰器
  → 传输方式：
    - Streamable HTTP：适合远程服务器部署（生产环境）
    - stdio：适合本地服务器（开发调试）
```

### 2.2 Phase 2-4：实现到评估

```
Phase 2：实现
  → 搭建项目结构
  → 实现每个工具的 handler
  → 关键：工具注解（Annotations）
    - readOnlyHint: true/false
    - destructiveHint: true/false ← 告诉 AI「这个操作可能会删除数据」
    - idempotentHint: true/false
    - openWorldHint: true/false

Phase 3：审查与测试
  → 代码质量检查
  → MCP Inspector 测试（MCP 官方测试工具）
  → 每个工具能否正常响应？

Phase 4：创建评估 ⚠️ 另一个容易被跳过
  → 编写 10 个复杂场景的评估问题
  → 验证「LLM 能不能通过这个 MCP 服务器完成真实世界任务」
  → 评估问题的 6 个标准：
    1. Independent（不依赖其他问题）
    2. Read-only（只查询不修改——安全）
    3. Complex（需要多次工具调用）
    4. Realistic（真实场景）
    5. Verifiable（答案可验证）
    6. Stable（数据不会变化）
```

---

## 3. 错误处理设计：区分好 MCP 和烂 MCP

### 3.1 为什么错误处理特别重要

```
传统 API 的错误处理：
  → 返回 HTTP 状态码 + 错误消息
  → 调用方是人类开发者 → 能理解 404/500 → 会看文档

MCP 的错误处理：
  → 调用方是 AI → AI 收到错误后会尝试不同的参数重试
  → 如果错误信息不清晰 → AI 进入「猜测-重试-失败」死循环
  → 浪费大量 token + 最终还是失败
```

### 3.2 错误信息对比

```
烂的错误信息：
  "Error 404"
  → AI 只知道出错了
  → 不知道为什么错
  → 不知道下一步该做什么
  → 陷入猜测循环

好的错误信息：
  "User 'user_12345' not found. 
   Try order_list_users to see available users, 
   or check if the user ID format is correct 
   (should be 'user_' prefix + 5 digits)."
  
  包含三个要素：
  1. 具体什么出错了（哪个用户没找到）
  2. 为什么可能出错（ID 格式不正确）
  3. 下一步可以做什么（试试 list_users 工具）
```

### 3.3 注解（Annotations）设计

```
四个 annotation hints 告诉 AI 如何安全使用工具：

readOnlyHint: true
  → 工具不会修改任何数据
  → AI 可以放心反复调用

destructiveHint: true  ⚠️ 最重要
  → 工具可能会删除/修改数据
  → AI 应该在调用前确认（需要用户授权）

idempotentHint: true
  → 重复调用结果是相同的（幂等）
  → AI 可以在失败后安全重试

openWorldHint: true
  → 工具需要联网访问外部资源
  → AI 应该预期可能因网络问题失败
```

---

## 4. 结构拆解：MCP 构建型 Skill 模板

```markdown
## MCP 构建型 Skill 模板

### 核心特征
→ 管理对象 = MCP 服务器（模型上下文协议工具接口）
→ 核心原则 = Phase 1 规划决定 70% 成败 + 错误信息给 AI 出路
→ 关键设计 = 四阶段流程 + 工具注解 + 评估 6 标准

### 通用结构

## Four-Phase Development（四阶段开发）
Phase 1: 深入研究与规划
  - 工具粒度（API 覆盖率 vs 工作流工具）
  - 命名规范（统一前缀 + 动词+名词）
  - 分页策略
  - 技术栈选择

Phase 2: 实现
  - 项目结构 + handler 实现 + 工具注解

Phase 3: 审查与测试（MCP Inspector）

Phase 4: 创建评估（10 个问题，6 条标准）

## Error Handling Design（错误处理设计）
每条错误信息三要素：
1. 具体出了什么错
2. 为什么可能出错
3. 下一步可以做什么

## Tool Annotations（工具注解）
- readOnlyHint / destructiveHint / idempotentHint / openWorldHint
```

---

## 5. 电商案例：订单/库存/物流数据 API 至 MCP 工具转换

某快消品电商公司（主营进口零食，已入驻天猫、京东、拼多多 3 个平台）的运营团队每天需要在 3 个商家后台之间切换查看订单状态、核实库存、追踪物流。运营主管想让 AI 助手能直接跨平台查询——「帮我看一下拼多多今天未发货的订单有多少」「京东仓的蛋黄酥还有多少库存」。

用 mcp-builder 构建了一个「电商订单与库存 MCP 服务器」，对接公司已有的 ERP 数据库。

**Phase 1 规划**：
- 工具粒度：API 覆盖率策略（按业务实体划分——订单查询、库存查询、物流追踪，而非每个平台一个工具）
- 命名规范：`order_` / `inventory_` / `logistics_` 前缀 + 动词+名词
  - `order_search`（按平台+状态+时间范围模糊搜索，分页 20 条）
  - `order_detail`（按订单号精确查，含商品明细/优惠减免/实付金额）
  - `inventory_check`（按 SKU+仓库查当前库存，含安全库存预警线）
  - `logistics_track`（按运单号查物流轨迹，含异常停留时间标注）
- 分页：order_search 默认 20 条/页（客服单次对话通常只看最新 20 笔），最大 100 条
- 技术栈：TypeScript + Streamable HTTP（部署于公司内网）

**关键工具设计示例**：

```typescript
// order_search —— 最常用工具，带分页和平台过滤
{
  name: "order_search",
  description: "Search orders by platform, status, and time range. Returns paginated results (default 20 per page).",
  inputSchema: {
    platform: "string (enum: tmall|jd|pinduoduo)",
    status: "string (enum: pending|paid|shipped|delivered|cancelled)",
    date_from: "string (ISO 8601, e.g. 2026-05-01)",
    date_to: "string (ISO 8601)",
    page: "number (default 1)",
    page_size: "number (default 20, max 100)"
  }
}

// inventory_check —— 带安全库存预警
{
  name: "inventory_check",
  description: "Check current inventory for a SKU across warehouses. Returns stock quantity + safety stock threshold. If below threshold, returns warning flag.",
  inputSchema: {
    sku: "string (e.g. DS-001 for 蛋黄酥经典装)",
    warehouse: "string (optional, omit for all warehouses)"
  }
}
```

**错误处理设计**：
```
Good error example:
"Order 'ORD-20260503-8842' not found in platform 'tmall'.
 Possible reasons:
 1. Order ID belongs to a different platform (try 'jd' or 'pinduoduo')
 2. Order was created before 2025-06-01 (archived data, use order_archive_search instead)
 3. Order ID format is incorrect (expected: ORD-YYYYMMDD-XXXX)
Suggestions: use order_search to fuzzy-search by date range instead."
```

**效果**：
- 运营主管输入「拼多多昨天发了多少单」→ AI 自动调用 `order_search(platform=pinduoduo, status=shipped, date_from=2026-05-04, date_to=2026-05-04)` → 即时返回数字——不需要登录拼多多商家后台
- 库存预警查询「蛋黄酥还有多少」→ AI 返回「京东仓 247 件（安全库存 200，正常），天猫仓 58 件（低于安全库存 100！⚠️ 建议补货）」——运营在缺货前 3 天就已发现

> 🔑 **启示**：MCP 工具对电商运营的价值不是「AI 能查数据」——后台本来就能查。价值在于「AI 能跨平台用自然语言一句话查到」——运营不用登录 3 个后台、切换菜单、输入筛选条件。另外，`inventory_check` 工具嵌入安全库存预警逻辑，让 AI 从「被动回答」变成「主动提醒」。

---

## 6. 掌握检验

**Q1**：MCP 服务器开发的四个阶段的正确顺序是什么？
- A) 实现 → 规划 → 审查 → 评估
- B) 深入研究与规划 → 实现 → 审查与测试 → 创建评估
- C) 评估 → 规划 → 实现 → 审查
- D) 规划 → 评估 → 实现 → 审查

**Q2**：MCP 工具注解中，`destructiveHint` 的含义是什么？
- A) 该工具可能会永久删除数据或造成不可逆的改变
- B) 该工具是用 TypeScript 写的
- C) 该工具需要管理员权限
- D) 该工具执行速度很慢

**Q3**：「API 覆盖率」策略和「专门工作流工具」策略各有什么优缺点？在什么场景下应优先选择哪个？

**Q4**：以下哪个是最佳的工具命名方案？
- A) `lookup_order`, `cancel_order`, `detail_order`
- B) `order_search`, `order_cancel`, `order_detail`
- C) `findOrder`, `deleteOrder`, `showOrder`
- D) `get_order_info`, `remove_order`, `see_order`

**Q5**：错误处理在 MCP 工具中为什么特别重要？请对比「烂的错误信息」和「好的错误信息」，并解释好的错误信息应该包含什么额外内容。

**Q6**：一个好的 MCP 评估问题需要满足 6 个标准。假设你为「电商订单查询」MCP 服务器写评估问题，请写一个满足全部 6 个标准的评估问题。

**Q7**：为什么 Phase 1 被课程称为「被大多数开发者跳过的阶段」？跳过 Phase 1 直接敲代码会导致哪些具体问题？

**Q8**：技术栈选择中，为什么 TypeScript 被推荐为首选语言？Streamable HTTP 和 stdio 分别适用于什么部署场景？

---

## 7. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** — Phase 1: 深入研究与规划；Phase 2: 实现；Phase 3: 审查与测试；Phase 4: 创建评估。

**Q2**：**A** — `destructiveHint: true` 告诉调用方该工具可能会执行破坏性操作（如删除、取消、退款等），调用方应谨慎处理。

**Q3**：参考答案 — 「API 覆盖率」策略每个 API 端点对应一个工具，优势是灵活性最高，适合 API 功能多样且使用模式不可预测的场景。「专门工作流工具」策略把常见多步骤打包成一个工具，优势是特定任务更方便高效，适合某个多步骤操作被频繁使用的场景（如「创建订单+扣库存+发邮件」三步总是一起做）。

**Q4**：**B** — `order_search`、`order_cancel`、`order_detail` 使用了统一的 `order_` 前缀 + 一致的动词+名词结构，AI 一眼就知道这是一组相关工具。

**Q5**：参考答案 — 烂的错误信息："Error 404"——AI 只知道出错了，不知道为什么会出错，更不知道下一步该做什么。好的错误信息包含三要素：(1) 具体什么出错了；(2) 为什么可能出错；(3) 下一步可以做什么。

**Q6**：参考答案示例 — "A customer ordered product P001 (2 units) and P002 (1 unit) on 2026-01-15 with order ID ORD-8842. The order used coupon code SAVE20. What was the final paid amount after coupon discount, and what is the current shipping status?" 满足：Independent（不依赖其他问题）、Read-only（只查询）、Complex（需要多次工具调用）、Realistic（真实客服场景）、Verifiable（答案可验证）、Stable（数据不变）。

**Q7**：参考答案 — Phase 1 被跳过是因为开发者急于看到代码运行。跳过后的具体问题：(1) 工具粒度混乱——有些工具太细、有些太粗，LLM 调用时效率极低；(2) 命名规范不统一——后续改名字需要把 SKILL.md、代码、文档、评估全部更新。

**Q8**：参考答案 — TypeScript 推荐原因：高质量 SDK 支持、兼容性好、AI 生成 TS 代码质量高。Python 备选原因：FastMCP 框架 + Pydantic 模型。传输方式：Streamable HTTP 适合远程服务器部署（生产环境）；stdio 适合本地服务器（开发调试）。

（6/8 通过）
</details>

---

## 8. 延伸阅读

- [[L5-07-Web应用自动化测试|上一课：Web应用自动化测试]]
- [[L5-09-Skill创建器|下一课：元技能——用AI创造AI技能]]
