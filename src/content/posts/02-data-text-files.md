---
title: Python 第 69-167 集：進制、文件與編碼相關
published: 2026-09-10
updated: 2026-09-14
description: 進制】文件
tags: [Python, 基礎篇]
category: Python 學習
draft: false
lang: zh_TW
---

# 第二章　把資料整理好，再保存到檔案（第 69–167 集）

這一章解決兩個問題：「多筆資料要放在哪裡？程式關閉後，如何把資料留下來？」學完後，你應能完成文字清洗、商品資料的增刪改查，以及 UTF-8 檔案讀寫。

前置知識：變數、基本資料型別、`print()`、`input()`、`if`、`for`、`while`。本章使用 Python 3；第 168 集的函式引入放在下一章。

每個 Python 程式碼框都是一個可單獨執行的例子。先預測結果，再執行，最後只改一處重跑。檔案例子會建立自己的暫存資料夾，結束後自動移除；例子的 `w` 模式只操作範例檔案。

## 1. 學習路線

| 對應範圍 | 要回答的問題 | 學習成果 |
|---|---|---|
| 69–74 | 數字能用哪些進制表示？ | 分清數值和文字表示 |
| 75–81 | 如何取出需要的文字？ | 索引、切片、清洗、拆分、組合 |
| 82–100 | 多筆資料如何組織？ | 選對列表、元組、字典、集合 |
| 101–125 | 文字如何變成位元組？ | 分清 `str`、`bytes`、編碼、解碼 |
| 126–167 | 如何保存、複製和修改檔案？ | 理解模式、位置、路徑和資料格式 |

上述是原筆記的概念分組，內容按理解順序重排，不是逐集轉錄。多版本共存的重複內容合併說明。

## 2. 進制：數值相同，寫法不同

二進制 `1010` 表示 `1×8 + 0×4 + 1×2 + 0×1`，所以是十進制的 10。數值沒有變，變的是表示方式。

```python
number = 10
print(bin(number))
print(oct(number))
print(hex(number))
print(int("1010", 2))
print(0b1010 == 10)
# 輸出：依次為 0b1010、0o12、0xa、10、True。
# 說明：前三個結果是字串；int 按指定進制解讀文字；0b1010 本身是整數。
```

追蹤第一個轉換：`number` 指向整數 10，`bin()` 取得這個值，返回字串 `"0b1010"`，`print()` 顯示字串。不是將整數改成另一種型別的「二進制整數」。

| 進制 | 可用數字 | 字面量前綴 | 轉成文字 |
|---|---|---|---|
| 二進制 | 0、1 | `0b` | `bin(n)` |
| 八進制 | 0–7 | `0o` | `oct(n)` |
| 十進制 | 0–9 | 無 | `str(n)` |
| 十六進制 | 0–9、a–f | `0x` | `hex(n)` |

`int(text, base)` 中，`text` 是交給函式處理的文字，`base` 指定它使用的進制，轉換結果稱為返回值。這只是使用現成函式，自訂函式下一章才開始。

常見錯誤：`int("102", 2)` 會失敗，因為二進制沒有數字 2；`int("10")` 預設按十進制解讀。

**練習：**心算 `int("1111", 2)`，再把 31 轉為十六進制字串。**自查簡答：**15、`"0x1f"`。`bin(10)` 是 `str`；`0b1010` 是 `int`。

## 3. 字串：先定位，再處理

### 3.1 索引取一個字元，切片取一段

把 `"Python"` 看成六個依序排列的字元。正索引從 0 起算；`-1` 是最後一個。切片格式為 `text[start:stop:step]`，包含起點，不包含終點。

```python
text = "Python"
print(text[0], text[-1])
print(text[1:4])
print(text[:2], text[2:])
print(text[::2])
print(text[::-1])
# 輸出：P n；yth；Py thon；Pto；nohtyP（五行）。
# 說明：切片的 stop 不包含在結果；負步長表示朝反方向取值。
```

追蹤 `text[1:4]`：取索引 1 的 `y`、2 的 `t`、3 的 `h`，到 4 就停，所以是三個字元。省略起點或終點時，Python 按步長方向選擇邊界。

索引超界會出錯；切片超過尾端通常只取到現有資料。空字串可以切片，不能取第 0 個字元。步長不能是 0。

```python
text = "cat"
print(text[:100])
try:
    print(text[100])
except IndexError:
    print("索引超出範圍")
# 輸出：cat；索引超出範圍。
# 說明：捕捉錯誤讓例子可以完整執行；例外處理在後續章節學習。
```

### 3.2 字串不可原地修改

```python
name = "cat"
new_name = "b" + name[1:]
print(name, new_name)
name = name.upper()
print(name)
# 輸出：cat bat；CAT。
# 說明：字串內容沒有原地改變；最後的賦值讓 name 改為指向新字串。
```

`name[0] = "b"` 會產生 `TypeError`。只寫 `name.upper()` 而不使用結果，`name` 仍指向原字串。

### 3.3 清洗 → 拆分 → 組合

```python
raw = "  apple,banana,orange  "
clean = raw.strip()
items = clean.split(",")
result = " / ".join(items)
print(repr(clean))
print(items)
print(result)
# 輸出：'apple,banana,orange'；['apple', 'banana', 'orange']；apple / banana / orange。
# 說明：repr 顯示引號便於觀察；strip 返回 str，split 返回 list，join 返回 str。
```

`raw` 是一整段文字；`split()` 把它拆成列表；`join()` 把列表裡的字串接起來。分隔符寫在 `join()` 前面，所有被連接的元素都必須是字串。

`strip("ab")` 從兩端移除屬於 a 或 b 的字元，不是刪掉固定前綴 `"ab"`。固定前綴可以用 `removeprefix()`；固定後綴用 `removesuffix()`（Python 3.9+）。

```python
print("abbahelloab".strip("ab"))
print("ababhello".removeprefix("ab"))
print("a  b\tc".split())
print("a,,b".split(","))
# 輸出：hello；abhello；['a', 'b', 'c']；['a', '', 'b']。
# 說明：無參數 split 合併連續空白；指定逗號拆分時保留空欄位。
```

### 3.4 常用方法與格式化

```python
text = "Python Programming"
print(text.lower())
print(text.upper())
print(text.replace("Python", "Go"))
print(text.startswith("Py"), text.endswith("ing"))
print(text.find("Program"), text.find("Java"))
print(text.count("m"))
# 輸出：python programming；PYTHON PROGRAMMING；Go Programming；True True；7 -1；2。
# 說明：find 找不到返回 -1；這些方法都不會原地改動 text。
```

只想知道文字是否存在，通常用 `"Python" in text`。不要直接把 `find()` 的結果當布林值：索引 0 是假，-1 卻是真。

```python
product = "筆記本"
price = 12.5
quantity = 2
print(f"{product} × {quantity} = {price * quantity:.2f} 元")
# 輸出：筆記本 × 2 = 25.00 元。
# 說明：:.2f 控制顯示兩位小數，並不將浮點數變成精確的金額型別。
```

**練習：**把 `"  Alice, 85  "` 變成姓名 `"Alice"` 和整數 85；取出 `"order-2026-001"` 的最後三個字元。

**自查簡答：**清理整段後用逗號拆分，再清理各欄位並 `int()`；最後三個字元用 `text[-3:]`。方法的返回值要使用或保存。

## 4. 四種容器：從需求決定結構

| 結構 | 如何取資料 | 可改容器本身嗎 | 重複與順序 |
|---|---|---|---|
| `list` 列表 | 位置索引 | 可以 | 有順序，允許重複 |
| `tuple` 元組 | 位置索引 | 不可替換項目 | 有順序，允許重複 |
| `dict` 字典 | 鍵，如商品 ID | 可以 | 鍵唯一；Python 3.7+ 保留插入順序 |
| `set` 集合 | 成員是否存在 | 可以 | 元素唯一，不承諾迭代順序 |

### 4.1 列表 CRUD：新增、查詢、修改、刪除

```python
products = ["鉛筆", "橡皮擦"]
products.append("筆記本")
products.insert(1, "尺")
print(products)
print(products[0])
products[0] = "自動鉛筆"
products.remove("尺")
removed = products.pop()
print(products, removed)
# 輸出：['鉛筆', '尺', '橡皮擦', '筆記本']；鉛筆；['自動鉛筆', '橡皮擦'] 筆記本。
# 說明：append 加一項；insert 按位置插入；remove 按值刪第一個符合項；pop 刪除並返回項目。
```

每操作一步就寫出列表內容，比背方法名稱有效。`remove(value)` 是按值刪除，找不到會報 `ValueError`；`pop(index)` 是按索引刪除，索引不存在會報 `IndexError`。`del products[0]` 也能刪除，但不把刪除項返回。

```python
numbers = [3, 1, 3]
numbers.append([8, 9])
print(numbers)
numbers = [3, 1, 3]
numbers.extend([8, 9])
print(numbers)
print(len(numbers), 8 in numbers, numbers.count(3))
# 輸出：[3, 1, 3, [8, 9]]；[3, 1, 3, 8, 9]；5 True 2。
# 說明：append 將整個物件當一項加入；extend 將可迭代資料的各項加入。
```

### 4.2 排序與反轉要看返回值

```python
numbers = [3, 1, 5]
new_numbers = sorted(numbers)
print(numbers, new_numbers)
result = numbers.sort()
print(numbers, result)
numbers.reverse()
print(numbers)
# 輸出：[3, 1, 5] [1, 3, 5]；[1, 3, 5] None；[5, 3, 1]。
# 說明：sorted 產生新列表；sort 和 reverse 修改原列表，返回 None。
```

不要寫 `numbers = numbers.sort()`，那會讓 `numbers` 指向 `None`。降冪排序可用 `numbers.sort(reverse=True)`；反轉只把目前順序倒過來，不等於降冪排序。

### 4.3 元組：固定的是項目的對應

```python
point = (10, 20)
x, y = point
one = (1,)
print(x, y)
print(type(one).__name__, type((1)).__name__)
record = ("Alice", [80, 90])
record[1].append(100)
print(record)
# 輸出：10 20；tuple int；('Alice', [80, 90, 100])。
# 說明：元組的項目不能被替換，但項目指向的列表仍可修改；單元素元組需要逗號。
```

「元組不可變」不代表裡面所有物件都不可變。`record[1] = []` 不合法；`record[1].append(100)` 改的是內層列表，所以可以。元組適合座標、固定欄位的紀錄、函式的多個結果。

### 4.4 字典：用有意義的鍵定位

```python
product = {"id": "P001", "name": "筆記本", "stock": 8}
product["stock"] = 6
product["price_cents"] = 1250
print(product["name"])
print(product.get("color", "未提供"))
for key, value in product.items():
    print(key, value)
# 輸出：筆記本；未提供；接著為 id P001、name 筆記本、stock 6、price_cents 1250。
# 說明：新鍵是新增，已有鍵是更新；items 每次提供一組鍵和值。
```

`key in product` 檢查鍵。直接迭代字典也是取得鍵；`keys()`、`values()`、`items()` 分別提供鍵、值、鍵值對。`get()` 找不到時只返回預設值，不會新增鍵。`pop(key)` 刪除並返回值；不確定存在時可用 `pop(key, None)`。

鍵必須可雜湊；常用字串或整數。元組只有在所有元素都可雜湊時才可當鍵，包含列表的元組不行。「不可變」和「可雜湊」不能當成完全相同的定義。

多筆商品可用字典套字典，方便按 ID 查找：

```python
products = {
    "P001": {"name": "筆記本", "stock": 8},
    "P002": {"name": "鉛筆", "stock": 20},
}
product_id = "P002"
if product_id in products:
    products[product_id]["stock"] -= 2
print(products["P002"])
# 輸出：{'name': '鉛筆', 'stock': 18}。
# 說明：先用商品 ID 取得一筆商品，再用 stock 取得欄位。
```

若重點是按順序處理多筆記錄，也可用「列表裡放字典」。資料格式要服務查詢需求。

### 4.5 集合：關心有沒有，不關心第幾個

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
# 輸出：['book', 'pen', 'ruler']；['book']；['pen']；1。
# 說明：| 是聯集，& 是交集，- 是差集；sorted 只為穩定展示，集合本身不承諾順序。
```

`{}` 是空字典，空集合用 `set()`。集合元素也必須可雜湊。`remove()` 刪不存在的元素會報錯，`discard()` 不會。`list(set(data))` 能去重，但不能保證保留原順序。

### 4.6 堆疊與佇列：取出的規則不同

```python
stack = []
stack.append("A")
stack.append("B")
print(stack.pop())
queue = ["A", "B"]
print(queue.pop(0))
# 輸出：B；A。
# 說明：堆疊後進先出，佇列先進先出；列表 pop(0) 需移動後面的項目，只適合小型示範。
```

大量佇列操作通常用 `collections.deque`。現在先理解規則，不必立即引入新結構。

**練習：**按商品 ID 查詢選什麼？找兩人共同買過的商品用什麼？

**自查簡答：**按 ID 查詢適合字典；共同商品適合集合交集。列表按索引取值；字典按鍵取值，數字鍵也不是列表索引。

## 5. 引用與複製：為什麼改 B，A 也變了？

### 5.1 賦值建立名稱關係

```python
a = [1, 2]
b = a
b.append(3)
print(a, b)
print(a is b)
b = [9]
print(a, b)
# 輸出：[1, 2, 3] [1, 2, 3]；True；[1, 2, 3] [9]。
# 說明：b = a 不複製列表；append 改共享物件，b = [9] 只重新繫結 b。
```

畫兩個箭頭 `a`、`b` 指向同一列表，就容易看懂。`==` 比較值是否相等；`is` 比較是否同一物件。一般數字、字串的相等判斷使用 `==`。

### 5.2 淺拷貝與深拷貝

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
# 輸出：[[1, 2, 9], [3, 4]]；[[1, 2, 9], [3, 4], [5, 6]]；[[1, 2], [3, 4, 8]]。
# 說明：淺拷貝只建新外層；深拷貝讓本例的巢狀列表也分開。
```

`shallow.append()` 改新外層，`a` 沒多一項；`shallow[0].append()` 改共享內層，`a[0]` 也變了。這裡討論一般巢狀資料，`deepcopy()` 並不是所有檔案資源或外部連線都能複製的工具。

### 5.3 `+=` 不總等於 `+` 再賦值

```python
a = [1]
b = a
a += [2]
print(a, b, a is b)
a = [1]
b = a
a = a + [2]
print(a, b, a is b)
# 輸出：[1, 2] [1, 2] True；[1, 2] [1] False。
# 說明：列表 += 原地擴充；列表 + 建立新列表，再由賦值改變 a 的指向。
```

整數 `count += 1` 可先理解為算出新值再繫結，但不能把這個模型套到所有型別。

**練習：**用一句話說明 `a.copy()` 為什麼不能隔離內層列表的修改。

**自查簡答：**只複製外層容器，內層項目仍指向原物件。除錯先分清「重新繫結名稱」與「修改共享物件」。

## 6. 編碼：文字與位元組之間的轉換

`str` 表示文字，`bytes` 表示位元組序列。保存或傳輸文字時需要編碼，讀回位元組時需要解碼。

```text
str ── encode("utf-8") ──→ bytes
str ←── decode("utf-8") ── bytes
```

```python
text = "你好"
data = text.encode("utf-8")
print(data)
print(len(text), len(data))
print(data.decode("utf-8"))
# 輸出：b'\xe4\xbd\xa0\xe5\xa5\xbd'；2 6；你好。
# 說明：本例每個中文字占三個 UTF-8 位元組；len(str) 和 len(bytes) 的單位不同。
```

`len(str)` 計算 Unicode 碼位，不保證等於畫面看見的完整字形數；例如某些 emoji 含多個碼位。初學先用一般中英文字理解。

ASCII 主要涵蓋英文和數字；Unicode 定義字元及碼位；UTF-8、UTF-16 是把 Unicode 文字編成位元組的方式；GBK 是常見的傳統中文編碼。Unicode 和 UTF-8 不在同一層。

```python
data = "你好".encode("utf-8")
try:
    print(data.decode("ascii"))
except UnicodeDecodeError:
    print("這些位元組不能按 ASCII 解碼")
# 輸出：這些位元組不能按 ASCII 解碼。
# 說明：刻意使用不相容的 ASCII；錯用其他編碼也可能不報錯卻得到亂碼。
```

排查順序：確認來源編碼 → 檢查讀寫的 `encoding` → 檢查編輯器與終端機。不要為了不報錯而隨便使用 `errors="ignore"`，它會丟掉資料。

Python 2 已停止支援；影片的 `raw_input()`、Python 2 字串行為作歷史理解。Python 3 使用 `input()`。多版本共存時，確認編輯器選的直譯器與安裝套件用的是同一個環境。

### 6.1 原始碼檔案頭和外部檔案編碼不是同一件事

Python 3 原始碼預設按 UTF-8 解讀，通常不必寫編碼宣告。舊教材中的 `# coding: utf-8` 是在指定**這個 Python 原始碼檔案**的編碼，不會替 `open()` 讀寫的所有外部檔案設定編碼。

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
print("原始碼中的中文")
# 輸出：原始碼中的中文。
# 說明：編碼宣告示範放在前兩行；shebang 在支援它的啟動環境用來選擇直譯器，對 Python 語法而言是註解。
```

shebang 常見於 Unix 類系統可直接執行的腳本；Windows 的 Python launcher 也可能解讀它，但不應假定任何編輯器或啟動方式都據此選版本。執行 `python 檔名.py` 時，先由你選的 `python` 決定執行環境。Python 3 仍可能因外部資料編碼或終端機顯示設定不符而亂碼，不能把「Python 3」當作永不亂碼的保證。

**練習：**兩個中文字為什麼可能占六個位元組？**自查簡答：**文字長度和 UTF-8 位元組長度的單位不同；編碼是 `str → bytes`，解碼反向。

## 7. 檔案最小流程：開啟 → 使用 → 關閉

### 7.1 不依賴外部檔案的第一個例子

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "note.txt"
    with open(path, "w", encoding="utf-8") as file:
        count = file.write("你好\nPython")
    with open(path, "r", encoding="utf-8") as file:
        content = file.read()
    print(count)
    print(content)
    print(file.closed)
# 輸出：9；你好；Python；True（四行）。
# 說明：write 返回字元數；read 返回文字；內層 with 關閉檔案，外層結束後移除暫存資料。
```

`TemporaryDirectory()` 提供暫存資料夾；`Path(folder) / "note.txt"` 組成路徑。它們是讓範例可安全重跑的準備工具，目前先理解用途即可。

`with open(...) as file` 將檔案物件交給 `file`；離開區塊會關閉檔案，包括因一般例外離開的情況。`with` 是上下文管理語法，不是所有用途都等同於手寫 `try/finally`；對內建檔案物件，重點是可靠關閉資源。

### 7.2 路徑相對於哪裡？

相對路徑依**目前工作目錄**解讀，不必然依 `.py` 所在位置。絕對路徑指定完整位置；`.` 是目前目錄，`..` 是上一層。找不到檔案時先確認工作目錄、檔名和副檔名。

```python
from pathlib import Path

print(Path.cwd())
print(Path("data") / "users.txt")
# 輸出：第一行是本次執行的工作目錄；第二行是使用目前平台分隔符的 data/users.txt 路徑。
# 說明：只顯示路徑，不建立檔案；Path 處理平台分隔符。
```

### 7.3 模式：`+` 不代表追加

| 模式 | 讀寫 | 不存在時 | 已存在時 |
|---|---|---|---|
| `r` | 讀 | 報錯 | 從開頭讀 |
| `w` | 寫 | 建立 | **開啟時立即清空** |
| `a` | 寫 | 建立 | 寫入追加到尾端 |
| `x` | 寫 | 建立 | 報 `FileExistsError` |
| `r+` | 讀寫 | 報錯 | 不清空，從開頭開始 |
| `w+` | 讀寫 | 建立 | **開啟時立即清空** |
| `a+` | 讀寫 | 建立 | 寫入追加；讀取前通常需 `seek(0)` |

`t` 是文字模式（預設），`b` 是二進制模式，例如 `rb`、`wb`。`+` 是同時讀寫，追加要看 `a`。文字模式建議明寫 `encoding="utf-8"`，二進制模式不能指定 `encoding`。

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
# 輸出：'A\nB\n'；'C\n'。
# 說明：a 保留原內容並追加；第二次 w 在開啟時就清空 A 和 B。
```

### 7.4 讀取後，位置往前移

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
# 輸出：'A\n'；['B\n', 'C']；''。
# 說明：readline 讀一行；readlines 讀剩餘各行；到檔尾後再 read 返回空字串。
```

| 方法 | 文字模式返回值 | 使用場合 |
|---|---|---|
| `read()` | 剩餘全部 `str` | 小檔案全文處理 |
| `read(n)` | 最多 n 個字元 | 限量讀取 |
| `readline()` | 下一行 `str` | 一次一行，通常含換行 |
| `readlines()` | 剩餘各行的列表 | 確實需要完整行列表 |
| `for line in file` | 每次取得一行 | 大檔案逐行處理 |

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "names.txt"
    path.write_text(" Alice \nBob\n", encoding="utf-8")
    with open(path, "r", encoding="utf-8") as file:
        for line in file:
            print(repr(line.rstrip("\r\n")))
# 輸出：' Alice '；'Bob'。
# 說明：只去掉換行，保留原有空格；strip 會額外移除首尾其他空白。
```

### 7.5 寫入不會自動補換行

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
# 輸出：AB；C；1 None（三行）。
# 說明：兩個方法都不自動補換行；write 返回字元數，writelines 返回 None。
```

常見轉義包含換行、定位字元、反斜線和引號。文字模式通常會處理平台換行差異，所以「寫入字元數」不必然等於「硬碟位元組數」。

回車 `\r` 原意是回到行首，換行 `\n` 原意是移到下一行；Windows 文字檔常見 `\r\n`，Unix 類系統常見 `\n`。在終端機顯示時，單獨回車可能讓後續文字從行首覆寫，看起來與真正換行不同。

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
# 輸出：'A\nB\nC\n'；'A\r\nB\rC\n'。
# 說明：預設 newline=None 讀取時統一換行；newline="" 保留讀到的換行形式。
```

文字寫入時，`newline=None` 可能把換行轉成平台形式；二進制模式則原樣處理位元組。一般文字先用預設，只有需求明確時才手動控制 `newline`。

### 7.6 `tell()`、`seek()`：位置不是文字索引

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "position.txt"
    path.write_text("你好ABC", encoding="utf-8")
    with open(path, "r", encoding="utf-8") as file:
        print(file.read(1))
        position = file.tell()
        print(file.read(1))
        file.seek(position)
        print(file.read(1))
        file.seek(0)
        print(file.read(2))
# 輸出：你；好；好；你好。
# 說明：tell 的值可供 seek 恢復位置；文字模式不要把它當成第幾個字元。
```

文字模式的 `tell()` 是可用於恢復位置的值，不要對它做任意字元偏移運算。先熟悉 `seek(0)` 與儲存後再恢復 `tell()` 的結果。二進制模式則按位元組定位。

### 7.7 緩衝與 `flush()`：交出去，不代表已經可靠落盤

為降低 I/O 次數，Python 可能先把寫入資料放在緩衝區；作業系統也可能保留自己的快取。機械硬碟有尋道等成本，固態硬碟沒有同樣的機械尋道，但也仍有 I/O 延遲。批量寫入、逐行處理與分段搬運，是在延遲、記憶體用量與程式簡單度之間取平衡。

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "buffered.bin"
    with open(path, "wb") as writer:
        writer.write(b"ready\n")
        writer.flush()
        print(path.read_bytes())
# 輸出：b'ready\n'。
# 說明：flush 將 Python 寫入緩衝交給下層；正常關閉也會刷新，但它們都不等於保證斷電後資料已持久保存。
```

需要更強的持久化要求時，會再涉及 `os.fsync()`、檔案系統與硬體保證；本章先分清 `write()`、`flush()`、關閉與持久保存不是同一件事。也不要每寫一個字元就 `flush()`，那可能降低緩衝的效益。

### 7.8 讀取後續新增資料：有限次的監看模型

普通檔案讀到檔尾後，若之後有新資料追加，可以從目前位置再讀。以下用兩次追加模擬日誌新增，不會啟動無限迴圈。

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
# 輸出：True；new 1；new 2。
# 說明：先定位末尾跳過舊內容；每次追加並關閉寫入者後，讀取者從原位置讀到新行。
```

追蹤：初次檔尾讀到空位元組 → 追加第一行 → 從原檔尾讀第一行 → 再追加第二行 → 接著讀第二行。真實輪詢若暫時沒資料，要有短暫等待與停止條件，避免空轉；還需考慮半行資料、檔案截斷和替換。這個教學例只處理同一檔案追加完整行的情況。

**練習：**讀完後第二次 `read()` 為何為空？`w+` 能保護原內容嗎？

**自查簡答：**位置已到檔尾，重新讀前用 `seek(0)`；`w+` 仍會清空原內容。

## 8. 二進制模式、複製與修改

### 8.1 同一檔案，兩種解讀方式

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as folder:
    path = Path(folder) / "note.txt"
    path.write_bytes("你好".encode("utf-8"))
    with open(path, "r", encoding="utf-8") as file:
        text = file.read()
    with open(path, "rb") as file:
        data = file.read()
    print(type(text).__name__, len(text))
    print(type(data).__name__, len(data))
# 輸出：str 2；bytes 6。
# 說明：文字模式自動解碼，二進制模式保留原始位元組。
```

圖片、壓縮檔、音訊通常用二進制模式。文字 `write()` 需要 `str`；二進制 `write()` 需要如 `bytes` 的位元組類資料。

### 8.2 分段複製：一次搬一小箱

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
# 輸出：b'ABCDEFGHIJ'；True。
# 說明：依次讀 ABCD、EFGH、IJ、空位元組；空值代表讀完。
```

本例每次 4 位元組便於觀察，實際可用 `1024 * 1024`（1 MiB）等區塊。目的主要是限制記憶體使用，不是保證更快。I/O 常比記憶體操作慢，避免逐字元搬運和反覆開關檔案。

### 8.3 修改：先寫新檔，再替換

文字檔通常不能直接在中間插入任意長度資料。小檔案可整個讀進記憶體修改；大檔案可逐行讀原檔、寫新檔。

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
# 輸出：P001,6；P002,20（兩行）。
# 說明：新檔完整寫好並關閉後才替換原檔；只操作自己的暫存資料。
```

重要資料還需備份與失敗恢復；這只是一個基本保存策略，並非完整交易系統。

## 9. 綜合：讀取帳號、登入與註冊

先約定格式：每行 `帳號:練習密碼`，按第一個冒號分隔。帳號不能含冒號或換行，密碼不能含換行。真實帳號不能明文保存密碼，這裡只練解析和寫檔。

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
    print("無效行數：", invalid_count)
    print("登入成功：", users.get("alice") == "demo123")
    new_name, new_password = "carol", "demo456"
    if new_name in users:
        print("帳號已存在")
    else:
        users[new_name] = new_password
        with open(path, "a", encoding="utf-8") as file:
            file.write(f"{new_name}:{new_password}\n")
        print("註冊完成：", new_name)
# 輸出：{'alice': 'demo123', 'bob': 'abc:123'}；無效行數： 1；登入成功： True；註冊完成： carol。
# 說明：略過空行，拒絕格式錯誤與重複帳號；split(":", 1) 保留密碼中的冒號。
```

追蹤 `"bob:abc:123"`：最多拆一次，得到 `"bob"` 和 `"abc:123"`，再存進字典。此例使用確定符合規則的新增資料；改成 `input()` 時，必須先驗證帳號和密碼格式。尚未處理多人同時寫檔。

再用學完的字串與字典完成單字計數：

```python
sentence = "Python is easy and Python is useful"
words = sentence.lower().split()
counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1
print(counts)
# 輸出：{'python': 2, 'is': 2, 'easy': 1, 'and': 1, 'useful': 1}。
# 說明：第一次 get 返回 0，存入 1；再次遇到同字時沿用舊次數加一。
```

這個版本不處理標點，也不是中文分詞器。擴充前先定義「單字」的規則，避免把不清楚的需求藏進程式。

## 10. 常見錯誤與排查

| 現象 | 先查哪裡 | 修正方向 |
|---|---|---|
| `IndexError` | 索引與長度 | 檢查長度或使用合理切片 |
| `KeyError` | 鍵是否存在 | 存在性判斷或 `get()` |
| `ValueError` | 數字格式、拆分項數 | 驗證後再轉換或解包 |
| `TypeError` | 操作與型別 | 留意 `str`/`bytes`、不可變字串、可雜湊限制 |
| `FileNotFoundError` | 路徑與初始化 | 確認工作目錄和檔名 |
| `UnicodeDecodeError` | 來源編碼 | 使用實際編碼 |
| 列表變成 `None` | 是否接了 `sort()` 結果 | 分清原地修改與返回新物件 |
| 檔案內容消失 | 是否開了 `w` | 分清覆寫和追加 |
| 改 B 連 A 也改 | 可變物件是否共享 | 畫引用關係，選合適複製方式 |

## 11. 章末練習與自查

1. **文字清洗：**將 `"  apple, banana,, apple  "` 清洗成非空品項列表，統計次數。驗收：apple 2 次、banana 1 次。
2. **商品容器：**建立三筆商品，含 ID、名稱、庫存。新增、修改、刪除各一筆，再查詢不存在的 ID。驗收：不存在時提示，不中斷。
3. **保存資料：**把 ID 和庫存寫入練習檔，重新讀回。驗收：庫存讀回後是整數。
4. **複製工具：**分段複製自己建立的檔案。驗收：來源與目標位元組相同，空檔案也能處理。
5. **學生管理草稿：**先列格式、空資料和重複姓名規則，再實作新增、查詢、平均分。下一章再拆成函式。

自查簡答：

- 為何拆分後還要清理欄位？整段 `strip()` 只處理最外側，不處理每個逗號旁的空白。
- 重複字典鍵會如何？後次賦值覆蓋前值；若不允許重複，要先檢查。
- `read(4)` 一定讀四位元組嗎？文字模式是最多四個字元，二進制模式才是最多四位元組。
- 檔案關閉後 `content` 還能用嗎？可以；關閉檔案資源不會移除已讀入記憶體的資料。
- 怎樣算學會？能解釋容器選擇、預測引用修改結果，並完成寫入後重新讀回。

下一章將重複流程命名為函式，讓每一段處理都有清楚的輸入和輸出。
