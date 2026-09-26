---
title: Python 函数：参数、返回值与代码复用
published: 2026-09-11
updated: 2026-09-14
description: 理解函数的定义和调用，掌握参数绑定、默认值、可变参数、返回值，以及如何拆分程序责任。
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第三章　函数：替一段工作取名字（第 168–195 集）

当新增学生、验证分数、统计平均分的程序开始重复，应该把它们拆成有名字的小工作。本章解决的是：「如何让一段程序可以重复使用，并清楚知道它需要什么、产生什么？」

前置知识：判断、循环、列表、字典、字符串拆分。学习路线是：定义与调用 → 返回值 → 参数绑定 → 默认值 → 收集与解包 → 拆分一个完整流程。先不学装饰器。

每个 Python 程序代码框都可单独执行。先预测画面输出，再执行；特别分清「画面打印了什么」与「调用者拿到什么」。

## 1. 函数是有名字的一组步骤

### 1.1 定义是准备，调用才执行

```python
def greet():
    print("好，Python！")


print("先准备")
greet()
print("已完成")
# 输出：先准备；好，Python！；已完成。
# 说明：def 创建函数对象并绑定名称；执行 greet() 才进入函数本体。
```

逐步追踪：

1. Python 执行到 `def`，创建函数，让名称 `greet` 指向它；不执行本体里的 `print()`。
2. 显示「先准备」。
3. 遇到 `greet()`，进入函数本体，显示问候。
4. 函数结束，回到调用之后的位置，显示「已完成」。

`greet` 是函数对象，`greet()` 是调用动作。空函数可以暂用 `pass` 占位。函数本体的缩排决定哪些步骤属于函数。

### 1.2 参数让同一份步骤处理不同数据

```python
def greet_user(name):
    print(f"好，{name}！")


greet_user("小明")
greet_user("小华")
# 输出：好，小明！；好，小华！
# 说明：每次调用会将该次传入的文字绑定到 name，函数本体只需写一次。
```

定义括号里的 `name` 叫**形参**，是等待数据的名称；调用括号里的 `"小明"` 叫**实参**，是这次提供的数据。形参不是「所有调用共用一个全局变量」，每次调用都有自己的局部绑定。

**练习：**写 `show_product(name, stock)`，显示商品名称与库存。**自查简答：**两个形参在 `def` 中，两个实参在调用时提供；只定义不调用不会显示商品。

## 2. `print()` 显示数据，`return` 交还数据

### 2.1 计算结果要能接着使用

```python
def add(a, b):
    return a + b


result = add(3, 5)
print(result)
print(result * 2)
# 输出：8；16。
# 说明：return 将 8 交还调用者；result 保存它，之后还可以参与计算。
```

追踪 `result = add(3, 5)`：先调用 `add` → 形参为 `a=3`、`b=5` → 算出 8 → `return 8` → 调用表达式得到 8 → 将 8 赋值给 `result`。`return` 本身不会在终端显示结果。

```python
def show_total(a, b):
    print(a + b)


value = show_total(3, 5)
print(value)
# 输出：8；None。
# 说明：函数显示了 8，但没有 return 值；执行完毕时隐式返回 None。
```

这是初学者最容易混淆的一点：**看见 8，不代表调用者得到 8。**没有 `return`，或只写 `return`，都会返回 `None`。`None` 是「没有具体结果」的特殊值，不是字符串 `"None"`，也不是数字 0。

### 2.2 `return` 立即结束本次函数调用

```python
def divide(a, b):
    if b == 0:
        return None
    return a / b


print(divide(10, 2))
print(divide(10, 0))
# 输出：5.0；None。
# 说明：除数为 0 时先返回，不执行除法；本例约定 None 表示无法计算。
```

返回之后同一路径上的后续步骤不再执行。`return` 结束函数；`break` 只跳出最近一层循环；`continue` 只进入下一次循环。不要把它们混为一谈。

### 2.3 多个结果其实是一个元组

```python
def analyze(scores):
    if not scores:
        return None
    return min(scores), max(scores), sum(scores) / len(scores)


result = analyze([70, 80, 90])
print(result)
minimum, maximum, average = result
print(minimum, maximum, average)
print(analyze([]))
# 输出：(70, 90, 80.0)；70 90 80.0；None。
# 说明：逗号将三个结果打包为元组；左侧三个名称再解包；空列表有明确返回规则。
```

调用者应先判断结果是否为 `None`，再解包，不能把空数据的结果当成三个值。常用 `result is None` 判断这个特殊值。

**练习：**写 `find_max_min(numbers)`，非空返回最大与最小值，空列表返回 `None`。**自查简答：**要两个结果仍只使用一次 `return maximum, minimum`；分两行 `return`，第二行不会被执行。

## 3. 参数是如何对上的？

### 3.1 位置与关键字是两种传入方式

```python
def introduce(name, age):
    return f"{name} 今年 {age} 岁"


print(introduce("小明", 18))
print(introduce(age=18, name="小明"))
print(introduce("小明", age=18))
# 输出：三次都是 小明 今年 18 岁。
# 说明：位置实参依顺序绑定，关键字实参依名称绑定；这些形参都允许两种方式。
```

一般调用时，先写普通位置实参，再写关键字实参。关键字名称必须对得上形参，除非函数另外使用 `**kwargs` 收集它。

以下捕捉两种错误，方便观察而不中断例子：

```python
def introduce(name, age):
    return f"{name} 今年 {age} 岁"


try:
    introduce("小明", name="小华", age=18)
except TypeError:
    print("name 被提供了两次")
try:
    introduce("小明")
except TypeError:
    print("缺少 age")
# 输出：name 被提供了两次；缺少 age。
# 说明：第一个位置已绑定 name，不能再给同名关键字；必要参数也不能漏掉。
```

### 3.2 默认值是「没提供时用什么」

```python
def greet(name, message="欢迎"):
    return f"{message}，{name}！"


print(greet("小明"))
print(greet("小华", "早安"))
print(greet("小明", message=""))
# 输出：欢迎，小明！；早安，小华！；，小明！
# 说明：只有没传 message 时才使用默认值；明确传空字符串仍是有效的传入值。
```

对一般可按位置传入的形参，没有默认值的放前，有默认值的放后。此规则不应简化成「任何必要参数都只能在所有默认参数前面」；仅关键字参数有不同规则，下一小节补充。

### 3.3 阅读补充：`*` 之后只能用关键字

```python
def make_label(name, *, prefix="商品"):
    return f"{prefix}：{name}"


print(make_label("笔记本", prefix="新品"))
# 输出：新品：笔记本。
# 说明：星号之后的 prefix 是仅限关键字参数，不能以第二个普通位置实参传入。
```

若看见 `def f(a, /, b)`，斜线之前的 `a` 只能按位置传入；`*` 之后只能按关键字传入。初学先能读懂，不必为每个函数加入这些限制。

**自查简答：**形参指函数定义中的名称；位置／关键字描述调用时如何给值。不能把「有默认值」误当成「只能用关键字」。

## 4. 默认值只在定义时创建一次

这一节解决「明明调用两次，为什么第二次混入第一次的数据？」

```python
def add_item(item, items=[]):
    items.append(item)
    return items


print(add_item("A"))
print(add_item("B"))
# 输出：['A']；['A', 'B']。
# 说明：这是刻意展示的错误设计；默认列表在定义时创建一次，多次省略 items 会共享它。
```

不是所有默认参数都有问题；问题在于修改了跨调用共享的可变默认对象。希望每次默认使用新列表，可用 `None` 表示尚未提供。

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items


print(add_item("A"))
print(add_item("B"))
existing = ["C"]
print(add_item("D", existing))
print(existing)
# 输出：['A']；['B']；['C', 'D']；['C', 'D']。
# 说明：省略时每次建新列表；明确传入 existing 时，本例按约定修改调用者的列表。
```

追踪：第一次 `items` 为 `None`，创建新列表加 A；第二次也从 `None` 开始，所以得到另一个列表。第三次收到 `existing` 的引用，没有新建，会改动它。

不要随便写成 `items = items or []`，那会把调用者刻意提供的空列表也替换掉。`is None` 才精确对应「未提供」的约定。

**练习：**把默认字典 `options={}` 改成每次独立创建。**自查简答：**默认设为 `None`，函数内 `if options is None: options = {}`。

## 5. 传入列表后，函数能不能改到外面？

Python 调用函数时，形参会绑定到实参所代表的对象。不是把整个列表复制一份，也不是一律「只能传值／只能传址」的简单二分。

```python
def append_score(scores):
    scores.append(90)


def replace_scores(scores):
    scores = [100]
    return scores


original = [80]
append_score(original)
print(original)
new_scores = replace_scores(original)
print(original, new_scores)
# 输出：[80, 90]；[80, 90] [100]。
# 说明：append 修改共同列表；scores = [100] 只重新绑定函数内的名称。
```

若函数按设计要修改传入的列表或字典，名称和说明要让调用者知道。若想保持原数据，可先复制，并确认浅拷贝是否足够。

## 6. `*args` 与 `**kwargs`：收集未被接走的参数

### 6.1 定义时 `*args` 收集位置实参

```python
def total(*numbers):
    print(numbers)
    result = 0
    for number in numbers:
        result += number
    return result


print(total(1, 2, 3))
print(total())
# 输出：(1, 2, 3)；6；()；0。
# 说明：星号将位置实参收集为元组；名称不必是 args，numbers 在本例更有意义。
```

`args` 是惯用名称，不是关键字；真正控制收集的是前面的 `*`。

### 6.2 `**kwargs` 收集关键字实参

```python
def build_student(name, score, **extra):
    student = {"name": name, "score": score}
    student.update(extra)
    return student


print(build_student("小明", 88, city="台北", club="程序社"))
# 输出：{'name': '小明', 'score': 88, 'city': '台北', 'club': '程序社'}。
# 说明：name、score 先由普通形参接收，其余关键字收集成 extra 字典。
```

可变长参数适合数量真的不固定的场合。若函数只需要商品 ID 与数量，直接写两个明确形参通常更好读。

### 6.3 混用时，按类别接收

```python
def describe(title, *items, separator=" / ", **options):
    print(title)
    print(separator.join(items))
    print(options)


describe("购物车", "铅笔", "笔记本", separator="、", color="green")
# 输出：购物车；铅笔、笔记本；{'color': 'green'}。
# 说明：title 先接第一项，items 收集其余位置项，separator 仅限关键字，options 收集剩余关键字。
```

追踪不能只数括号里有几个值，还要看它们是位置实参或关键字实参、是否已被具名形参接走。

## 7. 调用时 `*`、`**` 是展开，不是收集

```python
def add(a, b, c):
    return a + b + c


numbers = [1, 2, 3]
options = {"a": 1, "b": 2, "c": 3}
print(add(*numbers))
print(add(**options))
# 输出：6；6。
# 说明：调用中的 * 展开为位置实参；** 展开字典为关键字实参。
```

| 出现位置 | `*` | `**` |
|---|---|---|
| 函数定义 | 收集多个位置实参为元组 | 收集多个关键字实参为字典 |
| 函数调用 | 将可迭代数据展成位置实参 | 将映射展成关键字实参 |

`add(numbers)` 只传了一个列表，不等于 `add(*numbers)`。`**options` 的键必须是字符串，且最终参数要符合被调用函数的规则；与已提供的参数重复仍会出错。

**练习：**用 `values=[2,3]` 调用 `power(base, exponent)`。**自查简答：**`power(*values)`，等同 `power(2, 3)`；星号不会把函数自动改成接受任意参数。

## 8. 函数设计：一个名称对应一项主要责任

一个函数「只有一项主要责任」，不代表只能有一行；而是用户能用一句话说清它做什么。例如：把文字转成分数列表、分析分数、显示分析结果。这三件事变更的理由不同，分开后更容易验证。

先写三项约定：

| 函数 | 输入 | 返回与副作用 |
|---|---|---|
| `parse_scores(text)` | 空白分隔的整数文字 | 合法返回列表，错误返回 `None`；不打印 |
| `analyze_scores(scores)` | 分数列表 | 非空返回三项元组，空列表返回 `None` |
| `show_analysis(result)` | 统计结果或 `None` | 显示结果，隐式返回 `None` |

```python
def parse_scores(text):
    scores = []
    for item in text.split():
        try:
            score = int(item)
        except ValueError:
            return None
        if not 0 <= score <= 100:
            return None
        scores.append(score)
    return scores


def analyze_scores(scores):
    if not scores:
        return None
    return min(scores), max(scores), sum(scores) / len(scores)


def show_analysis(result):
    if result is None:
        print("没有分数可以统计")
        return
    minimum, maximum, average = result
    print(f"最低分：{minimum}")
    print(f"最高分：{maximum}")
    print(f"平均分：{average:.2f}")


text = "70 80 90"
scores = parse_scores(text)
if scores is None:
    print("请输入 0 到 100 的整数分数")
else:
    show_analysis(analyze_scores(scores))
# 输出：最低分：70；最高分：90；平均分：80.00。
# 说明：数据依序通过转换、分析、显示；None 表示非法输入，空列表表示没有输入分数。
```

此处 `try/except ValueError` 只为辨识整数转换失败，后续异常章会深入。先用固定文字让例子可重跑，再将 `text = "70 80 90"` 改成 `text = input("请输入分数：")`。这样输入方式改变时，分析函数不用重写。

用以下输入自己追踪一次：

| `text` | 解析结果 | 后续行为 |
|---|---|---|
| `"70 80 90"` | `[70, 80, 90]` | 正常分析 |
| `""` | `[]` | 显示没有分数 |
| `"80 abc"` | `None` | 显示输入错误 |
| `"101"` | `None` | 显示超范围的输入错误 |

若把输入、转换、计算、打印和文件保存全塞进一个函数，要测一次平均值就得走完全部流程。分开后，`analyze_scores([70, 80, 90])` 可直接验证。

## 9. 常见错误查阅表

| 现象 | 原因 | 修正方向 |
|---|---|---|
| 定义了却没输出 | 没调用 | 分清 `func` 和 `func()` |
| 打印正确值，接到 `None` | 只有 `print` 没 `return` | 计算函数返回结果 |
| `TypeError` 缺少参数 | 调用不符合形参 | 依位置和名称画绑定表 |
| 重复提供同一参数 | 位置已提供，又写关键字 | 移除重复 |
| 第二次混入第一次数据 | 修改了可变默认对象 | `None` 作默认，调用内新建 |
| 改传入列表影响外部 | 修改共享对象 | 明确约定副作用，必要时复制 |
| 函数有时返回值、有时没有 | 分支漏了 `return` | 检查所有分支的返回规则 |
| 空列表统计出错 | `min`、`max`、除法没有空数据规则 | 先处理空列表 |

## 10. 自查简答
自查简答：

- 定义和调用差在哪里？定义创建函数对象；调用才执行本体。
- `return` 后还会执行同一路径后面的程序吗？不会；它结束本次调用。
- `print()` 能替代 `return` 吗？不能；显示和交还数据是不同事情。
- `*args`、`**kwargs` 在函数内是什么？元组、字典；名称可以更换。
- 默认值何时创建？执行函数定义时，不是每次调用时。
- 50 行函数一定不好吗？不能只看行数；看责任、数据流和分支是否清楚。
- 怎样才算能进下一章？能不看示例写函数，追踪参数如何对应，清楚区分返回值与副作用。

下一章将回答：「函数内的名称到哪里找？函数能否像数据一样保存、传递，甚至包装另一个函数？」
