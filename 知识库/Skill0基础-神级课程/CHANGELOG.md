# Changelog

## [v0.3.0] - 2026-05-06

### Added
- **新增 3 门课程**，课程总数从 160 → 166 门
  - **L0-03：给AI立规矩 —— CLAUDE.md 项目配置与Karpathy行为规范**（Level 0 先修课）
    - 深入讲解 CLAUDE.md、SKILL.md、System Prompt 三层配置体系
    - 完整解读 andrej-karpathy-skills 的四大行为原则（Think Before Coding / Simplicity First / Surgical Changes / Goal-Driven Execution）
    - 含源码精读（65行 CLAUDE.md 逐行分析）、结构拆解、手写模仿、变体练习
  - **L5-25：超能力框架 Superpowers**（Level 5 大师）
    - 14 个互锁 Skill 的完整方法学框架（brainstorm→plan→TDD→review→ship）
    - 深入 "1%规则"、Iron Law、Red Flags 表、SessionStart Hook 机制
    - Rule vs Gate vs Hook 设计哲学对比
  - **L5-26：OpenSpec 规范驱动开发**（Level 5 大师）
    - Propose→Apply→Archive 三阶段工作流
    - Delta-based Specs、Brownfield-first、动态指令生成等核心创新
    - 11 个命令完整解析（Core Profile + Custom Profile）

### Changed
- Level 0 课程数 2→3 门，Level 5 课程数 24→26 门
- 更新 `Skill大师课-总纲.md` 课程地图、Level 0/5 课程表
- 更新 `Level5-大师/L5-课程总览.md` 增加方法学框架系列和毕业标准

### Knowledge Base
- 新增 7 个知识库页面：3 个来源摘要 + 3 个实体页 + 1 个概念页
- 更新 `知识库/index.md` 和 `知识库/log.md`

### Skip
- **gstack** 已在现有教程 Level 4-7 中覆盖 78 处，跳过

## [v0.2.0] - 2026-05-06

### Added
- **全课程英中术语翻译**：为 Level 0~7 全部 168 篇课程笔记中的英文技术术语添加中文解释
- 按学员阶段分级标注：
  - **L0-L2（入门阶段）**：详细解释，格式 `（术语：详细中文说明——做什么、为什么重要）`
  - **L3-L4（进阶阶段）**：简要解释，格式 `（术语：简要说明）`
  - **L5-L7（高级阶段）**：极简解释，仅标注极冷门术语

### 术语覆盖
- L0 先修课：YAML、Markdown、API、token、System Prompt、Progressive Disclosure、Fine-tuning、npx、npm、git clone、marketplace 等
- L1 新手村：description、CLI、CDP、DOM、CSS、HTML、WebP、PNG、JPEG、JSON、Playwright、TypeScript、bun、SDK 等
- L2 学徒工：Prompt、SEO、B2B、CTA、lead、pipeline、SaaS、SDK、Mermaid、SVG 等
- L3 熟练工：Schema、ASO、ROI、KPI、CRO、CRM、A/B test、decision tree、copywriting、Core Web Vitals 等
- L4 专家：Canary、Playwright、MCP、CLI 等
- L5 大师：MCP、Playwright、CLI 等
- L6-L7 宗师/神级：OWASP、STRIDE、TTHW、Iron Law、ngrok、ADR 等

### 翻译规则
- ✅ 翻译：中文段落中的英文教学文字
- ⛔ 保留：代码块、CLI 命令、YAML 示例、Mermaid 图表、Wiki-links、URL
- 🔖 补偿：段落末尾按阶段添加中文术语解释

## [v0.1.0] - 2026-05-03

### Added
- 初始化 Skill0 大师课仓库
- 8 个 Level、168 篇课程笔记
- README 课程体系介绍
- Skill大师课-总纲（学习路线图）
