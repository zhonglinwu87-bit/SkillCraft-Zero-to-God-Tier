---
tags: [教程, Skill, 大师课, Level1, baoyu, 工具型]
created: 2026-05-03
updated: 2026-05-03
course_number: L1-02
prerequisites: ["[[L1-01-智能图片压缩]]"]
next_course: "[[L1-03-Markdown格式化排版]]"
source_skill: "[[../../../原始资料/Skills/baoyu-skills/skills/baoyu-url-to-markdown/SKILL.md]]"
---

# L1-02：网页转Markdown —— 把任何网页变成干净的文本

> 🎯 **学什么**：学会封装复杂 CLI（CLI/命令行接口：在终端里通过打字操作电脑的方式，比图形界面更快） 工具的 Skill——含偏好系统、首次设置、质量闸门。
> 💡 **难易度**：⭐⭐ | ⏱️ **预计时间**：50 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**： 你有没有遇到过想保存一篇网页，结果复制下来全是广告、导航栏、推荐阅读？这个Skill就像"网页榨汁机"——把网页扔进去，出来的是纯纯的干净文字（Markdown（Markdown：用简单符号如 # = 标题来排版的纯文本格式，是写文档的通用语言）格式），广告和边栏全被过滤掉了。
> 💡 **白话小课堂**： 不同的AI平台问问题的方式不一样——就像微信、QQ、钉钉各有各的表情包格式。为了让你写的Skill在所有平台都能工作，你要先说清楚"用什么方式问用户问题"。
> ⚠️ **避坑指南**： 新手经常把脚本路径写死成 `/home/user/skills/xxx/script.ts`，换台电脑就坏了。一定要用 `{baseDir}` 占位符——Skill装在哪里，路径就指向哪里。
> 🔑 **核心秘诀**： BLOCKING 的意思是"必须停下来等用户选"。AI 擅自替用户选择"不下载图片"可能导致数据丢失——有些决策必须由人来做。
> 翻译：关键规则——将无头浏览器抓取结果视为"暂定版本"。每次无头模式运行后，必须检查保存的Markdown内容质量（参考 references/quality-gate.md 完整清单），因为某些网站可能悄悄返回低质量内容而不报错。

**这是 Skill 工程化的标志**。简单 Skill 只管"做成没做成"，工程化 Skill 还管"做得好不好"。

**质量闸门的三个层次**：

| 层次 | 检查什么 | 例子 |
|------|---------|------|
| 完成检查 | 脚本有没有报错？ | 进程 exit code = 0 |
| 内容检查 | 输出内容完整吗？ | 页面正文是否 > 50 词 |
| 质量检查 | 输出质量好吗？ | 有没有被 headless 模式截断的懒加载内容 |

***

## 3. 结构拆解：带交互的工具型 Skill 模板

从 L1-01 模板进化而来，新增部分用 `★` 标注：

```markdown
---
name: [skill-name]
description: [详细描述 + 触发词]
---

# [功能名]

[一句话描述]

## User Input Tools        ★ 新增：跨平台用户交互
[声明优先使用什么工具问用户问题]

## CLI Setup               ★ 新增：运行时依赖管理
[1. 确定路径 2. 检测运行时 3. 安装依赖 4. 占位符替换]

## Preferences (EXTEND.md) ★ 增强：首次设置检查
[如果配置不存在 → BLOCKING 询问用户 → 写入配置 → 继续]

## Usage
[命令行用法]

## Options
[参数表]

## Agent Quality Gate      ★ 新增：质量保障
[运行后必须检查输出质量]

## Examples
[示例]
```

***

## 4. 深度思考

### 4.1 EXTEND.md vs 环境变量：什么时候用哪个？

```
EXTEND.md（偏好文件）：
  → 适用于持久化、可共享的用户偏好
  → 可以版本控制（项目级配置）
  → 他人可以阅读和修改

环境变量（如 BAOYU_CHROME_PROFILE_DIR）：
  → 适用于敏感的运行时配置
  → 无法版本控制
  → 每个机器可能不同（如 Chrome 路径）
```

**这个 Skill 两者都用**：EXTEND.md 管用户偏好，环境变量管系统配置。

### 4.2 为什么输出路径要 Agent 自己构造？

```markdown
## Output Path Generation

The agent must construct the output file path — baoyu-fetch does not 
auto-generate paths.

Algorithm:
1. base directory from EXTEND.md
2. Extract domain from URL
3. Generate slug from URL path
4. Construct: {base_dir}/{domain}/{slug}/{slug}.md
5. Conflict: append timestamp
```

**这是「关注点分离」的体现**：
- CLI 负责「抓取+转换」，不关心文件组织
- Skill 负责「文件组织策略」，不关心抓取细节
- 让 CLI 简单可测试，让 Skill 灵活可配置

***

## 5. 电商案例：1688供应商商品页批量转Markdown——搭建选品数据库

> 🛒 **实战案例**：某淘宝店主做家居收纳品类，每周选品时要浏览1688上50+供应商的商品页——产品参数、起批量、价格阶梯、材质描述。过去是逐个页面Ctrl+C/Ctrl+V到Excel，每周花8小时，而且复制的内容图片丢失、格式错乱，很难横向对比。
>
> **用 baoyu-url-to-markdown 搭建选品数据库**：
> 1. 配置 EXTEND.md：输出目录 `选品库/`，图片策略 "always download"（供应商随时可能改价改图，必须本地存档）
> 2. 每周将50+个1688商品链接批量转换，按"供应商+品类"自动组织文件夹结构：`选品库/收纳盒/义乌XX塑业/产品名.md`
> 3. 图片自动下载到本地，保留供应商原图的尺寸/材质/细节特写
> 4. 转换后的Markdown保留完整价格表（5件/50件/200件阶梯价）和规格参数
> 5. 结合 L1-03 Markdown格式化 Skill，进一步优化排版，统一规格表格格式
>
> **效果**：
> - 选品信息收集从每周8小时 → 40分钟（减少92%）
> - 50个供应商的产品可全文搜索——搜"PP材质+带盖"秒出全部匹配项
> - 横评时不再来回切换浏览器标签——Obsidian中并排打开3个Markdown页面直接对比价格和起批量
> - 图片本地存档——供应商下架或改价后仍有原始数据可回溯
> - 3个月积累了600+供应商产品档案，形成可复用选品数据库
>
> > 🔑 **启示**：选品的核心不是"看得多"，而是"比得快"。把1688的产品页变成结构化本地文件，你就能在Obsidian里同时对比15个供应商的价格、材质、起批量——而不是开15个浏览器标签来回切。网页会下架，本地文件永远在。

***

## 6. 常见错误

| ❌ 错误 | ✅ 正确 |
|---------|--------|
| 跳过首次设置，静默创建默认配置 | BLOCKING 标注，强制等待用户选择 |
| 不检查 CLI 输出质量 | 每次运行后检查内容完整性 |
| 参数表太简单，漏掉重要选项 | 覆盖核心场景（timeout、interaction、debug） |
| 不写 Troubleshooting | 列出常见问题和解决方案 |

***

## 7. 掌握检验

**Q1**：CLI Setup 的四步初始化流程中，哪一步是为了解决"脚本换台电脑就跑不了"的问题？
- A) 步骤 1：确定 {baseDir} 目录路径
- B) 步骤 2：检测 bun/npx 运行时
- C) 步骤 3：安装 node_modules 依赖
- D) 步骤 4：用实际路径替换文档中的占位符

**Q2**：以下哪个选项正确描述了 Quality Gate 的三个层次（从低到高）？
- A) 完成检查（exit code）→ 质量检查（懒加载内容）→ 内容检查（字数）
- B) 内容检查（字数）→ 完成检查（exit code）→ 质量检查（懒加载内容）
- C) 完成检查（exit code）→ 内容检查（内容完整性）→ 质量检查（是否高质量）
- D) 质量检查（是否高质量）→ 内容检查（完整性）→ 完成检查（exit code）

**Q3**：为什么首次设置标注为 ⛔ BLOCKING？如果 AI 静默选择"不下载图片"会导致什么后果？

**Q4**：EXTEND.md 和环境变量分别适合存放什么类型的配置？baoyu-url-to-markdown 如何同时使用两者？

**Q5**：为什么输出路径要由 Agent 自己构造而不是由 CLI 自动生成？这体现了什么设计原则？

**Q6**：与 L1-01（简单工具型）相比，L1-02 新增了哪三个关键模块？每一个模块的名字和作用是什么？

**Q7**：某电商运营官想用 baoyu-url-to-markdown 搭建竞品情报库。从配置 EXTEND.md 到最终形成可搜索的竞品档案，完整的操作流程包括哪几个关键步骤？每个步骤解决什么问题？

***

## 8. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**D** — 步骤 4 将 SKILL.md 中的 `${READER}` 等占位符替换为实际路径值，确保 Skill 安装在任何路径下都能正常运行。步骤 1 确定了 baseDir 但如果不做替换，文档中的路径仍然是占位符文本。步骤 2-3 处理的是运行环境问题而非路径可移植性问题。

**Q2**：**C** — 三个层次从低到高依次为：完成检查（看脚本有没有报错，只看 exit code）、内容检查（看输出内容是否完整，如正文是否 > 50 词）、质量检查（看输出质量是否好，如 headless 模式下懒加载内容是否被截断）。A 和 B 的顺序都有误。

**Q3**：BLOCKING 的作用是强制 AI 停下来等待用户明确选择，因为"图片下载策略"这类决策是不可逆的——如果 AI 静默选择了"不下载图片"然后运行抓取，原始网页可能已经发生变化或下线，用户永远失去了下载图片的机会。这种决策必须由人来做出，AI 不能越俎代庖。

**Q4**：EXTEND.md 适用于持久化、可共享的用户偏好（如默认输出目录、图片策略），可纳入版本控制；环境变量适用于敏感或机器特定的运行时配置（如 Chrome 配置文件路径），每台机器可能不同。baoyu-url-to-markdown 两者都用——EXTEND.md 管理输出目录和图片策略，环境变量管理 Chrome 路径等系统级配置。

**Q5**：这体现了「关注点分离」原则。CLI（baoyu-fetch）只负责"抓取 + 转换"，不关心文件怎么组织；Skill 层负责"文件组织策略"，不关心抓取细节。这样 CLI 可以保持简单、易于测试，而文件组织策略可以根据需求灵活调整，两者互不干扰。

**Q6**：与 L1-01 相比，L1-02 新增了三个关键模块：1) **User Input Tools**——声明跨平台用户交互工具，确保 Skill 在不同 AI 平台都能正常工作；2) **CLI Setup**——运行时依赖管理（路径确定→运行时检测→依赖安装→占位符替换），解决脚本可移植性问题；3) **Quality Gate（质量闸门）**——运行后自动检查输出质量，弥补简单 exit code 检查无法发现的内容质量问题。

**Q7**：完整流程包括五个关键步骤：1) **配置 EXTEND.md**——设定输出目录（如 `竞品情报/`）和图片策略（always download），让后续操作自动化；2) **批量 URL 收集**——每周收集 20-30 个竞品页面 URL，统一提交；3) **自动抓取与转换**——Skill 按域名+slug 自动组织文件夹结构，图片下载到本地；4) **全文搜索启用**——所有 Markdown 文件形成可搜索的文本库，可按品牌、成分、卖点等关键词检索；5) **时间线追溯**——持续积累形成竞品变化档案，每次活动、每次新品发布都有时间戳。核心价值：把信息收集从每周 6 小时降到 30 分钟（减少 92%），让运营官的时间真正用在分析而非收集上。

（6/7 通过）

</details>

***

## 9. 手写模仿

**不看源码，写一个「GitHub README 下载器」Skill 的 SKILL.md。**

要求：
- name: `readme-downloader`
- 功能：输入 GitHub repo URL，下载其 README.md → 保存到本地
- 首次设置：询问默认输出目录、是否同时下载截图
- 质量闸门：检查下载的内容是否是有效的 Markdown

<details>
<summary>点击查看参考写法</summary>

````markdown
---
name: readme-downloader
description: Downloads README from any GitHub repository and saves as local markdown. Use when user asks to "download readme", "save readme", "get github readme", "clone readme", "下载README".
---

# README Downloader

Downloads README.md from GitHub repos via GitHub API.

## User Input Tools

Prefer built-in user-input tools. Batch questions when possible.

## CLI Setup

脚本路径：`scripts/fetch-readme.ts`（`{baseDir}` = SKILL.md 所在目录，装到哪里都能跑）

1. 选择运行时：检测系统有 bun 还是 npx → 填入 `${BUN}` 占位符
2. 装依赖（如果需要）：`{baseDir}/scripts` 目录下执行 `npm install`

> 上面第 2 步的原始写法是 `${BUN} install --cwd {baseDir}/scripts`，理解为「在脚本目录里装依赖」就行。

## Preferences (EXTEND.md)

Check in priority: project > XDG > user home.

> 查找优先级：项目级 > 系统配置级 > 用户级。跟 L1-01 学的三级查找机制一样。

### First-Time Setup ⛔ BLOCKING

When EXTEND.md not found, ask:
- Q1: Default output directory?
- Q2: Download repo screenshots (social preview)?

Write answers → confirm → continue.

## Usage

```bash
# ${BUN} = 运行时选择器（bun/npx）  |  {baseDir} = Skill 所在目录
${BUN} {baseDir}/scripts/fetch-readme.ts <github-url> [options]
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `<url>` | GitHub repo URL | Required |
| `--output` | Output path | {repo}-README.md |
| `--lang` | Language (if multiple READMEs) | en |
| `--screenshots` | Download social preview image | false |

## Quality Gate

After download, verify:
- [ ] File > 0 bytes
- [ ] Contains markdown headers (#)
- [ ] Not a 404 page

## Examples

```bash
${BUN} {baseDir}/scripts/fetch-readme.ts https://github.com/facebook/react
${BUN} {baseDir}/scripts/fetch-readme.ts https://github.com/torvalds/linux --output linux-intro.md
```
````

> 这是一个完整的Skill SKILL.md示例（readme-downloader），遵循了url-to-markdown的工具型Skill模板结构：YAML头部声明、用户输入工具声明、CLI依赖管理、偏好系统（EXTEND.md+首次设置）、参数表、质量闸门和示例。

</details>

***

## 10. 延伸阅读

- [[L1-03-Markdown格式化排版|下一课：Markdown格式化]] — 6 步流水线，从分析到排版
- [[L1-01-智能图片压缩|上一课：智能图片压缩]] — 最简单工具型 Skill
