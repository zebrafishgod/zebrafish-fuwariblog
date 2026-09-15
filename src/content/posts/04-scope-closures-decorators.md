---
title: Python 第 196-241 集：名稱空間、裝飾器相關
published: 2026-09-14
updated: 2026-09-17
description: 名稱空間、裝飾器相關
tags: [Python, 基礎篇]
category: Python 學習
draft: false
lang: zh_TW
---

# 第四章　名稱從哪裡來？函式如何記住資料與包裝行為？（第 196–241 集）

本章要解決三個問題：同名變數到底用哪一個？函式結束後能否保留某些狀態？如何替多個函式統一增加日誌或計時，而不用到處複製程式碼？

前置知識：`def`、參數、`return`、`*args`、`**kwargs`、引用與可變物件。若仍分不清 `func` 和 `func()`，先回第三章；裝飾器困難往往來自這個區別。

每段 Python 都可單獨執行。請先預測輸出順序，再執行。此章只學常用的函式式裝飾器；類別、`staticmethod`、`classmethod`、`property` 放在附錄，屬於本基礎主線以外的延伸。

## 1. 學習路線

| 原筆記範圍 | 問題 | 核心概念 |
|---|---|---|
| 196–201 | Python 到哪裡找名稱？ | 名稱空間、LEGB |
| 202–208 | 內外層同名時怎麼辦？ | 作用域、`global`、`nonlocal` |
| 209–214 | 函式能像資料一樣使用嗎？ | 保存、傳入、返回函式 |
| 215–216 | 如何保留外層函式的資料？ | 閉包 |
| 217–233 | 如何增加共同功能？ | 包裝、裝飾器、`wraps` |
| 234–241 | 配置與多層包裝如何運作？ | 裝飾器工廠、求值與套用順序 |

## 2. 名稱空間：名稱到物件的對照關係

`price = 10` 可理解為建立名稱 `price` 與整數物件 10 的關係。名稱空間保存這些關係；作用域決定在某一段程式可以如何找到名稱。兩個詞相關，但不是完全同義。

常見的名稱來源：

- **L，Local：**目前函式這次呼叫的局部名稱，包括形參。
- **E，Enclosing：**詞法上包住它的外層函式名稱。
- **G，Global：**函式所屬模組的全域名稱。
- **B，Built-in：**Python 提供的 `len`、`print` 等內建名稱。

這就是一般函式名稱查找的 **LEGB**。它描述這類場景的規則，不是要把之後的類別屬性查找等所有機制都塞進同一套縮寫。

### 2.1 同名時，先找較近的層

```python
name = "全域"


def outer():
    name = "外層函式"

    def inner():
        name = "內層函式"
        print(name)

    inner()
    print(name)


outer()
print(name)
# 輸出：內層函式；外層函式；全域。
# 說明：三個作用域各有自己的 name；內層同名名稱遮蔽外層，沒有刪除或改寫外層。
```

追蹤 `inner()` 裡的 `print(name)`：先找 L，已找到「內層函式」，就不繼續向外找。將內層的賦值刪除，才會用 E 的「外層函式」。全部找不到會產生 `NameError`。

### 2.2 查找看定義位置，不看誰呼叫它

```python
message = "模組的文字"


def show():
    print(message)


def caller():
    message = "呼叫者的文字"
    show()


caller()
# 輸出：模組的文字。
# 說明：show 定義在模組中；caller 的局部名稱不會自動變成 show 的外層作用域。
```

這個差別很重要：外層是程式碼中的嵌套關係，不是執行時「上一個呼叫我的函式」。此外，`if`、`for`、`while` 不會在一般程式區塊中另建函式式局部作用域。

不要取名 `list`、`str`、`sum`、`print` 來保存自己的資料，否則會遮蔽內建名稱。遇到「物件不能呼叫」時，檢查是否把函式名重新賦值成了普通資料。

## 3. 賦值與讀取不同：為什麼出現 `UnboundLocalError`？

```python
count = 10


def broken():
    print(count)
    count = 20


try:
    broken()
except UnboundLocalError:
    print("局部 count 還沒有值")
print(count)
# 輸出：局部 count 還沒有值；10。
# 說明：函式內有對 count 的賦值，Python 將它判定為局部名稱；前面的讀取不會改去找全域值。
```

不是讀到 `count = 20` 才開始局部化。對一般函式，Python 會根據函式整體的繫結操作判定局部名稱。`count += 1` 也有賦值成分，因此同樣要注意。

### 3.1 `global`：重新繫結模組名稱

```python
count = 0


def increase():
    global count
    count += 1


increase()
increase()
print(count)
# 輸出：2。
# 說明：global 讓此函式中的 count 指向模組全域名稱，而非建立同名局部名稱。
```

只讀全域值不需要 `global`。修改全域名稱指向的列表內容，也不等於重新繫結全域名稱：

```python
records = []


def add_record():
    records.append("完成")


add_record()
print(records)
# 輸出：['完成']。
# 說明：append 修改共享列表，沒有把 records 這個名稱重新指向另一個物件，所以不需 global。
```

`global` 可用，但會增加隱藏資料流。若只是計算新數量，`new_count = increase(old_count)` 這種參數與返回值通常更容易理解。

### 3.2 `nonlocal`：重新繫結外層函式名稱

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
# 輸出：11；12。
# 說明：nonlocal 指向最近一層已存在的外層函式 count，讓兩次呼叫持續更新同一份狀態。
```

`nonlocal` 的名稱必須已存在於外層函式作用域，不能直接指向模組全域。它用於**重新繫結名稱**；若只讀外層值、或只對外層列表 `append()`，不需要它。

**練習：**說明 `count += 1` 與 `records.append(...)` 的差別。

**自查簡答：**前者對名稱進行賦值，後者修改物件。若要重新繫結外層函式的整數名稱，用 `nonlocal`；重新繫結模組名稱，用 `global`。

## 4. 函式也是物件：能保存、傳入與返回

### 4.1 保存函式，不要提前呼叫

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


operations = {"+": add, "*": multiply}
operation = operations["*"]
print(operation(4, 5))
print(operations["+"](4, 5))
# 輸出：20；9。
# 說明：字典保存函式物件；取出後加括號才呼叫。也可將函式保存於列表中逐一呼叫。
```

若寫 `operations = {"+": add(4, 5)}`，保存的是已計算的 9，不再是可供下次呼叫的函式。

### 4.2 函式作為參數與返回值

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
# 輸出：5。
# 說明：choose_operation 返回函式；apply_operation 接收並呼叫它；都沒有複製函式本體。
```

函式能像資料一樣使用，常稱為「一等物件」。理解這件事後，裝飾器就只是函式接收與返回的進一步應用。

## 5. 閉包：函式保留對外層變數的存取

閉包可理解為：內層函式使用了外層函式的局部變數，並保留對那些變數的存取。**外層返回內層函式是常見用法，不是必要條件。**關鍵是引用外層函式的變數；只有讀取模組全域值，不算本章討論的這種閉包。

### 5.1 建立帶不同配置的函式

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply


times_two = make_multiplier(2)
times_three = make_multiplier(3)
print(times_two(5))
print(times_three(5))
# 輸出：10；15。
# 說明：兩次建立的 multiply 分別保留對各自 factor 的存取；外層結束後仍可使用。
```

追蹤 `times_two = make_multiplier(2)`：外層本次形參 `factor=2` → 建立使用它的 `multiply` → 返回函式物件 → `times_two` 指向它。直到 `times_two(5)` 才執行乘法。

### 5.2 沒有 `return inner` 也能形成閉包

```python
callbacks = []


def register_message(message):
    def show():
        print(message)

    callbacks.append(show)


register_message("稍後顯示")
callbacks[0]()
# 輸出：稍後顯示。
# 說明：show 透過列表保存，而非由外層返回；它仍引用外層局部 message，所以仍是閉包。
```

### 5.3 保存的是變數關係，不是永遠固定的照片

```python
def make_reader():
    value = 1

    def read():
        return value

    value = 2
    return read


reader = make_reader()
print(reader())
# 輸出：2。
# 說明：read 在呼叫時讀取所保留的 value；建立 read 當下的 1 並沒有被永久拍成快照。
```

這也解釋了迴圈建立多個閉包時常見的「全部讀到最後一個值」。需要每次獨立配置，可以每次呼叫一個工廠函式；不要靠死背特殊寫法掩蓋繫結規則。

**練習：**寫 `make_power(exponent)`，返回計算某數字次方的函式；再建兩個獨立計數器。

**自查簡答：**`square = make_power(2)` 取得函式，`square(3)` 才返回 9。每次呼叫 `make_counter()` 建一份新外層狀態，兩個計數器不必共享數量。

## 6. 裝飾器：先手動包裝，再看 `@`

假設很多函式都要印「開始／結束」，直接在每個函式內重複寫兩行，日後要改格式就得改很多處。包裝函式能把共同步驟集中。

此處的**函式式裝飾器**接收原函式，返回一個包裝函式。這是最常見的學習形式；Python 裝飾器不被限制為一定要返回新函式，也不只這一種實作。

### 6.1 最小例：先不使用 `@`

```python
def log_call(func):
    def wrapper():
        print("開始")
        result = func()
        print("結束")
        return result

    return wrapper


def say_hello():
    print("Hello")
    return "完成"


say_hello = log_call(say_hello)
print(say_hello())
# 輸出：開始；Hello；結束；完成。
# 說明：log_call 返回 wrapper；名字 say_hello 改指向 wrapper，原函式由閉包中的 func 保存。
```

兩個 `return` 職責不同：

| 位置 | 返回什麼 | 何時執行 |
|---|---|---|
| 外層 `return wrapper` | 函式物件 | 裝飾時 |
| 內層 `return result` | 原函式的結果 | 每次呼叫包裝器時 |

不要寫 `return wrapper()`：那會在裝飾時直接執行包裝器，把它的執行結果交出去。先能手動追蹤這段，才繼續下一節。

### 6.2 `@` 把重新賦值寫得更靠近定義

```python
def log_call(func):
    def wrapper():
        print("開始")
        result = func()
        print("結束")
        return result

    return wrapper


@log_call
def say_hello():
    print("Hello")
    return "完成"


print(say_hello())
# 輸出：開始；Hello；結束；完成。
# 說明：在此簡單情況，@log_call 可用定義後 say_hello = log_call(say_hello) 理解。
```

裝飾在執行到這段定義時發生，不是每次呼叫都重新包裝。若定義本身位於另一個函式或迴圈內，每次再次執行定義仍會再次裝飾。使用 `@` 後不要再手動包一次，否則會重複增加行為。

### 6.3 定義時也可能有畫面輸出

```python
def announce(func):
    print("正在裝飾")

    def wrapper():
        print("正在呼叫")
        return func()

    return wrapper


@announce
def work():
    print("原函式工作")


print("定義已完成")
work()
# 輸出：正在裝飾；定義已完成；正在呼叫；原函式工作。
# 說明：裝飾器本體在定義階段執行；wrapper 本體在之後呼叫階段執行。
```

所以不能說「只有呼叫被裝飾函式才會出現裝飾器效果」。先確認程式碼是在外層裝飾函式，還是在內層包裝函式。

## 7. 常用包裝器的三個契約

若你的目標是「增加日誌但保留原功能」，通常要保留：參數如何傳入、返回值如何傳出、例外如何傳遞。

### 7.1 接收後再原樣轉交

```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print("開始")
        result = func(*args, **kwargs)
        print("完成")
        return result

    return wrapper


@log_call
def add(a, b=0):
    return a + b


print(add(2, b=3))
# 輸出：開始；完成；5。
# 說明：wrapper 收集 args=(2,) 和 kwargs={'b': 3}，呼叫 func 時再展開轉交。
```

這是常用的同步函式包裝形式，不是「自動相容一切函式」的保證；生成器或非同步函式還有其他行為需要考慮，本章先不展開。

### 7.2 忘記內層 `return`，結果會丟失

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)

    return wrapper


@bad_decorator
def get_number():
    return 42


print(get_number())
# 輸出：None。
# 說明：這是可執行的錯誤設計示範；原函式返回 42，但 wrapper 沒有把它返回給呼叫者。
```

### 7.3 需要出錯也執行的後置動作，用 `finally`

```python
def trace(func):
    def wrapper(*args, **kwargs):
        print("進入")
        try:
            return func(*args, **kwargs)
        finally:
            print("離開")

    return wrapper


@trace
def divide(a, b):
    return a / b


try:
    divide(10, 0)
except ZeroDivisionError:
    print("呼叫者收到除零錯誤")
# 輸出：進入；離開；呼叫者收到除零錯誤。
# 說明：finally 在一般返回或例外離開時執行；此包裝器沒有吞掉原函式的例外。
```

把後置 `print()` 放在 `func()` 後面，只在原函式正常返回時走得到。`finally` 的完整語法在例外章學習，這裡只要能看出「正常結束後做」與「離開時做」不同。

## 8. `wraps` 保留資訊，不會替你執行原函式

包裝之後，外部名稱指向 `wrapper`，預設的 `__name__`、`__doc__` 也會變成包裝器資訊。`functools.wraps` 把原函式的重要中繼資料複製過去，並設定 `__wrapped__` 供工具追溯。

```python
from functools import wraps


def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("開始")
        return func(*args, **kwargs)

    return wrapper


@log_call
def greet(name):
    """產生問候文字。"""
    return f"你好，{name}"


print(greet.__name__)
print(greet.__doc__)
print(greet("小明"))
# 輸出：greet；產生問候文字。；開始；你好，小明。
# 說明：wraps 保留名稱與文件；真正執行原函式的是 func(*args, **kwargs)。
```

`wraps` 不會把包裝器變回原函式，不會自動轉交參數，也不會補上你忘記寫的 `return`。沒有 `wraps`，功能可能照常運作，但除錯與文件工具看到的資訊較難理解。

第 230 集的「實時模板」是 IDE 幫你快速插入程式骨架的便利功能，不是新的 Python 語法。可以把常用裝飾器骨架存成範本，但每次仍要確認參數、返回值和例外是否符合需求。先能徒手解釋骨架，再使用模板節省打字。第 233 集重複多版本共存主題，可回第二章的版本環境說明查閱。

## 9. 有參裝飾器：先配置，再包裝，再呼叫

想指定日誌標籤，不應每次呼叫原函式都多傳一個與業務無關的參數。可以先呼叫工廠建立一個記住標籤的裝飾器。

```python
from functools import wraps


def tagged(label):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"[{label}] 開始")
            return func(*args, **kwargs)

        return wrapper

    return decorator


@tagged("學習")
def add(a, b):
    return a + b


print(add(2, 3))
# 輸出：[學習] 開始；5。
# 說明：tagged 接配置，decorator 接原函式，wrapper 接本次原函式所需參數。
```

這個常見寫法有三層，因為有三次不同目的的呼叫：

```text
tagged("學習")        → 得到 decorator
decorator(原始 add)  → 得到 wrapper
wrapper(2, 3)       → 執行包裝與加法，得到 5
```

三層是容易學習的實作形式，不是有參裝飾器唯一合法的寫法。配置也可以是重複次數、角色名稱、日誌等級等。

### 9.1 重複執行時，先定義次數與結果規則

```python
from functools import wraps


def repeat(times):
    if type(times) is not int or times < 1:
        raise ValueError("times 必須是正整數")

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
# 輸出：Hi；Hi；Hi；完成。
# 說明：times 在配置時驗證；每次呼叫重複三次，約定返回最後一次結果；遇到例外直接向外傳遞。
```

重複執行是增加副作用，不可隨意拿來包裝扣款、扣庫存等動作。此例適合問候等可重複的示範。重試與重複不是同一件事，重試只在指定失敗後再次嘗試，附錄另作說明。

## 10. 疊加：求值、套用、呼叫是三種順序

看到 `@A`、`@B` 不要只背一句「由下往上」。需要分三件事：

1. **裝飾器表達式求值：由上到下。**例如先計算 `make_trace("A")`，再計算 `make_trace("B")`。
2. **將裝飾器套用到函式：由下到上。**先由 B 包住原函式，再由 A 包住結果。
3. **實際呼叫：按包裝器程式碼執行。**常見情況先進 A，再進 B，再原函式，然後 B、A 返回；若某層不呼叫下一層，就會在該處停止。

```python
from functools import wraps


def make_trace(label):
    print(f"求值 {label}")

    def decorator(func):
        print(f"套用 {label}")

        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"進入 {label}")
            result = func(*args, **kwargs)
            print(f"離開 {label}")
            return result

        return wrapper

    return decorator


@make_trace("A")
@make_trace("B")
def work():
    print("原函式")


print("準備呼叫")
work()
# 輸出：求值 A；求值 B；套用 B；套用 A；準備呼叫；進入 A；進入 B；原函式；離開 B；離開 A。
# 說明：表達式求值和裝飾器套用方向不同；呼叫順序由 wrapper 是否呼叫下一層決定。
```

在裝飾器值 `A`、`B` 已取得後，最終包裝關係可用 `work = A(B(原始 work))` 理解。但若原本寫的是有副作用的裝飾器表達式，不能只看巢狀呼叫就忽略其求值時機。

順序有實際影響：日誌在權限檢查外層，通常連未授權嘗試也能記錄；權限檢查若先擋住，內層日誌不一定執行。不要把「裝飾器越多」當作設計越好。

## 11. 綜合例：計時但保留返回值與例外

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
            print(f"{func.__name__} 耗時 {elapsed:.6f} 秒")

    return wrapper


@timer
def calculate_total(numbers):
    return sum(numbers)


print(calculate_total(range(5)))
# 輸出：先顯示 calculate_total 耗時若干秒，再顯示 10。
# 說明：耗時依電腦與當次執行變化；finally 記錄離開時間，原返回值和例外仍向外傳遞。
```

原函式只負責加總，計時放在共同包裝層。若只是單一流程計時一次，普通前後記錄也可以，不必強行裝飾器化。

## 12. 常見錯誤與修正

| 問題 | 原因 | 修正 |
|---|---|---|
| `UnboundLocalError` | 局部賦值前讀取 | 分清作用域，選參數／返回值或合適宣告 |
| 以為閉包必須 `return inner` | 把常見用法當定義 | 檢查是否保留對外層函式變數的存取 |
| 使用 `global` 才能 append | 混淆名稱與物件 | append 是修改物件，非重新繫結 |
| 裝飾後變成 `None` | 外層沒返回包裝器 | 檢查 `return wrapper` |
| 原結果變成 `None` | 內層沒返回結果 | 檢查 `return func(...)` 或 `return result` |
| 定義時意外執行原函式 | 外層呼叫 `func()` 或返回 `wrapper()` | 分清裝飾時與呼叫時 |
| 帶參數函式不能用 | wrapper 沒接收或沒轉交 | 接收與展開都要對上 |
| 加了 wraps 還不執行 | 誤把 wraps 當委派呼叫 | 原函式仍需 `func(...)` |
| 呼叫順序與預期不同 | 混淆求值、套用與執行 | 分三階段逐行追蹤 |

## 13. 練習與自查簡答

1. **名稱查找：**自己寫 L、E、G 同名的例子，逐一刪去內層繫結並預測輸出。
2. **閉包：**寫 `make_counter(start)`，建立從 0 和從 100 起算的兩個計數器；交替呼叫，確認互不干擾。
3. **日誌：**寫支援位置和關鍵字參數的裝飾器，顯示函式名與結果；不要隨意記錄密碼等資料。
4. **配置：**寫 `prefix_log(label)`，用不同標籤裝飾兩個函式。
5. **權限練習：**寫 `require_role("admin")`，接收含 `role` 的使用者字典；符合才呼叫原函式。不符合時的返回規則必須說清楚。這只是邏輯練習，不是實際身分驗證系統。
6. **疊加：**交換兩個有輸出提示的裝飾器，分別寫出求值、套用、呼叫順序，不只抄最後畫面。

自查簡答：

- LEGB 的 E 指誰？程式碼中包住當前函式的外層函式，不是呼叫者。
- `nonlocal` 能用來直接改模組全域名稱嗎？不能，外層函式需已有該名稱。
- 閉包是否保存數值快照？一般是保留變數繫結的存取；呼叫時值可能已變。
- 兩個 `return` 各交還什麼？外層交還包裝函式，內層交還一次業務執行的結果。
- `wraps` 是否會呼叫原函式？不會，它處理中繼資料與可追溯資訊。
- 裝飾器是否必須三層？有配置的函式工廠常用三層，但不是唯一合法形式。
- 疊加哪個先發生？表達式由上到下求值；套用由下到上；實際呼叫按包裝器內容。

達標標準：不用看模板，也能說清楚每個變數指向哪個物件、每個括號是在哪一階段呼叫、每個返回值交給誰。能做到這三點，比背熟裝飾器形式更重要。
