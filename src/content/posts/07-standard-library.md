---
title: Python 第 309～339 集：时间、随机数、系统与路径、压缩、数据序列化、设定文件、子行程，以及猴子补丁与第三方包安装
published: 2026-09-17
updated: 2026-09-19
description: 时间、随机数、系统与路径、压缩、数据序列化、设定文件、子行程，以及猴子补丁与第三方包安装
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第 7 章｜标准库：把基础语法接到真实任务

本章对应基础篇目录约第 309～339 集：时间、随机数、系统与路径、压缩、数据序列化、设定文件、子行程，以及猴子补丁与第三方包安装。内容依已核对的影片目录重新整理，没有宣称逐集观看或逐字重现影片。

已经学过数据型别、条件、循环、函数、文件和模块。这一章要解决的是：**同一件常见工作，Python 是否已经提供可靠的工具？**

学完后，应该能：

- 为订单记录时间，并分清楚「日历上的时间」与「经过几秒」。
- 安全地在练习目录中创建文件、复制文件及制作压缩文件。
- 将购物车保存为 JSON，重新载入后继续处理。
- 读取设定文件，知道文字设定为甚么还需要型别转换。
- 启动另一个 Python 程序，取得它打印的文字与执行结果。
- 知道标准库、第三方包与执行环境分别扮演甚么角色。

本章的文件例子都会自行创建样本，放在 `TemporaryDirectory` 管理的临时目录里。离开 `with` 区块时，这个临时目录会自动清理。

---

## 7.1 先懂使用方式：模块提供甚么？

可以把模块理解为一个有名字的工具箱：

```python
import math

answer = math.sqrt(81)
print(answer)

# 输出：
# 9.0
# 说明：
# math.sqrt(81) 返回 9.0，answer 接住这个返回值。
# print(answer) 才把 9.0 显示出来；print 本身返回 None。
```

遇到一个新函数，先确认三件事：

1. 它要甚么输入？例如路径、字符串、数字或文件对象。
2. 它返回甚么？例如新字符串、列表、数字，或者 `None`。
3. 它有没有额外动作？例如写文件、修改原列表或启动另一个程序。

例如 `json.dumps()` 返回字符串；`json.dump()` 则把文字写入文件。名字只差一个字母，工作方向却不同。

自己创建的练习文件不要取名为 `random.py`、`json.py`、`logging.py` 等标准库名字，否则 `import` 可能载入自己的文件。

## 7.2 time：表示时间点，或量测经过时间

初学时最容易混淆两个问题：

| 想问的问题 | 适合的工具 | 返回值的意思 |
|---|---|---|
| 现在在时间轴上的甚么位置？ | `time.time()` | 从 Unix 纪元起算的秒数 |
| 这段程序花了多久？ | `time.perf_counter()` | 高精度计时读数，两次相减才有意义 |
| 让当前程序稍候一下 | `time.sleep(秒数)` | 暂停执行，返回 `None` |

Unix 纪元通常指 UTC 的 1970-01-01 00:00:00。时间戳不是「2026 年 9 月 15 日」这种人类易读的日历文字，也不是本地时区名称。

### 最小例：量测一段计算

```python
import time

started = time.perf_counter()
total = sum(range(10000))
elapsed = time.perf_counter() - started

print(total)
print(f"经过 {elapsed:.6f} 秒")

# 输出：
# 49995000
# 经过 0.000xxx 秒（示意；实际数字取决于电脑及当次执行）
# 说明：
# perf_counter() 返回计时读数，第二次减第一次得到秒数。
# sum() 返回总和；这里打印总和，方便确认被计时的计算确实完成。
```

量测耗时时优先用 `perf_counter()`，因为系统时钟可能被校正，而计时器适合比较经过时间。不要把单次测得的极小差异当成程序性能结论。

`time.sleep(1)` 的意思是让程序等待约 1 秒；实际恢复时间可能更晚。它不会让某个计算「自动执行 1 秒」，也不适合拿来替代对用户输入的等待。

### 读懂旧教材中的 time 写法

有些教材使用 `time.localtime()` 取得本地时间的结构，再用 `time.strftime()` 转成文字：

```python
import time

parts = time.strptime("2026-09-15 08:30:00", "%Y-%m-%d %H:%M:%S")
text = time.strftime("%Y/%m/%d %H:%M", parts)
print(text)

# 输出：
# 2026/09/15 08:30
# 说明：
# strptime() 把符合格式的文字解析成时间结构。
# strftime() 把时间结构格式化为新字符串，print() 负责显示。
```

不要急著背下所有 `time` 函数。涉及日期加减和订单时间时，下一节的 `datetime` 通常更好表达。

## 7.3 datetime：日期、时间与时间差

`datetime` 模块里常用的名字有：

| 名字 | 表示甚么 | 例子 |
|---|---|---|
| `date` | 只有年月日 | 到期日 |
| `datetime` | 年月日与时分秒 | 下单时间 |
| `timedelta` | 两个时间之间的长度 | 7 天、30 分钟 |
| `timezone` | 固定的 UTC 时差 | UTC+8 |

### 最小例：算出七天后的日期

```python
from datetime import date, timedelta

created_on = date(2026, 9, 15)
expires_on = created_on + timedelta(days=7)
remaining = expires_on - created_on

print(expires_on)
print(remaining.days)

# 输出：
# 2026-09-22
# 7
# 说明：
# 日期加上 timedelta，返回新的日期。
# 两个日期相减，返回 timedelta；.days 取得其中的天数。
```

日期不是普通整数。不要把 `20260915 + 7` 当成日期计算；月底、年底和闰年都可能使这种写法失效。

### 格式化与解析是相反方向

```python
from datetime import datetime

original = "2026-09-15 08:30:00"
moment = datetime.strptime(original, "%Y-%m-%d %H:%M:%S")
display = moment.strftime("%Y/%m/%d %H:%M")

print(display)
print(moment.year)

# 输出：
# 2026/09/15 08:30
# 2026
# 说明：
# strptime：文字 → datetime；strftime：datetime → 文字。
# moment 是日期时间数据，display 是字符串，两者型别与用途不同。
```

常用格式：`%Y` 是四位年份，`%m` 是月份，`%d` 是日期，`%H` 是 24 小时制的小时，`%M` 是分钟，`%S` 是秒。**月份 `%m` 与分钟 `%M` 的大小写不同。**

### 订单时间要交代时区

```python
from datetime import datetime, timedelta, timezone

utc_plus_8 = timezone(timedelta(hours=8))
ordered_at = datetime(2026, 9, 15, 9, 0, tzinfo=utc_plus_8)
ordered_at_utc = ordered_at.astimezone(timezone.utc)

print(ordered_at.isoformat())
print(ordered_at_utc.isoformat())

# 输出：
# 2026-09-15T09:00:00+08:00
# 2026-09-15T01:00:00+00:00
# 说明：
# 两行表示同一个时间点，只是使用不同时区显示。
# isoformat() 返回字符串，适合在数据中保存带时区的时间。
```

`datetime.now(utc_plus_8)` 能取得当前 UTC+8 的时间，结果随执行时刻变动。上例使用固定日期，是为了让能核对输出。

没有时区资讯的 `datetime` 常称为 naive datetime；有时区资讯的常称为 aware datetime。初学阶段不必背术语，但同一个系统里应使用一致的时间约定，不要混著比较。

### 时间戳、日期时间、文字：同一资讯的不同表示

有时接口给的是 `1720000000` 这类时间戳，画面却要显示日期。先辨认起点和终点，再选择转换方式：

| 从哪里到哪里 | 常用方式 |
|---|---|
| 时间戳 → 带时区的日期时间 | `datetime.fromtimestamp(seconds, tz=...)` |
| 日期时间 → 时间戳 | `moment.timestamp()` |
| 日期时间 → 一般格式文字 | `moment.strftime(format)` |
| 一般格式文字 → 日期时间 | `datetime.strptime(text, format)` |
| 日期时间 → ISO 格式文字 | `moment.isoformat()` |
| ISO 格式文字 → 日期时间 | `datetime.fromisoformat(text)` |

```python
from datetime import datetime, timezone

moment = datetime.fromtimestamp(0, tz=timezone.utc)
saved_text = moment.isoformat()
restored = datetime.fromisoformat(saved_text)

print(saved_text)
print(restored.timestamp())
print(restored == moment)

# 输出：
# 1970-01-01T00:00:00+00:00
# 0.0
# True
# 说明：
# 0 是固定的 Unix 时间戳；明确使用 UTC，避免本地时区改变显示结果。
# isoformat() 返回文字，fromisoformat() 把 ISO 文字解析回日期时间。
```

旧教材可能使用 `time.localtime(seconds)` 转成本地时间结构，再用 `time.mktime(parts)` 转回时间戳。注意 `mktime` 把输入当作**本地时间**；不能把 `time.gmtime` 得到的 UTC 结构直接交给它，却期待所有时区都得到原数值。对新程序，带明确时区的 `datetime` 转换通常更容易读懂。

## 7.4 random：让测试数据有变化

`random` 适合抽签、游戏、模拟及产生练习数据。常见函数的差别如下：

| 函数 | 作用 | 边界或特征 |
|---|---|---|
| `randint(a, b)` | 随机整数 | 包含 `a` 与 `b` |
| `randrange(start, stop)` | 从范围抽一个整数 | 不包含 `stop` |
| `random()` | 随机小数 | `0.0 <= 结果 < 1.0` |
| `choice(items)` | 抽一项 | 非空序列才可抽 |
| `sample(items, k)` | 抽 `k` 个位置且不重复取同一位置 | `k` 不可大于母体数量 |
| `shuffle(items)` | 打乱原列表 | 直接修改原列表，返回 `None` |

### 最小例：打乱与抽取有甚么不同？

```python
import random

products = ["铅笔", "笔记本", "橡皮擦", "尺"]
picked = random.choice(products)
two_products = random.sample(products, 2)
result = random.shuffle(products)

print(picked in products)
print(len(two_products))
print(len(set(two_products)))
print(sorted(products) == sorted(["铅笔", "笔记本", "橡皮擦", "尺"]))
print(result)

# 输出：
# True
# 2
# 2
# True
# None
# 说明：
# 实际选到的商品及打乱后顺序会变，但这五个检查的结果固定。
# choice() 与 sample() 返回抽取结果；shuffle() 修改原列表并返回 None。
```

`sample()` 保证不重复抽同一个位置；如果母体本来是 `["A", "A", "B"]`，结果仍可能有两个 `"A"`。上例母体没有重复商品名称，所以也能验证抽到的值不重复。

### seed：让重新得到同一段随机序列

```python
import random

random.seed(42)
first = random.randint(1, 100)
random.seed(42)
second = random.randint(1, 100)

print(first == second)

# 输出：
# True
# 说明：
# 同一执行环境中，重设相同种子并做相同操作，可以重现随机结果。
# seed() 不是「每次只抽到同一个数」；不重设种子，序列会继续前进。
```

种子适合让练习和测试重现问题。`random` 的结果可预测，不适合产生登入凭证或重设密码的连结；那类需求使用 `secrets`。

## 7.5 pathlib：先把「路径」当成数据

`pathlib` 是现代 Python 常用的路径写法；这是对旧教材 `os.path` 写法的补充。这里的 `Path` 可以先理解为「代表路径的工具」，不需要先学完整的对象导向。

一个路径存在，不代表路径所指的文件已经存在。`Path("notes.txt")` 只是创建路径数据；`write_text()` 才会写文件。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    folder = Path(temp_dir) / "shop"
    folder.mkdir()

    file_path = folder / "note.txt"
    count = file_path.write_text("好，Python", encoding="utf-8")
    content = file_path.read_text(encoding="utf-8")

    print(file_path.name)
    print(file_path.suffix)
    print(file_path.exists())
    print(count)
    print(content)

# 输出：
# note.txt
# .txt
# True
# 9
# 好，Python
# 说明：
# / 在这里用来拼接 Path；不是对字符串做除法。
# write_text() 写入文字并返回字符数；read_text() 返回读到的字符串。
# 此处临时目录会在 with 结束时清理，没有留下练习文件。
```

上例 `count` 计算的是字符数，不是 UTF-8 编码后的字节数。

常用操作：

| 操作 | 意思 |
|---|---|
| `path.name` | 最后一段文件名，例如 `note.txt` |
| `path.stem` | 去掉最后副文件名后的名称，例如 `note` |
| `path.suffix` | 最后的副文件名，例如 `.txt` |
| `path.parent` | 上一层目录的路径 |
| `path.exists()` | 是否存在 |
| `path.is_file()`、`path.is_dir()` | 是否为文件、目录 |
| `folder.mkdir(parents=True, exist_ok=True)` | 必要时创建父目录；目录已存在也接受 |
| `folder.iterdir()` | 逐一产生直接子项的路径 |

`write_text()` 会覆盖同名文件。若想追加内容，应回到已学过的 `open(..., "a", encoding="utf-8")`，而不是假设所有写文件函数都会追加。

## 7.6 os、os.path 与 sys：分清楚操作系统和 Python 本身

可以先用一句话区分：

- `os` 与 `os.path`：文件系统、路径和操作系统提供的资讯。
- `sys`：当前 Python 程序的执行环境与启动参数。

### os.path：读懂旧教材的路径操作

```python
import os
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    file_path = os.path.join(temp_dir, "notes.txt")
    with open(file_path, "w", encoding="utf-8") as file:
        file.write("这是样本")

    print(os.path.basename(file_path))
    print(os.path.splitext("notes.txt"))
    print(os.path.isfile(file_path))
    print(sorted(os.listdir(temp_dir)))

# 输出：
# notes.txt
# ('notes', '.txt')
# True
# ['notes.txt']
# 说明：
# join() 返回拼接后的路径字符串；listdir() 返回子项名称的列表。
# listdir() 的顺序没有保证，范例用 sorted() 让显示稳定。
```

Windows 和其他系统的路径分隔符可能不同。用 `os.path.join()` 或 `Path / 名称` 拼接，比手动加入 `"\\"` 或 `"/"` 更容易维护。

`os.getcwd()` 返回当前工作目录。**工作目录不一定等于程序文件所在的目录。** 例如从不同文件夹启动同一个程序时，`open("data.json")` 可能指向不同位置。

初学时可固定从作业文件夹启动程序，并把数据保存位置写清楚。不要用随意切换 `os.chdir()` 的方式掩盖路径问题。

### sys：确认正在用哪个 Python

```python
import sys

print(sys.version.split()[0])
print(sys.executable)
print(len(sys.argv) >= 1)

# 输出：
# 当前 Python 版本，例如 3.13.14（依环境而异）
# 当前 Python 直译器的完整路径（依环境而异）
# True
# 说明：
# sys.version、sys.executable、sys.argv 都是数据属性，不是需要调用的函数。
# sys.argv 是启动参数列表；第一项通常与被执行的程序或启动方式有关。
```

若执行 `python shop.py Alice`，通常 `sys.argv` 会是 `["shop.py", "Alice"]`。`input()` 是执行途中再询问；启动参数是在程序开始时就交给它，两者不同。

`sys.path` 是汇入模块时会搜索的路径列表。遇到找不到包，先确认直译器与安装环境是否一致，避免立刻手动修改 `sys.path`。

## 7.7 shutil 与 zipfile：复制、整理与压缩

两个工具箱的关注点不同：

- `shutil` 常用于复制文件、复制目录，以及高阶压缩操作。
- `zipfile` 让明确控制 ZIP 内的文件名称，并读取或解压缩 ZIP。

### 最小例：复制一份文件

```python
from pathlib import Path
from tempfile import TemporaryDirectory
import shutil

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    root = Path(temp_dir)
    source = root / "cart.txt"
    backup = root / "cart_backup.txt"
    source.write_text("铅笔 x 2", encoding="utf-8")

    copied_to = shutil.copy2(source, backup)

    print(Path(copied_to).name)
    print(backup.read_text(encoding="utf-8"))
    print(source.exists())

# 输出：
# cart_backup.txt
# 铅笔 x 2
# True
# 说明：
# copy2() 复制内容并尝试保留文件的部分中继数据，返回目的路径。
# 复制完成后，来源文件仍存在。
```

`copy2()` 不是备份版本管理：目的位置已有同名文件时可能被覆盖。目录整体复制可用 `copytree()`，本章先熟悉单一文件即可。

### 最小例：制作自己知道内容的 ZIP，再读回

```python
from pathlib import Path
from tempfile import TemporaryDirectory
import zipfile

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    root = Path(temp_dir)
    source = root / "readme.txt"
    archive = root / "lesson.zip"
    destination = root / "unpacked"
    source.write_text("Python 练习", encoding="utf-8")

    with zipfile.ZipFile(archive, "w", compression=zipfile.ZIP_DEFLATED) as zipped:
        zipped.write(source, arcname="readme.txt")

    with zipfile.ZipFile(archive, "r") as zipped:
        print(zipped.namelist())
        zipped.extractall(destination)

    print((destination / "readme.txt").read_text(encoding="utf-8"))

# 输出：
# ['readme.txt']
# Python 练习
# 说明：
# arcname 决定 ZIP 内显示的相对名称，避免把临时目录的完整路径塞进 ZIP。
# 此例只解压刚刚由自己创建、内容已知的压缩文件。
```

把 ZIP 当作容器来理解：电脑上的来源路径与压缩文件内的名称是两件事。解压未知压缩文件前，还需要检查成员路径、大小与目的地；不要直接把此教学例子套用到任意下载文件。

### shutil 的高阶压缩写法

```python
from pathlib import Path
from tempfile import TemporaryDirectory
import shutil
import zipfile

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    root = Path(temp_dir)
    source = root / "source"
    source.mkdir()
    (source / "note.txt").write_text("样本", encoding="utf-8")

    archive_path = shutil.make_archive(
        str(root / "backup"),
        "zip",
        root_dir=source,
    )

    print(Path(archive_path).name)
    with zipfile.ZipFile(archive_path, "r") as zipped:
        print(zipped.namelist())

# 输出：
# backup.zip
# ['note.txt']
# 说明：
# make_archive() 创建压缩文件，返回压缩文件路径。
# 第一个参数是文件名前缀，不必自己再加 .zip。
```

此处把压缩文件放在来源目录外面，避免打包时把正在创建的压缩文件也包进来源内容。

## 7.8 JSON：把内存数据变成可交换的文字

假设购物车是：

`{"P001": 2, "P002": 1}`

程序关闭后，内存中的字典就不存在了。要下次继续使用，需要先把它转成适合保存的格式。这个转换叫做**序列化**；把保存形式还原成数据叫做**反序列化**。

JSON 是文字格式，许多程序语言都认识。它和 Python 字典外观相近，但不是同一种语法。例如 JSON 使用 `true`、`false`、`null`，Python 使用 `True`、`False`、`None`。

### 四个名字先看方向

| 函数 | 数据流向 | 需要提供甚么 |
|---|---|---|
| `json.dumps(data)` | Python 数据 → JSON 字符串 | Python 数据 |
| `json.loads(text)` | JSON 字符串 → Python 数据 | 含 JSON 的字符串 |
| `json.dump(data, file)` | Python 数据 → 文件 | 数据、已开启的文字文件 |
| `json.load(file)` | 文件 → Python 数据 | 已开启的文字文件 |

内存方式：这四个 JSON 函数中，尾端的 `s` 可以理解为「处理 string」。`loads("cart.json")` 并不会替开文件；它会尝试把 `cart.json` 这几个字符当成 JSON 解析。

### 最小例：来回转换后，型别可能改变

```python
import json

original = {
    "姓名": "小明",
    "标签": ("新手", "会员"),
    1: "编号数据",
    "启用": True,
}

text = json.dumps(original, ensure_ascii=False)
restored = json.loads(text)

print(isinstance(text, str))
print(restored["标签"])
print(restored["1"])
print(type(restored["启用"]).__name__)
print(original == restored)

# 输出：
# True
# ['新手', '会员']
# 编号数据
# bool
# False
# 说明：
# dumps() 返回 JSON 字符串；loads() 返回解析后的 Python 数据。
# JSON 没有 tuple，元组会变成阵列，读回 Python 后是 list。
# JSON 对象的 key 是字符串，原本整数 key 1 读回后变成 "1"。
```

这也是商品 ID 使用 `"P001"` 这类字符串的好理由：保存前后的识别值容易保持一致。

`ensure_ascii=False` 让中文以可阅读的字符写在 JSON 文字中。它不会替设定文件编码；开文件时仍应使用 `encoding="utf-8"`。

### 最小例：保存购物车并读回

```python
import json
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    file_path = Path(temp_dir) / "cart.json"
    cart = {
        "items": [
            {"product_id": "P001", "quantity": 2, "unit_price_cent": 350},
        ]
    }

    with file_path.open("w", encoding="utf-8") as file:
        written = json.dump(cart, file, ensure_ascii=False, indent=2)

    with file_path.open("r", encoding="utf-8") as file:
        restored = json.load(file)

    item = restored["items"][0]
    total_cent = item["quantity"] * item["unit_price_cent"]
    print(written)
    print(restored == cart)
    print(total_cent)

# 输出：
# None
# True
# 700
# 说明：
# dump() 已经执行写文件动作，返回值是 None；load() 返回读回的数据。
# 金额以整数「分」保存，避免直接用二进位浮点数累加货币金额。
```

读文件成功只表示「文字可以被解析」，不表示内容符合的业务规则。载入后仍需验证：

- 最外层是不是字典？有没有 `items`？
- 数量是不是正整数？
- 商品 ID 是否存在？
- 金额是否使用约定的单位？

### JSON 的四个常见坑

1. **一个文件连续 `dump` 两次。** 这通常会得到两份 JSON 直接黏在一起，`load` 无法把它当成一个完整 JSON 值。先组成一个列表或字典，再一次写入。
2. **以为任何 Python 对象都能保存。** `set`、`datetime`、函数等不能直接用预设 JSON 编码器保存。先转成列表、ISO 时间字符串等约定格式。
3. **整数 key 与字符串 key 混用。** `1` 和 `"1"` 在 Python 字典中可以同时存在，但转成 JSON key 后可能冲突。
4. **把解析当成验证。** JSON 语法合法，不代表 `quantity=-5` 就是合理的购买数量。

文件不存在或 JSON 损坏时如何处理，见 [错误处理与验证桥接](bridge-errors-testing.md)。当前先掌握正常数据的保存与读回。

## 7.9 pickle：保存 Python 专用数据

JSON 强调跨语言交换；`pickle` 著重保存 Python 支援的数据结构，因此能保留某些 JSON 不保留的型别，例如元组。

`pickle` 使用字节数据。文件必须用 `"wb"` 写入、`"rb"` 读取。它的 `dumps()` 返回 `bytes`，与 `json.dumps()` 返回 `str` 不同。

```python
import pickle
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    file_path = Path(temp_dir) / "sample.pkl"
    original = {"position": (3, 5), "tags": {"a", "b"}}

    with file_path.open("wb") as file:
        pickle.dump(original, file)

    with file_path.open("rb") as file:
        restored = pickle.load(file)

    print(restored["position"])
    print(isinstance(restored["tags"], set))
    print(restored == original)

# 输出：
# (3, 5)
# True
# True
# 说明：
# 此例只载入本次程序自己创建的可信样本。
# pickle.load() 返回还原后的 Python 数据；pickle.dump() 返回 None。
```

`pickle.load()` 在还原数据时可能执行程序代码，所以不要载入陌生人提供或来源不明的 pickle 文件。对本系列的购物车与订单练习，优先使用容易查看与验证的 JSON。

## 7.10 configparser：设定文件读出来，预设仍是文字

设定文件适合保存可调整的选项，例如商店名称、库存下限、是否显示提示。它和订单数据不同：设定描述程序如何运作，订单描述已经发生甚么。

INI 格式长得像这样：

```ini
[shop]
name = 练习商店
low_stock = 5
debug = no
```

`[shop]` 是区段名称，下面是键值配对。

```python
import configparser
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    file_path = Path(temp_dir) / "settings.ini"
    file_path.write_text(
        "[shop]\nname = 练习商店\nlow_stock = 5\ndebug = no\n",
        encoding="utf-8",
    )

    config = configparser.ConfigParser()
    loaded_files = config.read(file_path, encoding="utf-8")

    print(len(loaded_files))
    print(config["shop"]["name"])
    print(type(config["shop"]["low_stock"]).__name__)
    print(config.getint("shop", "low_stock") + 1)
    print(config.getboolean("shop", "debug"))
    print(config.get("shop", "currency", fallback="CNY"))

# 输出：
# 1
# 练习商店
# str
# 6
# False
# CNY
# 说明：
# read() 返回成功读到的文件名称列表；它不是设定内容本身。
# 直接取值通常得到字符串；getint()、getboolean() 会依规则转换型别。
```

特别注意：`bool("no")` 的结果是 `True`，因为它是非空字符串。读 INI 里的布尔设定时，使用 `getboolean()`；它能理解 `yes/no`、`true/false`、`on/off` 和 `1/0` 等形式。

`config.read()` 对不存在的文件通常不会直接抛出「找不到文件」错误，所以正式程序若一定要有设定文件，应检查 `loaded_files` 是否为空。

`fallback` 用于设定缺失时提供预设值；它不能把 `low_stock=abc` 这种错误数字自动变成有效整数。要保存改好的设定，可用文字模式开文件后调用 `config.write(file)`。

## 7.11 subprocess：启动另一个程序，并拿回结果

普通函数调用仍在当前这个 Python 程序里执行。`subprocess` 则可以启动另一个行程。

先分清楚三种结果：

| 名字 | 意思 |
|---|---|
| `stdout` | 子程序正常输出的文字 |
| `stderr` | 子程序的错误或诊断输出 |
| `returncode` | 子程序结束时的整数状态码，惯例上 `0` 表示成功 |

### 最小例：让另一个 Python 打印 5

```python
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "-c", "print(2 + 3)"],
    capture_output=True,
    text=True,
    check=True,
    timeout=5,
    shell=False,
)

print(result.stdout.strip())
print(result.returncode)
print(result.stderr == "")

# 输出：
# 5
# 0
# True
# 说明：
# run() 返回包含执行结果的 CompletedProcess，并非直接返回数字 5。
# 子程序的 print() 写入 stdout；主程序读 result.stdout 后才显示它。
# sys.executable 指向当前的 Python，因此不依赖系统是否有其他命令。
```

逐项理解：

- 参数使用列表，每个元素是一个独立参数。
- `capture_output=True` 收集输出，而不是直接把子程序输出送到当前画面。
- `text=True` 让输出以文字处理，而不是原始 `bytes`。
- `check=True` 表示非零结束码会抛出异常。
- `timeout=5` 限制等待时间。
- `shell=False` 直接执行指定程序，不经命令壳层重新解读字符串。

上例里的 `.strip()` 去除 `print` 产生的结尾换行；原本的 `stdout` 是 `"5\n"`。

不要把用户输入拼成一大串命令再交给 `shell=True`。需要传入商品名称等数据时，把它作为参数列表中的一个元素，并让接收程序把它当数据处理。失败、逾时的异常处理可接着阅读 [错误处理与验证桥接](bridge-errors-testing.md)。

## 7.12 猴子补丁：执行时替换既有属性

「猴子补丁」通常指程序执行时，把既有模块或对象的属性替换掉。它不是新的语法；核心仍是已学过的赋值。

初学阶段的重点是：**看懂它会改变后续查找到的函数，而且知道要恢复原状。**

```python
import time

original_time = time.time

try:
    time.time = lambda: 123.0
    print(time.time())
finally:
    time.time = original_time

print(time.time is original_time)

# 输出：
# 123.0
# True
# 说明：
# original_time 保存原函数本身，没有加括号，因此没有先调用它。
# 赋值后，time.time 指向新函数；finally 会把原函数恢复。
```

`try/finally` 在这里表示「无论前面成功或出错，都执行恢复」。详细异常语法见 [错误处理与验证桥接](bridge-errors-testing.md)。

这种修改会影响同一行程中后续使用 `time.time` 的地方，容易使程序难以追踪。自己的函数若需要「可控制的时间」，通常把时间作为参数传入更清楚。也要注意：若某处早已用 `from time import time` 保存了原函数引用，替换模块属性不一定会改到那个引用。

这一节以理解现有程序为目标，不要求在购物系统作业中使用猴子补丁。

## 7.13 标准库、第三方包、pip 与 PyCharm

本章大部分工具随 Python 一起提供，通常不需要另外安装。`pip` 主要负责安装和管理第三方包；例如下载别人提供的数据分析或网路包。

三者关系要分清楚：

| 名称 | 角色 |
|---|---|
| Python 直译器 | 执行的程序 |
| pip | 把第三方包安装到某个 Python 环境 |
| PyCharm | 编写、执行与除错程序的编辑环境；它需要选定 Python 直译器 |

同一台电脑可以有多个 Python 环境。在 A 环境安装的包，不会自然出现在 B 环境。

### 读懂安装命令，不必为本章额外安装

下面是**终端命令示意**，不是 Python 程序，也不用为了本章执行最后一行：

```text
python -m venv .venv
python -m pip --version
python -m pip install 包名称
```

`python -m pip` 表示「用这个 Python 执行它所属的 pip」。比起只打 `pip`，更容易辨认安装目标。但第一行创建 `.venv` 后，不会自动把之后的 `python` 切换到新环境；还需要启用环境，或直接指定环境中的直译器。

例如在 Windows 的作业文件夹里，可以使用：

```text
.\.venv\Scripts\python.exe -m pip --version
```

在 macOS 或 Linux 常见的是：

```text
./.venv/bin/python -m pip --version
```

以上命令只查看 pip 资讯；只有 `install` 才会安装包。

PyCharm 中的选单名称可能随版本改变，但判断原则不变：查看专案选用的 Python Interpreter，确认它与安装包的环境一致。若「明明安装了，却无法 import」，先打印前面学过的 `sys.executable` 来确认。

虚拟环境可以理解为每个专案自己的包区域。它不会自动改写的程序，也不是用来保存订单或用户数据。

## 7.14 把工具串起来：一笔订单的数据旅程

现在可以把前面的知识接成一条清楚的流程：

1. `input()` 得到用户输入的文字。
2. 自己的函数验证商品 ID 与数量，计算整数金额。
3. `datetime` 产生带时区的下单时间，再转成 ISO 字符串。
4. 字典和列表保存订单数据。
5. `json.dump()` 把数据写入明确的文件路径。
6. 下次使用 `json.load()` 读回，再做栏位与业务验证。

这里每个工具只负责一小段工作。不要因为本章学了压缩、子行程和猴子补丁，就把它们全部塞进同一个小作业。先把正确的数据流做清楚。

## 7.15 动手练习

### 练习 A｜七天后到期

输入一个 `YYYY-MM-DD` 形式的日期，用 `datetime.strptime` 或 `date.fromisoformat` 解析，计算七天后的日期。

验收数据：

- `2026-09-15` → `2026-09-22`
- `2026-12-28` → `2027-01-04`
- `2024-02-25` → `2024-03-03`

先完成合法输入；学完错误处理后，再处理 `2026-02-30`。

### 练习 B｜购物车 JSON 往返

在临时目录创建一份包含至少两种商品的购物车 JSON。每项有字符串商品 ID、正整数数量和以分表示的单价。

要求：

- 先保存，再从文件载入，**不要直接拿原变量算总额**。
- 从读回数据计算总额。
- 修改一项数量，再完整覆写保存，并重新载入确认。
- 中文商品名称能正常显示。

思考：为甚么不应该直接向同一文件连续追加两次 `json.dump`？

### 练习 C｜设定驱动提示

自己产生 INI 设定，包含商店名、库存下限与是否显示提醒。载入后，当商品库存低于下限而提醒已开启时，打印补货提示。

至少核对三种情况：提醒开启且库存不足、提醒关闭且库存不足、库存充足。不要用 `bool("false")` 解析设定。

### 练习 D｜创建练习备份

在临时目录中产生两个纯文字文件，打包成 ZIP，列出 ZIP 中的文件名，再解压到另一个空目录。核对解压后内容与来源相同。

练习只处理自己在这次程序执行中创建的数据。

## 7.16 自查与简答

先遮住右栏，试著用自己的话回答。

| 问题 | 简答 |
|---|---|
| 计算程序耗时，为甚么用 `perf_counter`？ | 它适合量测经过时间；两次读数相减得到耗时。 |
| `strftime` 和 `strptime` 分别做甚么？ | 前者把时间转文字，后者按格式把文字解析成时间。 |
| `randint(1, 6)` 可能得到 6 吗？ | 可以，两端都包含。 |
| `result = random.shuffle(items)` 后，`result` 是甚么？ | `None`；被打乱的是原列表 `items`。 |
| 创建 `Path("a.txt")` 是否已经创建文件？ | 没有，它只是路径数据。 |
| `json.loads("cart.json")` 会读文件吗？ | 不会，它尝试解析这段字符串；读文件用开文件后的 `json.load(file)`。 |
| 元组和整数 key 经 JSON 往返会怎样？ | 元组读回成列表，整数 key 读回成字符串。 |
| `dump` 和 `print` 的共同返回值是甚么？ | 这里的 `json.dump` 与 `print` 都返回 `None`，但额外动作不同。 |
| `bool("no")` 为甚么是 `True`？ | 非空字符串为真；INI 布尔值应用 `getboolean` 解析。 |
| `subprocess.run` 返回的就是子程序打印的文字吗？ | 不是，返回结果对象；文字在 `stdout` 属性中。 |
| 为甚么范例用 `sys.executable`？ | 明确使用当前的 Python，避免误用另一个环境。 |
| pickle 比 JSON 保留更多态别，就应该总用 pickle 吗？ | 不应；还要考虑可信来源、可读性与跨语言需求。购物作业优先用 JSON。 |
| 创建虚拟环境后，编辑器会自动选它吗？ | 不一定，仍要确认专案所选直译器。 |

读下一章前，请确认能不看范例完成「数据 → JSON 文件 → 数据 → 计算结果」这个小循环。下一章会用杂凑检查文件、用正则检查文字，并用日志记录程序发生过的事。
