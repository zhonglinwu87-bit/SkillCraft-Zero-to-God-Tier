---
tags: [教程, Skill, 大师课, Level6, gstack, Skill 创建, 自动化]
created: 2026-05-03
updated: 2026-05-05
course_number: L6-49
prerequisites: ["[[L6-48-项目知识积累]]"]
next_course: "[[L6-50-创业深度诊断]]"
---

# L6-49：经验Skill化 —— 把你重复做的事一键打包成可复用技能

> **学 gstack 的 Skill 创建系统**：不是从零写 SKILL.md，而是从项目中反复出现的操作模式中自动提取 Skill 模板——配置、命令、工作流——把"我每次部署后都做这几步"变成"一个 Skill 自动完成"。
> **预计时间**：20 分钟

> [!tip] **初学提示**：如果你还无法从操作中识别出重复模式，从最简单的任务开始——比如"每次提交前跑的命令"——把它变成你的第一个 Skill。

---

> [!quote]- 原Skill精要
> **技能名称**：skillify
> **核心功能**：将最近成功执行的 /scrape 流程固化为一个持久化的 browser-skill 存到磁盘。回溯对话、合成 script.ts + script.test.ts + fixture、在临时目录中运行测试、询问后提交。未来相同意图的 /scrape 调用将运行已固化的脚本（约 200ms），而非重新驱动页面。
> **所属体系**：gstack 虚拟工程团队
> **协作关系**：固化后的 skill 可被任何 gstack Agent 调用，实现从"重复操作"到"一键完成"的转变

---

## §1 白话小课堂：什么是"经验Skill化"？

你在项目中是否经历过这样的循环？

- 每次上线前，手动跑一遍测试、检查 Redis 容量、确认 CDN 预热、验证数据库连接池...
- 每次提交代码前，手动跑 lint、format、typecheck...
- 每次大促前，手动检查一堆环境指标...

这些操作有两个成本：**执行成本**（你得花时间做）和**记忆成本**（你得记得做，还得记得怎么做）。

**skillify 做的事**：把你的操作序列自动变成一个可复用的 Skill。以后你只需要说一句话（如 `/presale-check`），所有检查自动执行，输出 PASS/FAIL 表格。而且这个 Skill 创建一次后，所有 Agent（你的 Agent、CI 中的 Agent、同事的 Agent）都能调用——投资回报率随使用次数线性增长。

> [!note] **类比：从"手动挡"到"自动挡"**
> 你发现你每天早上都在做同样的事：打开咖啡机、等 2 分钟、倒咖啡、加奶。你买了一个定时咖啡机——设定好时间，它自动完成前三步。你只需要倒咖啡和加奶。skillify 就是这个"定时咖啡机"——把重复的手动操作变成一键自动化。

---

## §2 结构拆解：skillify 的完整架构

```skill
# skillify Skill Template

## 触发词
/skillify "<描述>"、"把这个变成 Skill"、"固化这个流程"

## 工具列表
- Bash: 执行操作序列中的命令
- Read: 读取 SKILL.md 模板
- Write: 生成 SKILL.md、script.ts、script.test.ts
- Glob: 扫描项目文件以提取上下文

## 核心特征
- **管理对象**：重复操作序列 → 持久化可复用 Skill（SKILL.md + 脚本 + 测试）
- **核心原则**：模式提取 > 模板生成 —— 识别核心模式，参数化变体
- **关键设计**：
  - 操作序列 → Skill 模板：自动识别输入/输出/依赖/环境
  - 参数化：区分固定值（项目特有）和变量（跨项目可配置）
  - 权限最小化：初始生成时只有"刚好够执行原始操作"的权限
  - 版本管理：Skill 包含 version 字段，输入/输出契约变化时 bump 版本
  - 适用范围标记：project + compatible_with 防止跨项目误用

## 通用架构
输入层：
  → 操作序列（一组 bash 命令或特定操作步骤）
  → Skill 描述 + 触发词
  → 所需权限（Bash / Read / Write / Glob / WebFetch）
  ├── 模式识别引擎
  │   ├── 提取操作序列中的共同模式（交集 = 核心步骤）
  │   ├── 识别变体（并集 = 可选参数）
  │   └── 区分固定值 vs 变量（参数化）
  ├── Skill 生成
  │   ├── SKILL.md 模板（描述、触发词、工具列表、工作流）
  │   ├── script.ts（固化后的执行脚本，约 200ms vs 原驱动页面）
  │   ├── script.test.ts（固化脚本的测试）
  │   └── fixture（测试 fixture 数据）
  ├── 权限配置
  │   ├── 初始权限：刚好够执行原始操作
  │   ├── 权限扩展需显式审批（防止"权限蠕变"）
  │   └── 网络/文件系统权限需用户确认
  └── 版本与适用范围
      ├── version 字段（契约变化时 bump）
      ├── project 标记（防止跨项目误用）
      └── compatible_with 列表（白名单项目）
输出层：
  → SKILL.md（持久化 Skill 定义）
  → 固化脚本（未来调用约 200ms）
  → 测试通过确认
```

---

## §3 Skillify 的核心原理：从"重复"到"复用"

### 3.1 操作模式识别

skillify 的第一步不是"生成文件"，而是**识别模式**。这比听起来难：

```
场景：一个项目中，开发者 A 每次部署前做 A-B-C 三步，开发者 B 做 A-B-D 三步。

傻办法：分别生成 "A-B-C Skill" 和 "A-B-D Skill" —— 两个几乎一样的 Skill
聪明办法：识别核心模式 "A-B"（所有部署的共同需求）
         C = 检查 CDN 缓存预热（A 负责前端，所以需要）
         D = 检查消息队列积压（B 负责异步任务）
         → 生成一个 Skill：核心步骤 A-B + 可选参数 --check-cdn / --check-mq

模式识别的本质：找出交集（核心）+ 并集作为可选（扩展）。
不是硬编码所有变体，而是设计一个可扩展的框架让用户按场景启用/禁用可选步骤。
```

### 3.2 参数化：区分"固定值"和"变量"

这是 skillify 最容易被低估的一步。考虑以下操作序列：

```
redis-cli INFO memory | grep used_memory_human  # 检查 Redis 容量
```

如果 skillify 直接把这个命令固化到 Skill 里，生成的 Skill 将包含硬编码的 Redis 地址和端口（来自当前环境的默认值）。当这个 Skill 在另一个项目（使用不同的 Redis 配置）中运行时，会失败。

**参数化的过程**：

```markdown
## 原始命令
redis-cli -h redis.mega-shop.internal -p 6379 INFO memory | grep used_memory_human

## 提取参数
- redis-host: redis.mega-shop.internal（固定值？变量？）
  → 如果在 mega-shop 项目内使用 → 固定值
  → 如果可能跨项目使用 → 变量（--redis-host）
- redis-port: 6379 → 变量（--redis-port，默认 6379）
- 阈值: 16GB → 变量（--redis-min-gb，默认 16）

## 生成的 Skill 调用方式
/presale-check → 使用默认值（mega-shop 项目内）
/presale-check --redis-host redis.staging.internal → 在 staging 环境使用
/presale-check --redis-min-gb 8 → 小项目使用更低的阈值
```

**参数化 = 复用性**。没有参数化的 Skill = 一次性的脚本。参数化后的 Skill = 可跨项目、跨环境使用的工具。

### 3.3 模板生成：SKILL.md 的五要素

skillify 生成的 SKILL.md 不是随意的——它包含五个标准要素：

```markdown
1. 描述（Description）
   → 这个 Skill 做什么？一句话说清楚
   → "自动化大促前的五项环境检查，输出 PASS/FAIL 表格"

2. 触发词（Triggers）
   → 用户说什么时调用这个 Skill？
   → "大促检查"、"封网前检查"、"pre-sale check"、/presale-check

3. 工具列表（Tools）
   → 这个 Skill 需要哪些工具权限？
   → Bash（执行 shell 命令）、Read（读取配置文件）

4. 工作流（Workflow）
   → Skill 执行的步骤序列，每步的输入/输出
   → Step 1: 检查 Redis 容量 → redis-cli INFO memory → 提取 used_memory_human
   → Step 2: 检查 DB 连接池 → psql -c "SHOW max_connections"
   → ...

5. 输出格式（Output）
   → Skill 的输出是什么格式？
   → Markdown 表格：| 检查项 | 状态 | 详情 |
```

### 3.4 权限模型：最小权限 + 显式扩展

```
初始生成：
  → 分析原始操作序列中使用了哪些工具
  → 只授予"刚好够用"的权限
  → 例：大促检查 Skill 只需要 Bash（5 个 shell 命令）
  → tools: [Bash]

权限扩展：
  → 如果有人想添加"发送 Slack 通知"步骤
  → 需要 WebFetch 权限（不在原始工具列表中）
  → 系统阻止并提示："此 Skill 正在请求添加网络访问权限——是否允许？"
  → 用户确认后才在 SKILL.md 的 tools 中添加 WebFetch

防止"权限蠕变"：
  → 一个最初只需要 Bash 的 Skill
  → 不应该在多次修改后无声地获得网络访问、文件写入等额外权限
  → 每次权限扩展 = 显式审批
```

---

## §4 版本管理与适用范围控制

### 4.1 版本管理

```markdown
## Skill 版本策略

Skill 被创建后不是"永恒不变"的。当底层逻辑变化时：

场景：大促检查 Skill v1 包含"确认数据库连接池 >= 50"
     但新架构中数据库连接池由 Kubernetes 自动管理 → 这个检查不再需要

处理方式：
  Step 1: 不直接删除——在新版本（v2）的 SKILL.md 中
    将旧步骤标记为 [DEPRECATED] "已在 v2 中移除"
  Step 2: 用户升级时自动 bump version → 移除过时步骤
  Step 3: 如果用户选择不升级 → 运行时提示"此 Skill 使用已过时检查项（v1），建议升级到 v2"

关键原则：Skill 的输入/输出契约发生变化时必须 bump 版本
用户显式选择是否升级——给过渡期，不确定覆盖
```

### 4.2 适用范围标记

```
防止跨项目误用：

mega-shop 的"大促前检查"Skill 包含:
  redis-cli -h redis.mega-shop.internal INFO memory

如果在 mini-shop 项目中调用:
  → Agent 尝试连接 redis.mega-shop.internal（不存在或指向错误 Redis）
  → 连接超时或返回错误数据
  → 检查结果完全错误，但 Agent 可能不知道（假阳性/假阴性）

适用范围标记设计:
  SKILL.md 中添加:
    project: mega-shop
    compatible_with: [mega-shop, mega-shop-staging]

  其他项目的 Agent 调用时:
    → 提示"此 Skill 配置为 mega-shop 专用，当前项目为 mini-shop——是否仍要运行？"
    → 降误用但允许有意识的跨项目复用（如果用户确认两个项目的 Redis 架构相同）
```

---

## §5 电商 Skill 案例：电商大促前自动检查Skill的创建与演化

> 🛒 **实战案例**：某电商运营团队每次大促前手动执行5项环境检查——Redis容量确认(≥16GB)、数据库连接池确认(≥200)、CDN缓存预热确认、秒杀商品库存确认、支付链路E2E测试。每次耗时15-20分钟，在双11高压场景下遗漏任何一步都可能造成线上事故。使用`/skillify`将这套操作模式转化为自动化Skill。

> - **skillify过程（5步）**：Step 1模式识别——识别出这是"环境检查"类操作模式，5个命令全部是Bash工具（`redis-cli INFO memory`/`psql -c "SHOW max_connections"`/`curl https://cdn.mega-shop.com/health`/`curl https://api.mega-shop.com/flash-sale/stock-check`/`bun test test/e2e/payment.test.ts`），输入独立互不依赖，输出每个命令返回检查结果。Step 2参数提取——Redis host `redis.mega-shop.internal`→变量`--redis-host`、Redis容量阈值16GB→变量`--redis-min-gb`(默认16)、DB连接池阈值200→变量`--db-min-connections`(默认50)、CDN health URL→变量`--cdn-url`、秒杀库存API→变量`--stock-api`、E2E测试命令→变量`--e2e-test-cmd`。参数化确保这个Skill不硬编码mega-shop的值，可以用于其他项目或同一项目的staging/production不同环境。
> - **Step 3-4 生成SKILL.md和固化脚本**：SKILL.md——描述"自动化大促前的五项环境检查，输出PASS/FAIL表格"、触发词"大促检查/封网前检查/pre-sale check//presale-check"、工具Bash only(无需WebFetch/Write)、工作流依次执行5项检查→汇总结果→输出表格。固化脚本`script.ts`——封装5个检查步骤的执行逻辑和结果解析(Redis: 解析`used_memory_human`与阈值对比、DB: 解析`max_connections`、CDN: HTTP状态码200+响应内容含"warm"、库存: JSON响应中的stock数值、E2E: 测试通过率100%)。
> - **测试与验证**：在临时目录运行`script.test.ts`——模拟每个检查的成功和失败场景(Redis内存不足→WARN/DB连接数低于阈值→WARN/CDN未预热→FAIL/库存为0→FAIL/E2E失败→FAIL)→所有测试通过→用户确认→提交到`gstack-presale-check/`。
> - **使用效果与演化**：之后每次大促前运行`/presale-check`→输出PASS/FAIL表格，从原来手动15-20分钟缩减到自动化8秒。但第一次双11使用后发现了新的检查项——"优惠券服务健康检查"和"物流查询API响应时间"也需要检查→通过`/skillify --update /presale-check`将新检查项添加到Skill→Skill在实战中持续演化。创建Skill花了5分钟，此后每次大促节省15-20分钟→年回报率600%+。

> 🔑 **启示**：skillify的核心价值不是"快速执行"——而是"消除记忆成本"。在大促准备的高压场景下，人的记忆力是最不可靠的环节——遗漏任何一项检查都可能造成线上事故。Skill确保五项检查一个不落——不是"执行更快了"，是"不会忘记了"。参数化是复用的关键——不参数化的Skill是代码hardcoding，参数化后的Skill是应用的函数：同一个`/presale-check`可以用于staging(--redis-host staging.redis.internal --redis-min-gb 4)、production(--redis-host prod.redis.internal --redis-min-gb 16)、甚至其他项目。一个Team Lead创建Skill，整个团队受益——跨Agent复用是skillify的投资回报率放大器。

> [!info] 参考来源
> Skill 模式识别参考了 [gstack SKILL.md template system](https://github.com/garrytan/gstack) 的 gen-skill-docs 生成流程。操作模式识别参考了 [Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns) 的模式提取方法论。
## §6 掌握检验

**Q1**：skillify 的核心功能是什么？
- A) 从零开始手动编写 SKILL.md 文件
- B) 从项目中反复出现的操作模式中自动提取 Skill 模板——识别操作序列 → 提取参数 → 生成 SKILL.md → 配置工具列表和触发词
- C) 下载社区贡献的 Skill 模板
- D) 将 Skill 转换为 bash 脚本

**Q2**：skillify 生成 SKILL.md 时，"识别参数"为什么特别重要？
- A) 参数让 Skill 更美观
- B) 参数区分"固定值"和"变量"——如果大促检查 Skill 中 Redis 容量阈值固定为 16GB，当不同项目使用不同阈值时 Skill 无法复用；参数化（`--redis-min-gb`）让同一个 Skill 适配不同项目
- C) 参数是 GitHub 的要求
- D) 参数让 Skill 文件大小减小

**Q3**：电商案例中将"大促前环境检查"转化为 Skill。原始操作包含 5 个检查步骤——如果其中某个检查步骤在新的大促场景中不再需要（如"确认数据库连接池 >= 50"但在新架构中数据库连接池由 Kubernetes 自动管理），Skill 应该如何处理过时的步骤？是删除该步骤？还是保留但标记为"可选"？Skill 的"版本管理"应该如何设计——当底层检查逻辑变化时，如何确保使用旧版 Skill 的人不会执行过时的检查？

**Q4**：从"重复操作"到"可复用 Skill"的转化中，操作模式识别的关键挑战是什么？假设一个项目中开发者 A 每次部署前手动做 A-B-C 三步，开发者 B 手动做 A-B-D 三步（C 和 D 不同）。skillify 应该提取出什么 Skill？是"A-B-C"还是"A-B-D"还是"A-B（C/D 可选）"？如何从多个相似但不完全相同的操作序列中发现真正的"核心模式"？

**Q5**：skillify 生成的 Skill 需要配置"允许的工具列表"（Bash/Read/Write/Glob 等）。如果大促检查 Skill 的原始操作包括 `redis-cli INFO memory`（Bash），但 Skill 生成后有人添加了一个步骤"发送 Slack 通知"（需要 HTTP 请求工具，不在原始工具列表中）——Skill 的权限模型是否会阻止这个新步骤？如果会，如何在不重新生成整个 Skill 的前提下扩展权限？

**Q6**：skillify 将一个操作序列固化为 Skill 后，这个 Skill 可以被"任何 gstack Agent"调用。这带来了什么好处（复用）和什么风险（过度泛化）？如果一个 Skill 是为特定项目（mega-shop）的大促场景创建的，但在另一个项目（mini-shop）中被错误地调用——可能有什么后果？如何设计 Skill 的"适用范围"标记来防止这种跨项目误用？

---

## §7 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** —— skillify 的核心是"模式提取"——识别项目中反复出现的操作序列，提取参数（固定值 vs 变量），生成标准化的 SKILL.md 模板（包含描述、触发词、工具列表、工作流），让"每次手动重复的操作"变成"说一句话自动完成"。

**Q2**：**B** —— 参数化 = 复用性。

**Q3**：处理过时步骤：1. 删除——如果步骤完全不再需要（如架构变更导致数据库连接池不需要手动检查），在 Skill 的新版本中删除该步骤。2. 标记为可选+deprecated——在 SKILL.md 中将该步骤标记为 `[DEPRECATED]`（"已在 v2 中移除"），保留 old version 的 Skill 文档但不执行，给用户过渡期。版本管理：Skill 文件包含 `version` 字段 → 生成时自动 bump → 用户升级 Skill 时运行 migration（类似 gstack-upgrade）→ 自动移除过时步骤 → 如果用户选择了不升级 → 运行时显示"此 Skill 使用已过时的检查项（v1），建议升级到 v2"。关键原则：Skill 的输入/输出契约发生变化时必须 bump 版本 → 用户显式选择是否升级。

**Q4**：核心模式 = "A-B"（部署前检查基础步骤：redis + db 连接验证 → 这是所有部署的共同需求）。C 是"检查 CDN 缓存预热"——A 负责前端模块，所以需要；D 是"检查消息队列积压"——B 负责异步任务模块。skillify 应该生成：核心 Skill = "A-B"（所有部署前必做）→ C 和 D 作为"可选参数"（`--check-cdn`、`--check-mq`）。模式识别 = 找出交集（核心）+ 并集作为可选（扩展）。不是硬编码所有变体——而是设计一个可扩展的框架让用户按场景启用/禁用可选步骤。

**Q5**：权限模型会阻止 Slack 通知——因为 HTTP 请求工具不在原始工具列表中。扩展权限的方法：1. 在 SKILL.md 的 `tools` 字段中添加 `WebFetch` 或 `bash`（允许 curl）→ 不要重新生成整个 Skill，只需编辑 SKILL.md 的 tools 列表。2. 权限扩展需要审批——如果是"自动化 Skill 添加了网络访问权限"，应该提示用户确认（"此 Skill 正在请求添加网络访问权限以发送 Slack 通知——是否允许？"）。3. Skill 的权限应该是最小权限原则——初始生成时只有"刚好够执行原始操作"的权限，后续扩展需要显式审批。防止"权限蠕变"——一个最初只需要 Bash 的 Skill 不应该无声地获得网络访问、文件写入等额外权限。

**Q6**：好处：Skill 一次创建后，所有 Agent（开发机上的 Claude、CI 中的 Agent、同事的 Agent）都能调用——投资回报率随使用次数线性增长。风险：mega-shop 的"大促前检查"Skill 包含 `redis-cli -h redis.mega-shop.internal`——这是一个硬编码的内部地址。如果在 mini-shop 中调用 → Agent 尝试连接 `redis.mega-shop.internal`（不存在或指向错误的 Redis）→ 连接超时或返回错误数据 → 检查结果完全错误但 Agent 可能不知道（假阳性或假阴性）。适用范围标记：SKILL.md 中添加 `project: mega-shop` 和 `compatible_with: [mega-shop, mega-shop-staging]` → 其他项目的 Agent 调用时提示"此 Skill 配置为 mega-shop 专用，当前项目为 mini-shop——是否仍要运行？" → 降低误用但允许有意识的跨项目复用（如果用户确认两个项目的 Redis 架构相同）。

（6/6 通过）
</details>

---

## 延伸阅读
- [[L6-48-项目知识积累|上一课：项目知识积累]]
- [[L6-50-创业深度诊断|下一课：创业深度诊断]]
- [[L6-15-PR代码审查|相关：PR 代码审查]] —— Skill 固化的检查流程可以被 review 系统调用
- [[L6-36-项目复盘大师|相关：项目复盘大师]] —— retro 可以识别出"值得 skillify 的重复模式"
