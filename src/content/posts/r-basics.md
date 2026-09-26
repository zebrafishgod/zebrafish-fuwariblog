---
title: R 语言基础：从对象、向量到数据框
published: 2026-09-26
updated: 2026-09-26
description: 以可运行示例学习 R 的对象、向量、列表、数据框、条件、循环、函数、缺失值和文件读写。
tags: [R, 基础篇]
category: R 学习
draft: false
lang: zh_CN
---

# R 语言基础学习笔记

这篇笔记记录 R 语言入门阶段最常用的知识。R 常用于统计分析、数据处理和可视化，所以学习时不能只记语法，还要知道数据是什么类型、每一步返回什么，以及结果是否符合预期。

示例可以逐段复制到 R 控制台或保存为 `.R` 文件运行。代码中的注释说明输出结果，实际小数显示可能因为版本和环境略有差异。

## 1. 第一个 R 程序

R 控制台会直接显示表达式的结果，但写入脚本后，通常使用 `print()` 明确输出。

```r
print("Hello, R!")
name <- "小林"
message <- paste("你好，", name, "！")
print(message)
# 输出：
# [1] "Hello, R!"
# [1] "你好， 小林 ！"
```

`#` 后面是注释。R 常用 `<-` 赋值，也支持 `=`，初学时建议统一使用 `<-`。赋值语句本身通常不会在脚本中显示结果。

```r
price <- 8
quantity <- 3
total <- price * quantity
quantity <- 5
print(total)
print(price * quantity)
# 输出：24；40
```

这里的 `total` 在赋值时已经计算完成，之后修改 `quantity` 不会自动重新计算它。

## 2. 对象和基本类型

R 中几乎所有数据都可以看作对象。对象有值，也有类型和长度。常用的基本类型包括：

| 类型 | 示例 | 说明 |
|---|---|---|
| `numeric` | `3.14`、`10` | 数值，R 默认把普通数字当作双精度数 |
| `integer` | `10L` | 整数，字面量后加 `L` |
| `character` | `"R"` | 字符串 |
| `logical` | `TRUE`、`FALSE` | 逻辑值 |
| `complex` | `1 + 2i` | 复数 |
| `NULL` | `NULL` | 没有对象 |
| `NA` | `NA` | 缺失值 |

```r
value <- 10
print(typeof(value))
print(class(value))
print(length(value))
# 输出：double；numeric；1

value <- "10"
print(typeof(value))
print(value + "2")
# 第二行会报错，因为字符串不能直接做数值加法
```

`typeof()` 更接近 R 内部类型，`class()` 更接近对象在使用时的类别。排查问题时，可以先检查 `typeof()`、`class()` 和 `length()`。

## 3. 向量：R 最重要的数据结构

R 的很多操作都是针对向量设计的。用 `c()` 可以把多个值连接成一个向量。

```r
scores <- c(70, 85, 90, 60)
print(scores + 5)
print(scores >= 80)
print(mean(scores))
# 输出：[1] 75 90 95 65
# 输出：[1] FALSE TRUE TRUE FALSE
# 输出：76.25
```

`scores + 5` 会把 5 应用到每一个元素，这叫向量化运算。它通常比手写循环更简洁。

```r
names <- c("Alice", "Bob", "Carol")
print(names[1])
print(names[c(1, 3)])
print(names[-2])
# 输出："Alice"；"Alice" "Carol"；"Alice" "Carol"
```

R 的向量索引从 **1** 开始，不是从 0 开始。正索引选择元素，负索引排除元素。

```r
numbers <- 1:5
print(numbers[2:4])
print(numbers[numbers %% 2 == 0])
# 输出：[1] 2 3 4；[1] 2 4
```

`%%` 是取余运算。`numbers %% 2 == 0` 会先生成逻辑向量，再用这个逻辑向量筛选元素。

### 3.1 向量的类型转换

一个原子向量通常只能保存一种基本类型。如果把不同类型放在一起，R 会按规则自动转换。

```r
mixed <- c(1, TRUE, "3")
print(mixed)
print(typeof(mixed))
# 输出："1" "TRUE" "3"
# 输出：character
```

常见转换方向大致是 `logical -> numeric -> character`。自动转换方便，但也可能隐藏错误，所以从文件读取数据后要检查类型。

```r
text_numbers <- c("10", "20", "30")
numbers <- as.numeric(text_numbers)
print(sum(numbers))
# 输出：60
```

`as.numeric()` 是转换，不会修改原来的字符串向量。

## 4. 列表：保存不同结构的数据

向量要求元素类型相对一致，列表则可以保存不同类型、不同长度的对象。

```r
student <- list(
  name = "小林",
  scores = c(80, 90, 85),
  active = TRUE
)

print(student$name)
print(student[["scores"]])
print(student[["scores"]][2])
# 输出：小林；80 90 85；90
```

`student["scores"]` 得到的仍然是列表，`student[["scores"]]` 才是其中保存的向量。这是列表初学时最容易混淆的区别。

```r
print(student[1])
print(student[[1]])
# 第一行仍是一个列表
# 第二行是字符串 "小林"
```

列表适合表示一笔结构复杂的数据，例如一个学生、一笔订单或一组配置。

## 5. 数据框：按列保存的表格

数据框 `data.frame` 是 R 中最常用的表格结构。每一列是一个向量，同一列的元素类型通常一致，各列长度相同。

```r
students <- data.frame(
  name = c("小林", "小王", "小陈"),
  score = c(88, 76, 92),
  passed = c(TRUE, TRUE, TRUE)
)

print(students)
print(students$name)
print(students[students$score >= 80, ])
# 最后一行筛选出小林和小陈
```

数据框的索引格式是 `data[行, 列]`。逗号左边是行，右边是列；省略一边表示保留全部。

```r
students$level <- ifelse(students$score >= 90, "优秀", "合格")
students$score[students$name == "小王"] <- 80
print(students)
```

`$` 可以按列名访问和新增列。修改数据前要确认筛选条件是否选中了正确的行。

### 5.1 因子

分类数据有时会被保存为因子 `factor`。因子保存类别水平，不应直接当作普通字符串或数字使用。

```r
skill_level <- factor(c("初级", "中级", "初级"))
print(levels(skill_level))
# 输出："初级" "中级"
```

如果只是处理文字分类，先确认列是否为 `character` 或 `factor`，不要猜测类型。

## 6. 条件判断和循环

### 6.1 `if`：条件成立时执行

```r
score <- 85
if (score >= 90) {
  level <- "优秀"
} else if (score >= 60) {
  level <- "合格"
} else {
  level <- "不合格"
}
print(level)
# 输出："合格"
```

R 的条件必须得到单个 `TRUE` 或 `FALSE`。如果条件是一个向量，应使用 `ifelse()` 或先明确处理每个元素。

```r
scores <- c(55, 70, 90)
result <- ifelse(scores >= 60, "通过", "未通过")
print(result)
# 输出："未通过" "通过" "通过"
```

### 6.2 `for` 和 `while`

```r
total <- 0
for (number in 1:5) {
  total <- total + number
}
print(total)
# 输出：15

count <- 3
while (count > 0) {
  print(count)
  count <- count - 1
}
```

`for` 适合依次处理一组值，`while` 适合只知道停止条件的情况。使用 `while` 时必须确保状态最终会改变，否则会产生无限循环。

## 7. 函数：把一段操作命名

函数定义了输入、处理过程和返回值。R 中最后一个表达式会自动成为返回值，也可以明确使用 `return()`。

```r
add <- function(a, b) {
  a + b
}

result <- add(3, 5)
print(result)
# 输出：8
```

函数体最后一行 `a + b` 的结果会返回给调用者。函数只打印结果而不返回结果时，后续代码不能继续使用这个结果。

```r
show_total <- function(price, quantity) {
  print(price * quantity)
}

value <- show_total(10, 2)
print(value)
# 第一行输出 20，第二行输出 NULL
```

默认参数可以让调用者省略常用选项：

```r
greet <- function(name, prefix = "你好") {
  paste(prefix, name)
}

print(greet("小林"))
print(greet("小王", prefix = "早上好"))
# 输出："你好 小林"；"早上好 小王"
```

R 的函数参数可以按位置传递，也可以写出参数名。参数名较多时，使用命名参数更容易读懂。

## 8. 缺失值和逻辑判断

`NA` 表示缺失值，和空字符串、0、`NULL` 不是一回事。对包含 `NA` 的向量直接计算，结果通常也会是 `NA`。

```r
scores <- c(80, NA, 90)
print(mean(scores))
print(mean(scores, na.rm = TRUE))
# 输出：NA；85
```

`na.rm = TRUE` 的意思是计算前移除缺失值。是否应该移除，要根据业务含义决定，不能为了得到数字就无条件忽略缺失值。

不要使用 `score == NA` 判断缺失值，因为结果仍然是 `NA`。应使用 `is.na()`：

```r
print(is.na(scores))
print(scores[!is.na(scores)])
# 输出：FALSE TRUE FALSE；80 90
```

## 9. 字符串和文件读写

常用字符串函数包括 `paste()`、`paste0()`、`strsplit()`、`tolower()` 和 `toupper()`。

```r
first <- "Ada"
last <- "Lovelace"
print(paste(first, last))
print(paste0(first, "_", last))
print(toupper(first))
# 输出："Ada Lovelace"；"Ada_Lovelace"；"ADA"
```

`paste()` 默认使用空格连接，`paste0()` 默认不添加分隔符。

```r
lines <- c("apple,2", "banana,3")
writeLines(lines, "fruit.txt")
content <- readLines("fruit.txt", encoding = "UTF-8")
print(content)
file.remove("fruit.txt")
# 输出：[1] "apple,2" "banana,3"
```

写文件前要明确路径和编码。练习时可以使用 `tempfile()` 创建临时文件，避免覆盖重要数据。

```r
path <- tempfile(fileext = ".csv")
write.csv(students, path, row.names = FALSE, fileEncoding = "UTF-8")
loaded <- read.csv(path, fileEncoding = "UTF-8")
print(loaded)
unlink(path)
```

`write.csv()` 保存表格，`read.csv()` 读回表格。读回后仍要检查列名、类型和缺失值，不能假设文件内容一定正确。

## 10. 向量化处理和基础统计

很多数据处理可以直接用向量化函数完成：

```r
prices <- c(12, 5, 20, 8)
print(sum(prices))
print(min(prices))
print(max(prices))
print(sort(prices, decreasing = TRUE))
# 输出：45；5；20；20 12 8 5
```

筛选、排序和统计通常可以组合起来：

```r
available <- data.frame(
  product = c("笔", "书", "杯子"),
  stock = c(5, 0, 12),
  price = c(3, 20, 15)
)

in_stock <- available[available$stock > 0, ]
in_stock <- in_stock[order(in_stock$price, decreasing = TRUE), ]
print(in_stock)
# 结果按价格从高到低显示杯子和笔
```

`order()` 返回排序后的行位置。先筛选再排序，能够把每一步的意图写得更清楚。

## 11. 常见错误排查

| 现象 | 常见原因 | 排查方向 |
|---|---|---|
| `object not found` | 名称拼写错误或尚未赋值 | 检查执行顺序和变量名 |
| `subscript out of bounds` | 索引超出范围 | 检查 `length()` 和索引位置 |
| 结果变成 `NA` | 数据中含缺失值 | 使用 `is.na()` 检查，决定是否 `na.rm = TRUE` |
| 条件判断报错 | `if` 收到多个逻辑值 | 改用 `ifelse()` 或明确取一个值 |
| 字符串无法相加 | 字符串不是数值 | 使用 `as.numeric()` 并检查转换结果 |
| 数据框筛选结果异常 | 行列索引或条件写错 | 明确 `data[行, 列]` 的位置 |
| 文件读写失败 | 路径、编码或权限错误 | 检查 `getwd()`、文件是否存在和编码 |
| `NA` 没有被筛掉 | 使用了 `x == NA` | 改用 `is.na(x)` |

## 12. 自查

1. R 的向量索引从几开始？`scores <- c(70, 80, 90)` 中如何取出 80？
2. `scores + 5` 为什么可以一次给每个元素加 5？
3. 列表中的 `x["scores"]` 和 `x[["scores"]]` 有什么区别？
4. 数据框 `data[行, 列]` 中，逗号两边分别表示什么？
5. 为什么 `mean(c(80, NA, 90))` 默认得到 `NA`？
6. 判断缺失值为什么要用 `is.na(x)`，而不是 `x == NA`？
7. 函数最后一行表达式和 `return()` 的关系是什么？
8. `ifelse()` 和 `if` 分别适合处理什么情况？
9. `write.csv()` 写入文件后，为什么还要用 `read.csv()` 读回检查？
10. 使用 `while` 循环时，怎样避免无限循环？

自查要点：索引从 1 开始；向量运算会逐元素处理；`[[ ]]` 取出列表中的对象而 `[ ]` 通常保留列表结构；数据框索引为行和列；`NA` 表示缺失并会传播到统计结果；缺失值用 `is.na()` 判断；函数最后一个表达式可以作为返回值；`ifelse()` 适合向量化条件；文件读回用于验证保存结果；循环状态必须朝停止条件变化。
