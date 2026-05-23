---
tags: [教程, Skill, 大师课, Level5, Anthropic, 数据]
created: 2026-05-03
updated: 2026-05-05
course_number: L5-16
prerequisites: ["[[L5-15-PPT幻灯片生成]]"]
next_course: "[[L5-17-ClaudeAPI最佳实践]]"
---

# L5-16：Excel表格魔法师 —— 公式、图表、数据透视一键完成

> 🎯 **学什么**：像华尔街分析师一样创建专业的 Excel 模型——不是把数据"硬塞进格子"，而是用 Excel 原生公式让表格自动计算，遵循金融行业四色标准，生成零错误的专业电子表格。核心洞察：(1) "公式而非硬编码"是 xlsx Skill 最重要的原则——电子表格的生命周期不是"建完就完了"，它会被打开、修改、更新几十次，硬编码的数字一旦源数据变化就变成了谎言；(2) 金融模型四色编码（蓝=输入/黑=公式/绿=跨表/红=外部）是华尔街 30 年的行业标准——不是审美选择，是防止灾难性错误的信号系统；(3) "零公式错误交付"是一个硬核标准——任何一个 #REF! 都会让接收方对你的专业性产生怀疑。
> 💡 **难易度**：⭐⭐⭐⭐ | ⏱️ **预计时间**：50 分钟

***

## 1. 课程概览

> 💡 **白话小课堂**：很多人用 Excel 是"手动计算器"——在算好数字后手动敲进格子里（"销售额 5000 元"）。但这就像开一辆自动挡汽车却永远自己用手推——Excel 真正的威力是"公式"。你告诉它"B10 那一格永远等于 B2 到 B9 的总和"，以后你改了 B2-B9 的数据，B10 自动更新。这就是 xlsx Skill 的核心——让电子表格"活"起来。

> **来源**：Anthropic 官方 Skills 仓库

---

## 2. 公式而非硬编码：电子表格的生死线

### 2.1 为什么公式是原则而非偏好

```
❌ 错误做法：在 Python 中计算，硬编码结果
  total = df['Sales'].sum()
  sheet['B10'] = total  # 硬编码 5000 ❌
  
  问题链：
    1. 今天 B2-B9 的销售额加起来是 5000
    2. 明天运营更新了 B5 从 800 → 950
    3. B10 仍然是 5000——变成了一个谎言
    4. 接收方不知道这是一个"过时的计算结果"
    5. 此后所有引用 B10 的公式都在基于错误数据运算

✅ 正确做法：使用 Excel 公式，让电子表格自行计算
  sheet['B10'] = '=SUM(B2:B9)'  # Excel 公式 ✅
  
  好处链：
    1. B5 从 800 → 950 → B10 自动变为 5150
    2. 接收方看到 B10 知道这是一个"实时计算结果"
    3. 审计可以追踪公式逻辑：点击 B10 → 看到高亮的 B2:B9
    4. 电子表格的生命周期被充分尊重——它会活很久

这适用于所有计算：
  总计 = =SUM(range)
  百分比 = =A1/B1
  增长率 = =(B1-A1)/A1
  加权平均 = =SUMPRODUCT(weights, values)/SUM(weights)
```

### 2.2 recalc.py 的必要性

```
openpyxl 的一个关键限制：
  → openpyxl 创建文件时，公式只存为字符串，计算值是空的
  → 如果直接发给客户，客户打开后所有公式单元格显示为空白或 0
  → Excel 打开时需要一定条件才触发重算，不保证完整

recalc.py 的解决方案：
  → 使用 LibreOffice 自动重算所有公式
  → 将计算值写入每个公式单元格
  → 确保交付时每个公式单元格都有正确的计算值

交付前必须运行 recalc.py——这是"零公式错误交付"的前提。
```

---

## 3. 金融模型四色编码

### 3.1 华尔街 30 年的信号系统

```
四色编码不是审美选择——是防止灾难性错误的信号系统：

🔵 蓝色文字 = 硬编码输入（用户可修改的假设值）
  → 如：增长率假设 15%、利润率 22%、贷款利率 4.5%
  → 含义：「这些数字是手动输入的，不是公式算出来的」
  → 安全性：修改蓝色数字时必须谨慎——它们是整个模型的"源假设"
  → 类比：汽车的方向盘——由你控制

⚫ 黑色文字 = 公式和计算结果
  → 如：=SUM(B2:B9)、=B10*C10
  → 含义：「这些是自动计算的，源数据变了它会自动更新」
  → 安全性：不要手动修改黑色数字——你改了一个公式，相关公式全错
  → 类比：汽车的车速表——显示实时速度，你不能手动调

🟢 绿色文字 = 跨工作表链接
  → 如：='Sheet2'!B10
  → 含义：「这个数字是从同文件的其他工作表引用来的」
  → 安全性：修改源工作表会影响这个格——两者有依赖关系

🔴 红色文字 = 外部文件链接
  → 如：='[外部文件.xlsx]Sheet1'!$A$1
  → 含义：「这个数字来自另一个 Excel 文件」
  → 安全性：如果外部文件被移动/删除/重命名——链接断裂 → #REF!

🟡 黄色背景（叠加在以上颜色上）= 关键假设
  → 含义：「这个数字很重要，改之前先想清楚」
  → 典型：退货率假设、汇率假设、税率
```

### 3.2 四色编码在代码中的实现

```python
from openpyxl.styles import Font, PatternFill

# 蓝色文字 = 硬编码输入
blue_input = sheet['B5']
blue_input.value = 0.15  # 增长率 15%
blue_input.font = Font(color='0000FF')  # 蓝色
blue_input.fill = PatternFill(start_color='FFFF00', end_color='FFFF00', fill_type='solid')  # 黄色背景（关键假设）

# 黑色文字 = 公式结果
formula_cell = sheet['B10']
formula_cell.value = '=SUM(B2:B9)'  # Excel 公式
formula_cell.font = Font(color='000000')  # 黑色（默认）
```

---

## 4. 数据处理的五个常见陷阱

### 4.1 陷阱清单

```
陷阱 1：年份格式化为数字
  → 输入 2024 → Excel 自动格式化为 "2,024"
  → 看起来像两千零二十四，而不是年份二零二四
  → 修复：年份永远是文本字符串 "2024"，不是数字

陷阱 2：比例格式错误
  → 输入 0.25（期望显示 25%）→ Excel 可能显示 0.25 或 25%
  → 修复：显式设置格式为 '0.00%'

陷阱 3：data_only=True 的写入风险
  → data_only=True 模式读取的是缓存值，不是公式
  → 如果在此模式下修改并保存 → 所有公式丢失，只剩值
  → 规则：只读用 data_only=True，写回用默认模式

陷阱 4：NaN 处理
  → 源数据有空值 → 未经 pd.notna() 检查写入 Excel
  → 公式遇到 NaN → 返回 #VALUE! 错误
  → 修复：写入前用 df.fillna() 或检查 pd.notna()

陷阱 5：零除检查
  → 除法公式：分母为 0 → #DIV/0!
  → 如：某天销售额恰好为 0，利润率公式 =利润/销售额 爆炸
  → 修复：=IF(B1=0, "-", A1/B1)
```

### 4.2 列映射的陷阱

```
Excel 列字母 vs 数字的映射容易出错：

  → 第 64 列 = 列 BL（不是 BK）
  → 超过 26 列（Z 之后）开始两位数：AA, AB, AC...AZ, BA, BB...
  → 如果列映射错了——整个列的数据全部引用到错误位置

openpyxl 提供了安全的列转换：
  from openpyxl.utils import get_column_letter
  get_column_letter(64)  # → 'BL'
```

---

## 5. 公式验证清单

```
交付前必须检查的 7 项：

1. 公式覆盖检查
   → 所有合计、百分比、增长率都用公式？没有硬编码数字？
   → 检查方法：点击可疑单元格 → 看编辑栏是公式还是固定值

2. 四色编码合规
   → 蓝色输入、黑色公式、绿色跨表、红色外部——全部一致？
   → 检查方法：视觉扫描——任何不一致的颜色都是信号

3. 公式一致性
   → 同一列的所有行用同一个公式？（不是 B2=B3+B4, B3=B5+B6）
   → 检查方法：选中一列 → 不一致的公式会在状态栏报出

4. 零除保护
   → 所有除法公式有 IF 保护？
   → 检查方法：搜索 "/" → 确认每个除法旁边有 IF 兜底

5. NaN 处理
   → pd.notna() 检查过所有数据列？
   → 检查方法：df.isnull().sum() 确认没有 NaN

6. 列映射验证
   → 用 get_column_letter() 而非手动计算？
   → 检查方法：抽查边界列（Z 附近、BL 附近）

7. 运行 recalc.py
   → 交付前最后一步
   → 检查方法：recalc 完成后检查文件大小 > 原始文件（因为多了计算值）
```

---

## 6. 结构拆解：Excel 操作 Skill 模板

```markdown
## Excel 操作型 Skill 模板

### 核心特征
→ 管理对象 = Excel 电子表格（.xlsx/.xlsm/.csv/.tsv）
→ 核心原则 = 公式而非硬编码 + 四色编码 + 零错误交付
→ 关键设计 = openpyxl 公式引擎 + 金融行业标准 + 7 项验证清单

### 通用结构

## Core Principle（核心原则）
- 永远用 Excel 公式，不用硬编码
- 电子表格的寿命 > 创建时间——公式确保它的"活"性
- recalc.py 确保交付时公式有值

## Four-Color Code（金融四色编码）
- 🔵 蓝 = 硬编码输入（可修改假设）
- ⚫ 黑 = 公式和计算结果
- 🟢 绿 = 跨工作表链接
- 🔴 红 = 外部文件链接
- 🟡 黄底 = 关键假设（叠加色）

## Five Common Pitfalls（五个常见陷阱）
1. 年份 → 文本字符串
2. 比例 → 显式格式
3. data_only=True → 只读不写
4. NaN → 写入前填充
5. 零除 → IF 保护

## Seven-Point Validation Checklist（7 项验证清单）
公式覆盖/四色合规/一致性/零除保护/NaN/列映射/recalc
```

---

## 7. 电商案例：选品分析表格——竞品价格/销量/毛利三维对比

某抖音电商运营团队（主营家居日用，店铺 500 个 SKU）计划从 1688 拓展 30 个新品。运营需要做一个选品分析表：对 30 个候选品逐一评估——1688 采购价、同类抖音店铺售价、预估月销量、预估毛利、退货率风险。手工作表耗时 4 小时且极易漏算。

用 xlsx Skill 创建一个自动化的选品分析工作簿。

**用 openpyxl 创建（公式驱动 + 四色编码）**：

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
ws = wb.active
ws.title = "选品分析"

# 表头
headers = ["候选品名", "1688采购价", "抖音同款售价", "预估月销", "运费/件",
           "平台佣金率", "退货率(预估)", "毛利/件", "毛利率", "月毛利", 
           "预估月净利", "入选建议"]
for i, h in enumerate(headers, 1):
    ws.cell(row=1, column=i, value=h).font = Font(bold=True, size=11)

# 数据行（第 2-31 行，30 个候选品）
for row in range(2, 32):
    # 🔵 蓝色 = 手动输入的假设数据
    blue_font = Font(color='0000FF', size=11)
    ws.cell(row=row, column=1, value=f"候选品{row-1}").font = blue_font  # 品名
    ws.cell(row=row, column=2).font = blue_font   # 1688采购价（手动填）
    ws.cell(row=row, column=3).font = blue_font   # 抖音售价（手动调研）
    ws.cell(row=row, column=4).font = blue_font   # 预估月销（手动估）
    ws.cell(row=row, column=5, value=6).font = blue_font  # 运费
    ws.cell(row=row, column=6, value=0.05).font = blue_font  # 佣金率5%
    # 🟡 黄底 = 关键假设
    ws.cell(row=row, column=6).fill = PatternFill(start_color='FFFF00', end_color='FFFF00', fill_type='solid')
    ws.cell(row=row, column=7, value=0.08).font = blue_font   # 退货率预估
    ws.cell(row=row, column=7).fill = PatternFill(start_color='FFFF00', end_color='FFFF00', fill_type='solid')

    # ⚫ 黑色 = 公式自动计算
    black_font = Font(color='000000', size=11)
    # 毛利/件 = 售价 - 采购价 - 运费 - 售价×佣金率
    ws.cell(row=row, column=8).value = f'=C{row}-B{row}-E{row}-C{row}*F{row}'
    ws.cell(row=row, column=8).font = black_font
    # 毛利率（含零除保护）
    ws.cell(row=row, column=9).value = f'=IF(C{row}=0,"-",H{row}/C{row})'
    ws.cell(row=row, column=9).number_format = '0.00%'
    ws.cell(row=row, column=9).font = black_font
    # 月毛利 = 毛利/件 × 预估月销
    ws.cell(row=row, column=10).value = f'=H{row}*D{row}'
    ws.cell(row=row, column=10).font = black_font
    # 预估月净利 = 月毛利 × (1 - 退货率)
    ws.cell(row=row, column=11).value = f'=J{row}*(1-G{row})'
    ws.cell(row=row, column=11).font = black_font
    # 入选建议（条件公式）
    ws.cell(row=row, column=12).value = f'=IF(K{row}>=5000,"★ 入选",IF(K{row}>=2000,"△ 关注","✗ 放弃"))'
    ws.cell(row=row, column=12).font = black_font

# 汇总行
ws.cell(row=33, column=1, value="汇总").font = Font(bold=True)
ws.cell(row=33, column=12).value = '=COUNTIF(L2:L31,"★ 入选")&"个入选/"&COUNTA(L2:L31)&"个候选"'
```

**交付前验证**：
1. `recalc.py` 重算 → 所有公式单元格有计算值
2. 零除保护：毛利率公式 `=IF(C=0,"-",H/C)` 防止售价为空时 #DIV/0!
3. 四色编码检查：蓝色输入（7列）+ 黑色公式（5列），黄色关键假设（佣金率 + 退货率）

**效果**：
- 选品分析耗时：从 4 小时（手动输入 + 手工计算 + 格式调整）→ 20 分钟（调研填 30 行 × 3 个蓝色数字 + 公式自动完成全部运算）
- 漏算错误：从「上个月有 2 个品上架后才发现毛利算错」→ 零错误（公式统一计算，不存在手算笔误）
- 入选标准透明：团队一致使用「月净利 ≥ 5000 入选 / ≥ 2000 关注」的量化标准，不再凭「感觉」选品

---

## 8. 掌握检验

**Q1**：document-xlsx 的"公式而非硬编码"原则中，以下哪个是正确的做法？
- A) `total = df['Sales'].sum()` 然后 `sheet['B10'] = total`
- B) `sheet['B10'] = '=SUM(B2:B9)'`
- C) `sheet['B10'] = 5000`
- D) `sheet['B10'] = '=5000'`

**Q2**：金融模型四色编码中，蓝色文字代表什么？
- A) 公式和计算结果
- B) 硬编码输入——用户可以修改的假设值
- C) 跨工作表链接
- D) 外部文件链接

**Q3**：recalc.py 脚本的作用是什么？为什么 openpyxl 创建的文件必须运行 recalc.py？

**Q4**：为什么年份必须格式化为文本字符串（"2024"）而非数字（2,024）？

**Q5**：`data_only=True` 在 openpyxl 中的风险是什么？在什么场景下使用是安全的，什么场景下是危险的？

**Q6**：公式验证清单中包含 7 个检查项。请挑选其中 3 个并解释为什么每个都重要。

**Q7**：你为店铺建了一个"单品利润测算模型"。请写出至少 3 个应该使用公式而不是硬编码的计算，以及每个公式引用的假设单元格应该使用什么颜色。

**Q8**："零公式错误交付"为什么是一个硬核标准？自动检查和人工检查各自能发现什么问题？

---

## 9. 答案

<details>
<summary>点击查看答案</summary>

**Q1**：**B** — `sheet['B10'] = '=SUM(B2:B9)'`。正确做法是使用 Excel 原生公式。

**Q2**：**B** — 蓝色文字 = 硬编码输入，即用户可以修改的假设值。这是华尔街 30 年的行业标准。

**Q3**：参考答案 — recalc.py 使用 LibreOffice 自动重算所有公式并将计算值写入单元格。openpyxl 创建的公式只存为字符串，计算值是空的——直接发给客户公式单元格显示空白或 0。

**Q4**：参考答案 — 如果把"2024"当作数字输入，Excel 自动格式化为"2,024"——看起来像两千零二十四，而不是年份二零二四。规则：年份永远是文本字符串。

**Q5**：参考答案 — `data_only=True` 读取的是缓存的计算值而非公式。危险：在此模式下修改并保存会丢失所有公式。安全：只读不写（提取数据做分析）。

**Q6**：参考答案示例 — (1) 列映射：64=BL 不是 BK，错了整个列数据全部引用错误；(2) NaN 处理：空值未经检查写入 Excel，公式返回 #VALUE!；(3) 零除检查：分母为 0 产生 #DIV/0!，影响整个报表可信度。

**Q7**：参考答案 — (1) 净利润黑色公式，引用的采购成本等假设蓝色；(2) 毛利率黑色公式；(3) 盈亏平衡销量黑色公式，引用的售价和退货率蓝色+黄色背景。

**Q8**：参考答案 — 一个 #REF! 错误会让接收方质疑整个模型的可信度。自动检查（recalc.py）能发现 #REF!/#DIV/0!/#VALUE! 等技术错误；人工检查能发现"数据对但逻辑不对"的问题。两者必须配合使用。

（6/8 通过）
</details>

---

## 10. 延伸阅读

- [[L5-15-PPT幻灯片生成|上一课：PPT幻灯片生成]]
- [[L5-17-ClaudeAPI最佳实践|下一课：Claude API最佳实践]]
