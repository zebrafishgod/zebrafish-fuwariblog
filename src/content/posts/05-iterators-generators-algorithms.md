---
title: Python 第 242-276 集：迭代器、生成器及常用算法
published: 2026-09-15
updated: 2026-09-16
description: 迭代器 生成器 常用算法
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第 5 章｜迭代器、生成器与常用算法

> 对应既有学习路线的 242–276 节范围，按概念重新编排。本章是依课程主题整理的自学讲义，并非逐集影片逐字稿。示例以 Python 3 为准。

## 1. 本章解决什么问题？需要哪些前置知识？

这一章回答四个问题：

1. `for` 为什么能逐个读取列表、字符串及文件？
2. 如果数据很多，能不能「需要一个，才产生一个」？
3. 怎样用推导式、递回、二分搜索，把问题写得更清楚？
4. `lambda` 究竟是什么，为什么常和 `sorted()` 一起出现？

前置知识是列表 `list`、字典 `dict`、集合 `set`、条件、循环，以及函数的参数和 `return`。本章不需要先学类别。

读到捕捉 `StopIteration` 的例子时，先把 `try/except` 理解成「尝试取值；如果已经取完，就走另一段处理」。异常的完整说明见 [补充：错误与测试](bridge-errors-testing.md)。

建议分成三次学习：先掌握迭代与生成器，再学推导式与递回，最后做搜索与排序。`send()` 是选读，不是写出第一个完整程序的必要条件。

## 2. 先创建直觉

### 2.1 容器、可迭代对象、迭代器，是不同角度的名称

把一份商品清单想成一本目录：

- **可迭代对象（iterable）**：可以交给 `iter()`，取得一个「逐项读取工具」。列表、字符串、字典、`range()` 都是常见例子。
- **迭代器（iterator）**：这个工具本身会记住当前读到哪里。`next()` 让它往后读一项。
- **容器**：主要强调「保存数据」。不是所有可迭代对象都会预先存好全部数据，例如生成器可以边读边算。

一个列表可以产生多个互不影响的迭代器。迭代器通常只往前走；取完后，不会因为再写一个 `for` 就回到开头。

### 2.2 生成器是一种迭代器

普通函数用 `return` 交回结果，这一次调用就结束。含 `yield` 的函数是**生成器函数**：调用它会先创建生成器对象；之后每次索取下一项，才从上次暂停处继续执行。

| 动作 | 做了什么 | 是否一定显示在萤幕？ |
|---|---|---|
| `print(value)` | 把内容显示到输出 | 是，预设显示在终端 |
| `return value` | 把结果交回函数调用处，结束这次调用 | 否 |
| `yield value` | 交出本次迭代值，保留暂停位置 | 否 |

生成器常能省下「预先保存全部结果」的内存，但不保证运算比较快。如果最后仍写 `list(generator)`，所有结果仍会放进内存。

### 2.3 算法就是解题步骤

递回把问题拆成「同一种、更小的问题」；二分搜索在**已排序**的数据中，每次排除约一半候选。两者都不是「看起来比较高级就使用」，而是要看问题是否符合条件。

## 3. 最小可跑例：按顺序练习

以下程序代码区块除非特别注明，皆可各自存成一个 `.py` 文件执行。标记「输出」的注解描述 `print()` 显示的内容；`return` 和 `yield` 本身不负责显示。

### 3.1 `iter()` 创建读取位置，`next()` 读下一项

```python
# 输出：
# 苹果
# 香蕉
# 已取完
# 说明：next() 每次取得一个值；读完后会产生 StopIteration。
products = ["苹果", "香蕉"]
cursor = iter(products)

print(next(cursor))
print(next(cursor))

try:
    print(next(cursor))
except StopIteration:
    print("已取完")
```

`products` 仍然是完整列表；前进的是 `cursor` 的读取位置，并不是从列表删除了元素。最后一次 `next(cursor)` 没有成功取得值，所以第三个 `print()` 不会先打印什么，再发生错误。

若「取不到」是正常情况，可以为 `next()` 提供预设值：

```python
# 输出：
# 10
# 没有下一项
# 说明：next(迭代器, 预设值) 在耗尽时交回预设值。
cursor = iter([10])
print(next(cursor, "没有下一项"))
print(next(cursor, "没有下一项"))
```

注意：`next([10, 20])` 不成立，因为列表可迭代，却不是迭代器。先写 `cursor = iter([10, 20])`。若数据本身可能就是预设值，便不能只凭「结果等于预设值」判断是否耗尽。

### 3.2 `for` 在背后做什么？

平常这样写就好：

```python
# 输出：
# 甲
# 乙
# 说明：for 会取得迭代器，逐项读取，并在读完时自动停止。
names = ["甲", "乙"]
for name in names:
    print(name)
```

下面用明确步骤模拟核心行为，目的是理解，不是要求你以后把 `for` 改写得更长：

```python
# 输出：
# 甲
# 乙
# 说明：这是 for 的核心取值流程；只在 next() 周围捕捉耗尽讯号。
names = ["甲", "乙"]
cursor = iter(names)

while True:
    try:
        name = next(cursor)
    except StopIteration:
        break
    print(name)
```

同一个列表可以反复遍历，是因为每轮 `for name in names` 都取得新的读取位置。如果改为遍历同一个已耗尽的迭代器，第二轮就没有项目：

```python
# 输出：
# [1, 2]
# []
# [1, 2]
# 说明：list(cursor) 会消耗 cursor；原列表没有因此清空。
numbers = [1, 2]
cursor = iter(numbers)
print(list(cursor))
print(list(cursor))
print(list(numbers))
```

### 3.3 生成器函数与生成器对象

```python
# 输出：
# 已创建
# 开始计算
# 0
# 1
# 4
# 说明：square_numbers 是函数；stream 是一次调用产生的生成器对象。
def square_numbers(limit):
    print("开始计算")
    for number in range(limit):
        yield number * number


stream = square_numbers(3)
print("已创建")
print(next(stream))
print(next(stream))
print(next(stream))
```

重点是输出顺序：`stream = square_numbers(3)` 只创建对象，**不立即执行函数主体**。第一次 `next(stream)` 才开始执行；遇到 `yield 0` 暂停，把 `0` 交回给外层的 `print()`。

第二次 `next()` 会从第一次 `yield` 后面继续。它不是重新调用一次函数，也不会重新打印「开始计算」。

同一个生成器耗尽后不能直接倒回；再次调用生成器函数，会创建另一个新对象：

```python
# 输出：
# [0, 1, 4]
# []
# [0, 1, 4]
# 说明：两次 square_numbers(3) 创建的是不同生成器。
def square_numbers(limit):
    for number in range(limit):
        yield number * number


stream = square_numbers(3)
print(list(stream))
print(list(stream))
print(list(square_numbers(3)))
```

在生成器内，`return` 或执行到函数结尾表示「结束迭代」。`return 99` 也不等于「再产生一个 99」；要交出一个可遍历的项目应使用 `yield 99`。初学时，生成器中的 `return` 先只用来结束即可。

### 3.4 从普通循环理解推导式

推导式不是新的计算能力，而是把「遍历、筛选、组成容器」缩短。先看容易追踪的版本：

```python
# 输出：
# [4, 16]
# [4, 16]
# 说明：两种写法都先挑出偶数，再保存它们的平方。
numbers = [1, 2, 3, 4]

squares = []
for number in numbers:
    if number % 2 == 0:
        squares.append(number * number)
print(squares)

squares = [number * number for number in numbers if number % 2 == 0]
print(squares)
```

阅读顺序是：`for number in numbers` 逐项取得值 → `if ...` 决定保留与否 → 前面的 `number * number` 决定保存什么。

| 形式 | 结果 | 典型用途 |
|---|---|---|
| `[运算式 for 变量 in 数据]` | 列表 | 保存转换后的顺序数据 |
| `{键: 值 for 变量 in 数据}` | 字典 | 创建查找关系 |
| `{运算式 for 变量 in 数据}` | 集合 | 去除重复 |
| `(运算式 for 变量 in 数据)` | 生成器 | 之后逐项产生结果 |

```python
# 输出：
# {'apple': 5, 'banana': 6}
# ['apple', 'banana']
# 说明：字典把名称映射到长度；集合去重后用 sorted() 稳定显示顺序。
names = ["apple", "banana", "apple"]
length_by_name = {name: len(name) for name in names}
unique_names = {name for name in names}

print(length_by_name)
print(sorted(unique_names))
```

字典的键必须唯一，重复键会用后面的值覆盖前面的值。集合不提供位置索引，不应依赖它显示的顺序。本例用 `sorted()` 产生排序后的列表，让输出可核对。

先单独认识条件运算式，也常被称为「三元运算式」：`符合时的值 if 条件 else 不符合时的值`。它会计算出一个值，可放在赋值右侧；只有被选中的那一侧会求值。

```python
# 输出：
# 可购买
# 说明：条件运算式交回其中一个字符串；print() 才显示它。
stock = 3
message = "可购买" if stock > 0 else "已售完"
print(message)
```

另一种 `if` 放在推导式前面，是「每一项都保留，但决定这一项的值」：

```python
# 输出：
# ['奇数', '偶数', '奇数', '偶数']
# 说明：if/else 是选值；尾端没有筛选条件，因此输出仍有四项。
labels = ["偶数" if number % 2 == 0 else "奇数" for number in range(1, 5)]
print(labels)
```

如果一条推导式需要读两三遍才明白，改回普通 `for` 往往更好。尤其别把复杂操作和多层条件塞在同一行。

### 3.5 生成器表达式：把「先存好」改成「逐个给」

```python
# 输出：
# 30
# 0
# [1, 4, 9, 16]
# 说明：第一次 sum() 消耗生成器；再次求和时已没有项目。
stream = (number * number for number in range(1, 5))
print(sum(stream))
print(sum(stream))
print([number * number for number in range(1, 5)])
```

`sum(stream)` 不需要先创建平方结果列表，就能逐项加总。第二个 `0` 是「空的可迭代对象加总为 0」，不是生成器重新算了一次却算错。

需要随机取得第 100 项、反复遍历、确认长度时，列表通常更直接。只需扫过一次、数据很大时，再考虑生成器。生成器表达式的**结果元素**逐项计算，但最左侧 `for` 的可迭代运算式会在创建时求值；不要把「惰性」理解成所有东西都完全延后。

例如统计一段英文文字有多少个空白分隔的单词，可以逐行计算，再加总：

```python
# 输出：
# 7
# 说明：split() 不指定参数时按连续空白分隔；标点不另外分词。
lines = ["Python is fun.\n", "I write code.\n", "Great!\n"]
word_count = sum(len(line.split()) for line in lines)
print(word_count)
```

这个例子是 `3 + 3 + 1`。套用到先前学过的文件处理时，在 `with open(..., encoding="utf-8") as file:` 区块内，把 `lines` 换成 `file`，就能逐行读取并加总，避免先用 `readlines()` 创建完整行列表。必须在文件仍开启时完成消耗。这是「按空白分隔」的单词数，不是中文字数，也不是自然语言的精确分词；`len(text)` 计算的又是另一种长度，先定义需求再选算法。

### 3.6 选读：`send()` 不只取值，也能把值送回去

`next()` 只要求「继续执行」。`send(value)` 还把一个值送进当前暂停的 `yield` 运算式。

```python
# 输出：
# 0
# 5
# 8
# 说明：yield 交出当前总和；恢复时，send() 的参数成为 amount 的值。
def running_total():
    total = 0
    while True:
        amount = yield total
        total += amount


counter = running_total()
print(next(counter))
print(counter.send(5))
print(counter.send(3))
counter.close()
```

把 `amount = yield total` 拆成两个时刻：

1. **暂停前**：把 `total` 交给外面。
2. **恢复时**：外面送入的值，才会赋给 `amount`。

第一次要先 `next(counter)`，或 `counter.send(None)`，让它走到第一个暂停点。尚未启动就 `send(5)` 会产生 `TypeError`。本例启动后预期每次收到数字；若再调用 `next(counter)`，送入的会是 `None`，于是加法失败。`close()` 用来结束这个本来会一直等待输入的生成器。

这是双向互动的初步认识。一般数据遍历用 `for` 已足够，不需要特意改用 `send()`。

### 3.7 递回：先找到「不用再分解」的情况

计算 `4!`：`4 × 3 × 2 × 1`。可以把它写成 `4 × 3!`，而 `3!` 又是 `3 × 2!`。

```python
# 输出：
# 24
# 1
# 说明：factorial() 用 return 交回数字；外层 print() 才显示结果。
# 前提：n 是非负整数。
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)


print(factorial(4))
print(factorial(0))
```

递回必须有两件事：

- **基例**：这里是 `n == 0`，可以直接交回答案。
- **朝基例前进的规则**：这里每次改成 `n - 1`。

不能只检查「有没有写 `if`」。如果输入 `-1`，每次再减一会离 `0` 越来越远；最终通常会遇到 `RecursionError`。本例把非负整数列为调用前提。要接收用户输入，还需要先检查范围与型别。

每次递回调用都要保留尚未完成的工作。Python 有递回深度限制；递回也通常有额外开销。像阶乘这种直线型累积，使用循环同样清楚；本例主要用来练习「拆解、到底、返回」。不要用调高限制代替修正无限递回。

### 3.8 全排列：固定一个位置，再排列剩下的项目

`["A", "B", "C"]` 的全排列是「三个元素各用一次，所有可能的先后顺序」。

```python
# 输出：
# ABC
# ACB
# BAC
# BCA
# CAB
# CBA
# 说明：permutations() 交回排列列表；外层 for 和 print() 显示各排列。
# 前提：输入元素互不重复；本例适合少量数据。
def permutations(items):
    if len(items) == 0:
        return [[]]

    result = []
    for index in range(len(items)):
        first = items[index]
        remaining = items[:index] + items[index + 1:]
        for rest in permutations(remaining):
            result.append([first] + rest)
    return result


for arrangement in permutations(["A", "B", "C"]):
    print("".join(arrangement))
```

`items[:index] + items[index + 1:]` 是把「选中的那一项」排除后，组成新的列表；原始输入不会被修改。`"".join(arrangement)` 是把字符串列表接成一个字符串。

基例为什么是 `[[]]` 而不是 `[]`？因为「没有剩余元素」仍有**一种**排列方式：空排列。外层才能把最后选中的元素接上去。若直接交回 `[]`，上层 `for rest in ...` 一次也不执行，全部结果就消失了。

互不重复的 `n` 个元素有 `n!` 个排列。`10!` 就有 3,628,800 个结果。这份写法还会保存中间结果，所以只拿小型数据学习。若输入有重复值，可能产生内容重复的排列；去重是另一个问题。生成器可以减少一次保存的结果，却不能消除排列数量爆增的事实。

### 3.9 二分搜索：先确认数据已排序

在 `[3, 7, 12, 18, 25]` 找 `18`，先看中间 `12`；因为 `18 > 12`，只需要在右半边找。

```python
# 输出：
# 3
# -1
# -1
# 说明：找到时 return 索引，找不到时 return -1；print() 显示结果。
# 前提：numbers 已按从小到大排序。
def binary_search(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left <= right:
        middle = (left + right) // 2
        if numbers[middle] == target:
            return middle
        if numbers[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1


numbers = [3, 7, 12, 18, 25]
print(binary_search(numbers, 18))
print(binary_search(numbers, 9))
print(binary_search([], 9))
```

这里 `left` 和 `right` 都包含在搜索范围内，所以条件是 `left <= right`；只剩一个候选时，也要检查。更新成 `middle + 1` 或 `middle - 1`，表示已检查过的中间项目不必再看，搜索范围确实缩小。

几个实际限制：

- 未排序的数据不能直接套用这个算法。
- 函数返回 `-1` 表示找不到；不能把这个 `-1` 直接拿去索引，因为 Python 的 `numbers[-1]` 反而表示最后一项。
- 若有重复值，本例找到的是某个符合值的索引，不保证是第一个。
- 若先排序再搜索，返回的是排序后的位置。排序成本与原始索引的变化都要考虑。
- 对可快速按索引取得元素的列表，搜索步数约为 `log₂(n)`；不是每次都必须逐项看完。

### 3.10 `lambda`：小型匿名函数

`lambda 参数: 运算式` 创建函数，调用后交回运算式的值。它只有一个运算式，不能写多行叙述。较长逻辑就用 `def`，并取一个好懂的名字。

```python
# 输出：
# 7
# 7
# 说明：两个函数都交回加法结果；lambda 不需要再写 return。
def add_named(left, right):
    return left + right


add_short = lambda left, right: left + right
print(add_named(3, 4))
print(add_short(3, 4))
```

「高阶函数」在这里指会接收其他函数作为参数的函数。你可以把小型规则交给它，让它负责完整操作。

### 3.11 用 `key` 说清楚排序依据

```python
# 输出：
# [('笔', 5), ('杯子', 20), ('书', 30)]
# ('书', 30)
# [('书', 30), ('笔', 5), ('杯子', 20)]
# 说明：key 函数交回比较依据；sorted() 产生新列表，原列表顺序不变。
products = [("书", 30), ("笔", 5), ("杯子", 20)]

print(sorted(products, key=lambda product: product[1]))
print(max(products, key=lambda product: product[1]))
print(products)
```

`lambda product: product[1]` 表示「对每个商品，拿第二项价格作为比较依据」。`key` 接收的是**函数本身**，不是你先算好的一个价格。

想由高到低排序，加上 `reverse=True`。价格相同时，Python 的排序会保留原本的相对顺序，称为稳定排序。商品很多时，清楚的 `def price_of(product): ...` 和 `lambda` 都可以使用，功能不因名字不同而改变。

### 3.12 `map()` 转换，`filter()` 筛选

```python
# 输出：
# [20, 40, 60]
# [20, 30]
# []
# 说明：Python 3 的 map/filter 交回迭代器；list() 取得并保存结果。
prices = [10, 20, 30]
double_prices = map(lambda price: price * 2, prices)
expensive_prices = filter(lambda price: price >= 20, prices)

print(list(double_prices))
print(list(expensive_prices))
print(list(expensive_prices))
```

`map()` 不丢掉项目，而是逐项转换；`filter()` 用条件决定是否保留，保留的是原项目。第三行空列表，是因为同一个 `filter` 迭代器已被消耗。

这个例子也可以写成 `[price * 2 for price in prices]` 和 `[price for price in prices if price >= 20]`。优先选自己和读者都容易理解的形式，不必为了用 `lambda` 而强行改写。

### 3.13 类型提示：说明预期，不会自动检查输入

类型提示（type hints）把函数希望收到和交回的数据型别写清楚：`name: str` 表示预期字符串，`-> int` 表示预期交回整数。编辑器和额外的静态检查工具可用它提醒潜在问题；Python 执行程序时不会只因为写了注记，就自动转型或拒绝不符合的值。

```python
# 输出：
# 7
# 34
# 说明：第二次调用故意违反注记，展示型别提示本身不阻止字符串相加。
def add(left: int, right: int) -> int:
    return left + right


print(add(3, 4))
print(add("3", "4"))
```

第二行的 `34` 是两个字符串连接后的结果，与整数 `34` 显示相似，但型别不同。这次调用应由类型检查工具提醒，业务程序不应故意这样使用。若数据来自 `input()`，仍然要自己转型、检查内容并处理失败。

容器也能标示元素型别。以下写法需要 Python 3.9 或更新版本：

```python
# 输出：
# ['笔', '书']
# 说明：list[str] 表示预期为字符串列表；-> list[str] 表示返回值的预期型别。
def nonempty_names(names: list[str]) -> list[str]:
    return [name.strip() for name in names if name.strip()]


print(nonempty_names([" 笔 ", "", "书"]))
```

`strip()` 去除字符串两端空白；本例过滤空字符串或只有空白的名称。`list[str]` 是描述，不是创建列表的操作。只做显示、没有需要交回的结果时，可以把返回提示写成 `-> None`；这里的 `None` 也不等于「没有任何行为」。

类型提示不要和预设值混淆：`quantity: int = 1` 里，`int` 是预期型别，`1` 才是省略实参时使用的值。函数的输入范围、金额单位、是否允许负数，也不是单写 `int` 就能完整说明，仍需文件和检查。

## 4. 执行追踪：真正理解程序停在哪里

### 4.1 生成器追踪

以 3.3 的 `square_numbers(3)` 为例：

| 外面做的动作 | 函数里做的动作 | 交回的值 | 剩下的状态 |
|---|---|---|---|
| 创建 `stream` | 主体尚未执行 | 生成器对象 | 等待启动 |
| 第一次 `next(stream)` | 印「开始计算」，算 `0 * 0` | `0` | 暂停在第一次 `yield` |
| 第二次 `next(stream)` | 接续循环，算 `1 * 1` | `1` | 暂停在第二次 `yield` |
| 第三次 `next(stream)` | 接续循环，算 `2 * 2` | `4` | 暂停在第三次 `yield` |
| 再次 `next(stream)` | 循环结束、函数结束 | 没有下一项 | 产生 `StopIteration` |

第三次交出 `4` 的当下，函数仍暂停在 `yield`。下一次要求取值，才继续走到结尾。`for` 会自动处理最后的耗尽讯号。

### 4.2 递回追踪

`factorial(3)` 先一路拆解，再一层层返回：

| 阶段 | 尚未完成的运算 |
|---|---|
| 调用 `factorial(3)` | 等 `3 * factorial(2)` |
| 调用 `factorial(2)` | 等 `2 * factorial(1)` |
| 调用 `factorial(1)` | 等 `1 * factorial(0)` |
| `factorial(0)` 到基例 | 直接交回 `1` |
| 返回 `factorial(1)` | `1 * 1 = 1` |
| 返回 `factorial(2)` | `2 * 1 = 2` |
| 返回 `factorial(3)` | `3 * 2 = 6` |

对全排列 `permutations(["A", "B"])`：先选 `A` → 剩下 `["B"]` → 再选 `B` → 剩下 `[]` → 基例交回 `[[]]` → 组成 `["B"]` → 组成 `["A", "B"]`。之后回到第一层改选 `B`，得到 `["B", "A"]`。

### 4.3 二分搜索追踪

在 `[3, 7, 12, 18, 25]` 找 `18`：

| 轮次 | `left` | `right` | `middle` | 中间值 | 下一步 |
|---|---:|---:|---:|---:|---|
| 1 | 0 | 4 | 2 | 12 | 目标较大，`left = 3` |
| 2 | 3 | 4 | 3 | 18 | 找到，交回 `3` |

找 `9` 时会经过中间值 `12 → 3 → 7`，最后 `left = 2`、`right = 1`。搜索范围为空，才交回 `-1`。

## 5. 常见错误与修正

| 容易误会的地方 | 正确理解与修正 |
|---|---|
| 「只要能放进 for，就一定能直接 next」 | `for` 能接收可迭代对象；`next()` 要接收迭代器。必要时先 `iter()`。 |
| 「生成器函数调用时已算好全部值」 | 调用通常先创建生成器对象；取下一项时才执行主体。 |
| 「把生成器打印来就能看全部内容」 | `print(stream)` 主要显示对象表示；用 `for` 逐项读取，或小量数据时用 `list(stream)`。 |
| 「生成器用完还能重复使用」 | 同一个对象不能倒带；重新调用生成器函数创建新对象。 |
| 「有 yield，就一定更快」 | 主要优势通常是逐项产生与节省中间容器；实际速度需量测。 |
| 「`[x if 条件 for x in ...]` 可以只保留符合值」 | 尾端筛选写 `[x for x in ... if 条件]`；前端选值必须有 `else`。 |
| 「递回有基例就一定会停」 | 每次调用还必须朝基例前进，并符合输入前提。 |
| 「全排列的空输入应返回空列表」 | 组合用的基例需要一个空排列，故用 `[[]]`。 |
| 「二分搜索适用任何列表」 | 必须符合排序前提与一致的比较规则。 |
| 「找到 `-1` 就印 `numbers[-1]`」 | `-1` 是本函数自订的未找到讯号，先判断，再取值。 |
| 「lambda 是高级语法，应取代 def」 | 短规则用 lambda；较长或需命名解释的逻辑用 def。 |

## 6. 练习：先预测，再执行

### 练习 A｜读取位置

创建 `numbers = [2, 4, 6]` 与 `cursor = iter(numbers)`。先执行一次 `next(cursor)`，再打印 `list(cursor)`，最后打印 `list(cursor)`。执行前写下两次 `print()` 的预期内容，并解释原列表是否改变。

### 练习 B｜生成器

撰写 `even_numbers(stop)`，逐项产生 `0` 到 `stop` 之前的偶数，边界与 `range(stop)` 一致。验收：`list(even_numbers(7))` 是 `[0, 2, 4, 6]`；`list(even_numbers(0))` 是 `[]`。试著只取前两项，描述程序暂停的位置。

### 练习 C｜推导式

已知 `prices = [12, 5, 20, 5]`，用推导式创建：① 所有大于等于 `10` 的价格；② 去重后的价格集合；③ 以价格为键、两倍价格为值的字典。写下为何字典只有三个键。

### 练习 D｜递回与边界

先用循环、再用递回写 `sum_to(n)`，计算 `1 + 2 + ... + n`，前提是 `n` 为非负整数。验收：`sum_to(0)` 为 `0`，`sum_to(5)` 为 `15`。在递回版本旁标明基例，以及每一步如何接近基例。不要用很大的 `n` 压测递回。

### 练习 E｜二分搜索

用本章函数查找 `[2, 5, 8, 11, 14, 17]` 中的 `2`、`17`、`9`；在纸上列出每轮 `left/right/middle`。再测空列表与只有一项的列表。验收输出依次为 `0`、`5`、`-1`；空列表找不到。

### 练习 F｜商品排序与筛选

商品数据为 `[("笔", 5), ("书", 30), ("杯子", 20)]`。创建价格至少 `20` 的商品列表，再按价格由高到低排序。验收结果为 `[("书", 30), ("杯子", 20)]`，且原列表顺序不变。

### 选做｜全排列

用 3.8 的函数验证两个元素有 `2` 个排列、三个元素有 `6` 个排列、空列表有 `1` 个空排列。尝试输入 `["A", "A"]`，解释为何会出现两个相同结果。先用语言说明，不急著实作去重。

## 7. 自查与简答

先遮住右栏，能用自己的话回答，才算理解。

| 自查问题 | 简答 |
|---|---|
| iterable 和 iterator 的差别？ | 前者能让 `iter()` 取得迭代器；后者保存进度，能被 `next()` 逐项读取。 |
| `iter(iterator)` 会重设进度吗？ | 不会；迭代器的 `iter()` 通常返回它自己。 |
| `for` 遇到 `StopIteration` 会怎样？ | 结束遍历；通常不把它当作错误显示给用户。 |
| 生成器函数与生成器对象差在哪里？ | 函数是产生流程的定义；对象是一轮可暂停、可前进的执行状态。 |
| `yield 3` 一定会把 `3` 印到萤幕吗？ | 不会。它交出值；外面要用 `print()` 才会显示。 |
| 列表推导式与生成器表达式的主要差别？ | 前者立即创建结果列表；后者逐项产生结果且通常只能消耗一次。 |
| `send(5)` 的返回值是 `5` 吗？ | 不一定。它送入 `5`，然后交回生成器接下来 `yield` 的值。 |
| 写递回前先确认什么？ | 输入范围、基例，以及每一步确实让问题变小。 |
| 二分搜索为什么用 `middle + 1`？ | 中间值已确认不是答案，不必再纳入下一轮，且范围必须缩小。 |
| `key=lambda product: product[1]` 做什么？ | 告诉排序或比较函数：把每件商品的价格当作比较依据。 |
| `map()` 和 `filter()` 有何不同？ | 前者转换每一项，后者按条件保留原项目。 |
| 写了 `age: int`，用户输入会自动变成整数吗？ | 不会。类型提示不做执行时转型或验证；仍需自行转换与检查。 |

练习核对：A 的两次输出为 `[4, 6]`、`[]`，原列表仍是 `[2, 4, 6]`；C 的第一项是 `[12, 20]`，第二项有 `5、12、20`，第三项为 `{12: 24, 5: 10, 20: 40}`，相同键 `5` 只保留一份；D 的递回基例可设 `n == 0`，其余情况交回 `n + sum_to(n - 1)`。

完成本章的最低标准：能解释一次性迭代、写出简单生成器与推导式、说出递回停止条件、手动追踪二分搜索，并使用 `key` 实作商品排序。接下来在 [第 6 章：模块与包](06-modules-packages.md) 把多个函数整理成可维护的文件结构。
