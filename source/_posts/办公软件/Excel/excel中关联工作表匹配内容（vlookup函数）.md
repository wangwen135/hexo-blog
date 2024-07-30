---
title: excel中关联工作表匹配内容（vlookup函数）
date: 2020-09-26
tags: 
  - Excel
  - VLOOKUP
categories:
  - [办公软件, Excel]
---

要在 Excel 中实现根据 Sheet1 的值匹配 Sheet2 中的值并取出对应行中另一个单元格的内容，可以使用 VLOOKUP 或 INDEX+MATCH 函数组合。以下是两种方法的示例：


### 方法一：使用 VLOOKUP 函数

假设 Sheet1 中的值在 A 列，Sheet2 中要匹配的值也在 A 列，取出匹配行中的 B 列的值。

1. 在 Sheet1 中，选择要放置匹配结果的单元格，例如 B2
2. 输入以下公式：
```
=VLOOKUP(A2, Sheet2!A:B, 2, FALSE)
```

- A2 是要匹配的值。
- Sheet2!A:B 是要在 Sheet2 中查找的区域。
- 2 是返回的列号，即 Sheet2 中的 B 列。
- FALSE 表示精确匹配。




### 方法二：使用 INDEX 和 MATCH 函数组合

假设 Sheet1 中的值在 A 列，Sheet2 中要匹配的值在 A 列，取出匹配行中的 B 列的值。

1. 在 Sheet1 中，选择要放置匹配结果的单元格，例如 B2。
2. 输入以下公式：
```
=INDEX(Sheet2!B:B, MATCH(A2, Sheet2!A:A, 0))
```
- INDEX(Sheet2!B:B, ...) 表示返回 Sheet2 中 B 列的值。
- MATCH(A2, Sheet2!A:A, 0) 查找 A2 的值在 Sheet2 A 列中的位置，0 表示精确匹配。
 



