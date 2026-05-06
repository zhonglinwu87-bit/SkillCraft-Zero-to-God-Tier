---
tags: [教程, Skill, 大师课, Level5, Anthropic, 文档]
created: 2026-05-03
updated: 2026-05-05
course_number: L5-13
prerequisites: ["[[L5-12-文档协作]]"]
next_course: "[[L5-14-PDF文档处理]]"
---

# L5-13：Word文档全能助手 —— 创建、修改、批注一条龙

> 🎯 **学什么**：让 AI 创建专业的 Word 文档——带目录、页眉页脚、表格、自动编号列表。还能编辑已有的 Word 文档，像用 Track Changes（修订标记）一样精准修改，只动该动的字，保留原有格式。核心洞察：(1) .docx 文件本质是一个 ZIP 压缩包装满 XML——这意味着任何操作都可以精准到"字"的粒度；(2) docx-js 的尺寸单位 twips（1 英寸=1440 twips）是从 1980 年代沿用至今的标准——忘记它会让你所有布局错乱；(3) "绝不手动插入项目符号"是血的教训——手动 `•` 只是一个普通字符，没有悬挂缩进、不会自动编号、换行会断开。
> 💡 **难易度**：⭐⭐⭐⭐ | ⏱️ **预计时间**：50 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：你有没有遇到过这种场景——老板丢给你一份 50 页的合同模板，说"把公司名称、金额、日期替换一下"。你打开 Word，发现公司名称出现了 37 次，一个个手动替换要半小时，而且还怕漏改。docx Skill 解决的就是这个问题——创建新文档用 docx-js（JavaScript），编辑已有文档用"解包 XML→修改→重新打包"的三步流程。

> **来源**：Anthropic 官方 Skills 仓库

---

## 2. DOCX 的本质：ZIP 包装的 XML

### 2.1 理解 .docx 的文件结构

很多人以为 .docx 是一个"文件"，但它其实是一个"改名换姓的 ZIP 包"。做个小实验：把任何一个 .docx 文件改名为 .zip，然后解压——你会看到一堆文件夹和 XML 文件。

```
.docx 文件 = ZIP 压缩包，内部结构：

[Content_Types].xml          ← 文件类型声明
_rels/
  .rels                      ← 包级关系映射
docProps/
  app.xml                    ← 应用程序元数据
  core.xml                   ← 文档核心属性（作者、标题）
word/
  document.xml               ← 文档正文（最重要！所有文字内容在这里）
  styles.xml                 ← 样式定义
  settings.xml               ← 文档设置
  header1.xml                ← 页眉
  footer1.xml                ← 页脚
  media/                     ← 图片和其他媒体文件
  _rels/
    document.xml.rels        ← 文档级关系映射
```

这意味着什么？

```
创建新文档 → 用 docx-js（JavaScript）生成结构 → 打包为 .docx
  → 不需要 Word 本身
  → 可编程控制每一个元素（段落、表格、图片、目录）
  → 适合：从零生成报告、信函、发票

编辑已有文档 → 解包 → 编辑 XML → 重新打包
  → 精准到字的修改
  → 保留原有格式和样式
  → 适合：模板填充、批量替换、Track Changes

读取内容 → pandoc 或直接解包读 XML
  → 提取文本、表格、图片
  → 适合：数据迁移、内容分析
```

### 2.2 docx-js 中的单位系统

```
docx-js 中最容易踩坑的是单位系统：

twips（twentieth of a point）：
  → 1 twip = 1/20 点
  → 1 英寸 = 1440 twips
  → 这是微软从 1980 年代沿用至今的标准
  → 为什么用这么别扭的单位？因为 1440 可以被 2、3、4、5、6、8、9、10、
    12、15、16、18、20、24、30……整除——在排版领域非常灵活

关键换算：
  A4 纸张：11906 × 16838 twips（210mm × 297mm）
  US Letter：12240 × 15840 twips
  1 cm：567 twips
  1 英寸：1440 twips

常见错误：
  → 用像素作为宽度 → 在不同 DPI 下显示不一致
  → 用毫米但没转换 → `new NumberOfCentimeters(2)` 而不是直接写 2000
```

---

## 3. 从零创建：docx-js 核心操作

### 3.1 文档结构层次

```
docx-js 的文档结构层次（从大到小）：

Document
  └── Section（节——相当于 Word 中的"节"）
        ├── 页眉/页脚（Header/Footer）
        └── 内容块
              ├── Paragraph（段落）
              │     └── TextRun（文本片段——一个段落内不同格式的文字）
              ├── Table（表格）
              │     ├── TableRow（行）
              │     └── TableCell（单元格）
              └── Image（图片）

关键认知：
  → 一个 Paragraph 可以包含多个 TextRun
  → 每个 TextRun 可以有不同的格式（字体、大小、颜色、加粗）
  → 这就是为什么能在同一段里"部分文字加粗"——它们是不同的 TextRun
```

### 3.2 必须知道的三个规则

```
规则 1：目录（Table of Contents）的两个硬要求
  → 标题必须使用 HeadingLevel（HEADING_1, HEADING_2 等），不能用自定义样式
  → 必须包含 outlineLevel（H1=0, H2=1, H3=2）
  → 忽略后果：目录无法生成或层级全部混乱——Word 的目录生成器
    只识别 HeadingLevel 和 outlineLevel

规则 2：绝不手动插入项目符号
  ❌ 错误做法：new TextRun("• Item")
  → "•" 只是一个普通字符，后果：
    (1) 没有悬挂缩进——文字和符号对不齐
    (2) 换行时符号和文字会断开
    (3) 别人修改时点回车不会自动生成下一个项目符号
  ✅ 正确做法：使用 numbering config + LevelFormat.BULLET
  → docx-js 生成真正的 Word 项目符号，支持自动编号、多级列表

规则 3：表格宽度必须使用 WidthType.DXA（固定宽度）
  → 不能用 PERCENTAGE——Google Docs 等非 Word 平台无法渲染
  → 每个单元格必须同时设置 columnWidths 和 width
  → 表格总宽度 = columnWidths 之和
```

### 3.3 创建文档的最小闭环

```
"先跑通最小闭环"是学技术类 Skill 的最高效方法：

最小闭环 = "三件套"：
  1. 一个标题（HeadingLevel.HEADING_1）
  2. 三个段落（每段一个 TextRun）
  3. 一个目录（TableOfContents）

跑通这个闭环后，你验证了：
  → docx-js 安装正确
  → 文档可以生成
  → 目录可以自动生成
  → 文件可以用 Word 正常打开

从"三件套"扩展到 50 页报告，只是"加更多内容"，不会有架构问题。
```

---

## 4. 编辑已有文档：Track Changes 的精髓

### 4.1 三步编辑流程

```
编辑已有 .docx 文档的三步流程：

Step 1：解包（Unpack）
  python scripts/office/unpack.py document.docx unpacked/
  → 提取 ZIP 内容
  → 美化 XML 格式（可读）
  → 合并相邻的 text runs（同一格式的连续文字合并为一个 run）

Step 2：编辑 XML
  → 编辑 unpacked/word/document.xml
  → 精确控制每个字符的增删改
  → 使用 w:del（删除标记）和 w:ins（插入标记）

Step 3：打包（Pack）
  python scripts/office/pack.py unpacked/ output.docx --original document.docx
  → 验证 XML 格式
  → 自动修复常见问题（durableId 溢出、缺少 xml:space="preserve"）
  → 重新压缩为 .docx
```

### 4.2 Track Changes 的"最小编辑"原则

Track Changes（修订标记）就像"给文档做手术时有全程录像"——每删除一个字都有一条删除标记，每插入一个字都有插入标记。你的同事打开文档后能看到你改了哪里，然后一条条决定"接受"还是"拒绝"。

```
"最小编辑"原则：只标记实际变化的部分

❌ 错误做法：整段替换
  → 把整个段落标记为"删除旧版本+插入新版本"
  → 同事打开后看到一大片红色——"你到底改了什么？"
  → 实际上你只改了 30 → 60

✅ 正确做法：精准标记变化的字节
  改 "30" → "60" 的正确 XML：

  <w:r><w:t>The term is </w:t></w:r>
  <w:del w:id="1" w:author="Claude" w:date="...">
    <w:r><w:delText>30</w:delText></w:r>
  </w:del>
  <w:ins w:id="2" w:author="Claude" w:date="...">
    <w:r><w:t>60</w:t></w:r>
  </w:ins>
  <w:r><w:t> days.</w:t></w:r>

  → 同事看到：只改了一个数字，一眼就能决定"接受"
```

### 4.3 自动修复的边界

```
自动修复（pack.py 内置）能处理的：
  ✓ durableId 溢出 → 重新生成
  ✓ 缺少 xml:space="preserve" → 自动添加
  ✓ 部分 XML 格式小问题

自动修复不能处理的：
  ✗ 格式错误的 XML（标签不匹配）
  ✗ 无效的元素嵌套
  ✗ 缺失的关系引用（.rels 中的引用不存在）

如果 XML 格式错误（如标签不匹配），自动修复无法介入——
需要用 XML 校验器手动检查并修复后再打包。
```

---

## 5. 结构拆解：Word 文档操作 Skill 模板

```markdown
## Word 文档操作型 Skill 模板

### 核心特征
→ 管理对象 = .docx 文件（创建 + 编辑 + 读取）
→ 核心原则 = DOCX=ZIP+XML + 最小编辑 + 先跑通闭环
→ 关键设计 = docx-js 创建 + XML 编辑 + Track Changes

### 通用结构

## File Understanding（文件本质理解）
- .docx = ZIP 压缩包，内含 XML 文件
- 创建：docx-js (JavaScript) 生成结构
- 编辑：解包 → 编辑 XML → 打包
- 单位系统：twips（1 英寸=1440 twips）

## Creation Pattern（创建模式）
- 文档结构层次：Document → Section → Paragraph/Table → TextRun
- 三个硬规则：
  1. 目录需 HeadingLevel + outlineLevel
  2. 项目符号用 numbering config，不用手动 •
  3. 表格宽度用 WidthType.DXA
- 最小闭环：标题+3段落+目录

## Editing Pattern（编辑模式）
- 三步流程：unpack → edit XML → pack
- Track Changes：w:del + w:ins 精准标记
- 最小编辑原则：只标记实际变化的部分
- 自动修复边界：XML 格式错误需手动修复
```

---

## 6. 电商案例：供应商采购合同自动生成

某生鲜电商平台（社区团购模式，日均合作供应商 80+ 家）的采购团队每个季度需要与供应商签署季度采购合同。每份合同 80% 内容相同（品质标准、交付时间窗口、验收条款、结算账期 30 天），20% 不同（供应商名称、采购品类、季度预估采购量、阶梯价格表、保证金金额）。

传统方式人工复制上一份合同 → 逐个查找替换 → 漏改一个数字就出法律风险。60 份合同需要 3 个采购助理加班 2 天。

**方案：模板 + CSV 数据源批量生成**

Step 1：用 docx-js 创建标准合同模板
- 固定条款（品质标准/交付/验收/结算）用固定 TextRun
- 变量用占位符：`{{供应商名称}}`、`{{品类}}`、`{{预估采购量}}`、`{{阶梯价格表}}`、`{{保证金}}`

Step 2：采购助理维护一张 CSV（每行一个供应商的变量数据）

```
供应商名称,品类,预估采购量(吨),价格L1,价格L2,价格L3,保证金(万)
宏源农场,叶菜类,50,3.2,2.8,2.4,5
绿鲜达,根茎类,80,4.5,3.9,3.3,8
阳光果园,水果类,30,8.0,7.0,6.0,5
...
```

Step 3：脚本循环生成——对每行数据替换占位符，导出为 `{供应商名}_Q3采购合同.docx`

**Track Changes 场景验证**：

供应商「宏源农场」收到合同后要求：
- 预估采购量从 50 吨 → 65 吨
- 价格 L1 从 3.2 → 3.5 元/kg（因今年雨季成本上升）

用三步流程精准修订：
1. `unpack.py` 解包合同
2. 编辑 document.xml：只替换 `<w:t>50</w:t>` → `<w:t>65</w:t>` 和 `<w:t>3.2</w:t>` → `<w:t>3.5</w:t>`，使用 `w:del` + `w:ins` 保留 Track Changes 标记
3. `pack.py` 重新打包

供应商打开后看到两个精确的修订标记（原数字被删除线，新数字为蓝色），逐条点「接受」——全流程不到 2 分钟。

**效果**：
- 60 份合同生成时间：从 2 天（3 人 × 16 小时 = 48 人时）→ 3 分钟（脚本跑完）+ 30 分钟（人工抽查 5 份）
- 数据错误率：从「每批合同总有 2-3 处漏改被供应商指出」→ 零错误（变量来自 CSV 系统，不存在漏改）
- 合同修订来回：从平均 3 轮 → 1 轮（Track Changes 精确标记消除了「你到底改了什么」的反复确认）

> 🔑 **启示**：合同批量生成的关键不是「能生成 Word」——而是「变量源与文档模板分离」。CSV 是单一真相来源，采购助理改 CSV 不改文档——杜绝了「上一份合同里改了数字但忘了改三处引用」的错误。Track Changes 让对方信任修订过程——看得见的修改比不可见的修改更容易被接受。

---

## 7. 掌握检验

**Q1**：.docx 文件的本质是什么？
- A) 一个二进制的微软专有格式文件
- B) 一个 ZIP 压缩包，里面全是 XML 文件
- C) 一个加密的数据库
- D) 一个 JSON 格式的文件

**Q2**：docx-js 中页面尺寸的单位"twips"是什么意思？
- A) 1 twip = 1 像素
- B) 1 twip = 1/20 点，1 英寸 = 1440 twips
- C) 1 twip = 1 毫米
- D) 1 twip = 1 厘米

**Q3**：为什么不能手动插入项目符号（`new TextRun("• Item")`）？列出至少 2 个具体后果。正确做法是什么？

**Q4**：编辑已有 .docx 文档的三步骤流程是什么？每一步使用的脚本/方法是什么？

**Q5**：Track Changes（修订标记）的"最小编辑"原则是什么？如果你想改一个数字从 30 到 60，正确的 XML 标记方式应该是怎样的？

**Q6**：以下哪项关于表格的说法是正确的？
- A) 表格宽度应该使用 WidthType.PERCENTAGE 以确保响应式
- B) 每个单元格不需要单独设置宽度，只设置表格总宽度即可
- C) 表格宽度必须使用 WidthType.DXA（固定宽度），因为 Google Docs 不兼容 PERCENTAGE
- D) 表格的 columnWidths 可以任意设置，不需要与总宽度一致

**Q7**：目录（Table of Contents）在 docx-js 中有两个关键要求——标题必须使用什么？必须包含什么？如果忽略了这两个要求会有什么后果？

**Q8**：Track Changes 的自动修复功能能处理什么？不能处理什么？如果 XML 格式错误，你应该怎么办？

---

## 8. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** — .docx 本质上是一个 ZIP 压缩包，里面包含 XML 文件（文档内容、样式、关系映射、图片等）。

**Q2**：**B** — twips（twentieth of a point）是微软从 1980 年代沿用至今的单位。关键换算：A4 = 11906×16838 twips，US Letter = 12240×15840 twips。

**Q3**：参考答案 — 后果：(1) 对齐混乱——手动 `•` 只是一个普通字符，没有项目符号的缩进和悬挂缩进特性；(2) 排版时会断行在 `•` 和文字之间；(3) 别人修改文档时点回车不会自动生成下一个项目符号。正确做法：使用 numbering config + LevelFormat.BULLET。

**Q4**：参考答案 — Step 1: 解包 `python scripts/office/unpack.py document.docx unpacked/`；Step 2: 编辑 XML——编辑 `unpacked/word/` 中的 XML 文件；Step 3: 打包 `python scripts/office/pack.py unpacked/ output.docx --original document.docx`。

**Q5**：参考答案 — "最小编辑"原则：只标记实际变化的部分。改 30 → 60 的正确 XML：用 `w:del` 标记删除 30，`w:ins` 标记插入 60，周围文字不做任何标记。

**Q6**：**C** — 必须使用 WidthType.DXA（固定宽度，单位 twips）。使用 PERCENTAGE 会导致 Google Docs 等非 Word 平台无法正确渲染表格。

**Q7**：参考答案 — (1) 标题必须使用 HeadingLevel（HEADING_1, HEADING_2 等）；(2) 必须包含 outlineLevel（H1=0, H2=1, H3=2）。忽略后果：目录无法生成或目录层级全部混乱。

**Q8**：参考答案 — 自动修复能处理：durableId 溢出（重新生成）、缺少 xml:space="preserve"。不能处理：格式错误的 XML、无效的元素嵌套、缺失的关系引用。如果 XML 格式错误，需要用 XML 校验器手动检查并修复，确保 XML 格式良好后再打包。

（6/8 通过）
</details>

---

## 9. 延伸阅读

- [[L5-12-文档协作|上一课：人机协作写作 —— 你和AI一起写文档，像两个作者合作]]
- [[L5-14-PDF文档处理|下一课：PDF处理大师 —— 提取、合并、拆分、标注全能]]
