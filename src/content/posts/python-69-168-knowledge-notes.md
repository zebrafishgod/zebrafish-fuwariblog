---
title: Python 第 69–168 集知識整理
published: 2026-09-06
updated: 2026-09-13
description: 系統整理 Python 的數字與進制、字串、資料結構、字符編碼及檔案讀寫，並提供練習與自查重點。
tags: [Python, 基礎語法, 資料結構, 檔案處理]
category: Python 學習
draft: false
lang: zh_TW
---

> 適用範圍：合集「01-Python 基礎篇」第 69 集至第 168 集。  
> 這一段的主線是：**資料結構 → 文字與編碼 → 檔案 I/O → 函式的入口**。

> **代碼示例閱讀規則**：每個主要示例都在程式碼內標出預期輸出、返回值或資料類型；涉及輸入、檔案和時間的示例，則用注釋說明執行效果，而不是假裝輸出固定不變。示例中的 `# 輸出：` 和 `# 說明：` 注釋可以直接保留在練習程式中。

## 0. 本階段學習目標

完成本階段後，應能夠：

- 熟練處理數字、字串、列表、元組、字典和集合。
- 根據查找、排序、去重和保存需求選擇資料結構。
- 解釋 `str`、`bytes`、編碼和解碼之間的關係。
- 使用 `open()` 和 `with` 安全地讀寫文字及二進制檔案。
- 完成檔案版登入、註冊、拷貝和資料修改程式。
- 把問題拆成「輸入 → 處理 → 輸出 → 保存」的流程，為下一階段函式學習做準備。

---

## 1. 知識地圖與建議順序

| 模組 | 對應集數 | 核心主題 | 學習成果 |
|---|---:|---|---|
| A | 69–74 | 數字參數與進制 | 能處理數字運算和進制轉換 |
| B | 75–81 | 字串索引、切片與常用操作 | 能清洗、切分和格式化文字 |
| C | 82–100 | 列表、元組、字典、集合與資料結構總結 | 能依需求組織和處理多筆資料 |
| D | 101–125 | 多版本提示、字符編碼與 I/O | 能理解並排查常見亂碼問題 |
| E | 126–167 | 檔案讀寫、路徑、模式、指針與實戰 | 能用檔案保存資料並完成小型工具 |
| F | 168 | 函式引入 | 理解下一階段為何需要函式重構 |

> 合集中第 101、102、121、124 集等「多版本共存」內容有重複或補充性質。Python 3 學習時，掌握一次即可，重點放在環境選擇和版本管理概念。

---

## 2. 模組 A：數字與進制（69–74）

### 2.1 進制基本概念

常見進制：

- 二進制（base 2）：只使用 `0`、`1`，電腦底層常用。
- 八進制（base 8）：使用 `0–7`。
- 十進制（base 10）：日常使用。
- 十六進制（base 16）：使用 `0–9` 和 `a–f`，常用於記憶體、顏色和位元資料表示。

同一個數值可以有不同表示法，但數值本身相同。例如十進制 `10` 等於二進制 `1010`、十六進制 `a`。

```python
number = 10

print(bin(number))       # 0b1010
print(oct(number))       # 0o12
print(hex(number))       # 0xa
print(int("1010", 2))   # 10
print(int("a", 16))     # 10
# 輸出：依次為 0b1010、0o12、0xa、10、10
# 說明：前 3 個函式把十進制轉成其他進制的字串；int(text, base) 把指定進制的文字轉回十進制整數。
```

常用轉換：

| 目的 | 語法 |
|---|---|
| 十進制 → 二進制 | `bin(n)` |
| 十進制 → 八進制 | `oct(n)` |
| 十進制 → 十六進制 | `hex(n)` |
| 其他進制字串 → 十進制 | `int(text, base)` |

### 2.2 參數與返回值的初步概念

這裡的「參數與返回值」可先理解為：把資料交給一段可重複使用的運算，再取得結果。正式的自訂函式會在第 168 集之後學習。

```python
result = int("1011", 2)
# 輸出：result == 11
# 說明："1011" 是以 2 為基數的文字，2 是傳給 int() 的進制參數。
```

`"1011"` 和 `2` 是傳入的參數，`int()` 產生轉換後的結果。

### 2.3 自查

1. `int("1111", 2)` 的結果是多少？
2. `bin(10)`、`oct(10)` 和 `hex(10)` 分別表示什麼？
3. `int(text, base)` 中的 `base` 有什麼作用？

---

## 3. 模組 B：字串深入操作（75–81）

字串是不可變的字元序列。可以讀取、切片和產生新字串，但不能直接修改原字串中的單一字元。

### 3.1 索引

```python
text = "Python"

print(text[0])    # P
print(text[-1])   # n
# 輸出：先列印 P，再列印 n
# 說明：正索引從 0 開始，負索引 -1 表示最後一個字元。
```

- 正索引從 `0` 開始。
- 負索引從 `-1`（最後一個字元）開始。
- 超出範圍會產生 `IndexError`。

### 3.2 切片

語法：`sequence[start:stop:step]`，`stop` 不包含在結果中。

```python
text = "Python"

print(text[1:4])   # yth
print(text[:2])    # Py
print(text[2:])    # thon
print(text[::2])   # Pto
print(text[::-1])  # nohtyP
# 輸出：yth、Py、thon、Pto、nohtyP（各占一行）
# 說明：切片的 stop 邊界不包含在結果中；負步長可用來反轉序列。
```

### 3.3 清除與拆分

```python
raw = "  apple,banana,orange  "

clean = raw.strip()          # 去除首尾空白
items = clean.split(",")     # 拆成列表
result = " / ".join(items)   # 用分隔符重新拼接
# 輸出：result == "apple / banana / orange"
# 說明：strip() 產生清理後的新字串，split() 產生列表，join() 再把列表元素組合成字串。
```

注意：

- `strip()` 預設處理空白，也可指定字元集合；它不會修改原字串。
- `split()` 的返回值是列表。
- `join()` 的呼叫者是分隔字串，參數通常是可迭代的字串集合。

### 3.4 常用字串方法

```python
text = "Python Programming"

text.lower()                 # 全部小寫
text.upper()                 # 全部大寫
text.replace("Python", "Go")
text.startswith("Py")
text.endswith("ing")
text.find("Program")         # 找不到時返回 -1
text.count("m")              # 計數
# 輸出：這些表達式分別返回 "python programming"、"PYTHON PROGRAMMING"、"Go Programming"、True、True、7、2。
# 說明：方法不會原地修改 text；若要保存結果，必須寫成 new_text = text.lower() 等形式。
```

這些方法都會返回新字串或新結果，不會原地改變字串。

### 3.5 字串處理模式

面對一段文字，常見流程是：

```text
讀取原文 → 清理空白/大小寫 → 拆分 → 遍歷處理 → 組合或統計
```

例如統計句子中的單字：

```python
sentence = "Python is easy and Python is useful"
words = sentence.lower().split()

counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

print(counts)
# 輸出：{'python': 2, 'is': 2, 'easy': 1, 'and': 1, 'useful': 1}
# 說明：dict.get(word, 0) 讓第一次遇到單字時從 0 開始計數。
```

### 3.6 自查

1. `text[1:5]` 是否包含索引 5？
2. `strip()` 和 `replace()` 是否會直接改變原字串？
3. `split()` 的結果是什麼類型？
4. 如何取得字串最後三個字元？
5. 如何將多個字串用逗號連接？
6. 為什麼 `text[0] = "X"` 會失敗？

---

## 4. 模組 C：列表、元組、字典、集合（82–100）

### 4.1 列表（`list`）

列表是有順序、可修改、可重複的容器。

```python
numbers = [3, 1, 5]

numbers.append(8)       # 尾部新增
numbers.insert(1, 9)    # 指定位置插入
numbers[0] = 10         # 修改
numbers.remove(9)       # 按值刪除第一次出現的元素
last = numbers.pop()    # 刪除並返回元素
# 輸出：操作後 numbers == [10, 1, 5]，last == 8
# 說明：remove() 按值刪除；pop() 預設刪除最後一項並把被刪除的值返回。
```

常用操作：

```python
len(numbers)
5 in numbers
numbers.sort()           # 原地排序
numbers.reverse()       # 原地反轉
numbers[1:3]            # 切片，返回新列表
# 輸出：len(numbers) == 3、5 in numbers == True、numbers.sort() 後為 [1, 5, 10]，reverse() 後為 [10, 5, 1]。
# 說明：sort() 和 reverse() 會修改原列表；切片則產生新列表。
```

`remove(value)` 是按值刪除；`pop(index)` 是按索引刪除並返回元素。找不到 `remove` 的值或索引越界時，要注意例外情況。

### 4.2 元組（`tuple`）

元組是有順序但不可修改的序列。

```python
point = (10, 20)
x, y = point
# 輸出：x == 10，y == 20
# 說明：這是元組解包；元組本身不能用 point[0] = 99 這種方式修改。
```

適合用於：

- 不希望資料被修改的固定記錄。
- 函式或運算產生的多個固定結果。
- 作為某些字典鍵的組成部分。

單元素元組需要逗號：

```python
one = (1,)
# 輸出：type(one) 是 tuple
# 說明：單元素元組的逗號不能省略；(1) 只是普通整數表達式。
```

### 4.3 字典（`dict`）

字典保存「鍵 → 值」映射，適合根據唯一識別資訊快速查找。

```python
student = {"name": "小明", "score": 88}

student["score"] = 92
student["age"] = 18
print(student.get("class", "未知"))
# 輸出：未知
# 說明：get() 找不到 class 鍵時返回預設字串，不會拋出 KeyError。
```

遍歷方式：

```python
for key in student:
    print(key, student[key])

for key, value in student.items():
    print(key, value)
# 輸出：先列印 name 小明、score 92、age 18（順序以目前字典內容為準）
# 說明：items() 每次提供一個 (key, value) 鍵值對。
```

常用方法：

- `get(key, default)`：安全查找，不存在時返回預設值。
- `keys()`：取得鍵。
- `values()`：取得值。
- `items()`：取得鍵值對。
- `pop(key)`：刪除指定鍵並返回值。

字典的鍵必須是可雜湊、通常也是不可變的類型，例如字串、數字和元組；列表不能作為字典鍵。

### 4.4 集合（`set`）

集合是無序且不重複的元素集合，常用於去重和集合關係判斷。

```python
numbers = [1, 2, 2, 3, 3]
unique = set(numbers)

a = {1, 2, 3}
b = {3, 4, 5}

print(a | b)   # 聯集
print(a & b)   # 交集
print(a - b)   # 差集
# 輸出：unique == {1, 2, 3}；a | b == {1, 2, 3, 4, 5}；a & b == {3}；a - b == {1, 2}
# 說明：集合不保證對外展示順序，重點是成員關係而不是列印順序。
```

不要依賴集合保存順序；需要順序時應使用列表。

### 4.5 隊列與堆疊的概念

- 堆疊：後進先出（LIFO），可用列表的 `append()` + `pop()` 模擬。
- 隊列：先進先出（FIFO），初學階段可用列表理解概念；大量資料時應使用專門的佇列結構。

```python
stack = []
stack.append("A")
stack.append("B")
print(stack.pop())  # B
# 輸出：B
# 說明：B 最後放入，所以先被取出，這就是後進先出（LIFO）。
```

### 4.6 引用、可變性與拷貝

直接賦值不會建立新列表：

```python
a = [1, 2]
b = a
b.append(3)
print(a)  # [1, 2, 3]
# 輸出：a == [1, 2, 3]
# 說明：b = a 只是建立另一個名稱，a 和 b 指向同一個列表。
```

淺拷貝只複製外層容器：

```python
a = [[1, 2], [3, 4]]
b = a.copy()
b.append([5, 6])       # 外層不同
b[0].append(9)         # 內層仍共享
# 輸出：b == [[1, 2, 9], [3, 4], [5, 6]]；a == [[1, 2, 9], [3, 4]]
# 說明：copy() 只複製外層，巢狀列表仍是同一批內層對象。
```

深拷貝會遞迴複製巢狀物件：

```python
import copy

b = copy.deepcopy(a)
# 輸出：b 是與 a 內容相同但內外層都獨立的新列表
# 說明：之後修改 b 的巢狀元素，不會影響 a。
```

第一輪學習要記住：

- `b = a`：同一個對象。
- `b = a.copy()`：只複製外層。
- `copy.deepcopy(a)`：遞迴複製巢狀資料。

### 4.7 資料結構選擇決策

| 需求 | 建議結構 |
|---|---|
| 有順序、可重複、需要修改 | 列表 |
| 有順序、資料固定不改 | 元組 |
| 按名稱或 ID 查找 | 字典 |
| 去重、判斷共同元素 | 集合 |
| 後進先出 | 列表模擬堆疊 |

### 4.8 自查

1. `remove()` 和 `pop()` 有什麼差別？
2. 何時應使用元組而不是列表？
3. 如何遍歷字典的鍵和值？
4. 為什麼集合不能用來保存有意義的順序？
5. `a = b` 為什麼可能造成意外修改？
6. 「列表裡放字典」適合表示哪類資料？

推薦表示多位學生：

```python
students = [
    {"name": "小明", "score": 88},
    {"name": "小華", "score": 95},
]
# 輸出：students 是一個包含兩個字典的列表
# 說明：列表保留學生順序，字典用 name 和 score 表示每位學生的欄位。
```

---

## 5. 模組 D：字符編碼、解碼與 I/O（101–125）

### 5.1 字符、字串與字節

- **字符**：人能理解的文字符號，例如「你」或 `A`。
- **字串（`str`）**：Python 用來表示文字的對象。
- **字節（`bytes`）**：電腦儲存和傳輸的位元組序列。
- **編碼（encode）**：`str → bytes`。
- **解碼（decode）**：`bytes → str`。

```python
text = "你好"
data = text.encode("utf-8")
print(data)

restored = data.decode("utf-8")
print(restored)
# 輸出：先列印 b'\xe4\xbd\xa0\xe5\xa5\xbd'，再列印 你好
# 說明：encode() 把 str 變成 bytes；decode() 使用相同編碼把 bytes 還原成 str。
```

編碼和解碼必須使用相容的字符集：

```python
data = "你好".encode("utf-8")
data.decode("gbk")   # 可能出錯或產生亂碼
# 輸出：通常會得到 UnicodeDecodeError，或在某些資料下得到錯誤文字
# 說明：解碼方式必須和編碼方式匹配；不要用任意編碼掩蓋資料問題。
```

### 5.2 常見字符集

- **ASCII**：早期英文字符集，主要覆蓋英文字母、數字和控制字符。
- **GBK**：中文環境常見的編碼方式。
- **Unicode**：為世界各種字符提供統一編碼標準。
- **UTF-8**：Unicode 的一種變長編碼，網路和現代 Python 程式最常用。

Unicode 是字符標準，UTF-8/UTF-16 等是具體的編碼方式。實際寫檔案時，通常明確使用 UTF-8。

### 5.3 亂碼問題的本質

亂碼通常不是原始文字消失，而是「寫入時採用的編碼」和「讀取時採用的解碼」不一致。

```text
文字 → 以 UTF-8 編碼寫入 → 以 GBK 解碼讀取 → 亂碼或解碼錯誤
```

排查順序：

1. 確認原始資料使用什麼編碼。
2. 確認程式讀寫時指定相同編碼。
3. 確認編輯器、終端機和資料來源沒有再次轉碼。
4. 優先統一使用 UTF-8。

### 5.4 Python 2 與 Python 3 的注意事項

合集中的多版本內容有歷史背景。Python 2 的 `raw_input()` 在 Python 3 中已不存在；Python 3 使用 `input()`。學習新程式時以 Python 3 的 `str`、`bytes` 和 UTF-8 為主，不要把 Python 2 寫法混入新專案。

### 5.5 I/O 與儲存媒介的直觀理解

機械硬碟、固態硬碟和記憶體的速度不同，I/O 操作常比純 CPU 運算慢。實務上應注意：

- 不要反覆開關同一個檔案。
- 大檔案不要一次全部讀入記憶體。
- 能批量讀寫時，避免逐字元操作。
- 用緩衝和分段處理降低 I/O 成本。

### 5.6 自查

1. `encode()` 和 `decode()` 的方向分別是什麼？
2. `str` 和 `bytes` 有什麼差別？
3. 為什麼 UTF-8 是常見的預設選擇？
4. 讀取 UTF-8 檔案時用 GBK 解碼可能發生什麼？
5. 亂碼排查應先檢查哪些地方？

---

## 6. 模組 E：檔案、路徑與讀寫模式（126–167）

### 6.1 檔案處理的通用流程

```text
確定路徑 → 打開檔案 → 讀取/寫入 → 關閉檔案
```

推薦使用 `with`，讓檔案離開區塊後自動關閉：

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
# 輸出：印出 data.txt 的全部文字內容
# 說明：with 區塊結束後 file 會自動關閉，即使區塊內發生例外也較安全。
```

### 6.2 路徑概念

- **絕對路徑**：從磁碟根目錄開始的完整位置。
- **相對路徑**：相對於目前工作目錄的位置。
- `.`：目前目錄。
- `..`：上一層目錄。

```python
with open("data/users.txt", "r", encoding="utf-8") as file:
    data = file.read()
# 輸出：data 是檔案全文的 str；檔案不存在時會拋出 FileNotFoundError
# 說明：相對路徑相對於目前工作目錄，不一定是 .py 檔案所在目錄。
```

相對路徑的基準通常是程式的目前工作目錄，而不是一定等於 `.py` 檔案所在位置；遇到找不到檔案時，先確認工作目錄。

### 6.3 `open()` 的重要參數

```python
open(file, mode="r", encoding=None)
# 輸出：這是 open() 的呼叫形式，不會自行列印；實際返回一個檔案對象
# 說明：file 是路徑，mode 決定讀寫方式，encoding 只適用於文字模式。
```

- `file`：檔案路徑。
- `mode`：讀寫模式。
- `encoding`：文字編碼，文字檔案建議明確指定 `"utf-8"`。

### 6.4 主要模式

| 模式 | 意義 | 注意 |
|---|---|---|
| `r` | 讀取 | 檔案不存在會失敗 |
| `w` | 寫入 | 會清空原內容；檔案不存在則建立 |
| `a` | 追加 | 從檔案尾端寫入 |
| `x` | 獨佔建立 | 已存在時失敗 |
| `b` | 二進制 | 如 `rb`、`wb` |
| `+` | 同時讀寫 | 如 `r+`、`w+` |

組合示例：

```python
with open("log.txt", "a", encoding="utf-8") as file:
    file.write("new record\n")

with open("image.png", "rb") as file:
    binary_data = file.read()
# 輸出：new record 被追加到 log.txt；binary_data 的類型是 bytes
# 說明：rb 是二進制讀取，適合圖片等非文字檔案。
```

### 6.5 讀取方法

```python
with open("data.txt", "r", encoding="utf-8") as file:
    all_text = file.read()       # 全部內容，返回 str
# 輸出：all_text 是整個檔案內容的 str
```

```python
with open("data.txt", "r", encoding="utf-8") as file:
    first_line = file.readline() # 一行，返回 str
    rest = file.readlines()      # 剩餘各行組成的列表
# 輸出：first_line 是一行 str；rest 是後續各行組成的 list[str]
```

大檔案建議逐行處理：

```python
with open("large.txt", "r", encoding="utf-8") as file:
    for line in file:
        process_line = line.strip()
        # 逐行處理，不必一次載入整個檔案
# 輸出：每次迴圈處理一行；process_line 是去除首尾空白後的 str
```

### 6.6 寫入方法

```python
with open("result.txt", "w", encoding="utf-8") as file:
    file.write("第一行\n")
    file.write("第二行\n")
# 輸出：result.txt 會被覆寫為兩行文字；write() 返回寫入的字元數
```

`writelines()` 接受可迭代的字串，但不會自動補換行：

```python
lines = ["A\n", "B\n"]
with open("result.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
# 輸出：result.txt 內容為 A 和 B 兩行
# 說明：writelines() 不會自動加入換行，所以 lines 中要自行放入 \n。
```

### 6.7 換行與轉義

- `\n`：換行。
- `\t`：定位字元。
- `\\`：反斜線。
- `\"` 或 `\'`：在字串中表示引號。

Windows、Linux 和 macOS 的換行表示可能不同；讀寫文字時一般讓 Python 處理換行即可，跨平台程式不要硬編碼不必要的路徑分隔符。

### 6.8 文件指針：`tell()` 與 `seek()`

檔案被讀寫時有一個目前位置，稱為文件指針。

```python
with open("data.txt", "r", encoding="utf-8") as file:
    print(file.tell())
    first = file.read(5)
    print(file.tell())
    file.seek(0)
    print(file.read(5))
# 輸出：先輸出 0，再輸出讀取後的新位置，seek(0) 後再次輸出檔案前 5 個字元
# 說明：tell() 查看指針位置；seek(0) 把指針移回檔案開頭。
```

- `tell()`：取得目前指針位置。
- `seek(position)`：移動指針。
- 文字模式下位置通常不要自行假設為「字元數」，因為編碼後的位元組長度可能不同。

### 6.9 文本模式與二進制模式

```python
# 文本模式：得到 str
with open("note.txt", "r", encoding="utf-8") as file:
    text = file.read()

# 二進制模式：得到 bytes
with open("note.txt", "rb") as file:
    data = file.read()
# 輸出：text 的類型是 str，data 的類型是 bytes
# 說明：同一檔案用文本模式會自動解碼，用二進制模式則保留原始位元組。
```

- 文本模式適合文字，負責編碼/解碼。
- 二進制模式適合圖片、壓縮檔、可執行檔等原始位元組資料。
- 二進制模式下不能直接使用文字字串寫入，必須寫入 `bytes`。

### 6.10 文件拷貝

小檔案可以一次讀取；大檔案應分段拷貝：

```python
source_path = "source.bin"
target_path = "target.bin"

with open(source_path, "rb") as source, open(target_path, "wb") as target:
    while True:
        chunk = source.read(1024 * 1024)
        if not chunk:
            break
        target.write(chunk)
# 輸出：target.bin 與 source.bin 的位元組內容相同
# 說明：每次只讀取固定大小 chunk，避免大檔案一次佔滿記憶體。
```

這種寫法的核心是：每次只在記憶體中保留固定大小的 `chunk`。

### 6.11 文件修改的常見策略

文本檔案通常不能直接在中間插入或刪除任意長度的內容。常見做法是：

```text
讀取原檔案 → 在記憶體中修改內容 → 寫入暫存檔或覆寫原檔案
```

對大檔案則可逐行讀取、逐行寫入新檔案，完成後再替換原檔案。操作重要檔案時應先備份，避免程式中途失敗造成資料遺失。

### 6.12 帳號登入/註冊的資料模型

初學階段可以用簡單的文字格式保存：

```text
alice:123456
bob:abc123
```

讀取後轉成字典：

```python
users = {}

with open("users.txt", "r", encoding="utf-8") as file:
    for line in file:
        line = line.strip()
        if not line:
            continue
        username, password = line.split(":", 1)
        users[username] = password
# 輸出：users 會變成 {'alice': '123456', 'bob': 'abc123'}（實際內容依檔案而定）
# 說明：split(":", 1) 只按第一個冒號切分，避免密碼中再有冒號時過度拆分。
```

重要提醒：這只是練習檔案處理的簡化方案。真實系統不能明文保存密碼，後續應學習雜湊、加鹽和資料庫。

### 6.13 自查

1. `r`、`w`、`a`、`x` 分別適合什麼情況？
2. 為什麼推薦使用 `with open(...)`？
3. `read()`、`readline()`、`readlines()` 返回值有什麼不同？
4. `w` 模式為什麼有覆蓋資料的風險？
5. 如何逐行處理大檔案？
6. 文本模式和二進制模式的返回值分別是什麼？
7. `seek(0)` 的作用是什麼？
8. 相對路徑是相對於哪個工作目錄？

---

## 7. 建議完成的階段專案

### 專案一：文件版學生管理器

資料格式可先採用：

```text
小明,88
小華,95
```

功能：

1. 啟動時從檔案載入學生資料。
2. 新增學生，拒絕重複姓名。
3. 查詢、修改、刪除學生。
4. 顯示平均分、最高分和最低分。
5. 程式結束前將資料寫回檔案。

### 專案二：文件拷貝與統計工具

功能：

- 輸入來源和目標路徑。
- 以二進制分段拷貝檔案。
- 統計檔案大小、行數、字元數。
- 搜尋指定關鍵字。
- 將統計結果保存到報告檔案。

### 專案三：文字清洗與單字統計器

流程：

```text
讀取文字檔案 → 統一大小寫 → 去除標點 → split → 字典計數 → 輸出排名
```

這個專案能綜合字串、列表、字典、集合和檔案 I/O。

---

## 8. 常見錯誤與排查方式

### 8.1 `FileNotFoundError`

原因：路徑錯誤、工作目錄不對，或檔案尚未建立。

排查：

- 打印目前工作目錄。
- 確認檔案名稱和副檔名。
- 優先使用清楚的專案目錄結構。

### 8.2 `UnicodeDecodeError`

原因：讀取時使用了不匹配的編碼。

排查：

- 確認檔案實際編碼。
- 在 `open()` 中明確指定正確 `encoding`。
- 不要用「隨便換一個編碼」掩蓋問題。

### 8.3 `KeyError`

原因：直接使用不存在的字典鍵。

```python
value = data.get("missing", "default")
# 輸出：value == "default"
# 說明：get() 可在查找不存在的鍵時提供安全的預設值。
```

### 8.4 `ValueError`

原因：轉換或拆分格式不符合預期，例如 `int("abc")` 或一行資料缺少分隔符。

處理前先驗證資料格式，不要假設每一行都完美。

### 8.5 覆寫檔案

開啟 `w` 模式會清空原內容。涉及重要資料時：

- 先確認是否真的要覆寫。
- 先寫入暫存檔。
- 必要時保留備份。

---

## 9. 階段知識自查表

### 語法層面

- [ ] 能使用字串索引、切片和步長。
- [ ] 能使用 `strip()`、`split()`、`join()`、`replace()`。
- [ ] 能完成列表的增刪改查和排序。
- [ ] 能遍歷字典的鍵、值和鍵值對。
- [ ] 能使用集合去重與交集/聯集/差集。
- [ ] 能解釋直接引用、淺拷貝和深拷貝。
- [ ] 能使用 `bin()`、`oct()`、`hex()` 和 `int(text, base)`。
- [ ] 能使用 `open()`、`read()`、`write()` 和 `with`。

### 概念層面

- [ ] 能說明列表、元組、字典和集合的適用場景。
- [ ] 能說明 `str` 和 `bytes` 的區別。
- [ ] 能說明編碼和解碼的方向。
- [ ] 能解釋亂碼的主要原因。
- [ ] 能說明 `r`、`w`、`a`、`x`、`b`、`+` 模式的差異。
- [ ] 能說明文件指針和大檔案分段讀取的意義。
- [ ] 能說明為什麼 `with` 比手動 `close()` 更安全。

### 問題建模層面

- [ ] 拿到需求時，能先寫出資料格式。
- [ ] 能把流程拆成輸入、驗證、處理、輸出和保存。
- [ ] 能列出空資料、重複資料、不存在資料和錯誤格式等邊界情況。
- [ ] 能為同一問題比較兩種資料結構的優缺點。
- [ ] 能不看原答案重寫至少一個文件型專案。

---

## 10. 進入函式階段前的達標標準

建議至少滿足以下條件再深入第 168 集之後的函式內容：

1. 能獨立完成文件版學生管理器。
2. 能完成二進制分段拷貝工具。
3. 能處理 UTF-8 文字的讀寫和常見編碼錯誤。
4. 能解釋列表、元組、字典和集合的選擇理由。
5. 能處理檔案不存在、空行、重複帳號和格式錯誤。
6. 能把重複的資料處理流程說成清楚的步驟。

第 168 集「函式引入」是下一階段的轉折點。此後要把本階段寫過的登入器、學生管理器和文字統計器拆成多個函式，逐步學會：

```text
把大程序拆成小功能 → 給功能命名 → 傳入參數 → 返回結果 → 重複使用
```

這樣學習，才能從「能把程式寫出來」進一步提升到「能設計和維護程式」。
