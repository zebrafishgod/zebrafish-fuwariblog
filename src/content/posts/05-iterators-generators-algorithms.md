---
title: Python 第 242-276 集：迭代器、生成器及常用算法
published: 2026-09-15
updated: 2026-09-18
description: 迭代器 生成器 常用算法
tags: [Python, 基礎篇]
category: Python 學習
draft: false
lang: zh_TW
---

# 第 5 章｜迭代器、生成器與常用算法

> 對應既有學習路線的 242–276 節範圍，按概念重新編排。本章是依課程主題整理的自學講義，並非逐集影片逐字稿。示例以 Python 3 為準。

## 1. 本章解決什麼問題？需要哪些前置知識？

這一章回答四個問題：

1. `for` 為什麼能逐個讀取串列、字串及檔案？
2. 如果資料很多，能不能「需要一個，才產生一個」？
3. 怎樣用推導式、遞迴、二分搜尋，把問題寫得更清楚？
4. `lambda` 究竟是什麼，為什麼常和 `sorted()` 一起出現？

前置知識是串列 `list`、字典 `dict`、集合 `set`、條件、迴圈，以及函式的參數和 `return`。本章不需要先學類別。

讀到捕捉 `StopIteration` 的例子時，先把 `try/except` 理解成「嘗試取值；如果已經取完，就走另一段處理」。例外的完整說明見 [補充：錯誤與測試](bridge-errors-testing.md)。

建議分成三次學習：先掌握迭代與生成器，再學推導式與遞迴，最後做搜尋與排序。`send()` 是選讀，不是寫出第一個完整程式的必要條件。

## 2. 先建立直覺

### 2.1 容器、可迭代物件、迭代器，是不同角度的名稱

把一份商品清單想成一本目錄：

- **可迭代物件（iterable）**：可以交給 `iter()`，取得一個「逐項讀取工具」。串列、字串、字典、`range()` 都是常見例子。
- **迭代器（iterator）**：這個工具本身會記住目前讀到哪裡。`next()` 讓它往後讀一項。
- **容器**：主要強調「保存資料」。不是所有可迭代物件都會預先存好全部資料，例如生成器可以邊讀邊算。

一個串列可以產生多個互不影響的迭代器。迭代器通常只往前走；取完後，不會因為再寫一個 `for` 就回到開頭。

### 2.2 生成器是一種迭代器

普通函式用 `return` 交回結果，這一次呼叫就結束。含 `yield` 的函式是**生成器函式**：呼叫它會先建立生成器物件；之後每次索取下一項，才從上次暫停處繼續執行。

| 動作 | 做了什麼 | 是否一定顯示在螢幕？ |
|---|---|---|
| `print(value)` | 把內容顯示到輸出 | 是，預設顯示在終端 |
| `return value` | 把結果交回函式呼叫處，結束這次呼叫 | 否 |
| `yield value` | 交出本次迭代值，保留暫停位置 | 否 |

生成器常能省下「預先保存全部結果」的記憶體，但不保證運算比較快。如果最後仍寫 `list(generator)`，所有結果仍會放進記憶體。

### 2.3 算法就是解題步驟

遞迴把問題拆成「同一種、更小的問題」；二分搜尋在**已排序**的資料中，每次排除約一半候選。兩者都不是「看起來比較高級就使用」，而是要看問題是否符合條件。

## 3. 最小可跑例：按順序練習

以下程式碼區塊除非特別註明，皆可各自存成一個 `.py` 檔執行。標記「輸出」的註解描述 `print()` 顯示的內容；`return` 和 `yield` 本身不負責顯示。

### 3.1 `iter()` 建立讀取位置，`next()` 讀下一項

```python
# 輸出：
# 蘋果
# 香蕉
# 已取完
# 說明：next() 每次取得一個值；讀完後會產生 StopIteration。
products = ["蘋果", "香蕉"]
cursor = iter(products)

print(next(cursor))
print(next(cursor))

try:
    print(next(cursor))
except StopIteration:
    print("已取完")
```

`products` 仍然是完整串列；前進的是 `cursor` 的讀取位置，並不是從串列刪除了元素。最後一次 `next(cursor)` 沒有成功取得值，所以第三個 `print()` 不會先印出什麼，再發生錯誤。

若「取不到」是正常情況，可以為 `next()` 提供預設值：

```python
# 輸出：
# 10
# 沒有下一項
# 說明：next(迭代器, 預設值) 在耗盡時交回預設值。
cursor = iter([10])
print(next(cursor, "沒有下一項"))
print(next(cursor, "沒有下一項"))
```

注意：`next([10, 20])` 不成立，因為串列可迭代，卻不是迭代器。先寫 `cursor = iter([10, 20])`。若資料本身可能就是預設值，便不能只憑「結果等於預設值」判斷是否耗盡。

### 3.2 `for` 在背後做什麼？

平常這樣寫就好：

```python
# 輸出：
# 甲
# 乙
# 說明：for 會取得迭代器，逐項讀取，並在讀完時自動停止。
names = ["甲", "乙"]
for name in names:
    print(name)
```

下面用明確步驟模擬核心行為，目的是理解，不是要求你以後把 `for` 改寫得更長：

```python
# 輸出：
# 甲
# 乙
# 說明：這是 for 的核心取值流程；只在 next() 周圍捕捉耗盡訊號。
names = ["甲", "乙"]
cursor = iter(names)

while True:
    try:
        name = next(cursor)
    except StopIteration:
        break
    print(name)
```

同一個串列可以反覆遍歷，是因為每輪 `for name in names` 都取得新的讀取位置。如果改為遍歷同一個已耗盡的迭代器，第二輪就沒有項目：

```python
# 輸出：
# [1, 2]
# []
# [1, 2]
# 說明：list(cursor) 會消耗 cursor；原串列沒有因此清空。
numbers = [1, 2]
cursor = iter(numbers)
print(list(cursor))
print(list(cursor))
print(list(numbers))
```

### 3.3 生成器函式與生成器物件

```python
# 輸出：
# 已建立
# 開始計算
# 0
# 1
# 4
# 說明：square_numbers 是函式；stream 是一次呼叫產生的生成器物件。
def square_numbers(limit):
    print("開始計算")
    for number in range(limit):
        yield number * number


stream = square_numbers(3)
print("已建立")
print(next(stream))
print(next(stream))
print(next(stream))
```

重點是輸出順序：`stream = square_numbers(3)` 只建立物件，**不立即執行函式主體**。第一次 `next(stream)` 才開始執行；遇到 `yield 0` 暫停，把 `0` 交回給外層的 `print()`。

第二次 `next()` 會從第一次 `yield` 後面繼續。它不是重新呼叫一次函式，也不會重新印出「開始計算」。

同一個生成器耗盡後不能直接倒回；再次呼叫生成器函式，會建立另一個新物件：

```python
# 輸出：
# [0, 1, 4]
# []
# [0, 1, 4]
# 說明：兩次 square_numbers(3) 建立的是不同生成器。
def square_numbers(limit):
    for number in range(limit):
        yield number * number


stream = square_numbers(3)
print(list(stream))
print(list(stream))
print(list(square_numbers(3)))
```

在生成器內，`return` 或執行到函式結尾表示「結束迭代」。`return 99` 也不等於「再產生一個 99」；要交出一個可遍歷的項目應使用 `yield 99`。初學時，生成器中的 `return` 先只用來結束即可。

### 3.4 從普通迴圈理解推導式

推導式不是新的計算能力，而是把「遍歷、篩選、組成容器」縮短。先看容易追蹤的版本：

```python
# 輸出：
# [4, 16]
# [4, 16]
# 說明：兩種寫法都先挑出偶數，再保存它們的平方。
numbers = [1, 2, 3, 4]

squares = []
for number in numbers:
    if number % 2 == 0:
        squares.append(number * number)
print(squares)

squares = [number * number for number in numbers if number % 2 == 0]
print(squares)
```

閱讀順序是：`for number in numbers` 逐項取得值 → `if ...` 決定保留與否 → 前面的 `number * number` 決定保存什麼。

| 形式 | 結果 | 典型用途 |
|---|---|---|
| `[運算式 for 變數 in 資料]` | 串列 | 保存轉換後的順序資料 |
| `{鍵: 值 for 變數 in 資料}` | 字典 | 建立查找關係 |
| `{運算式 for 變數 in 資料}` | 集合 | 去除重複 |
| `(運算式 for 變數 in 資料)` | 生成器 | 之後逐項產生結果 |

```python
# 輸出：
# {'apple': 5, 'banana': 6}
# ['apple', 'banana']
# 說明：字典把名稱映射到長度；集合去重後用 sorted() 穩定顯示順序。
names = ["apple", "banana", "apple"]
length_by_name = {name: len(name) for name in names}
unique_names = {name for name in names}

print(length_by_name)
print(sorted(unique_names))
```

字典的鍵必須唯一，重複鍵會用後面的值覆蓋前面的值。集合不提供位置索引，不應依賴它顯示的順序。本例用 `sorted()` 產生排序後的串列，讓輸出可核對。

先單獨認識條件運算式，也常被稱為「三元運算式」：`符合時的值 if 條件 else 不符合時的值`。它會計算出一個值，可放在賦值右側；只有被選中的那一側會求值。

```python
# 輸出：
# 可購買
# 說明：條件運算式交回其中一個字串；print() 才顯示它。
stock = 3
message = "可購買" if stock > 0 else "已售完"
print(message)
```

另一種 `if` 放在推導式前面，是「每一項都保留，但決定這一項的值」：

```python
# 輸出：
# ['奇數', '偶數', '奇數', '偶數']
# 說明：if/else 是選值；尾端沒有篩選條件，因此輸出仍有四項。
labels = ["偶數" if number % 2 == 0 else "奇數" for number in range(1, 5)]
print(labels)
```

如果一條推導式需要讀兩三遍才明白，改回普通 `for` 往往更好。尤其別把複雜操作和多層條件塞在同一行。

### 3.5 生成器表達式：把「先存好」改成「逐個給」

```python
# 輸出：
# 30
# 0
# [1, 4, 9, 16]
# 說明：第一次 sum() 消耗生成器；再次求和時已沒有項目。
stream = (number * number for number in range(1, 5))
print(sum(stream))
print(sum(stream))
print([number * number for number in range(1, 5)])
```

`sum(stream)` 不需要先建立平方結果串列，就能逐項加總。第二個 `0` 是「空的可迭代物件加總為 0」，不是生成器重新算了一次卻算錯。

需要隨機取得第 100 項、反覆遍歷、確認長度時，串列通常更直接。只需掃過一次、資料很大時，再考慮生成器。生成器表達式的**結果元素**逐項計算，但最左側 `for` 的可迭代運算式會在建立時求值；不要把「惰性」理解成所有東西都完全延後。

例如統計一段英文文字有多少個空白分隔的單詞，可以逐行計算，再加總：

```python
# 輸出：
# 7
# 說明：split() 不指定參數時按連續空白分隔；標點不另外分詞。
lines = ["Python is fun.\n", "I write code.\n", "Great!\n"]
word_count = sum(len(line.split()) for line in lines)
print(word_count)
```

這個例子是 `3 + 3 + 1`。套用到先前學過的檔案處理時，在 `with open(..., encoding="utf-8") as file:` 區塊內，把 `lines` 換成 `file`，就能逐行讀取並加總，避免先用 `readlines()` 建立完整行串列。必須在檔案仍開啟時完成消耗。這是「按空白分隔」的單詞數，不是中文字數，也不是自然語言的精確分詞；`len(text)` 計算的又是另一種長度，先定義需求再選算法。

### 3.6 選讀：`send()` 不只取值，也能把值送回去

`next()` 只要求「繼續執行」。`send(value)` 還把一個值送進目前暫停的 `yield` 運算式。

```python
# 輸出：
# 0
# 5
# 8
# 說明：yield 交出目前總和；恢復時，send() 的參數成為 amount 的值。
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

把 `amount = yield total` 拆成兩個時刻：

1. **暫停前**：把 `total` 交給外面。
2. **恢復時**：外面送入的值，才會賦給 `amount`。

第一次要先 `next(counter)`，或 `counter.send(None)`，讓它走到第一個暫停點。尚未啟動就 `send(5)` 會產生 `TypeError`。本例啟動後預期每次收到數字；若再呼叫 `next(counter)`，送入的會是 `None`，於是加法失敗。`close()` 用來結束這個本來會一直等待輸入的生成器。

這是雙向互動的初步認識。一般資料遍歷用 `for` 已足夠，不需要特意改用 `send()`。

### 3.7 遞迴：先找到「不用再分解」的情況

計算 `4!`：`4 × 3 × 2 × 1`。可以把它寫成 `4 × 3!`，而 `3!` 又是 `3 × 2!`。

```python
# 輸出：
# 24
# 1
# 說明：factorial() 用 return 交回數字；外層 print() 才顯示結果。
# 前提：n 是非負整數。
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)


print(factorial(4))
print(factorial(0))
```

遞迴必須有兩件事：

- **基例**：這裡是 `n == 0`，可以直接交回答案。
- **朝基例前進的規則**：這裡每次改成 `n - 1`。

不能只檢查「有沒有寫 `if`」。如果輸入 `-1`，每次再減一會離 `0` 越來越遠；最終通常會遇到 `RecursionError`。本例把非負整數列為呼叫前提。要接收使用者輸入，還需要先檢查範圍與型別。

每次遞迴呼叫都要保留尚未完成的工作。Python 有遞迴深度限制；遞迴也通常有額外開銷。像階乘這種直線型累積，使用迴圈同樣清楚；本例主要用來練習「拆解、到底、返回」。不要用調高限制代替修正無限遞迴。

### 3.8 全排列：固定一個位置，再排列剩下的項目

`["A", "B", "C"]` 的全排列是「三個元素各用一次，所有可能的先後順序」。

```python
# 輸出：
# ABC
# ACB
# BAC
# BCA
# CAB
# CBA
# 說明：permutations() 交回排列串列；外層 for 和 print() 顯示各排列。
# 前提：輸入元素互不重複；本例適合少量資料。
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

`items[:index] + items[index + 1:]` 是把「選中的那一項」排除後，組成新的串列；原始輸入不會被修改。`"".join(arrangement)` 是把字串串列接成一個字串。

基例為什麼是 `[[]]` 而不是 `[]`？因為「沒有剩餘元素」仍有**一種**排列方式：空排列。外層才能把最後選中的元素接上去。若直接交回 `[]`，上層 `for rest in ...` 一次也不執行，全部結果就消失了。

互不重複的 `n` 個元素有 `n!` 個排列。`10!` 就有 3,628,800 個結果。這份寫法還會保存中間結果，所以只拿小型資料學習。若輸入有重複值，可能產生內容重複的排列；去重是另一個問題。生成器可以減少一次保存的結果，卻不能消除排列數量爆增的事實。

### 3.9 二分搜尋：先確認資料已排序

在 `[3, 7, 12, 18, 25]` 找 `18`，先看中間 `12`；因為 `18 > 12`，只需要在右半邊找。

```python
# 輸出：
# 3
# -1
# -1
# 說明：找到時 return 索引，找不到時 return -1；print() 顯示結果。
# 前提：numbers 已按從小到大排序。
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

這裡 `left` 和 `right` 都包含在搜尋範圍內，所以條件是 `left <= right`；只剩一個候選時，也要檢查。更新成 `middle + 1` 或 `middle - 1`，表示已檢查過的中間項目不必再看，搜尋範圍確實縮小。

幾個實際限制：

- 未排序的資料不能直接套用這個算法。
- 函式回傳 `-1` 表示找不到；不能把這個 `-1` 直接拿去索引，因為 Python 的 `numbers[-1]` 反而表示最後一項。
- 若有重複值，本例找到的是某個符合值的索引，不保證是第一個。
- 若先排序再搜尋，回傳的是排序後的位置。排序成本與原始索引的變化都要考慮。
- 對可快速按索引取得元素的串列，搜尋步數約為 `log₂(n)`；不是每次都必須逐項看完。

### 3.10 `lambda`：小型匿名函式

`lambda 參數: 運算式` 建立函式，呼叫後交回運算式的值。它只有一個運算式，不能寫多行敘述。較長邏輯就用 `def`，並取一個好懂的名字。

```python
# 輸出：
# 7
# 7
# 說明：兩個函式都交回加法結果；lambda 不需要再寫 return。
def add_named(left, right):
    return left + right


add_short = lambda left, right: left + right
print(add_named(3, 4))
print(add_short(3, 4))
```

「高階函式」在這裡指會接收其他函式作為參數的函式。你可以把小型規則交給它，讓它負責完整操作。

### 3.11 用 `key` 說清楚排序依據

```python
# 輸出：
# [('筆', 5), ('杯子', 20), ('書', 30)]
# ('書', 30)
# [('書', 30), ('筆', 5), ('杯子', 20)]
# 說明：key 函式交回比較依據；sorted() 產生新串列，原串列順序不變。
products = [("書", 30), ("筆", 5), ("杯子", 20)]

print(sorted(products, key=lambda product: product[1]))
print(max(products, key=lambda product: product[1]))
print(products)
```

`lambda product: product[1]` 表示「對每個商品，拿第二項價格作為比較依據」。`key` 接收的是**函式本身**，不是你先算好的一個價格。

想由高到低排序，加上 `reverse=True`。價格相同時，Python 的排序會保留原本的相對順序，稱為穩定排序。商品很多時，清楚的 `def price_of(product): ...` 和 `lambda` 都可以使用，功能不因名字不同而改變。

### 3.12 `map()` 轉換，`filter()` 篩選

```python
# 輸出：
# [20, 40, 60]
# [20, 30]
# []
# 說明：Python 3 的 map/filter 交回迭代器；list() 取得並保存結果。
prices = [10, 20, 30]
double_prices = map(lambda price: price * 2, prices)
expensive_prices = filter(lambda price: price >= 20, prices)

print(list(double_prices))
print(list(expensive_prices))
print(list(expensive_prices))
```

`map()` 不丟掉項目，而是逐項轉換；`filter()` 用條件決定是否保留，保留的是原項目。第三行空串列，是因為同一個 `filter` 迭代器已被消耗。

這個例子也可以寫成 `[price * 2 for price in prices]` 和 `[price for price in prices if price >= 20]`。優先選自己和讀者都容易理解的形式，不必為了用 `lambda` 而強行改寫。

### 3.13 類型提示：說明預期，不會自動檢查輸入

類型提示（type hints）把函式希望收到和交回的資料型別寫清楚：`name: str` 表示預期字串，`-> int` 表示預期交回整數。編輯器和額外的靜態檢查工具可用它提醒潛在問題；Python 執行程式時不會只因為寫了註記，就自動轉型或拒絕不符合的值。

```python
# 輸出：
# 7
# 34
# 說明：第二次呼叫故意違反註記，展示型別提示本身不阻止字串相加。
def add(left: int, right: int) -> int:
    return left + right


print(add(3, 4))
print(add("3", "4"))
```

第二行的 `34` 是兩個字串連接後的結果，與整數 `34` 顯示相似，但型別不同。這次呼叫應由類型檢查工具提醒，業務程式不應故意這樣使用。若資料來自 `input()`，仍然要自己轉型、檢查內容並處理失敗。

容器也能標示元素型別。以下寫法需要 Python 3.9 或更新版本：

```python
# 輸出：
# ['筆', '書']
# 說明：list[str] 表示預期為字串串列；-> list[str] 表示回傳值的預期型別。
def nonempty_names(names: list[str]) -> list[str]:
    return [name.strip() for name in names if name.strip()]


print(nonempty_names([" 筆 ", "", "書"]))
```

`strip()` 去除字串兩端空白；本例過濾空字串或只有空白的名稱。`list[str]` 是描述，不是建立串列的操作。只做顯示、沒有需要交回的結果時，可以把回傳提示寫成 `-> None`；這裡的 `None` 也不等於「沒有任何行為」。

類型提示不要和預設值混淆：`quantity: int = 1` 裡，`int` 是預期型別，`1` 才是省略實參時使用的值。函式的輸入範圍、金額單位、是否允許負數，也不是單寫 `int` 就能完整說明，仍需文件和檢查。

## 4. 執行追蹤：真正理解程式停在哪裡

### 4.1 生成器追蹤

以 3.3 的 `square_numbers(3)` 為例：

| 外面做的動作 | 函式裡做的動作 | 交回的值 | 剩下的狀態 |
|---|---|---|---|
| 建立 `stream` | 主體尚未執行 | 生成器物件 | 等待啟動 |
| 第一次 `next(stream)` | 印「開始計算」，算 `0 * 0` | `0` | 暫停在第一次 `yield` |
| 第二次 `next(stream)` | 接續迴圈，算 `1 * 1` | `1` | 暫停在第二次 `yield` |
| 第三次 `next(stream)` | 接續迴圈，算 `2 * 2` | `4` | 暫停在第三次 `yield` |
| 再次 `next(stream)` | 迴圈結束、函式結束 | 沒有下一項 | 產生 `StopIteration` |

第三次交出 `4` 的當下，函式仍暫停在 `yield`。下一次要求取值，才繼續走到結尾。`for` 會自動處理最後的耗盡訊號。

### 4.2 遞迴追蹤

`factorial(3)` 先一路拆解，再一層層返回：

| 階段 | 尚未完成的運算 |
|---|---|
| 呼叫 `factorial(3)` | 等 `3 * factorial(2)` |
| 呼叫 `factorial(2)` | 等 `2 * factorial(1)` |
| 呼叫 `factorial(1)` | 等 `1 * factorial(0)` |
| `factorial(0)` 到基例 | 直接交回 `1` |
| 返回 `factorial(1)` | `1 * 1 = 1` |
| 返回 `factorial(2)` | `2 * 1 = 2` |
| 返回 `factorial(3)` | `3 * 2 = 6` |

對全排列 `permutations(["A", "B"])`：先選 `A` → 剩下 `["B"]` → 再選 `B` → 剩下 `[]` → 基例交回 `[[]]` → 組成 `["B"]` → 組成 `["A", "B"]`。之後回到第一層改選 `B`，得到 `["B", "A"]`。

### 4.3 二分搜尋追蹤

在 `[3, 7, 12, 18, 25]` 找 `18`：

| 輪次 | `left` | `right` | `middle` | 中間值 | 下一步 |
|---|---:|---:|---:|---:|---|
| 1 | 0 | 4 | 2 | 12 | 目標較大，`left = 3` |
| 2 | 3 | 4 | 3 | 18 | 找到，交回 `3` |

找 `9` 時會經過中間值 `12 → 3 → 7`，最後 `left = 2`、`right = 1`。搜尋範圍為空，才交回 `-1`。

## 5. 常見錯誤與修正

| 容易誤會的地方 | 正確理解與修正 |
|---|---|
| 「只要能放進 for，就一定能直接 next」 | `for` 能接收可迭代物件；`next()` 要接收迭代器。必要時先 `iter()`。 |
| 「生成器函式呼叫時已算好全部值」 | 呼叫通常先建立生成器物件；取下一項時才執行主體。 |
| 「把生成器印出來就能看全部內容」 | `print(stream)` 主要顯示物件表示；用 `for` 逐項讀取，或小量資料時用 `list(stream)`。 |
| 「生成器用完還能重複使用」 | 同一個物件不能倒帶；重新呼叫生成器函式建立新物件。 |
| 「有 yield，就一定更快」 | 主要優勢通常是逐項產生與節省中間容器；實際速度需量測。 |
| 「`[x if 條件 for x in ...]` 可以只保留符合值」 | 尾端篩選寫 `[x for x in ... if 條件]`；前端選值必須有 `else`。 |
| 「遞迴有基例就一定會停」 | 每次呼叫還必須朝基例前進，並符合輸入前提。 |
| 「全排列的空輸入應回傳空串列」 | 組合用的基例需要一個空排列，故用 `[[]]`。 |
| 「二分搜尋適用任何串列」 | 必須符合排序前提與一致的比較規則。 |
| 「找到 `-1` 就印 `numbers[-1]`」 | `-1` 是本函式自訂的未找到訊號，先判斷，再取值。 |
| 「lambda 是高級語法，應取代 def」 | 短規則用 lambda；較長或需命名解釋的邏輯用 def。 |

## 6. 練習：先預測，再執行

### 練習 A｜讀取位置

建立 `numbers = [2, 4, 6]` 與 `cursor = iter(numbers)`。先執行一次 `next(cursor)`，再印出 `list(cursor)`，最後印出 `list(cursor)`。執行前寫下兩次 `print()` 的預期內容，並解釋原串列是否改變。

### 練習 B｜生成器

撰寫 `even_numbers(stop)`，逐項產生 `0` 到 `stop` 之前的偶數，邊界與 `range(stop)` 一致。驗收：`list(even_numbers(7))` 是 `[0, 2, 4, 6]`；`list(even_numbers(0))` 是 `[]`。試著只取前兩項，描述程式暫停的位置。

### 練習 C｜推導式

已知 `prices = [12, 5, 20, 5]`，用推導式建立：① 所有大於等於 `10` 的價格；② 去重後的價格集合；③ 以價格為鍵、兩倍價格為值的字典。寫下為何字典只有三個鍵。

### 練習 D｜遞迴與邊界

先用迴圈、再用遞迴寫 `sum_to(n)`，計算 `1 + 2 + ... + n`，前提是 `n` 為非負整數。驗收：`sum_to(0)` 為 `0`，`sum_to(5)` 為 `15`。在遞迴版本旁標明基例，以及每一步如何接近基例。不要用很大的 `n` 壓測遞迴。

### 練習 E｜二分搜尋

用本章函式查找 `[2, 5, 8, 11, 14, 17]` 中的 `2`、`17`、`9`；在紙上列出每輪 `left/right/middle`。再測空串列與只有一項的串列。驗收輸出依次為 `0`、`5`、`-1`；空串列找不到。

### 練習 F｜商品排序與篩選

商品資料為 `[("筆", 5), ("書", 30), ("杯子", 20)]`。建立價格至少 `20` 的商品串列，再按價格由高到低排序。驗收結果為 `[("書", 30), ("杯子", 20)]`，且原串列順序不變。

### 選做｜全排列

用 3.8 的函式驗證兩個元素有 `2` 個排列、三個元素有 `6` 個排列、空串列有 `1` 個空排列。嘗試輸入 `["A", "A"]`，解釋為何會出現兩個相同結果。先用語言說明，不急著實作去重。

## 7. 自查與簡答

先遮住右欄，能用自己的話回答，才算理解。

| 自查問題 | 簡答 |
|---|---|
| iterable 和 iterator 的差別？ | 前者能讓 `iter()` 取得迭代器；後者保存進度，能被 `next()` 逐項讀取。 |
| `iter(iterator)` 會重設進度嗎？ | 不會；迭代器的 `iter()` 通常回傳它自己。 |
| `for` 遇到 `StopIteration` 會怎樣？ | 結束遍歷；通常不把它當作錯誤顯示給使用者。 |
| 生成器函式與生成器物件差在哪裡？ | 函式是產生流程的定義；物件是一輪可暫停、可前進的執行狀態。 |
| `yield 3` 一定會把 `3` 印到螢幕嗎？ | 不會。它交出值；外面要用 `print()` 才會顯示。 |
| 串列推導式與生成器表達式的主要差別？ | 前者立即建立結果串列；後者逐項產生結果且通常只能消耗一次。 |
| `send(5)` 的回傳值是 `5` 嗎？ | 不一定。它送入 `5`，然後交回生成器接下來 `yield` 的值。 |
| 寫遞迴前先確認什麼？ | 輸入範圍、基例，以及每一步確實讓問題變小。 |
| 二分搜尋為什麼用 `middle + 1`？ | 中間值已確認不是答案，不必再納入下一輪，且範圍必須縮小。 |
| `key=lambda product: product[1]` 做什麼？ | 告訴排序或比較函式：把每件商品的價格當作比較依據。 |
| `map()` 和 `filter()` 有何不同？ | 前者轉換每一項，後者按條件保留原項目。 |
| 寫了 `age: int`，使用者輸入會自動變成整數嗎？ | 不會。類型提示不做執行時轉型或驗證；仍需自行轉換與檢查。 |

練習核對：A 的兩次輸出為 `[4, 6]`、`[]`，原串列仍是 `[2, 4, 6]`；C 的第一項是 `[12, 20]`，第二項有 `5、12、20`，第三項為 `{12: 24, 5: 10, 20: 40}`，相同鍵 `5` 只保留一份；D 的遞迴基例可設 `n == 0`，其餘情況交回 `n + sum_to(n - 1)`。

完成本章的最低標準：能解釋一次性迭代、寫出簡單生成器與推導式、說出遞迴停止條件、手動追蹤二分搜尋，並使用 `key` 實作商品排序。接下來在 [第 6 章：模組與包](06-modules-packages.md) 把多個函式整理成可維護的檔案結構。
