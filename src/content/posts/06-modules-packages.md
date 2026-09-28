---
title: Python 模块与包：组织代码、导入模块与管理依赖
published: 2026-09-18
updated: 2026-09-28
description: 从模块、包和 import 开始，理解 Python 如何查找、加载并复用代码
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第六章　模块与包：把程序拆成可复用的部分

当程序变长后，商品数据、价格计算、用户交互和启动逻辑混在一个文件里，修改一处容易影响另一处。模块和包解决三个问题：代码放在哪里、如何复用、程序从哪里启动。

## 1. 模块是什么

一个 `.py` 文件就是一个模块。模块有自己的名称空间，文件里的变量、函数和类默认保存在这里。导入模块时，Python 会执行模块顶层代码，并把模块对象绑定到当前文件的一个名称。

```python
# pricing.py
TAX_RATE = 0.06

def total(price_cents, quantity):
    return round(price_cents * quantity * (1 + TAX_RATE))
```

```python
import pricing
print(pricing.total(1000, 2))
# 输出：2120
```

`pricing` 指向模块对象，`pricing.total` 才是模块中的函数。代码没有被复制到当前文件，使用模块名访问成员还能看出名称来源。

## 2. 三种常见导入写法

```python
import math
print(math.sqrt(9))

from math import sqrt
print(sqrt(9))

from math import sqrt as root
print(root(9))
# 输出：3.0；3.0；3.0
```

- `import math`：当前文件得到名称 `math`。
- `from math import sqrt`：当前文件直接得到名称 `sqrt`。
- `as`：只修改当前文件中的别名，不会修改模块本身。

`from module import *` 会把大量名称放入当前作用域，容易覆盖已有名称。实际程序中应明确写出需要导入的名称。

## 3. 导入、执行与缓存

第一次导入模块时，Python 会查找模块、创建模块对象、执行顶层代码，再把对象绑定到导入语句指定的名称。模块对象会放入当前进程的 `sys.modules`。

```python
# noisy.py
print("模块顶层代码执行")
```

```python
import noisy
import noisy
# 输出：模块顶层代码执行
```

第二次导入会复用缓存，不会再次执行顶层代码。重新启动 Python 后，缓存消失，顶层代码会再次执行。`__pycache__` 中的字节码文件与 `sys.modules` 不是同一个东西。

## 4. 包与入口文件

包是用目录组织的一组模块：

```text
examples/
└─ module_demo/
   ├─ __init__.py
   ├─ __main__.py
   ├─ catalog.py
   └─ pricing.py
```

`__init__.py` 在首次导入普通包时执行；`__main__.py` 是使用 `python -m module_demo` 启动包时执行的入口。

## 5. 用商品目录理解模块分工

### 5.1 `catalog.py`：保存商品数据

```python
PRODUCTS = {
    "P001": {"name": "笔记本", "price_cents": 1250, "stock": 6},
    "P002": {"name": "钢笔", "price_cents": 800, "stock": 3},
}

def get_product(product_id):
    return PRODUCTS.get(product_id)
```

这个模块只负责保存和查询商品，不负责打印菜单或读取用户输入。

### 5.2 `pricing.py`：提供计算函数

```python
def line_total(price_cents, quantity):
    if quantity < 0:
        raise ValueError("数量不能为负数")
    return price_cents * quantity

def format_money(cents):
    return f"{cents / 100:.2f}"
```

函数返回计算结果，不直接显示文字。命令行程序和测试代码都可以复用这些函数。

### 5.3 `__init__.py`：定义公开入口

```python
from .catalog import get_product
from .pricing import line_total, format_money

__all__ = ["get_product", "line_total", "format_money"]
```

`__all__` 只是说明哪些名称适合通过 `from module_demo import *` 导出，不是权限控制，也不会自动加载所有子模块。

### 5.4 `__main__.py`：组织启动流程

```python
from .catalog import get_product
from .pricing import format_money, line_total

product = get_product("P001")
amount = line_total(product["price_cents"], 2)
print(product["name"], format_money(amount))
# 输出：笔记本 25.00
```

在 `examples` 目录运行 `python -m module_demo`。不要直接运行 `python module_demo/__main__.py`，因为直接运行时包上下文不存在，相对导入无法解析。

## 6. `__name__` 与执行入口

模块被导入时，`__name__` 是模块完整名称；文件被直接启动时，`__name__` 是 `"__main__"`。

```python
def main():
    print("启动程序")

if __name__ == "__main__":
    main()
```

导入这个文件时只会创建 `main`，不会自动启动程序；直接运行时才会调用它。

## 7. Python 如何寻找模块

Python 会结合内置模块、启动位置、`PYTHONPATH` 和安装环境中的目录搜索模块。可以查看当前进程的搜索顺序：

```python
import sys
for location in sys.path:
    print(location)
```

若要导入 `examples/module_demo`，搜索路径中应包含 `examples` 的上一层。不要把文件命名为 `json.py`、`random.py` 或 `math.py`，否则可能遮蔽标准库同名模块。遇到导入异常时，可查看 `module.__file__` 确认实际加载位置。

## 8. 相对导入、绝对导入与循环导入

`.catalog` 表示当前包下的模块，`..common` 表示上一级包中的模块。相对导入依赖包上下文，应使用 `python -m 包名` 启动。绝对导入写完整模块名：

```python
from module_demo.catalog import get_product
```

如果 `a.py` 导入 `b.py`，而 `b.py` 又立即导入 `a.py`，其中一个模块可能尚未创建所需名称。解决方法是把共用内容移到第三个模块，让入口模块组织流程，并保持依赖单向。

## 9. 重点对照

| 名称 | 作用 |
|---|---|
| `.py` 文件 | 保存一个模块的代码 |
| 包目录 | 组织多个相关模块 |
| `__init__.py` | 初始化普通包，可提供公开入口 |
| `__main__.py` | 定义 `python -m 包名` 的启动内容 |
| `sys.modules` | 当前进程已经加载的模块缓存 |
| `sys.path` | Python 查找模块的路径列表 |

不要把“导入模块”理解成复制文件内容。自己创建的目录同样可以是包，关键是目录结构和启动位置正确。

## 自查

1. `import pricing` 后，为什么要写 `pricing.total()`？
2. `import` 为什么通常不会在同一进程中重复执行模块顶层代码？
3. `__init__.py` 和 `__main__.py` 分别负责什么？
4. 为什么包内相对导入通常要用 `python -m` 启动？
5. 如果导入了错误的 `json.py`，应该检查什么？
6. 如何把一个混杂了数据、计算和输入的文件拆成三个模块？
