---
title: Python 第 277-308 集：模块与包
published: 2026-09-16
updated: 2026-09-18
description: 模块与包
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第 6 章｜模块与包：把程序整理成能找到、能重用的文件

> 对应既有学习路线的 277–308 节主题，并把后面第 358 节的包补充按主题并入。这是依课程目录重组的讲义，并非逐集逐字稿。示例使用 Python 3；本章不需要类别知识。

## 1. 本章解决什么问题？需要哪些前置知识？

如果所有商品数据、价格计算、用户输入都放在同一个文件，程序会逐渐难找、难修改。本章教你把工作拆成多个文件，并回答：

1. `import` 到底做了什么？它不是单纯把另一个文件文字贴进来。
2. `import module` 和 `from module import name` 有什么差别？
3. 为什么有些文件能直接执行，有些却必须用 `python -m ...`？
4. 为什么会找不到模块、重复执行顶层程序代码，或发生循环导入？

前置知识：函数、作用域、列表与字典、文件和文件夹。类型提示在 [第 5 章](05-iterators-generators-algorithms.md) 已介绍；看到 `price: int` 时，记得它是提示，不是自动验证。

这章附有实际可执行的 [module_demo 小项目](examples/module_demo/README.md)。你可以先执行一次，再按下文逐步拆解。所有例子都在本机运行，不需外部服务。

## 2. 先创建直觉

### 2.1 模块像一个有自己名字表的工具抽屉

对本章的自订程序来说，一个 `.py` 文件通常就是一个模块，例如 `pricing.py`。模块也可能来自 Python 内建功能或其他载入方式，所以不要把所有模块都限定成普通文字文件。

模块有自己的**名称空间（namespace）**：它的变量、常数、函数名字都存放在自己的环境里。假设你写 `import pricing`，当前程序取得一个名为 `pricing` 的模块参考；透过 `pricing.line_total(...)` 便可使用它的函数。

你可以把 `pricing.line_total` 理解为：「到 pricing 这个抽屉，找 line_total 这件工具」。不同模块可有同名函数，不一定互相冲突。

### 2.2 包／包是用来组织模块的层级

本章使用带有 `__init__.py` 的**一般包（regular package）**。例如 `module_demo` 文件夹里放 `catalog.py`、`pricing.py`，完整名称分别是 `module_demo.catalog`、`module_demo.pricing`。

| 名称 | 本章例子 | 用途 |
|---|---|---|
| 模块 | `pricing.py` | 保存一组相关函数或数据 |
| 包／包 | `module_demo/` | 把相关模块组成有层级的单位 |
| 包初始化文件 | `__init__.py` | 在首次导入一般包时执行，通常保持简短 |
| 包入口文件 | `__main__.py` | 执行 `python -m module_demo` 时启动 |

Python 也支援不含 `__init__.py` 的 namespace package；「文件夹没有 `__init__.py` 就永远不能导入」并不准确。初学时先使用一般包，目录关系最清楚。

### 2.3 导入是载入与绑定，不是复制全部程序代码

第一次用某个模块名称导入时，Python 会查找、创建模块对象并执行它的顶层程序代码。这些「顶层程序代码」包括函数定义、变量赋值，也包括没有放在函数内的 `print()` 或 `input()`。

同一个 Python 行程内，再用相同完整名称导入，通常会从 `sys.modules` 快取取得已有模块。模块已完成的顶层程序代码一般不会再执行一次。

因此，把 `input()`、启动选单、读写正式数据等动作随意放在模块顶层，会让别人一导入你的工具，就被迫启动整个程序。

## 3. 最小可跑例：先学导入，再看多文件项目

### 3.1 导入标准库：不用先安装

标准库是随 Python 提供的模块。以数学模块为例：

```python
# 输出：
# 3.0
# 4.0
# 说明：import 绑定模块名称；from import 可以绑定指定函数并取别名。
import math
from math import sqrt as square_root

print(math.sqrt(9))
print(square_root(16))
```

`math.sqrt` 和 `square_root` 在这里指向同一个平方根函数；别名只是当前文件采用的名称。导入时不加 `.py`，也不写文件系统路径。

| 写法 | 当前文件得到什么名字？ | 使用方式 |
|---|---|---|
| `import math` | `math` | `math.sqrt(9)` |
| `import math as m` | `m` | `m.sqrt(9)` |
| `from math import sqrt` | `sqrt` | `sqrt(9)` |
| `from math import sqrt as root` | `root` | `root(9)` |

`from module import *` 会把一批名字带进当前作用域，容易看不出来源或覆盖已有名字。学习和项目中优先明确列出需要的名字。

### 3.2 先把附带的小项目跑起来

目录结构如下，所有文件已随笔记提供：

```text
examples/
└─ module_demo/
   ├─ __init__.py
   ├─ __main__.py
   ├─ catalog.py
   ├─ pricing.py
   ├─ entry_demo.py
   ├─ load_demo.py
   ├─ binding_demo.py
   └─ README.md
```

在 Windows PowerShell 执行：

```powershell
Set-Location 'D:\GPT projec1\outputs\python-foundations\examples'
python -m module_demo
```

如果你把笔记搬到其他地方，第一行改成自己 `examples` 文件夹的路径。第二行的 `module_demo` 是包名称，没有 `.py`，也没有斜线。

预期输出：

```text
商品：笔
数量：3
小计：15.00 CNY
```

这里只演示商品查询与价格计算，还没有互动选单、库存、订单或付款流程。不要把这个演示误当作后面电商作业的完整答案。

### 3.3 文件一：`module_demo/catalog.py` 负责数据

来源文件：[catalog.py](examples/module_demo/catalog.py)。下面是完整内容：

```python
# 输出：无。
# 说明：保存教学商品数据；价格单位是分，不是元。
CURRENCY = "CNY"
PRODUCTS = {
    "P001": {"name": "笔", "price_cents": 500},
    "P002": {"name": "书", "price_cents": 3000},
}


def get_product(product_id):
    return PRODUCTS.get(product_id)
```

`get_product()` 交回商品字典；找不到时，`dict.get()` 预设交回 `None`。它不负责打印商品，也不要求用户输入。

本例直接交回原字典的参考，外部若修改内容，也会改到模块保存的数据；它不是自动复制的安全副本。这里只读取，后续要让其他文件修改数据时，应把修改入口和规则设计清楚。

### 3.4 文件二：`module_demo/pricing.py` 负责计算

来源文件：[pricing.py](examples/module_demo/pricing.py)。下面是完整内容：

```python
# 输出：无。
# 说明：只计算与格式化，不读取用户输入；前提是非负整数金额及数量。
def line_total(unit_price_cents: int, quantity: int) -> int:
    return unit_price_cents * quantity


def format_money(cents: int) -> str:
    yuan = cents // 100
    fen = cents % 100
    return f"{yuan}.{fen:02d}"
```

为避免直接用二进位浮点数表示金额，本例把 `5.00 元` 存成 `500 分`。`// 100` 取得整数元，`% 100` 取得余下的分；`:02d` 表示整数显示至少两位，不足时在前面补 `0`。

`line_total()` 交回整数分，`format_money()` 交回格式化字符串，两个函数本身都不显示输出。这份格式化函数以非负金额为前提，不是通用的金融格式化工具。类型提示也不会阻止负数、字符串等不符合预期的输入，互动程序需要在调用前验证。

### 3.5 文件三：`__init__.py` 描述这个包

来源文件：[__init__.py](examples/module_demo/__init__.py)。下面是完整内容：

```python
# 输出：无。
# 说明：一般包的初始化文件；保持简短，不启动互动选单。
APP_NAME = "教学商品查询"
__all__ = ["APP_NAME"]
```

`__init__.py` 可以是空的。本例多放一个包层级常数，让你知道 `import module_demo` 后可使用 `module_demo.APP_NAME`。

`__all__` 主要用来说明 `from module_demo import *` 要带出的名字；它不是存取权限或安全限制，也不表示所有子模块都自动载入。即使没有把 `catalog` 写进这个清单，仍可明确导入 `module_demo.catalog`。

### 3.6 文件四：`__main__.py` 把工作串起来

来源文件：[__main__.py](examples/module_demo/__main__.py)。**必须放在上述包结构内，并从 `examples` 执行 `python -m module_demo`。** 不要把本区块单独贴到任意 `.py` 文件后直接执行，因为它使用包内的相对导入。

```python
# 输出：
# 商品：笔
# 数量：3
# 小计：15.00 CNY
# 说明：从 examples 目录执行 python -m module_demo，才有包执行脉络。
from .catalog import CURRENCY, get_product
from .pricing import format_money, line_total


def main():
    product = get_product("P001")
    quantity = 3
    subtotal = line_total(product["price_cents"], quantity)
    print(f"商品：{product['name']}")
    print(f"数量：{quantity}")
    print(f"小计：{format_money(subtotal)} {CURRENCY}")


if __name__ == "__main__":
    main()
```

入口函数决定先查哪个商品、购买多少、最后显示什么。`P001` 是本例确定存在的固定数据；若改成用户输入，就必须先处理找不到商品的 `None`，不能直接取 `product["price_cents"]`。

现在你已经把工作分开了：数据模块不知道画面怎么显示，计算模块不知道用户如何输入，而入口负责安排流程。

### 3.7 `__name__`：现在是入口，还是被别人导入？

来源文件：[entry_demo.py](examples/module_demo/entry_demo.py)。这是独立的入口观察例子：

```python
# 输出：
# 你好，小明
# 当前模块名：__main__
# 说明：这是用 python -m module_demo.entry_demo 执行时的输出；导入时不显示。
def greet(name):
    return f"你好，{name}"


def main():
    print(greet("小明"))
    print(f"当前模块名：{__name__}")


if __name__ == "__main__":
    main()
```

在 `examples` 文件夹执行 `python -m module_demo.entry_demo`，会出现注解中的两行输出。它作为本次程序入口执行，所以 `__name__` 是字符串 `"__main__"`。

再开一个新 Python 行程，从 `examples` 执行以下片段。可以把片段存成 `examples/check_entry.py`：

```python
# 输出：
# module_demo.entry_demo
# 你好，小华
# 说明：先前的 main() 不会因导入而被调用；greet() 仍然可以使用。
from module_demo import entry_demo

print(entry_demo.__name__)
print(entry_demo.greet("小华"))
```

导入时仍会执行顶层的函数定义与 `if` 判断，只是条件不成立，因此不执行 `main()`。不要理解成「写了入口保护，整个文件在导入时就完全不执行」。

`__main__.py` 是文件名，而 `__name__ == "__main__"` 是执行时的条件，两者用途不同。一般模块，例如 `entry_demo.py`，也可以在作为入口时得到 `__name__ == "__main__"`。

### 3.8 首次执行与 `sys.modules` 快取

来源文件：[load_demo.py](examples/module_demo/load_demo.py)：

```python
# 输出：
# load_demo：顶层执行
# 说明：为观察首次导入特意加入顶层 print；正式工具模块通常避免这样做。
print("load_demo：顶层执行")
VALUE = 10
```

从 `examples` 在**新行程**执行下面的片段：

```python
# 输出：
# load_demo：顶层执行
# True
# True
# True
# 说明：相同完整模块名在同一行程中复用模块；del 只删掉当前的一个名字。
import sys
from module_demo import load_demo
import module_demo.load_demo as second_name

print(load_demo is second_name)
print(load_demo is sys.modules["module_demo.load_demo"])

del load_demo
import module_demo.load_demo as third_name
print(third_name is second_name)
```

`is` 比较两个名字是否指向同一个对象。虽然写了三次导入，顶层讯息只显示一次。

`del load_demo` 只移除当前作用域的名字；`second_name` 和 `sys.modules` 仍保有参考，所以不等于把模块卸载，也不保证立即回收名称空间。对象能否回收，要看是否还被其他地方引用。

快取是**每个 Python 行程自己的**，不是把程序执行结果永久存到磁碟。重新启动程序，顶层程序代码会重新执行。`__pycache__` 下的 `.pyc` 是字节码快取，和 `sys.modules` 中当前这次执行的模块对象不是同一回事。

快取也以模块名称为键。若把同一来源文件当 `__main__` 执行，又用一般模块名导入，可能得到两次执行与不同状态。初学时不要用修改 `sys.modules` 或反复 reload 的方式处理设计问题；先维持清楚、一致的入口。

### 3.9 名称空间：函数去哪里找全域变量？

来源文件：[binding_demo.py](examples/module_demo/binding_demo.py)：

```python
# 输出：无。
# 说明：用来观察模块名称、全域查找及 from 导入后的绑定关系。
PRICE = 10
TAGS = ["new"]


def get_price():
    return PRICE
```

从 `examples` 执行以下片段：

```python
# 输出：
# 999
# 10
# 说明：get_price() 的全域名称查找使用它定义时所在的模块。
from module_demo import binding_demo

PRICE = 999
print(PRICE)
print(binding_demo.get_price())
```

调用者的 `PRICE = 999` 不会改变 `binding_demo.PRICE`。这正是模块名称空间的好处：不同文件可以有相同名称，而不会只因名字一样就混在一起。

### 3.10 `from ... import ...` 不会创建「永远同步的变量」

仍使用 `binding_demo.py`，从 `examples` 的**新行程**执行：

```python
# 输出：
# 10
# 20
# ['new', 'sale']
# True
# 说明：from 绑定当时的对象；重新绑定整数与修改同一列表，是不同操作。
from module_demo import binding_demo
from module_demo.binding_demo import PRICE, TAGS

binding_demo.PRICE = 20
binding_demo.TAGS.append("sale")

print(PRICE)
print(binding_demo.PRICE)
print(TAGS)
print(TAGS is binding_demo.TAGS)
```

分开看这两条线：

- `PRICE` 最初和模块的 `PRICE` 都指向整数 `10`。把 `binding_demo.PRICE` 重新指向 `20`，不会自动重新绑定当前文件的 `PRICE`。
- `TAGS` 和 `binding_demo.TAGS` 都指向同一个列表。`append()` 修改的是共同指向的对象，所以从两边都能看到新增项目。

因此，`from ... import ...` 不是深拷贝，也不是每次读取时自动去模块查询。若想清楚地读取「模块当前的状态」，保留模块名称并使用 `module.attribute` 往往更直观。

### 3.11 绝对导入与相对导入

本章包内的同一个工具，可以用以下方式指定：

| 写法 | 如何解读？ | 放在哪里合适？ |
|---|---|---|
| `from module_demo.pricing import line_total` | 从可查找到的顶层包名开始 | 包外或包内，前提是能找到顶层包 |
| `from .pricing import line_total` | 从当前包的 `pricing` 子模块取得 | 包内，且执行时具有包脉络 |
| `from ..pricing import line_total` | 先往上一层包，再找 `pricing` | 更深一层的子包内 |

相对导入的「相对」是相对于**包名称**，不是相对于终端当前所在文件夹，也不是任意的硬盘相对路径。不能用多个点越过顶层包。

如果直接执行 `python module_demo/__main__.py`，文件会被当成独立脚本，通常没有这里需要的包脉络，`.catalog` 就会报「attempted relative import with no known parent package」。从 `examples` 用 `python -m module_demo`，Python 按包名称启动，便能理解相对导入。

`-m` 不是自动修复所有导入错误；前提仍然是当前的搜索路径能找到 `module_demo`。也不要把文件名写成 `python -m module_demo.py`，因为 `-m` 后面接模块名称，不是文件名。

### 3.12 `sys.path`：Python 到哪里找文件型模块？

一般情况下，Python 先查看已载入模块的快取；若需要载入，再透过导入机制查找。对本章的文件型模块，`sys.path` 是重要的搜索位置清单，但内建模块或特殊载入器不全都靠普通文件路径。

常见的 `sys.path` 来源包括：

- 脚本所在文件夹；使用 `python -m ...` 或互动模式时，通常包含当前工作文件夹。
- `PYTHONPATH` 环境变量提供的位置（如果有设定）。
- Python 标准库的位置。
- 当前 Python 环境安装第三方包的位置，例如 `site-packages`。

具体顺序和内容可能受虚拟环境、启动选项或环境设定影响。最常见的初学错误，是把「当前终端文件夹」「正在执行的文件所在文件夹」「专案顶层」当成永远相同的位置。

从 `examples` 执行以下片段，检查实际载入的文件是否来自预期示例。这里的 `__file__` 是文件型模块的来源路径；不保证所有种类的模块都有它。

```python
# 输出：
# True
# True
# 说明：Path 处理文件路径；此例检查模块来源，不显示用户电脑的完整路径。
from pathlib import Path
from module_demo import pricing

source_file = Path(pricing.__file__)
print(source_file.name == "pricing.py")
print(source_file.parent.name == "module_demo")
```

`pathlib` 是标准库；本例只把路径转成对象，读取 `.name`（文件名）与 `.parent.name`（上一层文件夹名），不创建或修改文件。

`sys.path` 是可修改列表，`sys.path.insert(0, 某个文件夹的字符串路径)` 能让当前行程优先查找该位置。若要找到 `module_demo`，应加入**包含 module_demo 的上一层文件夹**，而非只加入包内某个子模块所在位置。它不会替你安装包，也不会永久改变其他行程。

对这份小项目，从正确的 `examples` 目录以 `-m` 启动即可。不要把自己电脑的绝对路径散落到每个文件；搬到另一台电脑就可能失效，加入过度优先的路径也可能导入错误的同名模块。

常见例子是把自己的文件取名为 `json.py`、`random.py` 或 `math.py`，与标准库名称冲突；程序可能载入自己的文件。报错时可检查实际来源。也要避免创建 `module_demo.py` 与 `module_demo/` 同时存在于搜索位置，让自己难以判断要载入哪个。

### 3.13 循环导入：两个模块互相等待尚未定义的名字

假设 `a.py` 第一行就向 `b.py` 要一个名字，而 `b.py` 第一行又向 `a.py` 要稍后才会定义的名字。Python 不会等两个文件都执行完才处理导入；此时其中一个模块可能仍处于「只初始化了一部分」的状态。

以下是**不要照此设计**的示意；程序行保留在注解中，不会真的触发导入错误：

```python
# 输出：无。
# 说明：以下是两个文件的错误结构示意，全部保持注解，没有执行。
# a.py:
# from b import b_value
# a_value = 10
#
# b.py:
# from a import a_value
# b_value = 20
```

从导入 `a` 开始追踪：

1. Python 开始执行 `a.py`，但尚未执行 `a_value = 10`。
2. 第一行要求载入 `b`，因此开始执行 `b.py`。
3. `b.py` 要求从 `a` 取得 `a_value`，可是 `a` 还没定义这个名字。
4. 因而可能看到带有「partially initialized module」的 `ImportError`。

常用修正是把双方共用的数据或函数移到第三个模块，例如 `common.py`，让 `a` 和 `b` 都依赖它；或把流程安排集中到入口模块，工具模块只提供数据和函数，不反向导入入口。

把导入移到函数内，有时能延后时间，解除特定循环；但若互相调用的设计仍有问题，延后并不是根治。也不是任何形式的互相导入一定报错，关键在于读取名字时它是否已创建。初学阶段以单向、容易追踪的依赖关系为目标。

### 3.14 软件目录规范：先按用途分，不必一次建满

当前七个 `.py` 文件中，`entry_demo`、`load_demo`、`binding_demo` 都只是教学观察工具。真正商品示例只需要 `__init__.py`、`__main__.py`、`catalog.py`、`pricing.py` 四个文件。

之后把程序扩展成可保存商品的本机工具，可以采用这种结构。这是未来整理方式的示意，以下目录并未在附带项目内全部创建：

```text
shop_project/
├─ README.md              如何启动、功能、限制
├─ data/                  商品与订单等数据文件
├─ tests/                 核对程序行为的测试
└─ shop/
   ├─ __init__.py
   ├─ __main__.py         入口与操作流程
   ├─ catalog.py          商品查询、增删改规则
   ├─ pricing.py          纯金额计算
   ├─ storage.py          读文件、存文件
   └─ settings.py         共用设定
```

在 `shop_project` 目录执行 `python -m shop`。随著程序成长再创建需要的部分，不要求一开始就有所有文件。你应该能回答「这个文件解决哪一类问题」，而不只是照抄目录名称。

几个分工判断：

- 想改商品排序规则，主要找 `catalog.py`；想改金额计算，找 `pricing.py`。
- 想把 JSON 文件换成另一种保存方式，尽量集中修改 `storage.py`，不要让每个选单选项都直接读写文件。
- 共用设定可放 `settings.py`，设定模块不应反过来启动入口。
- `data/` 是程序使用或产生的数据，不是需要 `import` 的 Python 程序代码；文件路径与模块搜索是两件事。
- 读写数据时，单用相对文件名通常以「当前工作目录」为基准，并不会自动跟著程序文件走。正式加入存储功能时，要明确指定数据根目录，相关路径工具在后续标准库章节练习。

先让一个小功能可跑，再整理其职责和依赖，是比盲目增加目录更容易检查的方式。模块化的成果应是：别人能找到功能、工具能重用、改一处不需要到处复制同样程序。

## 4. 执行追踪：从命令到显示结果

### 4.1 `python -m module_demo` 的流程

| 步骤 | 发生什么 | 此时是否有萤幕输出？ |
|---|---|---|
| 1 | 从搜索位置找到 `module_demo` 包 | 无 |
| 2 | 首次载入包，执行 `__init__.py` | 无，只定义名字 |
| 3 | 把 `module_demo/__main__.py` 作为入口执行 | 尚无 |
| 4 | 相对导入 `catalog` 和 `pricing` | 无，只创建数据和函数 |
| 5 | 定义 `main()`，判断入口条件成立 | 尚无 |
| 6 | 调用 `main()`，取出 `P001` | 无，取得字典 |
| 7 | `line_total(500, 3)` 交回 `1500` | 无，得到整数分 |
| 8 | 前两次 `print()` | 显示商品、数量 |
| 9 | 第三次 `print()` 的参数先求值，其中 `format_money(1500)` 交回 `"15.00"` | 取得字符串，尚未显示小计 |
| 10 | 第三次 `print()` 使用组合好的字符串 | 显示小计 |

函数定义会创建函数，但不等于调用函数。这个区分正好接上前章的 `return`：计算结果先交回，直到外层选择 `print()` 才显示。

### 4.2 依赖方向

```text
__main__.py（安排流程与显示）
   ├─→ catalog.py（商品数据）
   └─→ pricing.py（金额计算）
```

`catalog.py` 和 `pricing.py` 不必反过来导入 `__main__.py`。这个例子很小，拆文件的目的是创建职责和依赖的直觉；不是规定每个函数都要独立一个文件。

## 5. 常见错误与排查顺序

| 情况 | 先检查什么？ | 常见修正 |
|---|---|---|
| `ModuleNotFoundError` | 名字拼写、包是否存在、启动位置、使用哪个 Python 环境 | 回到包上一层，用正确的模块名和 `-m` 启动 |
| 相对导入没有父包 | 是否直接执行了包内文件 | 按完整包名用 `-m` 执行 |
| `ImportError: cannot import name ...` | 目标名字是否存在、是否尚未定义、是否循环导入 | 核对名字与依赖方向，而不是先乱改搜索路径 |
| 一导入就跳出输入提示 | `input()` 或选单是否在顶层 | 放进 `main()`，以入口条件控制调用 |
| 模块内容和预期不同 | 是否同名遮蔽、来源路径是否正确 | 核对 `__file__`，重新命名冲突文件 |
| 改了文件，旧互动工作阶段没变 | 模块是否已被快取 | 初学先重新启动程序再验证，不把重复 import 当成重载 |
| `del module` 后再导入没有重新执行 | `sys.modules` 或其他名字是否仍保留对象 | 理解名称删除与对象生命周期不同 |
| `from ... import PRICE` 没同步更新 | 是修改共享对象，还是重新绑定模块属性 | 需要当前状态时使用 `module.PRICE` |
| 写了 `__all__` 仍可导入其他名字 | 把汇出清单误当成权限控制 | `__all__` 不是禁止存取的安全机制 |

读错误时先看最后一行的种类与讯息，再往上找自己写的文件。不要看到「找不到」就立刻安装包；自己写的模块根本不需要去网路安装。异常处理与测试补充见 [bridge-errors-testing.md](bridge-errors-testing.md)。

### 5.1 写模块时的实用规则

- 名字简短、描述用途，使用小写与底线，例如 `order_service.py`。
- 导入通常放在文件前部；把标准库、第三方包、自己项目的导入分组，方便看依赖。
- 定义可重用的函数时，避免一导入就启动选单或修改正式数据。
- 明确说明金额、时间、数量的单位及输入前提，别只靠名称猜。
- `__init__.py` 保持简短。只有需要方便用户导入时，才考虑清楚地重新汇出工具。
- 不用 `from ... import *` 隐藏来源，也不要把内建或常用标准库名称拿来当自己的文件名。

未来项目变大时，可以再分出 `tests/`、`data/` 和项目说明文件。包的顶层应清楚可找到；不要为了看起来专业就套用自己还无法解释的多层目录。

## 6. 练习：把功能放对地方

### 练习 A｜预测导入结果

不执行程序，先回答：第一次导入 `module_demo.load_demo`、第二次导入、重开一个新 Python 行程再导入，各会印几次「顶层执行」？然后在终端验证。

### 练习 B｜新增商品

只在 `catalog.py` 加入 `P003`：名称「杯子」，价格 `2000` 分。创建一个练习入口，显示购买两个杯子的小计。验收输出应包含「杯子」「2」「40.00 CNY」。原来的价格计算函数不需要为这件商品另写一份。

### 练习 C｜增加一个计算工具

在 `pricing.py` 加入 `total_with_shipping(subtotal_cents, shipping_cents)`，交回商品小计加运费，仍以分为单位。为它写类型提示，并把前提写在注解中。验收：`1500` 加 `600` 得到整数 `2100`，格式化后为字符串 `"21.00"`。

只测算术时不需要启动商品选单。请从另一个练习文件导入并调用它，证明这个模块真的可重用。

### 练习 D｜入口保护

把一个「询问姓名、打印问候」的小程序整理成 `greet(name)`、`main()` 和入口条件。验收：直接执行会询问姓名；被另一个文件导入时不询问姓名，但能调用 `greet("小明")` 取得字符串。

### 练习 E｜导入方式

在包内解释 `from .pricing import line_total` 的一个点代表什么。再从包外用绝对导入调用同一个函数。写下两种情境应使用的启动命令；不要靠到处添加绝对 `sys.path` 才成功。

### 练习 F｜设计依赖

假设 `users.py` 和 `orders.py` 都要读取同一份运费设定。画出文件依赖方向，说明应把设定放在哪里，避免这两个模块因为共用设定互相导入。此题可以先用文字或箭头回答，不要求新增业务功能。

## 7. 自查与简答

| 自查问题 | 简答 |
|---|---|
| `import pricing` 是把所有函数复制进当前文件吗？ | 不是。通常载入或取得模块对象，并绑定 `pricing` 这个名字。 |
| 模块的顶层 `print()` 什么时候执行？ | 首次载入时；同一行程相同名字的重复导入通常用快取。作为脚本重新启动又是新的一次执行。 |
| 导入函数后，它的全域变量在哪里找？ | 在定义该函数的模块全域名称空间，而非任意调用者的同名变量。 |
| `from module import value` 是深拷贝吗？ | 不是。它在当前作用域绑定当时取得的对象。 |
| `__name__` 等于文件名吗？ | 不总是。导入时通常是模块完整名称；作为入口执行时是 `"__main__"`。 |
| `__init__.py` 和 `__main__.py` 有何不同？ | 前者用于一般包初始化，后者提供 `python -m 包名` 的入口。 |
| 相对导入跟著当前工作目录走吗？ | 不是。它根据模块所属包来解析。 |
| `sys.path` 要放包文件夹，还是它的上一层？ | 要能搜索到顶层包，通常放包含它的上一层。 |
| 删除当前 `module` 名字，会立即卸载模块吗？ | 不会。快取与其他对象仍可能持有参考。 |
| `sys.modules` 和 `__pycache__` 一样吗？ | 不一样。前者是当前行程内的模块快取；后者通常保存编译后的字节码文件。 |
| 循环导入一定报错吗？ | 不一定；问题常是模块尚未初始化完成，就被索取尚未定义的名字。应优先整理依赖方向。 |
| `-m` 后面写 `module_demo.py` 吗？ | 不写。应是可导入的模块／包名称，例如 `module_demo`。 |

练习核对：A 在同一行程第一回印 `1` 次、第二回新增 `0` 次，重开行程后再印 `1` 次；C 应以 `return subtotal_cents + shipping_cents` 交回数字；F 可使用 `settings.py` 保存设定，让 `users.py`、`orders.py` 都单向导入它。

本章完成的最低标准：能从正确位置启动附带项目、说清楚四个主要文件的分工、理解导入快取与名称绑定，并能把自己的小程序拆成数据、计算和入口。后续增加功能时，先维持这种可解释的结构，再逐步扩大。
