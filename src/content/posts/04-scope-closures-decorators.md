---
title: Python 第 196-241 集：名称空间、装饰器相关
published: 2026-09-14
updated: 2026-09-17
description: 名称空间、装饰器相关
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第四章　名称从哪里来？函数如何记住数据与包装行为？（第 196–241 集）

本章要解决三个问题：同名变量到底用哪一个？函数结束后能否保留某些状态？如何替多个函数统一增加日志或计时，而不用到处复制程序代码？

前置知识：`def`、参数、`return`、`*args`、`**kwargs`、引用与可变对象。若仍分不清 `func` 和 `func()`，先回第三章；装饰器困难往往来自这个区别。

每段 Python 都可单独执行。请先预测输出顺序，再执行。此章只学常用的函数式装饰器；类别、`staticmethod`、`classmethod`、`property` 放在附录，属于本基础主线以外的扩展。

## 1. 学习路线

| 原笔记范围 | 问题 | 核心概念 |
|---|---|---|
| 196–201 | Python 到哪里找名称？ | 名称空间、LEGB |
| 202–208 | 内外层同名时怎么办？ | 作用域、`global`、`nonlocal` |
| 209–214 | 函数能像数据一样使用吗？ | 保存、传入、返回函数 |
| 215–216 | 如何保留外层函数的数据？ | 闭包 |
| 217–233 | 如何增加共同功能？ | 包装、装饰器、`wraps` |
| 234–241 | 配置与多层包装如何运作？ | 装饰器工厂、求值与套用顺序 |

## 2. 名称空间：名称到对象的对照关系

`price = 10` 可理解为创建名称 `price` 与整数对象 10 的关系。名称空间保存这些关系；作用域决定在某一段程序可以如何找到名称。两个词相关，但不是完全同义。

常见的名称来源：

- **L，Local：**当前函数这次调用的局部名称，包括形参。
- **E，Enclosing：**词法上包住它的外层函数名称。
- **G，Global：**函数所属模块的全域名称。
- **B，Built-in：**Python 提供的 `len`、`print` 等内建名称。

这就是一般函数名称查找的 **LEGB**。它描述这类场景的规则，不是要把之后的类别属性查找等所有机制都塞进同一套缩写。

### 2.1 同名时，先找较近的层

```python
name = "全域"


def outer():
    name = "外层函数"

    def inner():
        name = "内层函数"
        print(name)

    inner()
    print(name)


outer()
print(name)
# 输出：内层函数；外层函数；全域。
# 说明：三个作用域各有自己的 name；内层同名名称遮蔽外层，没有删除或改写外层。
```

追踪 `inner()` 里的 `print(name)`：先找 L，已找到「内层函数」，就不继续向外找。将内层的赋值删除，才会用 E 的「外层函数」。全部找不到会产生 `NameError`。

### 2.2 查找看定义位置，不看谁调用它

```python
message = "模块的文字"


def show():
    print(message)


def caller():
    message = "调用者的文字"
    show()


caller()
# 输出：模块的文字。
# 说明：show 定义在模块中；caller 的局部名称不会自动变成 show 的外层作用域。
```

这个差别很重要：外层是程序代码中的嵌套关系，不是执行时「上一个调用我的函数」。此外，`if`、`for`、`while` 不会在一般程序区块中另建函数式局部作用域。

不要取名 `list`、`str`、`sum`、`print` 来保存自己的数据，否则会遮蔽内建名称。遇到「对象不能调用」时，检查是否把函数名重新赋值成了普通数据。

## 3. 赋值与读取不同：为什么出现 `UnboundLocalError`？

```python
count = 10


def broken():
    print(count)
    count = 20


try:
    broken()
except UnboundLocalError:
    print("局部 count 还没有值")
print(count)
# 输出：局部 count 还没有值；10。
# 说明：函数内有对 count 的赋值，Python 将它判定为局部名称；前面的读取不会改去找全域值。
```

不是读到 `count = 20` 才开始局部化。对一般函数，Python 会根据函数整体的系结操作判定局部名称。`count += 1` 也有赋值成分，因此同样要注意。

### 3.1 `global`：重新系结模块名称

```python
count = 0


def increase():
    global count
    count += 1


increase()
increase()
print(count)
# 输出：2。
# 说明：global 让此函数中的 count 指向模块全域名称，而非创建同名局部名称。
```

只读全域值不需要 `global`。修改全域名称指向的列表内容，也不等于重新系结全域名称：

```python
records = []


def add_record():
    records.append("完成")


add_record()
print(records)
# 输出：['完成']。
# 说明：append 修改共享列表，没有把 records 这个名称重新指向另一个对象，所以不需 global。
```

`global` 可用，但会增加隐藏数据流。若只是计算新数量，`new_count = increase(old_count)` 这种参数与返回值通常更容易理解。

### 3.2 `nonlocal`：重新系结外层函数名称

```python
def make_counter(start=0):
    count = start

    def next_count():
        nonlocal count
        count += 1
        return count

    return next_count


counter = make_counter(10)
print(counter())
print(counter())
# 输出：11；12。
# 说明：nonlocal 指向最近一层已存在的外层函数 count，让两次调用持续更新同一份状态。
```

`nonlocal` 的名称必须已存在于外层函数作用域，不能直接指向模块全域。它用于**重新系结名称**；若只读外层值、或只对外层列表 `append()`，不需要它。

**练习：**说明 `count += 1` 与 `records.append(...)` 的差别。

**自查简答：**前者对名称进行赋值，后者修改对象。若要重新系结外层函数的整数名称，用 `nonlocal`；重新系结模块名称，用 `global`。

## 4. 函数也是对象：能保存、传入与返回

### 4.1 保存函数，不要提前调用

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


operations = {"+": add, "*": multiply}
operation = operations["*"]
print(operation(4, 5))
print(operations["+"](4, 5))
# 输出：20；9。
# 说明：字典保存函数对象；取出后加括号才调用。也可将函数保存于列表中逐一调用。
```

若写 `operations = {"+": add(4, 5)}`，保存的是已计算的 9，不再是可供下次调用的函数。

### 4.2 函数作为参数与返回值

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


def choose_operation(symbol):
    if symbol == "+":
        return add
    if symbol == "*":
        return multiply
    return None


def apply_operation(operation, a, b):
    return operation(a, b)


selected = choose_operation("+")
if selected is not None:
    print(apply_operation(selected, 2, 3))
# 输出：5。
# 说明：choose_operation 返回函数；apply_operation 接收并调用它；都没有复制函数本体。
```

函数能像数据一样使用，常称为「一等对象」。理解这件事后，装饰器就只是函数接收与返回的进一步应用。

## 5. 闭包：函数保留对外层变量的存取

闭包可理解为：内层函数使用了外层函数的局部变量，并保留对那些变量的存取。**外层返回内层函数是常见用法，不是必要条件。**关键是引用外层函数的变量；只有读取模块全域值，不算本章讨论的这种闭包。

### 5.1 创建带不同配置的函数

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply


times_two = make_multiplier(2)
times_three = make_multiplier(3)
print(times_two(5))
print(times_three(5))
# 输出：10；15。
# 说明：两次创建的 multiply 分别保留对各自 factor 的存取；外层结束后仍可使用。
```

追踪 `times_two = make_multiplier(2)`：外层本次形参 `factor=2` → 创建使用它的 `multiply` → 返回函数对象 → `times_two` 指向它。直到 `times_two(5)` 才执行乘法。

### 5.2 没有 `return inner` 也能形成闭包

```python
callbacks = []


def register_message(message):
    def show():
        print(message)

    callbacks.append(show)


register_message("稍后显示")
callbacks[0]()
# 输出：稍后显示。
# 说明：show 透过列表保存，而非由外层返回；它仍引用外层局部 message，所以仍是闭包。
```

### 5.3 保存的是变量关系，不是永远固定的照片

```python
def make_reader():
    value = 1

    def read():
        return value

    value = 2
    return read


reader = make_reader()
print(reader())
# 输出：2。
# 说明：read 在调用时读取所保留的 value；创建 read 当下的 1 并没有被永久拍成快照。
```

这也解释了循环创建多个闭包时常见的「全部读到最后一个值」。需要每次独立配置，可以每次调用一个工厂函数；不要靠死背特殊写法掩盖系结规则。

**练习：**写 `make_power(exponent)`，返回计算某数字次方的函数；再建两个独立计数器。

**自查简答：**`square = make_power(2)` 取得函数，`square(3)` 才返回 9。每次调用 `make_counter()` 建一份新外层状态，两个计数器不必共享数量。

## 6. 装饰器：先手动包装，再看 `@`

假设很多函数都要印「开始／结束」，直接在每个函数内重复写两行，日后要改格式就得改很多处。包装函数能把共同步骤集中。

此处的**函数式装饰器**接收原函数，返回一个包装函数。这是最常见的学习形式；Python 装饰器不被限制为一定要返回新函数，也不只这一种实作。

### 6.1 最小例：先不使用 `@`

```python
def log_call(func):
    def wrapper():
        print("开始")
        result = func()
        print("结束")
        return result

    return wrapper


def say_hello():
    print("Hello")
    return "完成"


say_hello = log_call(say_hello)
print(say_hello())
# 输出：开始；Hello；结束；完成。
# 说明：log_call 返回 wrapper；名字 say_hello 改指向 wrapper，原函数由闭包中的 func 保存。
```

两个 `return` 职责不同：

| 位置 | 返回什么 | 何时执行 |
|---|---|---|
| 外层 `return wrapper` | 函数对象 | 装饰时 |
| 内层 `return result` | 原函数的结果 | 每次调用包装器时 |

不要写 `return wrapper()`：那会在装饰时直接执行包装器，把它的执行结果交出去。先能手动追踪这段，才继续下一节。

### 6.2 `@` 把重新赋值写得更靠近定义

```python
def log_call(func):
    def wrapper():
        print("开始")
        result = func()
        print("结束")
        return result

    return wrapper


@log_call
def say_hello():
    print("Hello")
    return "完成"


print(say_hello())
# 输出：开始；Hello；结束；完成。
# 说明：在此简单情况，@log_call 可用定义后 say_hello = log_call(say_hello) 理解。
```

装饰在执行到这段定义时发生，不是每次调用都重新包装。若定义本身位于另一个函数或循环内，每次再次执行定义仍会再次装饰。使用 `@` 后不要再手动包一次，否则会重复增加行为。

### 6.3 定义时也可能有画面输出

```python
def announce(func):
    print("正在装饰")

    def wrapper():
        print("正在调用")
        return func()

    return wrapper


@announce
def work():
    print("原函数工作")


print("定义已完成")
work()
# 输出：正在装饰；定义已完成；正在调用；原函数工作。
# 说明：装饰器本体在定义阶段执行；wrapper 本体在之后调用阶段执行。
```

所以不能说「只有调用被装饰函数才会出现装饰器效果」。先确认程序代码是在外层装饰函数，还是在内层包装函数。

## 7. 常用包装器的三个契约

若你的目标是「增加日志但保留原功能」，通常要保留：参数如何传入、返回值如何传出、异常如何传递。

### 7.1 接收后再原样转交

```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print("开始")
        result = func(*args, **kwargs)
        print("完成")
        return result

    return wrapper


@log_call
def add(a, b=0):
    return a + b


print(add(2, b=3))
# 输出：开始；完成；5。
# 说明：wrapper 收集 args=(2,) 和 kwargs={'b': 3}，调用 func 时再展开转交。
```

这是常用的同步函数包装形式，不是「自动兼容一切函数」的保证；生成器或异步函数还有其他行为需要考虑，本章先不展开。

### 7.2 忘记内层 `return`，结果会丢失

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)

    return wrapper


@bad_decorator
def get_number():
    return 42


print(get_number())
# 输出：None。
# 说明：这是可执行的错误设计示范；原函数返回 42，但 wrapper 没有把它返回给调用者。
```

### 7.3 需要出错也执行的后置动作，用 `finally`

```python
def trace(func):
    def wrapper(*args, **kwargs):
        print("进入")
        try:
            return func(*args, **kwargs)
        finally:
            print("离开")

    return wrapper


@trace
def divide(a, b):
    return a / b


try:
    divide(10, 0)
except ZeroDivisionError:
    print("调用者收到除零错误")
# 输出：进入；离开；调用者收到除零错误。
# 说明：finally 在一般返回或异常离开时执行；此包装器没有吞掉原函数的异常。
```

把后置 `print()` 放在 `func()` 后面，只在原函数正常返回时走得到。`finally` 的完整语法在异常章学习，这里只要能看出「正常结束后做」与「离开时做」不同。

## 8. `wraps` 保留资讯，不会替你执行原函数

包装之后，外部名称指向 `wrapper`，预设的 `__name__`、`__doc__` 也会变成包装器资讯。`functools.wraps` 把原函数的重要中继数据复制过去，并设定 `__wrapped__` 供工具追溯。

```python
from functools import wraps


def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("开始")
        return func(*args, **kwargs)

    return wrapper


@log_call
def greet(name):
    """产生问候文字。"""
    return f"你好，{name}"


print(greet.__name__)
print(greet.__doc__)
print(greet("小明"))
# 输出：greet；产生问候文字。；开始；你好，小明。
# 说明：wraps 保留名称与文件；真正执行原函数的是 func(*args, **kwargs)。
```

`wraps` 不会把包装器变回原函数，不会自动转交参数，也不会补上你忘记写的 `return`。没有 `wraps`，功能可能照常运作，但除错与文件工具看到的资讯较难理解。

第 230 集的「实时模板」是 IDE 帮你快速插入程序骨架的便利功能，不是新的 Python 语法。可以把常用装饰器骨架存成范本，但每次仍要确认参数、返回值和异常是否符合需求。先能徒手解释骨架，再使用模板节省打字。第 233 集重复多版本共存主题，可回第二章的版本环境说明查阅。

## 9. 有参装饰器：先配置，再包装，再调用

想指定日志标签，不应每次调用原函数都多传一个与业务无关的参数。可以先调用工厂创建一个记住标签的装饰器。

```python
from functools import wraps


def tagged(label):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"[{label}] 开始")
            return func(*args, **kwargs)

        return wrapper

    return decorator


@tagged("学习")
def add(a, b):
    return a + b


print(add(2, 3))
# 输出：[学习] 开始；5。
# 说明：tagged 接配置，decorator 接原函数，wrapper 接本次原函数所需参数。
```

这个常见写法有三层，因为有三次不同目的的调用：

```text
tagged("学习")        → 得到 decorator
decorator(原始 add)  → 得到 wrapper
wrapper(2, 3)       → 执行包装与加法，得到 5
```

三层是容易学习的实作形式，不是有参装饰器唯一合法的写法。配置也可以是重复次数、角色名称、日志等级等。

### 9.1 重复执行时，先定义次数与结果规则

```python
from functools import wraps


def repeat(times):
    if type(times) is not int or times < 1:
        raise ValueError("times 必须是正整数")

    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result

        return wrapper

    return decorator


@repeat(3)
def say_hi():
    print("Hi")
    return "完成"


print(say_hi())
# 输出：Hi；Hi；Hi；完成。
# 说明：times 在配置时验证；每次调用重复三次，约定返回最后一次结果；遇到异常直接向外传递。
```

重复执行是增加副作用，不可随意拿来包装扣款、扣库存等动作。此例适合问候等可重复的示范。重试与重复不是同一件事，重试只在指定失败后再次尝试，附录另作说明。

## 10. 叠加：求值、套用、调用是三种顺序

看到 `@A`、`@B` 不要只背一句「由下往上」。需要分三件事：

1. **装饰器表达式求值：由上到下。**例如先计算 `make_trace("A")`，再计算 `make_trace("B")`。
2. **将装饰器套用到函数：由下到上。**先由 B 包住原函数，再由 A 包住结果。
3. **实际调用：按包装器程序代码执行。**常见情况先进 A，再进 B，再原函数，然后 B、A 返回；若某层不调用下一层，就会在该处停止。

```python
from functools import wraps


def make_trace(label):
    print(f"求值 {label}")

    def decorator(func):
        print(f"套用 {label}")

        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"进入 {label}")
            result = func(*args, **kwargs)
            print(f"离开 {label}")
            return result

        return wrapper

    return decorator


@make_trace("A")
@make_trace("B")
def work():
    print("原函数")


print("准备调用")
work()
# 输出：求值 A；求值 B；套用 B；套用 A；准备调用；进入 A；进入 B；原函数；离开 B；离开 A。
# 说明：表达式求值和装饰器套用方向不同；调用顺序由 wrapper 是否调用下一层决定。
```

在装饰器值 `A`、`B` 已取得后，最终包装关系可用 `work = A(B(原始 work))` 理解。但若原本写的是有副作用的装饰器表达式，不能只看巢状调用就忽略其求值时机。

顺序有实际影响：日志在权限检查外层，通常连未授权尝试也能记录；权限检查若先挡住，内层日志不一定执行。不要把「装饰器越多」当作设计越好。

## 11. 综合例：计时但保留返回值与异常

```python
from functools import wraps
from time import perf_counter


def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            elapsed = perf_counter() - start
            print(f"{func.__name__} 耗时 {elapsed:.6f} 秒")

    return wrapper


@timer
def calculate_total(numbers):
    return sum(numbers)


print(calculate_total(range(5)))
# 输出：先显示 calculate_total 耗时若干秒，再显示 10。
# 说明：耗时依电脑与当次执行变化；finally 记录离开时间，原返回值和异常仍向外传递。
```

原函数只负责加总，计时放在共同包装层。若只是单一流程计时一次，普通前后记录也可以，不必强行装饰器化。

## 12. 常见错误与修正

| 问题 | 原因 | 修正 |
|---|---|---|
| `UnboundLocalError` | 局部赋值前读取 | 分清作用域，选参数／返回值或合适宣告 |
| 以为闭包必须 `return inner` | 把常见用法当定义 | 检查是否保留对外层函数变量的存取 |
| 使用 `global` 才能 append | 混淆名称与对象 | append 是修改对象，非重新系结 |
| 装饰后变成 `None` | 外层没返回包装器 | 检查 `return wrapper` |
| 原结果变成 `None` | 内层没返回结果 | 检查 `return func(...)` 或 `return result` |
| 定义时意外执行原函数 | 外层调用 `func()` 或返回 `wrapper()` | 分清装饰时与调用时 |
| 带参数函数不能用 | wrapper 没接收或没转交 | 接收与展开都要对上 |
| 加了 wraps 还不执行 | 误把 wraps 当委派调用 | 原函数仍需 `func(...)` |
| 调用顺序与预期不同 | 混淆求值、套用与执行 | 分三阶段逐行追踪 |

## 13. 练习与自查简答

1. **名称查找：**自己写 L、E、G 同名的例子，逐一删去内层系结并预测输出。
2. **闭包：**写 `make_counter(start)`，创建从 0 和从 100 起算的两个计数器；交替调用，确认互不干扰。
3. **日志：**写支援位置和关键字参数的装饰器，显示函数名与结果；不要随意记录密码等数据。
4. **配置：**写 `prefix_log(label)`，用不同标签装饰两个函数。
5. **权限练习：**写 `require_role("admin")`，接收含 `role` 的用户字典；符合才调用原函数。不符合时的返回规则必须说清楚。这只是逻辑练习，不是实际身分验证系统。
6. **叠加：**交换两个有输出提示的装饰器，分别写出求值、套用、调用顺序，不只抄最后画面。

自查简答：

- LEGB 的 E 指谁？程序代码中包住当前函数的外层函数，不是调用者。
- `nonlocal` 能用来直接改模块全域名称吗？不能，外层函数需已有该名称。
- 闭包是否保存数值快照？一般是保留变量系结的存取；调用时值可能已变。
- 两个 `return` 各交还什么？外层交还包装函数，内层交还一次业务执行的结果。
- `wraps` 是否会调用原函数？不会，它处理中继数据与可追溯资讯。
- 装饰器是否必须三层？有配置的函数工厂常用三层，但不是唯一合法形式。
- 叠加哪个先发生？表达式由上到下求值；套用由下到上；实际调用按包装器内容。

达标标准：不用看模板，也能说清楚每个变量指向哪个对象、每个括号是在哪一阶段调用、每个返回值交给谁。能做到这三点，比背熟装饰器形式更重要。
