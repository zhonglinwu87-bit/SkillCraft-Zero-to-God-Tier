---
tags: [教程, Skill, 大师课, Level5, Anthropic, 文档]
created: 2026-05-03
updated: 2026-05-05
course_number: L5-14
prerequisites: ["[[L5-13-Word文档生成]]"]
next_course: "[[L5-15-PPT幻灯片生成]]"
---

# L5-14：PDF处理大师 —— 提取、合并、拆分、标注全能

> 🎯 **学什么**：把 PDF 当成可以任意拆卸的机器——提取文字表格、合并多个 PDF、拆分页面、添加水印、加密解密、OCR 扫描件识别。一个 Skill 覆盖 PDF 从生到死的全部操作。核心洞察：(1) "没有一把锤子能解决所有问题"——pypdf 适合合并拆分但提取表格不行，pdfplumber 是表格之王但创建 PDF 不行，reportlab 能创建但不能编辑——这个 Skill 的核心价值就是"知道什么任务该用什么工具"；(2) Unicode 上下标字符（如 ₂）不在 ReportLab 内置 14 种标准字体中——H₂O 会变成 HO——这个细节破坏了无数专业 PDF 的可信度。
> 💡 **难易度**：⭐⭐⭐⭐ | ⏱️ **预计时间**：45 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：想象你面前的 PDF 文件就像一叠打印好的纸。过去你只能一页页翻、一个个字抄、用剪刀把多份摊开再订在一起。document-pdf 给你四种"权力"：📖 阅读权（提取文字表格）、✂️ 编辑权（合并拆分旋转）、📝 创作权（从零生成 PDF）、🔒 主权权（加密解密 OCR）。

> **来源**：Anthropic 官方 Skills 仓库

---

## 2. 四类 PDF 操作全景

### 2.1 操作矩阵

```
📖 读取与提取（Read & Extract）
  → 文本提取：pdfplumber（保留布局）/ pypdf（纯文本）/ pdftotext（命令行）
  → 表格提取：pdfplumber → pandas DataFrame（结构化数据导出）
  → 图片提取：pdfimages（poppler-utils 批量提取）
  → 元数据读取：读取作者、创建日期、页数等

✂️ 合并与拆分（Merge & Split）
  → 多文件合并：pypdf（PdfWriter + PdfReader API）
  → 按页拆分：pypdf 逐页控制
  → 旋转页面：pypdf rotate 方法
  → 命令行合并：qpdf 单条命令搞定

📝 创建与编辑（Create & Edit）
  → 从零创建 PDF：reportlab（Canvas 模式 / Platypus 文档模式）
  → 添加水印：pypdf 页面叠加
  → 填写表单：pdf-lib 或 pypdf（见 FORMS.md）

🔒 安全与 OCR（Security & OCR）
  → 加密/解密：pypdf（writer.encrypt / reader.decrypt）
  → OCR 扫描件识别：pytesseract + pdf2image（PDF → 图像 → 文字）
```

### 2.2 工具选择矩阵

```
没有一把锤子能解决所有 PDF 问题：

| 任务 | 最佳工具 | 原因 |
|------|---------|------|
| 合并 PDF | pypdf | 简单直接的 API |
| 拆分 PDF | pypdf | 逐页控制 |
| 提取文本（带布局） | pdfplumber | 保留位置信息 |
| 提取表格 | pdfplumber → pandas | 结构化数据导出 |
| 创建 PDF | reportlab | Canvas/Platypus 双模式 |
| 命令行合并 | qpdf | 单条命令 |
| OCR 扫描件 | pytesseract + pdf2image | 图像→文字 |
| 填写表单 | pdf-lib 或 pypdf | 见 FORMS.md |
| 提取图片 | pdfimages（poppler-utils） | 批量提取 |

核心思想：
  → 不要问"哪个库最好"
  → 要问"这个任务最适合哪个库"
  → Skill 的价值在于封装了"什么任务用什么工具"的知识
```

---

## 3. 扫描件 vs 文本型 PDF：OCR 的工作原理

### 3.1 两种 PDF 的本质区别

```
文本型 PDF：
  → 文字以字符编码存储（"你好"存为 你好）
  → 可以直接用 pdfplumber/pypdf 提取文字
  → 你可以选中、复制、搜索文字
  → 验证方法：用 pdfplumber 提取一页 → 如果不为空，就是文本型

扫描件 PDF：
  → 本质是一张照片（像素矩阵）
  → 计算机看到的只是像素，不知道照片上写的是什么字
  → 无法直接提取文字
  → 验证方法：用 pdfplumber 提取一页 → 如果为空，就是扫描件

OCR 的工作原理：
  PDF（扫描件）→ pdf2image（将每页转为图像）→ pytesseract（将图像中的像素识别为文字）→ 文字输出
```

### 3.2 OCR 实战流程

```python
# OCR 扫描件的标准流程
from pdf2image import convert_from_path
import pytesseract

# Step 1: PDF → 图像
images = convert_from_path('scan.pdf', dpi=300)  # 300 DPI 保证识别精度

# Step 2: 图像 → 文字
for i, image in enumerate(images):
    text = pytesseract.image_to_string(image, lang='chi_sim+eng')  # 中英文混合
    print(f"第 {i+1} 页: {text[:100]}...")
```

---

## 4. ReportLab 创建 PDF 的陷阱

### 4.1 Unicode 上下标问题

```
这是被踩最多的坑——Unicode 上下标字符（如 ₂、¹²³）不在 ReportLab 内置的
14 种标准字体中（Times-Roman, Helvetica, Courier 等）。

❌ 错误做法：
  canvas.drawString(100, 100, "H₂O")
  → ₂ 是 Unicode 字符，不在内置字体中
  → 显示为黑色方块：HO
  → 整份专业 PDF 文档的专业感瞬间崩塌

✅ 正确做法：使用 ReportLab XML 标记
  → 下标：H<sub>2</sub>O
  → 上标：x<super>2</super>
  → ReportLab 的排版引擎将文字缩小并下移/上移
  → 真正的排版效果，不依赖 Unicode 字体支持
```

### 4.2 Canvas vs Platypus：双模式选择

```
ReportLab 提供两种创建模式：

Canvas 模式（自由绘制）：
  → canvas.drawString(x, y, "text")——精确到像素的位置控制
  → canvas.line(x1, y1, x2, y2)——任意线条
  → canvas.drawImage("logo.png", x, y, width, height)——图片
  → 适合：精确布局的票据、证书、名片

Platypus 模式（文档流）：
  → 像写 Word 一样"流式"添加段落、表格、图片
  → 自动分页、自动排版
  → 适合：报告、电子书、多页文档
  
选择标准：
  → 需要精确位置控制（如发票上的每个字段位置）→ Canvas
  → 需要自动排版和分页（如 50 页报告）→ Platypus
```

---

## 5. 加密与安全

### 5.1 正确加密方式

```python
# 正确做法：同时设置用户密码和所有者密码
writer.encrypt("userpassword", "ownerpassword")

# 用户密码（userpassword）：用于打开文档——读者需要输入这个才能查看
# 所有者密码（ownerpassword）：用于管理文档权限——可以禁止打印、复制、修改

# 常见错误：
writer.encrypt("userpass")  # 只设了用户密码，权限管理缺失
```

---

## 6. 结构拆解：PDF 操作 Skill 模板

```markdown
## PDF 操作型 Skill 模板

### 核心特征
→ 管理对象 = PDF 文件的全生命周期操作
→ 核心原则 = 工具选择矩阵（不同任务不同工具）+ 文本型 vs 扫描件判断
→ 关键设计 = 四类操作 + 工具选择指南 + 常见陷阱规避

### 通用结构

## Four Operation Categories（四类操作）
- 📖 读取提取：文本/表格/图片/元数据
- ✂️ 合并拆分：合并/拆分/旋转
- 📝 创建编辑：从零创建/水印/表单填写
- 🔒 安全OCR：加密解密/扫描件识别

## Tool Selection Matrix（工具选择矩阵）
每个任务 → 最佳工具 + 原因
核心原则：没有万能工具，不同任务用不同工具

## Common Pitfalls（常见陷阱）
- Unicode上下标：H₂O 用 <sub> 而不是 Unicode ₂
- Canvas vs Platypus：精确布局 vs 自动排版
- 加密双密码：userpassword + ownerpassword
- OCR条件判断：先试 pdfplumber，空则走 OCR
```

---

## 7. 电商案例：供应商报价单 PDF 批量提取与比价

某跨境电商采购团队（主营小家电出口，通过阿里巴巴 1688 和义乌小商品城采购）每次新品开发时需要向 15-20 家供应商发送询价，收到的报价单 PDF 格式五花八门——有的文本型（从 ERP 系统直接导出）、有的是扫描件（供应商手写表格后拍照转 PDF）、有的加密（供应商设置了编辑保护密码）。

采购专员手工比价流程：逐张打开 PDF → 眼睛找价格 → 复制到 Excel → 发现报价单格式不同（有的含税有的不含税、有的含运费有的不含）→ 重新调整 → 手动比价。20 张报价单比价耗时 3 小时。

**用 document-pdf 实现自动化**：

Step 1：判断 PDF 类型并选择提取策略

```python
import pdfplumber
from pdf2image import convert_from_path
import pytesseract

for pdf_file in quote_pdfs:
    with pdfplumber.open(pdf_file) as pdf:
        text = pdf.pages[0].extract_text()
    if text and text.strip():
        # 文本型：用 pdfplumber 直接提取表格
        with pdfplumber.open(pdf_file) as pdf:
            table = pdf.pages[0].extract_tables()[0]
            result = parse_table(table)
    else:
        # 扫描件：OCR 流程
        images = convert_from_path(pdf_file, dpi=300)
        text = pytesseract.image_to_string(images[0], lang='chi_sim+eng')
        result = parse_ocr_text(text)
```

Step 2：标准化提取字段（处理格式差异）

```python
# 不同供应商的报价单字段名不同，统一映射
field_mapping = {
    "单价": "unit_price", "含税单价": "unit_price_tax",
    "不含税单价": "unit_price_no_tax", "出厂价": "unit_price",
    "最小起订量": "moq", "MOQ": "moq", "起批量": "moq",
    "交货期": "lead_time", "交期": "lead_time",
    "运费": "shipping", "是否含运费": "shipping_included"
}
# 统一输出：SKU | 供应商 | 单价(含税) | MOQ | 交期 | 运费条款
```

Step 3：自动生成比价表（Excel 输出）

```python
# 按 SKU 分组，列出所有供应商报价，高亮最低价
comparison_df = pd.DataFrame(results)
comparison_df = comparison_df.pivot(index='SKU', columns='supplier', values='unit_price_tax')
# 添加行：最低价供应商 + 价差百分比
# 高亮：绿色 = 最低价，红色 = 高于最低价 30%+
comparison_df.to_excel('比价报告_20260505.xlsx')
```

**效果**：
- 比价耗时：从 3 小时 → 8 分钟（脚本 5 分钟 + 人工复核 3 分钟）
- 扫描件 OCR 识别准确率：中英文混合 95%（300 DPI），中文手写体 82%（采购专员补充手写体人工确认）
- 直接节省的采购成本：某款空气炸锅同一 SKU，3 家供应商报价分别为 ¥82/¥89/¥108（价差 32%），人工比价时被遗漏的第三家成为谈判筹码后，最终拿下了 ¥79 的批量价——仅这一个 SKU，月省 ¥6,000（月采购量 2000 台）

> 🔑 **启示**：比价不是「找最低价」——是「统一口径后找最低价」。「含税」「不含税」「含运费」「不含运费」的统一映射是这个自动化最有价值的 30 行代码——没有这一步，比价结果全是错的。另外，OCR 手写体准确率 82% 意味着必须保留人工复核环节——自动化不是零人参与，是让人只看那 18%。

---

## 8. 掌握检验

**Q1**：以下哪个工具最适合提取 PDF 中的表格数据并导出为结构化格式？
- A) pypdf
- B) pdfplumber
- C) qpdf
- D) reportlab

**Q2**：OCR 扫描件处理的正确流程是什么？
- A) 图片 → PDF → 文字
- B) PDF → 图像 → OCR 识别 → 文字
- C) 文字 → OCR → PDF
- D) 图像 → 文字 → PDF

**Q3**：在 ReportLab 中为什么不能使用 Unicode 上下标字符（如 Unicode 编码的 ₂）？会出现什么具体后果？正确做法是什么？

**Q4**：document-pdf 的"工具选择矩阵"体现了什么核心思想？为什么没有一个"万能工具"能覆盖所有 PDF 操作？

**Q5**：你现在有 5 个银行的月度流水 PDF，需要合并为一个文件并按客户分别加密后发送。请写出完成这个任务需要的工具组合和大致流程。

**Q6**：以下哪种加密方式是正确的？
- A) `writer.encrypt("userpass")`——只设置用户密码
- B) `writer.encrypt("userpassword", "ownerpassword")`——分别设置用户密码和所有者密码
- C) `writer.encrypt("")`——不设密码
- D) `writer.lock()`——锁定文档

**Q7**：pdfplumber 相比 pypdf 在文本提取上有什么优势？在什么场景下你会选择 pdfplumber 而不是 pypdf？

**Q8**：document-pdf 这个 Skill 的核心价值是什么——如果你已经知道 Python 有 pypdf、pdfplumber、reportlab 这些库，为什么还需要一个 Skill？

---

## 9. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** — pdfplumber 是表格提取之王。它与 pandas 结合可以将表格直接导出为 DataFrame。

**Q2**：**B** — PDF → 图像（pdf2image）→ OCR 识别（pytesseract）→ 文字输出。

**Q3**：参考答案 — Unicode 上下标字符（如 ₂）不在 ReportLab 内置的 14 种标准字体中，会显示为黑色方块——H₂O 变成 HO。正确做法：使用 ReportLab XML 标记——`H<sub>2</sub>O` 和 `x<super>2</super>`。

**Q4**：参考答案 — 核心思想是"没有一把锤子能解决所有问题"——每个工具各有所长。Skill 的价值就在于它"知道什么任务该用什么工具"——你只需要描述任务，AI 自动选择最佳工具组合。

**Q5**：参考答案 — (1) 合并：pypdf（PdfWriter + PdfReader）遍历 5 个文件；(2) 加密：`writer.encrypt("客户专属密码", "管理员密码")`；(3) 自动化：循环为每个客户分别执行合并+加密。

**Q6**：**B** — `writer.encrypt("userpassword", "ownerpassword")`——用户密码用于打开文档，所有者密码用于管理文档权限。

**Q7**：参考答案 — pdfplumber 保留了每个字符在页面上的位置坐标，这让它可以精准识别表格的列边界、合并单元格。选择 pdfplumber：需要提取表格数据或保留文字布局信息；选择 pypdf：只需要纯文本内容、合并拆分页面等操作。

**Q8**：参考答案 — Skill 的价值不在库本身，而在于：(1) 封装了"什么任务用什么工具"的知识；(2) 提供了标准化的操作模式和错误处理；(3) 将 PDF 操作能力变成了自然语言接口。

（6/8 通过）
</details>

---

## 10. 延伸阅读

- [[L5-13-Word文档生成|上一课：Word文档全能助手 —— 创建、修改、批注一条龙]]
- [[L5-15-PPT幻灯片生成|下一课：PPT幻灯片生成 —— 精美演示文稿一句话搞定]]
