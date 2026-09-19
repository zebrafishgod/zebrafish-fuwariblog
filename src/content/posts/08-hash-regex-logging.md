---
title: Python 第 340～363 集，哈希值、正则表达式，以及 logging 的设定、logger、handler、命名与日志分类
published: 2026-09-17
updated: 2026-09-19
description: 哈希值、正则表达式，以及 logging 的设定、logger、handler、命名与日志分类
tags: [Python, 基础篇]
category: Python 学习
draft: false
lang: zh_CN
---

# 第 8 章｜杂凑、正则与日志：检查数据，理解程序发生了甚么

本章对应基础篇目录约第 340～363 集，主题包括内建 `hash`、`hashlib`、正则表达式，以及 logging 的设定、logger、handler、命名与日志分类。影片中穿插的包补充，集中整理在 [第 6 章：模块与包](06-modules-packages.md)。本章依已核对的目录整理，不是逐集影片的逐字稿。

前置知识：字符串、字节的基本概念、字典、函数、文件与模块。本章需要处理少量异常；相关语法可配合 [错误处理与验证桥接](bridge-errors-testing.md) 阅读。

本章回答三个不同问题：

| 工具 | 它帮回答甚么？ | 例子 |
|---|---|---|
| 杂凑摘要 | 这份内容和之前是否一致？ | 备份文件与原文件是否相同 |
| 正则表达式 | 这段文字是否符合某种形式？ | 商品 ID 是否为 P 加三个数字 |
| 日志 | 程序何时做过甚么？哪一步失败？ | 订单创建成功、数据载入失败 |

学会这些工具后，程序会更容易验证与排错。它们仍然要创建在正确的数据结构与业务流程上。

---

## 8.1 内建 hash：先理解字典为甚么能按 key 查值

可以把杂凑理解为：根据一份数据，算出一个便于比较和查找的数值。

Python 的字典和集合会使用杂凑机制协助查找。内建 `hash()` 返回整数，但它主要服务 Python 执行期间的数据操作。

```python
product_id = "P001"
same_id = "".join(["P", "001"])

print(product_id == same_id)
print(hash(product_id) == hash(same_id))
print(isinstance(hash((1, 2)), int))

# 输出：
# True
# True
# True
# 说明：
# hash() 返回整数；这里比较返回值是否相同，没有显示不稳定的具体数字。
# 在同一行程中，相等且可杂凑的对象必须有相同的 hash。
```

但是，**hash 相同不代表数据必定相同**。不同数据有可能得到同一个杂凑值，这叫做碰撞。字典不会只靠杂凑数字决定两个 key 相等，还会进一步处理比较。

常见的可杂凑数据包括字符串、整数，以及所有成员都可杂凑的元组。列表、字典、集合通常不可杂凑，所以不能直接作字典 key。

注意元组本身不可变，不代表它里面的成员都可杂凑：`(1, 2)` 可以，包含列表的 `(1, [2])` 就不可以。

### 为甚么不能用 hash() 当永久文件指纹？

Python 预设会对字符串、字节等的杂凑加入行程相关的随机化。同一段文字，在下次重新启动 Python 后，`hash()` 的数字可能不同；不同环境也不应假设完全一致。

所以：

- 理解字典查找时，可以研究 `hash()`。
- 需要保存、交换、跨程序执行核对内容摘要时，使用 `hashlib`。

## 8.2 hashlib：把内容变成固定长度的摘要

以 SHA-256 为例，输入可以长可以短，输出摘要固定为 256 个位，也就是 32 个字节。若用十六进位文字显示，就会是 64 个字符。

「摘要」可以理解为内容的指纹，但不是把原文压缩成可还原的格式。 不能拿 SHA-256 摘要直接解回原始文件。

### 最小例：先编码，再计算

```python
import hashlib

text = "hello"
data = text.encode("utf-8")
hasher = hashlib.sha256(data)

print(hasher.hexdigest())
print(len(hasher.digest()))
print(len(hasher.hexdigest()))

# 输出：
# 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
# 32
# 64
# 说明：
# encode() 把 str 转为 bytes；hashlib 处理的是字节。
# digest() 返回 32 个原始字节；hexdigest() 返回 64 字符的十六进位字符串。
# 这些方法返回数据，不会自行打印结果。
```

`"hello"` 与 `"hello\n"` 的内容不同，摘要也不同。文件的换行、空格、编码或任一字节改变，都会影响摘要。

`hashlib.sha256("hello")` 不能直接使用普通字符串；需要先决定编码。跨系统核对文字时，双方的编码约定也要一致。

### update：可以分多次提供数据

```python
import hashlib

whole = hashlib.sha256(b"hello").hexdigest()

in_parts = hashlib.sha256()
in_parts.update(b"he")
in_parts.update(b"llo")

print(in_parts.hexdigest() == whole)

# 输出：
# True
# 说明：
# 依照原顺序 update(b"he")、update(b"llo")，等同处理 b"hello"。
# update() 更新摘要状态并返回 None；hexdigest() 返回当前的摘要文字。
```

分批输入保持了原始字节顺序，所以不会改变摘要。不能先把每个分块各自变成摘要，再把摘要串起来，并期待得到相同结果。

## 8.3 SHA-256 文件校验：一块一块读，避免一次载入整份大文件

小文件可以一次读完，但大文件可能占用很多内存。计算摘要不需要保留完整内容，只需要按原顺序读取每一块。

```python
import hashlib
from pathlib import Path
from tempfile import TemporaryDirectory


def sha256_file(file_path):
    hasher = hashlib.sha256()
    with open(file_path, "rb") as file:
        while True:
            chunk = file.read(8192)
            if not chunk:
                break
            hasher.update(chunk)
    return hasher.hexdigest()


with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    file_path = Path(temp_dir) / "sample.txt"
    sample = b"Python basics\n" * 5000
    file_path.write_bytes(sample)

    from_file = sha256_file(file_path)
    expected = hashlib.sha256(sample).hexdigest()

    print(len(from_file))
    print(from_file == expected)

# 输出：
# 64
# True
# 说明：
# read(8192) 每次最多读 8192 个字节；最后一块可以比较短。
# 读到文件结尾时得到 b""，if not chunk 成立，结束循环。
# sha256_file() 返回摘要字符串；调用端决定要比较、保存或显示。
```

这个函数只负责一件事：根据文件内容返回摘要。把「打印结果」留在函数外面，测试与重用都更容易。

**摘要的可信度要和来源一起看。** 若只想检查自己复制文件时是否出错，核对两份文件的摘要很实用。若下载站点给了摘要，应从可信管道取得它；攻击者若能同时替换文件与摘要，单纯比较两者无法辨别来源。

MD5 与 SHA-1 也经常出现在旧教材中，但已有碰撞安全问题；新例子以 SHA-256 为主。这不表示 SHA-256 适合所有用途，下一节的密码保存就是不同需求。

## 8.4 盐与密码摘要：为甚么不能只调用一次 sha256？

文件摘要希望计算快速，这样大文件也容易核对。密码验证却需要控制猜测成本，避免攻击者每秒大量尝试候选密码。

因此，**不要把 `sha256(password)` 或 `sha256(salt + password)` 当成正式的密码保存方案**。加入盐仍然没有改变它计算太快的问题。

这里分清楚三个概念：

- **盐（salt）**：每次创建密码纪录时产生的随机数据，让不同纪录即使使用相同密码，也能得到不同结果。
- **密码衍生函数**：例如 PBKDF2 或 scrypt，透过可调整的计算成本，使大量猜测更昂贵。
- **验证**：用输入的密码与保存的参数重新计算，再比较摘要；不是解密取回原密码。

盐不需要保密，但需要和算法、成本参数、摘要一起保存。

### 教学示例：使用 Python 标准库的 PBKDF2

这是理解数据流的独立示例，不是购物系统作业的必要前提，也不要求在基础篇实作登入系统。

```python
import hashlib
import hmac
import secrets


def make_password_record(password):
    salt = secrets.token_bytes(16)
    iterations = 600_000
    derived = hashlib.pbkdf2_hmac(
        "sha256",
        password.encode("utf-8"),
        salt,
        iterations,
    )
    return {
        "algorithm": "pbkdf2_hmac_sha256",
        "iterations": iterations,
        "salt_hex": salt.hex(),
        "digest_hex": derived.hex(),
    }


def verify_password(password, record):
    salt = bytes.fromhex(record["salt_hex"])
    candidate = hashlib.pbkdf2_hmac(
        "sha256",
        password.encode("utf-8"),
        salt,
        record["iterations"],
    )
    expected = bytes.fromhex(record["digest_hex"])
    return hmac.compare_digest(candidate, expected)


record = make_password_record("classroom-example")
print(verify_password("classroom-example", record))
print(verify_password("different-example", record))

# 输出：
# True
# False
# 说明：
# 盐每次执行都不同，因此保存的具体摘要也会不同；验证结果保持一致。
# make_password_record() 返回设定与摘要数据，不返回或保存明文密码。
# compare_digest() 返回比较结果，适合用于这类敏感摘要比较。
```

此处 `600_000` 是示范成本参数，不是适用所有装置和所有服务的永久设定。实际系统应采用成熟认证框架、依当时建议与性能测试选择参数，并验证保存纪录的格式和算法版本。

此刻需要记住的是：SHA-256 文件校验、Python 的 `hash()` 和密码保存，虽然都提到「杂凑」，却有不同要求。

## 8.5 正则表达式：用一条规则描述文字形式

正则表达式常简称 regex。Python 使用 `re` 模块。

例如商品 ID 规则是：

> 第一个字符必须是大写 P，后面刚好有三个 ASCII 数字。

对应的规则可以写成 `r"P[0-9]{3}"`。先读懂这个小例子，再学更多符号：

- `P`：字面上的大写 P。
- `[0-9]`：从 0 到 9 的一个字符。
- `{3}`：前一个单位重复三次。

### 最小例：验证完整输入

```python
import re

pattern = r"P[0-9]{3}"

print(re.fullmatch(pattern, "P001") is not None)
print(re.fullmatch(pattern, "P12") is not None)
print(re.fullmatch(pattern, "P001-extra") is not None)
print(re.fullmatch(pattern, "p001") is not None)

# 输出：
# True
# False
# False
# False
# 说明：
# fullmatch() 成功时返回 Match，失败时返回 None。
# 「is not None」把匹配是否成功转成明确的布尔结果。
```

符合文字形式，不表示商品存在。`"P999"` 可能通过正则，但字典里没有这个商品；后续仍要检查 `product_id in products`。

## 8.6 r 字符串：先分清 Python 的跳脱与正则的跳脱

同一个反斜线可能先被 Python 字符串语法解读，再交给正则引擎。为了少处理一层跳脱，正则通常使用 `r"..."` 原始字符串。

```python
import re

print(len("\n"))
print(len(r"\n"))
print(re.fullmatch(r"[0-9]+\.[0-9]{2}", "12.50") is not None)
print(re.fullmatch(r"[0-9]+\.[0-9]{2}", "12x50") is not None)

# 输出：
# 1
# 2
# True
# False
# 说明：
# "\n" 是一个换行字符；r"\n" 保存反斜线和 n 两个字符。
# 正则里的 \. 表示字面上的小数点；未跳脱的 . 有特殊意义。
```

`r` 只改变 Python 对字符串的处理方式，**不会关闭正则语法**。例如 `r"\d"` 仍会被正则引擎解读为数字类别。

另一个常见错误是用 `"\bword\b"` 想写单字边界，但 Python 会先把 `\b` 当成退格字符。此时应使用 `r"\bword\b"`。

## 8.7 常见语法：不要一次背完，按需求查

| 语法 | 含义 | 小例子 |
|---|---|---|
| `abc` | 依序匹配字面字符 | `abc` |
| `.` | 一个字符，预设不含换行 | `a.c` 可匹配 `abc` |
| `[abc]` | 列出可接受的单一字符 | 一个 a、b 或 c |
| `[^abc]` | 一个不是 a、b、c 的字符 | 中括号内开头的 `^` 表示排除 |
| `[0-9]` | 一个 ASCII 数字 | 商品编号常用 |
| `\d` | 一个 Unicode 十进位数字 | 比 `[0-9]` 的范围更广 |
| `\D` | 一个非 `\d` 字符 | 与 `\d` 相反 |
| `\w` | Unicode 文数字或底线 | 不只英文字母 |
| `\W` | 一个非 `\w` 字符 | 与 `\w` 相反 |
| `\s` | 空白字符 | 空格、tab、换行等 |
| `\S` | 非空白字符 | 与 `\s` 相反 |
| `*` | 前一单位出现 0 次以上 | `ab*` 可匹配 `a`、`ab` |
| `+` | 前一单位出现 1 次以上 | `ab+` 不匹配只有 `a` |
| `?` | 前一单位出现 0 次或 1 次 | `colou?r` |
| `{m}` | 刚好 m 次 | `[0-9]{3}` |
| `{m,n}` | 至少 m 次，至多 n 次 | `[0-9]{2,4}` |
| `^` | 字符串开头；受 MULTILINE 影响 | `^P` |
| `$` | 字符串尾端或末尾换行之前；受 MULTILINE 影响 | `end$` |
| `A\|B` | A 或 B 其中一种规则 | `red\|blue` |
| `(...)` | 分组并撷取内容 | `(P[0-9]{3})` |
| `(?:...)` | 分组但不撷取 | `(?:cat\|dog)` |
| `\b` | 单字边界 | 边界依 `\w` 与非 `\w` 判断 |

表格中的「前一单位」可以是一个字符、一个字符类别，或者一个分组。`ab{3}` 表示 a 后面跟三个 b；`(ab){3}` 才表示整组 ab 重复三次。

常见旗标：

- `re.IGNORECASE` 或 `re.I`：忽略大小写。
- `re.MULTILINE` 或 `re.M`：改变 `^`、`$` 对每一行的处理。
- `re.DOTALL` 或 `re.S`：让 `.` 也能匹配换行。
- `re.ASCII` 或 `re.A`：把 `\d`、`\w` 等相关类别限制为 ASCII 规则。

「多行」和「点号包含换行」是不同设定，不要把 `re.M` 与 `re.S` 当成同一件事。

### \d 不一定只代表键盘上的 0～9

```python
import re

text = "１２３"

print(re.fullmatch(r"\d{3}", text) is not None)
print(re.fullmatch(r"[0-9]{3}", text) is not None)

# 输出：
# True
# False
# 说明：
# text 是全形数字；预设 \d 接受这些 Unicode 十进位数字。
# 若规格明确要求 ASCII 商品编号，使用 [0-9] 更直观。
```

## 8.8 search、match、fullmatch、findall：要找一段，还是验证全部？

| 函数 | 查找范围或目的 | 返回值 |
|---|---|---|
| `re.search` | 从任意位置寻找第一个匹配 | Match 或 `None` |
| `re.match` | 必须从开头匹配，但不要求用完整段字符串 | Match 或 `None` |
| `re.fullmatch` | 整段字符串都必须符合 | Match 或 `None` |
| `re.findall` | 找到全部不重叠匹配 | 列表；元素受撷取分组影响 |
| `re.finditer` | 逐一取得所有不重叠匹配 | 产生 Match 的迭代器 |

```python
import re

searched = re.search(r"[0-9]+", "订单 A12 已创建")
matched = re.match(r"[0-9]+", "12abc")
fully_matched = re.fullmatch(r"[0-9]+", "12abc")
all_numbers = re.findall(r"[0-9]+", "买 2 枝笔和 3 本书")

print(searched.group() if searched is not None else "找不到")
print(matched.group() if matched is not None else "找不到")
print(fully_matched)
print(all_numbers)

# 输出：
# 12
# 12
# None
# ['2', '3']
# 说明：
# group() 返回这次匹配到的文字；不能对 None 调用 group()。
# findall() 找到的是字符串，若要相加仍需做 int() 等型别转换。
```

做商品 ID 等完整输入验证时，优先考虑 `fullmatch`。只用 `match`，可能会让 `"123abc"` 这种尾端多了文字的输入通过原本只想允许数字的检查。

使用 `fullmatch` 也能避免误解 `$` 对最后换行的特殊行为。不要把 `^...$` 当成所有情况下都等价于 `fullmatch`。

## 8.9 分组：先匹配整段，再取出要的部分

假设一行文字是 `P001:12`，要取出商品 ID 和数量。分组让这两段内容有各自的位置或名字。

```python
import re

pattern = r"(?P<product_id>P[0-9]{3}):(?P<quantity>[0-9]+)"
matched = re.fullmatch(pattern, "P001:12")

if matched is not None:
    print(matched.group(0))
    print(matched.group("product_id"))
    print(int(matched.group("quantity")) + 1)
    print(matched.groupdict())

# 输出：
# P001:12
# P001
# 13
# {'product_id': 'P001', 'quantity': '12'}
# 说明：
# group(0) 取得完整匹配；具名分组让取值目的更清楚。
# groupdict() 返回分组名到文字的字典，数量当前仍是 str。
```

普通分组可用 `group(1)`、`group(2)` 取值。分组增加后，数字位置容易变，具名分组有助于阅读。

### findall 的结果，为甚么有时是字符串，有时是元组？

```python
import re

text = "P001=2; P002=10"

print(re.findall(r"P[0-9]{3}=[0-9]+", text))
print(re.findall(r"(P[0-9]{3})=[0-9]+", text))
print(re.findall(r"(P[0-9]{3})=([0-9]+)", text))

# 输出：
# ['P001=2', 'P002=10']
# ['P001', 'P002']
# [('P001', '2'), ('P002', '10')]
# 说明：
# 无撷取分组：返回完整匹配；一组：返回该组字符串；多组：返回元组。
# 只是想把规则括成一组而不改变撷取结果，可以使用 (?:...)。
```

不要只看正则能不能匹配，也要确认它返回的数据形状。这和学习列表、元组、字典时的思路相同。

## 8.10 贪婪与非贪婪：同样能匹配时，要取多长？

`*`、`+`、`{m,n}` 等重复规则预设倾向取较长的匹配，但仍要让整个规则成功，这常称为贪婪匹配。在量词后加 `?`，则先尝试较短的匹配。

```python
import re

text = "[A][B]"

print(re.findall(r"\[.*\]", text))
print(re.findall(r"\[.*?\]", text))
print(re.findall(r"\[[^\]]*\]", text))

# 输出：
# ['[A][B]']
# ['[A]', '[B]']
# ['[A]', '[B]']
# 说明：
# .* 可以吃到最后一个 ] 前面；.*? 先尝试最短可成功的范围。
# [^\]]* 明确限制内容不能包含 ]，对这种简单分隔数据更直接。
```

「非贪婪」不是「最多一个字符」。它仍可能匹配很长的内容，只要完成整个规则需要那么长。

正则适合有明确文字规则的任务。遇到完整 JSON、巢状结构、HTML 文件等，优先使用对应解析器，避免把复杂数据格式硬塞进一条正则。

## 8.11 替换、切割与重用规则

```python
import re

date_text = "2026-09-15"
reformatted = re.sub(
    r"([0-9]{4})-([0-9]{2})-([0-9]{2})",
    r"\1/\2/\3",
    date_text,
)
parts = re.split(r"[,;]\s*", "P001, P002; P003")
product_pattern = re.compile(r"P[0-9]{3}")

print(reformatted)
print(parts)
print(product_pattern.fullmatch("P005") is not None)

# 输出：
# 2026/09/15
# ['P001', 'P002', 'P003']
# True
# 说明：
# sub() 返回替换后的新字符串，不会修改原字符串。
# 替换字符串中的 \1、\2、\3 引用匹配到的分组。
# compile() 返回可重复使用的规则对象，仍可调用 fullmatch() 等方法。
```

重用 `re.compile` 的好处之一是把规则放在有意义的变量名下，让后面的程序更容易读。这里不需要自己定义类别。

上例只改变日期文字的排列。`2026-99-88` 也能符合「四个数字、两个数字、两个数字」的形式；判断日期真实有效，应交给 `datetime` 解析。

## 8.12 logging：留下能回头追查的事件纪录

`print` 适合当前用户需要看见的画面，例如菜单、输入提示和结算金额。日志则可以记录操作与失败原因，并依严重程度、文件位置、模块名称调整输出。

同一个事件可以在画面上简短提示，在日志里保留更有用的技术资讯。

常见级别由低到高：

| 级别 | 通常用来记录 |
|---|---|
| `DEBUG` | 供开发时追查的细节 |
| `INFO` | 正常流程中的重要事件，例如订单已创建 |
| `WARNING` | 需要留意，但程序仍可继续，例如存量偏低 |
| `ERROR` | 某项操作失败，例如这笔数据无法保存 |
| `CRITICAL` | 严重到使主要功能无法正常运作的事件 |

级别不是文字颜色，也不是「用户做错就一律记 ERROR」。例如输入不合法可能只是正常互动中的可预期情况，应依需要记录。

### 最小例：只显示 INFO 及更严重的事件

```python
import logging
import sys

logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s: %(message)s",
    stream=sys.stdout,
    force=True,
)

logging.debug("这行供除错使用")
logging.info("商店已启动")
logging.warning("铅笔库存偏低")

# 输出：
# INFO: 商店已启动
# WARNING: 铅笔库存偏低
# 说明：
# DEBUG 低于设定门槛，不会显示；INFO、WARNING 会显示。
# 日志函数执行记录动作并返回 None，不是返回格式化后的字符串。
```

这里明确使用 `sys.stdout`，让例子的显示顺序容易核对。预设串流 handler 通常使用 `stderr`，所以有时会在编辑器的错误输出区看见一般日志；这不代表每一行都是错误。

`basicConfig` 一般只在根 logger 尚无 handler 时生效。本章的独立示例使用 `force=True`，使重跑范例时能重新设定；正式应用通常在程序入口集中配置一次，模块内只取得 logger 并使用。

## 8.13 Logger、Handler、Formatter：谁说话，送去哪，长甚么样

先用一个生活例子理解：

> 结帐模块发出「订单创建」事件；画面只看重要消息，文件保留更多细节；每条消息都加上级别与内容。

对应三种工具：

| 名称 | 负责的工作 |
|---|---|
| Logger | 给事件命名、接受日志调用，决定是否创建纪录 |
| Handler | 把纪录送到特定目的地，例如画面或文件，并再次按级别筛选 |
| Formatter | 决定输出文字的格式 |

一个 logger 可以有多个 handler。下面的例子让画面只显示 INFO 以上，文件保留 DEBUG 以上。

```python
import logging
import sys
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    log_path = Path(temp_dir) / "shop.log"
    logger = logging.getLogger("lesson.routing")
    logger.setLevel(logging.DEBUG)
    logger.propagate = False

    formatter = logging.Formatter("%(levelname)s: %(message)s")
    console = logging.StreamHandler(sys.stdout)
    console.setLevel(logging.INFO)
    console.setFormatter(formatter)

    file_handler = logging.FileHandler(log_path, encoding="utf-8")
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(formatter)

    logger.addHandler(console)
    logger.addHandler(file_handler)
    try:
        logger.debug("载入 2 个商品")
        logger.info("订单 %s 已创建", "O001")
    finally:
        logger.removeHandler(console)
        logger.removeHandler(file_handler)
        console.close()
        file_handler.close()

    print("文件内容：")
    print(log_path.read_text(encoding="utf-8").strip())

# 输出：
# INFO: 订单 O001 已创建
# 文件内容：
# DEBUG: 载入 2 个商品
# INFO: 订单 O001 已创建
# 说明：
# logger 先允许 DEBUG 以上，再由各 handler 决定送出的门槛。
# 两份内容出现同一个 INFO，是因为它分别进入画面和文件。
# 关闭文件 handler 后再让临时目录清理，Windows 上也能正常收尾。
```

本节使用库提供的工具对象，不要求会自订类别。先理解讯息如何通过这条路径：

**日志调用 → logger 的有效级别 → handler 的级别 → formatter → 输出位置。**

如果 logger 已经挡掉 DEBUG，即使文件 handler 设成 DEBUG，也拿不到那条纪录。

`logger.info("订单 %s 已创建", order_id)` 会让 logging 处理参数格式化。`%s` 是日志文字的占位符，与正则语法无关。不要把密码、登入凭证或完整个人数据直接放进日志。

## 8.14 Logger 命名与传播：点号创建阶层

下面三个名字形成父子关系：

- `shop`
- `shop.checkout`
- `shop.checkout.payment`

`shop.checkout` 可以把纪录向上交给 `shop` 的 handler，再往根 logger 传递，这个行为叫做 propagation。

在模块里通常使用 `logging.getLogger(__name__)`，让 logger 名称与模块名称一致。例如汇入 `shop.checkout` 模块时，`__name__` 就能反映模块来源；直接执行入口文件时通常是 `"__main__"`。模块与包的名称关系见 [第 6 章](06-modules-packages.md)。

### 同一行日志为甚么出现两次？

常见原因是：

1. 子 logger 自己有画面 handler。
2. 父 logger 或根 logger 也有画面 handler。
3. 子 logger 的 `propagate=True`，同一纪录经过两条路径送到同一画面。

两种清楚的配置方式：

- 模块 logger 不自行加 handler，集中交给父 logger 或根 logger 输出。
- 某个 logger 有自己专用的 handler 时，视设计把 `propagate` 设为 `False`，让纪录到此为止。

另外，重复执行设定函数、每次都 `addHandler` 却没有移除旧 handler，也会让日志重复。

### 一个容易误解的级别细节

logger 的级别若是 `NOTSET`，它会向父层寻找有效级别。但**纪录已由子 logger 创建后，向上传播时会直接交给祖先的 handler，祖先 logger 的级别不会再重新筛一次**。

例如子 logger 明确设为 DEBUG，祖先 logger 设为 WARNING，祖先的 handler 却设为 INFO，子 logger 创建的 INFO 纪录仍可能由这个 handler 输出。

初学时可先用清楚、集中且一致的配置，真的需要不同模块有不同详细程度时，再调整这些门槛。

## 8.15 dictConfig：把配置集中写成字典

当设定包含多种格式、多个 handler、不同模块 logger 时，一连串 `setLevel`、`addHandler` 容易分散。`logging.config.dictConfig` 可以把配置整理在一个字典中。

```python
import logging
import logging.config

config = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "simple": {
            "format": "%(levelname)s | %(name)s | %(message)s",
        },
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "INFO",
            "formatter": "simple",
            "stream": "ext://sys.stdout",
        },
    },
    "loggers": {
        "shop": {
            "level": "INFO",
            "handlers": ["console"],
            "propagate": False,
        },
    },
    "root": {
        "level": "WARNING",
        "handlers": ["console"],
    },
}

logging.config.dictConfig(config)
logger = logging.getLogger("shop.checkout")
logger.info("订单 %s 已创建", "O002")
logger.debug("详细运算过程")

# 输出：
# INFO | shop.checkout | 订单 O002 已创建
# 说明：
# shop.checkout 没有自行设定级别，因此从父 logger shop 取得 INFO 门槛。
# 纪录传给 shop 的 console handler；shop 的 propagate=False 阻止继续向上。
# dictConfig() 套用设定并返回 None。
```

读配置时按顺序找引用：

1. `formatters["simple"]` 定义文字格式。
2. `handlers["console"]` 引用 `"simple"`，并决定送到哪里。
3. `loggers["shop"]` 引用 `"console"`，并设定传播方式。

`version=1` 是这份配置格式的版本，不是 Python 版本。`disable_existing_loggers=False` 表示不因这次配置就自动停用原来已有的其他 logger。

`"class": "logging.StreamHandler"` 是配置使用的工具名称；当前只需按库规定填写，不必先理解类别继承。`"ext://sys.stdout"` 表示引用外部的 `sys.stdout` 输出串流。

配置可以集中，但不同模块仍应透过自己的 logger 名称发出事件，这样输出里的 `%(name)s` 才能帮定位来源。

## 8.16 记录异常：除了「失败」，还留下失败位置

`logger.error(...)` 会记录错误级别文字；在 `except` 里使用 `logger.exception(...)`，则会同时记录当前异常的 traceback。

```python
import logging
import sys

logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s: %(message)s",
    stream=sys.stdout,
    force=True,
)
logger = logging.getLogger("lesson.errors")

try:
    quantity = int("two")
except ValueError:
    logger.exception("数量无法转成整数")

print("程序继续执行")

# 输出：
# ERROR: 数量无法转成整数
# Traceback (most recent call last):
# ...（中间的文件路径、行号随执行方式而变）
# ValueError: invalid literal for int() with base 10: 'two'
# 程序继续执行
# 说明：
# 异常由 except 处理；logger.exception() 记录它，但不会自行重新抛出。
# 此调用应放在正在处理异常的 except 区块中。
```

日志和错误处理是两件工作：日志留下资讯；程序仍要决定重试、返回失败结果、提示用户，或重新抛出异常。

不要因为写了一行日志，就让后续程序假装失败的操作已经成功。异常处理与测试的完整桥接，见 [错误处理与验证桥接](bridge-errors-testing.md)。

## 8.17 日志轮替：文件变大后，自动保留有限份数

一直写入同一份文件，它会越来越大。`RotatingFileHandler` 可以依大小轮替，让当前文件保持可控，并保留指定份数的旧文件。

```python
import logging
from logging.handlers import RotatingFileHandler
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory(prefix="python_basics_") as temp_dir:
    root = Path(temp_dir)
    log_path = root / "shop.log"
    logger = logging.getLogger("lesson.rotation")
    logger.setLevel(logging.INFO)
    logger.propagate = False

    handler = RotatingFileHandler(
        log_path,
        maxBytes=80,
        backupCount=2,
        encoding="utf-8",
    )
    handler.setFormatter(logging.Formatter("%(message)s"))
    logger.addHandler(handler)

    try:
        for number in range(12):
            logger.info("order=%02d status=created", number)
    finally:
        logger.removeHandler(handler)
        handler.close()

    print(sorted(path.name for path in root.iterdir()))
    print("order=11" in log_path.read_text(encoding="utf-8"))

# 输出：
# ['shop.log', 'shop.log.1', 'shop.log.2']
# True
# 说明：
# maxBytes 刻意设得很小，让教学例子容易发生轮替；实际系统通常大得多。
# backupCount=2 表示最多保留两个轮替备份，另有当前的 shop.log。
# 最旧的轮替文件会被淘汰，所以日志轮替不是永久保存所有事件。
```

还有依时间轮替的 `TimedRotatingFileHandler`，适合每日或每小时切文件。先选择符合需求的一种即可，不必同时使用。

`maxBytes` 不是绝对不会超过的硬上限：如果单条消息本身就非常大，文件也可能超出设定。因此日志内容本身也应合理控制。

## 8.18 整合时，让每个工具只回答它擅长的问题

以「汇入购物数据」为例：

1. 路径工具定位数据文件。
2. JSON 解析器读入数据。
3. 普通条件判断检查外层是不是字典、数量是否为正整数。
4. 正则检查商品 ID 的文字形式。
5. 商品字典检查商品是否存在，库存条件检查是否能购买。
6. 成功时用 INFO 记录结果；发生文件或解析异常时记录必要资讯。
7. 若需要核对备份内容，再用 SHA-256 计算文件摘要。

这条流程里，正则不负责查库存，杂凑不负责解析 JSON，日志也不负责回滚交易。

把工具摆在正确位置，比在作业里尽量多用几个模块更能反映理解程度。

## 8.19 动手练习

### 练习 A｜文件摘要验证

在临时目录产生两份内容相同的文字文件：

- 使用分块读取函数计算两份 SHA-256，确认相同。
- 修改第二份文件的一个字符，再次比较，应该不同。
- 用不同分块大小，例如 1024 与 8192，计算同一份文件，确认摘要相同。

不要只测试空文件；至少有一份样本应大于分块大小，确实走过多轮读取。

### 练习 B｜商品 ID 与数量文字

写一个函数接收 `"P001:12"` 形式的文字。商品 ID 规则为大写 P 加三个 ASCII 数字，数量必须是正整数。

要求：

- 匹配整段字符串，不能接受尾端多余内容。
- 成功时返回商品 ID 与整数数量，或设计的清楚数据结构。
- 失败时返回清楚的失败结果，或使用已学过的异常处理。
- `P001:12` 应成功；`P001:0`、`p001:2`、`P01:2`、`P001:2abc` 应失败。

思考：`0` 可以通过「一个或多个数字」的正则，为甚么仍然不能购买 0 件商品？哪一层应该检查？

### 练习 C｜一份日志，两种目的地

创建一个小程序，依序模拟：

1. 载入商品数据，记 DEBUG。
2. 成功新增一项购物车商品，记 INFO。
3. 发现库存偏低，记 WARNING。
4. 尝试把不合法的数量文字转成整数，捕捉指定异常并记录。

要求：画面显示 INFO 以上，文件保留 DEBUG 以上。所有文件在自己创建的练习目录中。每个画面事件只显示一次，并且能从日志名称辨认来源。

### 练习 D｜解释重复日志

画出这个配置的流向：`shop.checkout` 有自己的画面 handler，`shop` 也有画面 handler，前者的 `propagate=True`。

先预测 INFO 事件会印几次，再修改配置，使每次只出现一次。用自己的话解释移除了哪一条重复路径。

## 8.20 自查与简答

| 问题 | 简答 |
|---|---|
| `hash(a) == hash(b)` 能推论 `a == b` 吗？ | 不能，可能碰撞。 |
| 为甚么不保存 `hash("P001")` 当永久校验值？ | 字符串 hash 跨行程可能改变，这不是它的用途。 |
| SHA-256 的 `digest` 和 `hexdigest` 有何差别？ | 前者是 32 个字节，后者是 64 字符的十六进位文字。 |
| 分两次 `update`，只要顺序一样，摘要会改变吗？ | 不会，等同一次输入串接后的字节。 |
| 加盐的单次 SHA-256 就适合保存密码吗？ | 仍不适合；需要 PBKDF2、scrypt 等专用密码衍生方案。 |
| 盐必须保密吗？ | 不需要，但应随机产生并保存，供日后验证。 |
| `re.match` 成功，代表整段字符串都合法吗？ | 不代表，它只要求从开头匹配。 |
| `r` 会让正则符号变成普通字符吗？ | 不会，它主要控制 Python 字符串的跳脱处理。 |
| `\d` 预设等于 `[0-9]` 吗？ | 不等于，它也能接受其他 Unicode 十进位数字。 |
| `findall` 加上分组，返回值可能怎样改变？ | 从完整匹配字符串，变成分组字符串或分组元组。 |
| `.*?` 是否只能取一个字符？ | 不是，它先尝试较短的可成功匹配。 |
| 商品 ID 通过正则，代表能买吗？ | 不代表，还需检查商品存在、数量、库存等。 |
| handler 设 DEBUG，logger 设 INFO，就能记 DEBUG 吗？ | 不能，纪录先被 logger 的有效级别挡掉了。 |
| `logger.exception` 除了文字还会留下甚么？ | 正在处理的异常 traceback。 |
| 同一日志印两次，先检查甚么？ | 重复添加 handler，以及子父 logger 同时输出且仍向上传播。 |
| 轮替后所有旧日志都会永久保存吗？ | 不会，超过保留份数的旧文件会被淘汰。 |

完成本章后，应能说清楚每个工具的输入、返回值与额外动作。接下来做基础篇总作业时，先用简单、可验证的流程完成需求，再按作业要求加入保存、验证与日志。
