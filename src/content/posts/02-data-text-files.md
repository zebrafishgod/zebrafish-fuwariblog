---
title: Python 第 69-167 集：进制、文件与编码相关
published: 2026-09-10
updated: 2026-09-14
description: 进制】文件
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第二章　把数据整理好，再保存到文件（第 69–167 集）

这一章解决两个问题：「多笔数据要放在哪里？程序关闭后，如何把数据留下来？」学完后，应能完成文字清洗、商品数据的增删改查，以及 UTF-8 文件读写。

前置知识：变量、基本数据型别、`print()`、`input()`、`if`、`for`、`while`。本章使用 Python 3；第 168 集的函数引入放在下一章。

每个 Python 程序代码框都是一个可单独执行的例子。先预测结果，再执行，最后只改一处重跑。文件例子会创建自己的暂存文件夹，结束后自动移除；例子的 `w` 模式只操作范例文件。

## 1. 学习路线

| 对应范围 | 要回答的问题 | 学习成果 |
|---|---|---|
| 69–74 | 数字能用哪些进制表示？ | 分清数值和文字表示 |
| 75–81 | 如何取出需要的文字？ | 索引、切片、清洗、拆分、组合 |
| 82–100 | 多笔数据如何组织？ | 选对列表、元组、字典、集合 |
| 101–125 | 文字如何变成字节？ | 分清 `str`、`bytes`、编码、解码 |
| 126–167 | 如何保存、复制和修改文件？ | 理解模式、位置、路径和数据格式 |

上述是原笔记的概念分组，内容按理解顺序重排，不是逐集转录。多版本共存的重复内容合并说明。

## 2. 进制：数值相同，写法不同

二进制 `1010` 表示 `1×8 + 0×4 + 1×2 + 0×1`，所以是十进制的 10。数值没有变，变的是表示方式。

```python
number = 10
print(bin(number))
print(oct(number))
print(hex(number))
print(int("1010", 2))
print(0b1010 == 10)
# 输出：依次为 0b1010、0o12、0xa、10、True。
# 说明：前三个结果是字符串；int 按指定进制解读文字；0b1010 本身是整数。
```

追踪第一个转换：`number` 指向整数 10，`bin()` 取得这个值，返回字符串 `"0b1010"`，`print()` 显示字符串。不是将整数改成另一种型别的「二进制整数」。

| 进制 | 可用数字 | 字面量前缀 | 转成文字 |
|---|---|---|---|
| 二进制 | 0、1 | `0b` | `bin(n)` |
| 八进制 | 0–7 | `0o` | `oct(n)` |
| 十进制 | 0–9 | 无 | `str(n)` |
| 十六进制 | 0–9、a–f | `0x` | `hex(n)` |

`int(text, base)` 中，`text` 是交给函数处理的文字，`base` 指定它使用的进制，转换结果称为返回值。这只是使用现成函数，自订函数下一章才开始。

常见错误：`int("102", 2)` 会失败，因为二进制没有数字 2；`int("10")` 预设按十进制解读。

**练习：**心算 `int("1111", 2)`，再把 31 转为十六进制字符串。**自查简答：**15、`"0x1f"`。`bin(10)` 是 `str`；`0b1010` 是 `int`。

## 3. 字符串：先定位，再处理

### 3.1 索引取一个字符，切片取一段

把 `"Python"` 看成六个依序排列的字符。正索引从 0 起算；`-1` 是最后一个。切片格式为 `text[start:stop:step]`，包含起点，不包含终点。

```python
text = "Python"
print(text[0], text[-1])
print(text[1:4])
print(text[:2], text[2:])
print(text[::2])
print(text[::-1])
# 输出：P n；yth；Py thon；Pto；nohtyP（五行）。
# 说明：切片的 stop 不包含在结果；负步长表示朝反方向取值。
```

追踪 `text[1:4]`：取索引 1 的 `y`、2 的 `t`、3 的 `h`，到 4 就停，所以是三个字符。省略起点或终点时，Python 按步长方向选择边界。

索引超界会出错；切片超过尾端通常只取到现有数据。空字符串可以切片，不能取第 0 个字符。步长不能是 0。

```python
text = "cat"
print(text[:100])
try:
    print(text[100])
except IndexError:
    print("索引超出范围")
# 输出：cat；索引超出范围。
# 说明：捕捉错误让例子可以完整执行；异常处理在后续章节学习。
```

### 3.2 字符串不可原地修改

```python
name = "cat"
new_name = "b" + name[1:]
print(name, new_name)
name = name.upper()
print(name)
# 输出：cat bat；CAT。
# 说明：字符串内容没有原地改变；最后的赋值让 name 改为指向新字符串。
```

`name[0] = "b"` 会产生 `TypeError`。只写 `name.upper()` 而不使用结果，`name` 仍指向原字符串。

### 3.3 清洗 → 拆分 → 组合

```python
raw = "  apple,banana,orange  "
clean = raw.strip()
items = clean.split(",")
result = " / ".join(items)
print(repr(clean))
print(items)
print(result)
# 输出：'apple,banana,orange'；['apple', 'banana', 'orange']；apple / banana / orange。
# 说明：repr 显示引号便于观察；strip 返回 str，split 返回 list，join 返回 str。
```

`raw` 是一整段文字；`split()` 把它拆成列表；`join()` 把列表里的字符串接起来。分隔符写在 `join()` 前面，所有被连接的元素都必须是字符串。

`strip("ab")` 从两端移除属于 a 或 b 的字符，不是删掉固定前缀 `"ab"`。固定前缀可以用 `removeprefix()`；固定后缀用 `removesuffix()`（Python 3.9+）。

```python
print("abbahelloab".strip("ab"))
print("ababhello".removeprefix("ab"))
print("a  b\tc".split())
print("a,,b".split(","))
# 输出：hello；abhello；['a', 'b', 'c']；['a', '', 'b']。
# 说明：无参数 split 合并连续空白；指定逗号拆分时保留空栏位。
```

### 3.4 常用方法与格式化

```python
text = "Python Programming"
print(text.lower())
print(text.upper())
print(text.replace("Python", "Go"))
print(text.startswith("Py"), text.endswith("ing"))
print(text.find("Program"), text.find("Java"))
print(text.count("m"))
# 输出：python programming；PYTHON PROGRAMMING；Go Programming；True True；7 -1；2。
# 说明：find 找不到返回 -1；这些方法都不会原地改动 text。
```

只想知道文字是否存在，通常用 `"Python" in text`。不要直接把 `find()` 的结果当布尔值：索引 0 是假，-1 却是真。

```python
product = "笔记本"
price = 12.5
quantity = 2
print(f"{product} × {quantity} = {price * quantity:.2f} 元")
# 输出：笔记本 × 2 = 25.00 元。
# 说明：:.2f 控制显示两位小数，并不将浮点数变成精确的金额型别。
```

**练习：**把 `"  Alice, 85  "` 变成姓名 `"Alice"` 和整数 85；取出 `"order-2026-001"` 的最后三个字符。

**自查简答：**清理整段后用逗号拆分，再清理各栏位并 `int()`；最后三个字符用 `text[-3:]`。方法的返回值要使用或保存。

## 4. 四种容器：从需求决定结构

| 结构 | 如何取数据 | 可改容器本身吗 | 重复与顺序 |
|---|---|---|---|
| `list` 列表 | 位置索引 | 可以 | 有顺序，允许重复 |
| `tuple` 元组 | 位置索引 | 不可替换项目 | 有顺序，允许重复 |
| `dict` 字典 | 键，如商品 ID | 可以 | 键唯一；Python 3.7+ 保留插入顺序 |
| `set` 集合 | 成员是否存在 | 可以 | 元素唯一，不承诺迭代顺序 |

### 4.1 列表 CRUD：新增、查询、修改、删除

```python
products = ["铅笔", "橡皮擦"]
products.append("笔记本")
products.insert(1, "尺")
print(products)
print(products[0])
products[0] = "自动铅笔"
products.remove("尺")
removed = products.pop()
print(products, removed)
# 输出：['铅笔', '尺', '橡皮擦', '笔记本']；铅笔；['自动铅笔', '橡皮擦'] 笔记本。
# 说明：append 加一项；insert 按位置插入；remove 按值删第一个符合项；pop 删除并返回项目。
```

每操作一步就写出列表内容，比背方法名称有效。`remove(value)` 是按值删除，找不到会报 `ValueError`；`pop(index)` 是按索引删除，索引不存在会报 `IndexError`。`del products[0]` 也能删除，但不把删除项返回。

```python
numbers = [3, 1, 3]
numbers.append([8, 9])
print(numbers)
numbers = [3, 1, 3]
numbers.extend([8, 9])
print(numbers)
print(len(numbers), 8 in numbers, numbers.count(3))
# 输出：[3, 1, 3, [8, 9]]；[3, 1, 3, 8, 9]；5 True 2。
# 说明：append 将整个对象当一项加入；extend 将可迭代数据的各项加入。
```

### 4.2 排序与反转要看返回值

```python
numbers = [3, 1, 5]
new_numbers = sorted(numbers)
print(numbers, new_numbers)
result = numbers.sort()
print(numbers, result)
numbers.reverse()
print(numbers)
# 输出：[3, 1, 5] [1, 3, 5]；[1, 3, 5] None；[5, 3, 1]。
# 说明：sorted 产生新列表；sort 和 reverse 修改原列表，返回 None。
```

不要写 `numbers = numbers.sort()`，那会让 `numbers` 指向 `None`。降幂排序可用 `numbers.sort(reverse=True)`；反转只把当前顺序倒过来，不等于降幂排序。

### 4.3 元组：固定的是项目的对应

```python
point = (10, 20)
x, y = point
one = (1,)
print(x, y)
print(type(one).__name__, type((1)).__name__)
record = ("Alice", [80, 90])
record[1].append(100)
print(record)
# 输出：10 20；tuple int；('Alice', [80, 90, 100])。
# 说明：元组的项目不能被替换，但项目指向的列表仍可修改；单元素元组需要逗号。
```

「元组不可变」不代表里面所有对象都不可变。`record[1] = []` 不合法；`record[1].append(100)` 改的是内层列表，所以可以。元组适合座标、固定栏位的纪录、函数的多个结果。

### 4.4 字典：用有意义的键定位

```python
product = {"id": "P001", "name": "笔记本", "stock": 8}
product["stock"] = 6
product["price_cents"] = 1250
print(product["name"])
print(product.get("color", "未提供"))
for key, value in product.items():
    print(key, value)
# 输出：笔记本；未提供；接着为 id P001、name 笔记本、stock 6、price_cents 1250。
# 说明：新键是新增，已有键是更新；items 每次提供一组键和值。
```

`key in product` 检查键。直接迭代字典也是取得键；`keys()`、`values()`、`items()` 分别提供键、值、键值对。`get()` 找不到时只返回预设值，不会新增键。`pop(key)` 删除并返回值；不确定存在时可用 `pop(key, None)`。

键必须可杂凑；常用字符串或整数。元组只有在所有元素都可杂凑时才可当键，包含列表的元组不行。「不可变」和「可杂凑」不能当成完全相同的定义。

多笔商品可用字典套字典，方便按 ID 查找：

```python
products = {
    "P001": {"name": "笔记本", "stock": 8},
    "P002": {"name": "铅笔", "stock": 20},
}
product_id = "P002"
if product_id in products:
    products[product_id]["stock"] -= 2
print(products["P002"])
# 输出：{'name': '铅笔', 'stock': 18}。
# 说明：先用商品 ID 取得一笔商品，再用 stock 取得栏位。
```

若重点是按顺序处理多笔记录，也可用「列表里放字典」。数据格式要服务查询需求。

### 4.5 集合：关心有没有，不关心第几个

```python
a = {"pen", "book"}
b = {"book", "ruler"}
print(sorted(a | b))
print(sorted(a & b))
print(sorted(a - b))
empty = set()
empty.add("pen")
empty.add("pen")
empty.discard("missing")
print(len(empty))
# 输出：['book', 'pen', 'ruler']；['book']；['pen']；1。
# 说明：| 是联集，& 是交集，- 是差集；sorted 只为稳定展示，集合本身不承诺顺序。
```

`{}` 是空字典，空集合用 `set()`。集合元素也必须可杂凑。`remove()` 删不存在的元素会报错，`discard()` 不会。`list(set(data))` 能去重，但不能保证保留原顺序。

### 4.6 堆叠与伫列：取出的规则不同

```python
stack = []
stack.append("A")
stack.append("B")
print(stack.pop())
queue = ["A", "B"]
print(queue.pop(0))
# 输出：B；A。
# 说明：堆叠后进先出，伫列先进先出；列表 pop(0) 需移动后面的项目，只适合小型示范。
```

大量伫列操作通常用 `collections.deque`。现在先理解规则，不必立即引入新结构。

**练习：**按商品 ID 查询选什么？找两人共同买过的商品用什么？

**自查简答：**按 ID 查询适合字典；共同商品适合集合交集。列表按索引取值；字典按键取值，数字键也不是列表索引。

## 5. 引用与复制：为什么改 B，A 也变了？

### 5.1 赋值创建名称关系

```python
a = [1, 2]
b = a
b.append(3)
print(a, b)
print(a is b)
b = [9]
print(a, b)
# 输出：[1, 2, 3] [1, 2, 3]；True；[1, 2, 3] [9]。
# 说明：b = a 不复制列表；append 改共享对象，b = [9] 只重新系结 b。
```

画两个箭头 `a`、`b` 指向同一列表，就容易看懂。`==` 比较值是否相等；`is` 比较是否同一对象。一般数字、字符串的相等判断使用 `==`。

### 5.2 浅拷贝与深拷贝

```python
import copy

a = [[1, 2], [3, 4]]
shallow = a.copy()
deep = copy.deepcopy(a)
shallow.append([5, 6])
shallow[0].append(9)
deep[1].append(8)
print(a)
print(shallow)
print(deep)
# 输出：[[1, 2, 9], [3, 4]]；[[1, 2, 9], [3, 4], [5, 6]]；[[1, 2], [3, 4, 8]]。
# 说明：浅拷贝只建新外层；深拷贝让本例的巢状列表也分开。
```

`shallow.append()` 改新外层，`a` 没多一项；`shallow[0].append()` 改共享内层，`a[0]` 也变了。这里讨论一般巢状数据，`deepcopy()` 并不是所有文件资源或外部连线都能复制的工具。

### 5.3 `+=` 不总等于 `+` 再赋值

```python
a = [1]
b = a
a += [2]
print(a, b, a is b)
a = [1]
b = a
a = a + [2]
print(a, b, a is b)
# 输出：[1, 2] [1, 2] True；[1, 2] [1] False。
# 说明：列表 += 原地扩充；列表 + 创建新列表，再由赋值改变 a 的指向。
```

整数 `count += 1` 可先理解为算出新值再系结，但不能把这个模型套到所有型别。

**练习：**用一句话说明 `a.copy()` 为什么不能隔离内层列表的修改。

**自查简答：**只复制外层容器，内层项目仍指向原对象。除错先分清「重新系结名称」与「修改共享对象」。

## 6. 编码：文字与字节之间的转换

`str` 表示文字，`bytes` 表示字节序列。保存或传输文字时需要编码，读回字节时需要解码。

```text
str ── encode("utf-8") ──→ bytes
str ←── decode("utf-8") ── bytes
```

```python
text = "好"
data = text.encode("utf-8")
print(data)
print(len(text), len(data))
print(data.decode("utf-8"))
# 输出：b'\xe4\xbd\xa0\xe5\xa5\xbd'；2 6；好。
# 说明：本例每个中文字占三个 UTF-8 字节；len(str) 和 len(bytes) 的单位不同。
```

`len(str)` 计算 Unicode 码位，不保证等于画面看见的完整字形数；例如某些 emoji 含多个码位。初学先用一般中英文字理解。

ASCII 主要涵盖英文和数字；Unicode 定义字符及码位；UTF-8、UTF-16 是把 Unicode 文字编成字节的方式；GBK 是常见的传统中文编码。Unicode 和 UTF-8 不在同一层。

```python
data = "好".encode("utf-8")
try:
    print(data.decode("ascii"))
except UnicodeDecodeError:
    print("这些字节不能按 ASCII 解码")
# 输出：这些字节不能按 ASCII 解码。
# 说明：刻意使用不兼容的 ASCII；错用其他编码也可能不报错却得到乱码。
```

排查顺序：确认来源编码 → 检查读写的 `encoding` → 检查编辑器与终端。不要为了不报错而随便使用 `errors="ignore"`，它会丢掉数据。

Python 2 已停止支援；影片的 `raw_input()`、Python 2 字符串行为作历史理解。Python 3 使用 `input()`。多版本共存时，确认编辑器选的直译器与安装包用的是同一个环境。

### 6.1 源代码文件头和外部文件编码不是同一件事

Python 3 源代码预设按 UTF-8 解读，通常不必写编码宣告。旧教材中的 `# coding: utf-8` 是在指定**这个 Python 源代码文件**的编码，不会替 `open()` 读写的所有外部文件设定编码。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
print("源代码中的中文")
# 输出：源代码中的中文。
# 说明：编码宣告示范放在前两行；shebang 在支援它的启动环境用来选择直译器，对 Python 语法而言是注解。
```

shebang 常见于 Unix 类系统可直接执行的脚本；Windows 的 Python launcher 也可能解读它，但不应假定任何编辑器或启动方式都据此选版本。执行 `python 文件名.py` 时，先由选的 `python` 决定执行环境。Python 3 仍可能因外部数据编码或终端显示设定不符而乱码，不能把「Python 3」当作永不乱码的保证。

**练习：**两个中文字为什么可能占六个字节？**自查简答：**文字长度和 UTF-8 字节长度的单位不同；编码是 `str → bytes`，解码反向。

## 7. 文件最小流程：开启 → 使用 → 关闭

### 7.1 不依赖外部文件的第一个例子

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "note.txt"
    with open(path, "w", encoding="utf-8") as file:
        count = file.write("好\nPython")
    with open(path, "r", encoding="utf-8") as file:
        content = file.read()
    print(count)
    print(content)
    print(file.closed)
# 输出：9；好；Python；True（四行）。
# 说明：write 返回字符数；read 返回文字；内层 with 关闭文件，外层结束后移除暂存数据。
```

`TemporaryDirectory()` 提供暂存文件夹；`Path(folder) / "note.txt"` 组成路径。它们是让范例可安全重跑的准备工具，当前先理解用途即可。

`with open(...) as file` 将文件对象交给 `file`；离开区块会关闭文件，包括因一般异常离开的情况。`with` 是上下文管理语法，不是所有用途都等同于手写 `try/finally`；对内建文件对象，重点是可靠关闭资源。

### 7.2 路径相对于哪里？

相对路径依**当前工作目录**解读，不必然依 `.py` 所在位置。绝对路径指定完整位置；`.` 是当前目录，`..` 是上一层。找不到文件时先确认工作目录、文件名和副文件名。

```python
from pathlib import Path

print(Path.cwd())
print(Path("data") / "users.txt")
# 输出：第一行是本次执行的工作目录；第二行是使用当前平台分隔符的 data/users.txt 路径。
# 说明：只显示路径，不创建文件；Path 处理平台分隔符。
```

### 7.3 模式：`+` 不代表追加

| 模式 | 读写 | 不存在时 | 已存在时 |
|---|---|---|---|
| `r` | 读 | 报错 | 从开头读 |
| `w` | 写 | 创建 | **开启时立即清空** |
| `a` | 写 | 创建 | 写入追加到尾端 |
| `x` | 写 | 创建 | 报 `FileExistsError` |
| `r+` | 读写 | 报错 | 不清空，从开头开始 |
| `w+` | 读写 | 创建 | **开启时立即清空** |
| `a+` | 读写 | 创建 | 写入追加；读取前通常需 `seek(0)` |

`t` 是文字模式（预设），`b` 是二进制模式，例如 `rb`、`wb`。`+` 是同时读写，追加要看 `a`。文字模式建议明写 `encoding="utf-8"`，二进制模式不能指定 `encoding`。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "log.txt"
    with open(path, "w", encoding="utf-8") as file:
        file.write("A\n")
    with open(path, "a", encoding="utf-8") as file:
        file.write("B\n")
    print(repr(path.read_text(encoding="utf-8")))
    with open(path, "w", encoding="utf-8") as file:
        file.write("C\n")
    print(repr(path.read_text(encoding="utf-8")))
# 输出：'A\nB\n'；'C\n'。
# 说明：a 保留原内容并追加；第二次 w 在开启时就清空 A 和 B。
```

### 7.4 读取后，位置往前移

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "lines.txt"
    path.write_text("A\nB\nC", encoding="utf-8")
    with open(path, "r", encoding="utf-8") as file:
        first = file.readline()
        rest = file.readlines()
        after_end = file.read()
    print(repr(first))
    print(rest)
    print(repr(after_end))
# 输出：'A\n'；['B\n', 'C']；''。
# 说明：readline 读一行；readlines 读剩余各行；到文件末尾后再 read 返回空字符串。
```

| 方法 | 文字模式返回值 | 使用场合 |
|---|---|---|
| `read()` | 剩余全部 `str` | 小文件全文处理 |
| `read(n)` | 最多 n 个字符 | 限量读取 |
| `readline()` | 下一行 `str` | 一次一行，通常含换行 |
| `readlines()` | 剩余各行的列表 | 确实需要完整行列表 |
| `for line in file` | 每次取得一行 | 大文件逐行处理 |

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "names.txt"
    path.write_text(" Alice \nBob\n", encoding="utf-8")
    with open(path, "r", encoding="utf-8") as file:
        for line in file:
            print(repr(line.rstrip("\r\n")))
# 输出：' Alice '；'Bob'。
# 说明：只去掉换行，保留原有空格；strip 会额外移除首尾其他空白。
```

### 7.5 写入不会自动补换行

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "result.txt"
    with open(path, "w", encoding="utf-8") as file:
        count = file.write("A")
        result = file.writelines(["B\n", "C\n"])
    print(path.read_text(encoding="utf-8"), end="")
    print(count, result)
# 输出：AB；C；1 None（三行）。
# 说明：两个方法都不自动补换行；write 返回字符数，writelines 返回 None。
```

常见转义包含换行、定位字符、反斜线和引号。文字模式通常会处理平台换行差异，所以「写入字符数」不必然等于「硬盘字节数」。

回车 `\r` 原意是回到行首，换行 `\n` 原意是移到下一行；Windows 文字文件常见 `\r\n`，Unix 类系统常见 `\n`。在终端显示时，单独回车可能让后续文字从行首覆写，看起来与真正换行不同。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "newlines.txt"
    path.write_bytes(b"A\r\nB\rC\n")
    with open(path, "r", encoding="utf-8", newline=None) as file:
        print(repr(file.read()))
    with open(path, "r", encoding="utf-8", newline="") as file:
        print(repr(file.read()))
# 输出：'A\nB\nC\n'；'A\r\nB\rC\n'。
# 说明：预设 newline=None 读取时统一换行；newline="" 保留读到的换行形式。
```

文字写入时，`newline=None` 可能把换行转成平台形式；二进制模式则原样处理字节。一般文字先用预设，只有需求明确时才手动控制 `newline`。

### 7.6 `tell()`、`seek()`：位置不是文字索引

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "position.txt"
    path.write_text("好ABC", encoding="utf-8")
    with open(path, "r", encoding="utf-8") as file:
        print(file.read(1))
        position = file.tell()
        print(file.read(1))
        file.seek(position)
        print(file.read(1))
        file.seek(0)
        print(file.read(2))
# 输出：；好；好；好。
# 说明：tell 的值可供 seek 恢复位置；文字模式不要把它当成第几个字符。
```

文字模式的 `tell()` 是可用于恢复位置的值，不要对它做任意字符偏移运算。先熟悉 `seek(0)` 与存储后再恢复 `tell()` 的结果。二进制模式则按字节定位。

### 7.7 缓冲与 `flush()`：交出去，不代表已经可靠落盘

为降低 I/O 次数，Python 可能先把写入数据放在缓冲区；操作系统也可能保留自己的快取。机械硬盘有寻道等成本，固态硬盘没有同样的机械寻道，但也仍有 I/O 延迟。批量写入、逐行处理与分段搬运，是在延迟、内存用量与程序简单度之间取平衡。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "buffered.bin"
    with open(path, "wb") as writer:
        writer.write(b"ready\n")
        writer.flush()
        print(path.read_bytes())
# 输出：b'ready\n'。
# 说明：flush 将 Python 写入缓冲交给下层；正常关闭也会刷新，但它们都不等于保证断电后数据已持久保存。
```

需要更强的持久化要求时，会再涉及 `os.fsync()`、文件系统与硬件保证；本章先分清 `write()`、`flush()`、关闭与持久保存不是同一件事。也不要每写一个字符就 `flush()`，那可能降低缓冲的效益。

### 7.8 读取后续新增数据：有限次的监看模型

普通文件读到文件末尾后，若之后有新数据追加，可以从当前位置再读。以下用两次追加模拟日志新增，不会启动无限循环。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "events.log"
    path.write_bytes(b"old record\n")
    with open(path, "rb") as reader:
        reader.seek(0, 2)
        print(reader.readline() == b"")
        for message in [b"new 1\n", b"new 2\n"]:
            with open(path, "ab") as writer:
                writer.write(message)
            line = reader.readline()
            print(line.decode("utf-8").rstrip("\r\n"))
# 输出：True；new 1；new 2。
# 说明：先定位末尾跳过旧内容；每次追加并关闭写入者后，读取者从原位置读到新行。
```

追踪：初次文件末尾读到空字节 → 追加第一行 → 从原文件末尾读第一行 → 再追加第二行 → 接着读第二行。真实轮询若暂时没数据，要有短暂等待与停止条件，避免空转；还需考虑半行数据、文件截断和替换。这个教学例只处理同一文件追加完整行的情况。

**练习：**读完后第二次 `read()` 为何为空？`w+` 能保护原内容吗？

**自查简答：**位置已到文件末尾，重新读前用 `seek(0)`；`w+` 仍会清空原内容。

## 8. 二进制模式、复制与修改

### 8.1 同一文件，两种解读方式

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "note.txt"
    path.write_bytes("好".encode("utf-8"))
    with open(path, "r", encoding="utf-8") as file:
        text = file.read()
    with open(path, "rb") as file:
        data = file.read()
    print(type(text).__name__, len(text))
    print(type(data).__name__, len(data))
# 输出：str 2；bytes 6。
# 说明：文字模式自动解码，二进制模式保留原始字节。
```

图片、压缩文件、音讯通常用二进制模式。文字 `write()` 需要 `str`；二进制 `write()` 需要如 `bytes` 的字节类数据。

### 8.2 分段复制：一次搬一小箱

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    source_path = Path(folder) / "source.bin"
    target_path = Path(folder) / "target.bin"
    source_path.write_bytes(b"ABCDEFGHIJ")
    with open(source_path, "rb") as source, open(target_path, "wb") as target:
        while True:
            chunk = source.read(4)
            if not chunk:
                break
            target.write(chunk)
    print(target_path.read_bytes())
    print(source_path.read_bytes() == target_path.read_bytes())
# 输出：b'ABCDEFGHIJ'；True。
# 说明：依次读 ABCD、EFGH、IJ、空字节；空值代表读完。
```

本例每次 4 字节便于观察，实际可用 `1024 * 1024`（1 MiB）等区块。目的主要是限制内存使用，不是保证更快。I/O 常比内存操作慢，避免逐字符搬运和反复开关文件。

### 8.3 修改：先写新文件，再替换

文字文件通常不能直接在中间插入任意长度数据。小文件可整个读进内存修改；大文件可逐行读原文件、写新文件。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    original = Path(folder) / "products.txt"
    temporary = Path(folder) / "products.new.txt"
    original.write_text("P001,8\nP002,20\n", encoding="utf-8")
    with open(original, "r", encoding="utf-8") as source:
        with open(temporary, "w", encoding="utf-8") as target:
            for line in source:
                product_id, stock = line.rstrip("\r\n").split(",")
                if product_id == "P001":
                    stock = "6"
                target.write(f"{product_id},{stock}\n")
    temporary.replace(original)
    print(original.read_text(encoding="utf-8"), end="")
# 输出：P001,6；P002,20（两行）。
# 说明：新文件完整写好并关闭后才替换原文件；只操作自己的暂存数据。
```

重要数据还需备份与失败恢复；这只是一个基本保存策略，并非完整交易系统。

## 9. 综合：读取帐号、登入与注册

先约定格式：每行 `帐号:练习密码`，按第一个冒号分隔。帐号不能含冒号或换行，密码不能含换行。真实帐号不能明文保存密码，这里只练解析和写文件。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "users.txt"
    path.write_text("alice:demo123\n\nbad_line\nbob:abc:123\n", encoding="utf-8")
    users = {}
    invalid_count = 0
    with open(path, "r", encoding="utf-8") as file:
        for line in file:
            line = line.rstrip("\r\n")
            if not line:
                continue
            if ":" not in line:
                invalid_count += 1
                continue
            username, password = line.split(":", 1)
            if not username or not password or username in users:
                invalid_count += 1
                continue
            users[username] = password
    print(users)
    print("无效行数：", invalid_count)
    print("登入成功：", users.get("alice") == "demo123")
    new_name, new_password = "carol", "demo456"
    if new_name in users:
        print("帐号已存在")
    else:
        users[new_name] = new_password
        with open(path, "a", encoding="utf-8") as file:
            file.write(f"{new_name}:{new_password}\n")
        print("注册完成：", new_name)
# 输出：{'alice': 'demo123', 'bob': 'abc:123'}；无效行数： 1；登入成功： True；注册完成： carol。
# 说明：略过空行，拒绝格式错误与重复帐号；split(":", 1) 保留密码中的冒号。
```

追踪 `"bob:abc:123"`：最多拆一次，得到 `"bob"` 和 `"abc:123"`，再存进字典。此例使用确定符合规则的新增数据；改成 `input()` 时，必须先验证帐号和密码格式。尚未处理多人同时写文件。

再用学完的字符串与字典完成单字计数：

```python
sentence = "Python is easy and Python is useful"
words = sentence.lower().split()
counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1
print(counts)
# 输出：{'python': 2, 'is': 2, 'easy': 1, 'and': 1, 'useful': 1}。
# 说明：第一次 get 返回 0，存入 1；再次遇到同字时沿用旧次数加一。
```

这个版本不处理标点，也不是中文分词器。扩充前先定义「单字」的规则，避免把不清楚的需求藏程序。

## 10. 常见错误与排查

| 现象 | 先查哪里 | 修正方向 |
|---|---|---|
| `IndexError` | 索引与长度 | 检查长度或使用合理切片 |
| `KeyError` | 键是否存在 | 存在性判断或 `get()` |
| `ValueError` | 数字格式、拆分项数 | 验证后再转换或解包 |
| `TypeError` | 操作与型别 | 留意 `str`/`bytes`、不可变字符串、可杂凑限制 |
| `FileNotFoundError` | 路径与初始化 | 确认工作目录和文件名 |
| `UnicodeDecodeError` | 来源编码 | 使用实际编码 |
| 列表变成 `None` | 是否接了 `sort()` 结果 | 分清原地修改与返回新对象 |
| 文件内容消失 | 是否开了 `w` | 分清覆写和追加 |
| 改 B 连 A 也改 | 可变对象是否共享 | 画引用关系，选合适复制方式 |

## 11. 章末练习与自查

1. **文字清洗：**将 `"  apple, banana,, apple  "` 清洗成非空品项列表，统计次数。验收：apple 2 次、banana 1 次。
2. **商品容器：**创建三笔商品，含 ID、名称、库存。新增、修改、删除各一笔，再查询不存在的 ID。验收：不存在时提示，不中断。
3. **保存数据：**把 ID 和库存写入练习文件，重新读回。验收：库存读回后是整数。
4. **复制工具：**分段复制自己创建的文件。验收：来源与目标字节相同，空文件也能处理。
5. **学生管理草稿：**先列格式、空数据和重复姓名规则，再实作新增、查询、平均分。下一章再拆成函数。

自查简答：

- 为何拆分后还要清理栏位？整段 `strip()` 只处理最外侧，不处理每个逗号旁的空白。
- 重复字典键会如何？后次赋值覆盖前值；若不允许重复，要先检查。
- `read(4)` 一定读四字节吗？文字模式是最多四个字符，二进制模式才是最多四字节。
- 文件关闭后 `content` 还能用吗？可以；关闭文件资源不会移除已读入内存的数据。
- 怎样算学会？能解释容器选择、预测引用修改结果，并完成写入后重新读回。

下一章将重复流程命名为函数，让每一段处理都有清楚的输入和输出。
