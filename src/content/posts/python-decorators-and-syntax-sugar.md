---
title: Python 裝飾器與語法糖詳盡筆記
published: 2026-09-14
updated: 2026-09-14
description: 從函式是一等對象、閉包到裝飾器與語法糖，逐步理解 Python 的包裝模型、參數傳遞、執行順序和實用設計模式。
tags: [Python, 裝飾器, 閉包, 語法糖]
category: Python 學習
draft: false
lang: zh_TW
---

> 本筆記承接第 196–241 集，專門整理：函式作為對象、閉包、裝飾器、@ 語法糖、裝飾器工廠、有參裝飾器、裝飾器疊加，以及常見 Python 語法糖。

> **代碼示例閱讀規則**：每個主要 Python 代碼塊都包含輸出和說明注釋。建議先根據代碼預測輸出，再執行驗證。

---

## 1. 學習目標

完成本筆記後，應能夠：

- 說明函式為什麼可以被賦值、傳遞和返回。
- 從零寫出一個最基本的裝飾器。
- 理解 @decorator 與手動重新賦值的等價關係。
- 寫出能接收任意參數並保留返回值的通用裝飾器。
- 使用 functools.wraps 保留原函式的名稱和文件說明。
- 寫出有參裝飾器，理解三層函式的結構。
- 判斷多個裝飾器的包裝與執行順序。
- 寫出計時、日誌、權限、快取、重試等實用裝飾器。
- 理解常見 Python 語法糖，知道它們背後展開成什麼形式。
- 避免丟失參數、丟失返回值、元資料被覆蓋和重複執行等錯誤。

---

## 2. 函式是一等對象

裝飾器的基礎不是特殊語法，而是 Python 將函式視為普通對象。函式可以：

1. 賦值給另一個名稱。
2. 放入列表、字典或元組。
3. 作為參數傳給另一個函式。
4. 作為另一個函式的返回值。

### 2.1 函式賦值

~~~python
def greet(name):
    return f"你好，{name}"


say_hello = greet
print(say_hello("小明"))
# 輸出：你好，小明
# 說明：greet 和 say_hello 指向同一個函式對象；賦值不會複製函式。
~~~

### 2.2 函式作為參數

~~~python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


def apply_operation(operation, a, b):
    return operation(a, b)


print(apply_operation(add, 2, 3))
print(apply_operation(multiply, 2, 3))
# 輸出：依次為 5、6
# 說明：apply_operation 不需要知道 operation 的具體實作，只要它可以被呼叫即可。
~~~

### 2.3 函式作為返回值

~~~python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


def choose_operation(operator):
    if operator == "+":
        return add
    if operator == "*":
        return multiply
    return None


operation = choose_operation("*")
print(operation(4, 5))
# 輸出：20
# 說明：choose_operation 返回一個函式；operation(4, 5) 才是執行被返回的函式。
~~~

這三種能力共同構成裝飾器的基礎：裝飾器接收一個函式，建立一個新函式，再把新函式返回。

---

## 3. 閉包：裝飾器的前置知識

閉包是指：內層函式引用外層函式的局部變數，並且外層函式返回內層函式。即使外層函式已經執行完，內層函式仍能記住被引用的外層資料。

~~~python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply


times_two = make_multiplier(2)
times_three = make_multiplier(3)

print(times_two(5))
print(times_three(5))
# 輸出：依次為 10、15
# 說明：times_two 記住 factor == 2；times_three 記住 factor == 3。兩個函式保存的是不同外層環境。
~~~

### 3.1 閉包保存可變狀態

~~~python
def make_counter(start=0):
    count = start

    def count_up():
        nonlocal count
        count += 1
        return count

    return count_up


counter = make_counter(10)
print(counter())
print(counter())
# 輸出：依次為 11、12
# 說明：nonlocal 允許內層函式修改外層的 count；狀態會在多次呼叫之間保留。
~~~

裝飾器中的 wrapper 正是這種內層函式：它引用外層的 func，因此能在之後呼叫原函式。

---

## 4. 裝飾器到底是什麼？

裝飾器是一個符合以下形式的函式：

~~~text
接收原函式 → 建立包裝函式 → 返回包裝函式
~~~

抽象表示：

~~~python
def decorator(func):
    def wrapper(*args, **kwargs):
        # 前置操作
        result = func(*args, **kwargs)
        # 後置操作
        return result

    return wrapper
# 輸出：decorator 接收一個函式，最後返回 wrapper。
# 說明：真正的裝飾效果會在 wrapper 被呼叫時發生。
~~~

裝飾器不一定要叫 decorator，包裝函式也不一定要叫 wrapper；這些只是命名慣例。

### 4.1 最小可用裝飾器

~~~python
def log_call(func):
    def wrapper():
        print("函式開始")
        result = func()
        print("函式結束")
        return result

    return wrapper


def say_hello():
    print("Hello")


say_hello = log_call(say_hello)
say_hello()
# 輸出：依次為 函式開始、Hello、函式結束
# 說明：原本的 say_hello 被替換成 wrapper；wrapper 在原函式前後增加了兩次列印。
~~~

### 4.2 裝飾器的關鍵位置

~~~python
def log_call(func):                 # 1. 接收原函式
    def wrapper(*args, **kwargs):   # 2. 接收原函式的所有參數
        print("開始")                # 3. 加入前置行為
        result = func(*args, **kwargs)
        print("結束")                # 4. 加入後置行為
        return result                # 5. 返回原函式結果

    return wrapper                  # 6. 返回包裝函式
# 輸出：一個可重用的通用裝飾器。
# 說明：忘記任一個 return 或參數轉交，都可能破壞原函式的介面。
~~~

常見失誤：

- 忘記 return wrapper：被裝飾的名稱可能變成 None。
- 忘記 *args、**kwargs：帶參數的原函式無法被呼叫。
- 忘記 return result：原函式的返回值會丟失。
- 在 wrapper 外呼叫 func：原函式可能在裝飾階段就提前執行。

---

## 5. @ 是什麼語法糖？

@decorator 是裝飾器的語法糖，沒有引入另一種底層機制。

### 5.1 手動寫法

~~~python
def log_call(func):
    def wrapper():
        print("開始")
        result = func()
        print("結束")
        return result

    return wrapper


def work():
    print("工作中")


work = log_call(work)
work()
# 輸出：依次為 開始、工作中、結束
# 說明：先建立 work，再手動把 work 傳給 log_call，最後用返回的 wrapper 覆蓋 work。
~~~

### 5.2 @ 寫法

~~~python
def log_call(func):
    def wrapper():
        print("開始")
        result = func()
        print("結束")
        return result

    return wrapper


@log_call
def work():
    print("工作中")


work()
# 輸出：依次為 開始、工作中、結束
# 說明：Python 在完成 work 定義後，自動執行 work = log_call(work)。
~~~

### 5.3 精確展開時機

下面先用可執行示例觀察「裝飾階段」和「呼叫階段」：

~~~python
def decorator(func):
    print("執行裝飾器")

    def wrapper():
        print("執行包裝函式")
        return func()

    return wrapper


@decorator
def function():
    print("執行原函式")


print("定義完成")
function()
# 輸出：執行裝飾器、定義完成、執行包裝函式、執行原函式
# 說明：decorator(function) 在定義完成時執行一次；wrapper 在每次呼叫 function() 時執行。
~~~

其中 @decorator 的核心等價展開是：

~~~python
def function():
    pass


function = decorator(function)
# 輸出：假設 decorator 已定義，這一行會在定義階段完成包裝。
# 說明：這是等價展開式，用於理解語法，不是完整的獨立示例。
~~~

因此：

- 裝飾器接收原函式的那一層通常執行一次。
- 包裝器內的程式則每次呼叫都執行。
- 如果模組被重新載入，定義階段可能再次執行。

---

## 6. 通用裝飾器：支援任意參數與返回值

最初的 wrapper 只能包裝沒有參數的函式。通用裝飾器使用 *args 和 **kwargs 轉交所有位置與關鍵字參數。

~~~python
from functools import wraps


def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"開始執行：{func.__name__}")
        result = func(*args, **kwargs)
        print(f"執行結果：{result}")
        return result

    return wrapper


@log_call
def add(a, b):
    return a + b


print(add(2, 3))
# 輸出：開始執行：add、執行結果：5、5
# 說明：wrapper 收到 (2, 3)，再透過 func(*args, **kwargs) 還原成 add(2, 3)。
~~~

### 6.1 為什麼要返回結果？

錯誤示例：

~~~python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)

    return wrapper


@bad_decorator
def get_number():
    return 42


print(get_number())
# 輸出：None
# 說明：原函式返回 42，但 wrapper 沒有 return，因此返回值被丟失。
~~~

正確示例：

~~~python
def good_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result

    return wrapper


@good_decorator
def get_number():
    return 42


print(get_number())
# 輸出：42
# 說明：包裝器保留並返回原函式結果，因此裝飾前後的函式介面一致。
~~~

### 6.2 參數、返回值和例外要透明

品質良好的通用裝飾器通常遵循：

~~~text
接收什麼參數，就原樣轉交什麼參數；
原函式返回什麼，就返回什麼；
原函式拋出的例外，不要無意中吞掉。
~~~

如果裝飾器確實要攔截例外，應明確記錄、轉換或重新拋出，而不是默默忽略。

---

## 7. functools.wraps：保留原函式元資料

裝飾後，名稱 func 實際上指向 wrapper。如果不處理，__name__、__doc__ 等資訊可能變成包裝器的資料。

~~~python
def simple_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper


@simple_decorator
def greet():
    """顯示問候語。"""
    return "Hello"


print(greet.__name__)
print(greet.__doc__)
# 輸出：依次為 wrapper、None
# 說明：沒有 wraps 時，外部看到的是 wrapper 的名稱和文件字串。
~~~

使用 functools.wraps：

~~~python
from functools import wraps


def better_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper


@better_decorator
def greet():
    """顯示問候語。"""
    return "Hello"


print(greet.__name__)
print(greet.__doc__)
# 輸出：依次為 greet、顯示問候語。
# 說明：wraps 把原函式的重要元資料複製到 wrapper，方便除錯、文件生成和測試工具使用。
~~~

wraps 不會複製函式的執行邏輯；它主要修正包裝後可見的元資料，原函式仍由 wrapper 呼叫。

---

## 8. 前置、後置與環繞型裝飾器

### 8.1 前置操作

~~~python
from functools import wraps


def require_positive(func):
    @wraps(func)
    def wrapper(number):
        if number <= 0:
            return "輸入必須大於 0"
        return func(number)

    return wrapper


@require_positive
def square(number):
    return number * number


print(square(-2))
print(square(3))
# 輸出：依次為 輸入必須大於 0、9
# 說明：不符合條件時不執行原函式；符合條件時才把 number 傳入 square。
~~~

### 8.2 後置操作

~~~python
from functools import wraps


def announce_result(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"結果是：{result}")
        return result

    return wrapper


@announce_result
def add(a, b):
    return a + b


print(add(2, 3))
# 輸出：結果是：5，接著列印 5
# 說明：wrapper 先取得原函式結果，再執行後置列印，最後仍返回 5。
~~~

### 8.3 環繞型操作

環繞型裝飾器在原函式前後都執行程式碼，常用於計時、日誌、鎖和交易流程。

~~~python
from functools import wraps


def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("進入函式")
        try:
            return func(*args, **kwargs)
        finally:
            print("離開函式")

    return wrapper


@trace
def work():
    print("工作中")
    return "完成"


print(work())
# 輸出：進入函式、工作中、離開函式、完成
# 說明：finally 無論原函式正常返回或拋出例外，都會執行清理操作。
~~~

---

## 9. 有參裝飾器：裝飾器工廠

普通裝飾器接收原函式；有參裝飾器先接收配置，再返回一個真正接收原函式的裝飾器。因此通常有三層函式：

~~~text
最外層：接收裝飾器配置
中間層：接收原函式
最內層：接收原函式參數並執行包裝邏輯
~~~

### 9.1 重複執行裝飾器

~~~python
from functools import wraps


def repeat(times):
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


say_hi()
# 輸出：連續列印三次 Hi
# 說明：repeat(3) → decorator → wrapper；3 是被閉包保存的配置。
~~~

語法糖的精確展開是：

~~~python
say_hi = repeat(3)(say_hi)
# 輸出：repeat(3) 先返回 decorator，decorator(say_hi) 再返回 wrapper。
# 說明：外層括號傳配置，內層括號傳原函式；這就是有參裝飾器需要三層結構的原因。
~~~

### 9.2 權限裝飾器

~~~python
from functools import wraps


def require_role(required_role):
    def decorator(func):
        @wraps(func)
        def wrapper(user, *args, **kwargs):
            if user.get("role") != required_role:
                return "權限不足"
            return func(user, *args, **kwargs)

        return wrapper

    return decorator


@require_role("admin")
def delete_user(user, username):
    return f"已刪除 {username}"


admin = {"name": "Alice", "role": "admin"}
guest = {"name": "Bob", "role": "guest"}
print(delete_user(admin, "Tom"))
print(delete_user(guest, "Tom"))
# 輸出：依次為 已刪除 Tom、權限不足
# 說明：required_role 由外層閉包保存；每次呼叫只需傳入當前 user。
~~~

### 9.3 重試裝飾器的設計提醒

重試裝飾器應明確指定：

- 哪些例外可以重試。
- 最多重試幾次。
- 每次重試是否需要等待。
- 最後一次失敗時返回什麼或重新拋出什麼。

不要對所有例外無條件重試；輸入錯誤、權限錯誤等通常重試也不會成功。

---

## 10. 裝飾器疊加與執行順序

### 10.1 等價展開

~~~python
@decorator_a
@decorator_b
def work():
    print("原函式")
# 輸出：此處是語法結構示意，假設 decorator_a 和 decorator_b 已定義。
# 說明：最靠近 def 的 decorator_b 先包裝原函式。
~~~

等價展開為：

~~~python
work = decorator_a(decorator_b(work))
# 輸出：先得到 decorator_b(work)，再把結果傳給 decorator_a。
# 說明：這是等價展開式；實際程式中不應在使用 @ 後再手動包裝一次。
~~~

### 10.2 呼叫順序

~~~python
from functools import wraps


def outer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("outer before")
        result = func(*args, **kwargs)
        print("outer after")
        return result

    return wrapper


def inner(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("inner before")
        result = func(*args, **kwargs)
        print("inner after")
        return result

    return wrapper


@outer
@inner
def work():
    print("work")


work()
# 輸出：outer before、inner before、work、inner after、outer after
# 說明：outer 是最外層包裝器，因此最先進入、最後離開。
~~~

### 10.3 順序設計問題

裝飾器順序會影響結果，例如：

- 先驗證權限，再記錄日誌：未授權請求是否應被記錄？
- 先快取，再計時：計時的是快取查找，還是實際計算？
- 先重試，再記錄：每次嘗試記錄一次，還是整體流程記錄一次？

多裝飾器不是越多越好，應先寫清楚各層責任和順序。

---

## 11. 實用裝飾器模式

### 11.1 計時裝飾器

~~~python
import time
from functools import wraps


def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} 耗時 {elapsed:.6f} 秒")
        return result

    return wrapper


@timer
def calculate_total(numbers):
    return sum(numbers)


print(calculate_total(range(5)))
# 輸出：先列印類似 calculate_total 耗時 0.00000x 秒，再列印 10
# 說明：耗時會因電腦而變化；sum(range(5)) 的固定結果是 10。
~~~

### 11.2 日誌裝飾器

~~~python
from functools import wraps


def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"呼叫 {func.__name__}，位置參數={args}，關鍵字參數={kwargs}")
        result = func(*args, **kwargs)
        print(f"返回 {result}")
        return result

    return wrapper


@log_call
def add(a, b):
    return a + b


add(2, 3)
# 輸出：呼叫 add，位置參數=(2, 3)，關鍵字參數={}；返回 5
# 說明：記錄參數時要注意敏感資料，真實系統不要把密碼、令牌等寫入日誌。
~~~

### 11.3 快取裝飾器

標準庫 functools.lru_cache 是現成裝飾器：

~~~python
from functools import lru_cache


@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)


print(fibonacci(10))
# 輸出：55
# 說明：相同參數的結果會被快取；參數必須是可雜湊的，列表通常不能直接作為快取鍵。
~~~

### 11.4 內置裝飾器

本節涉及類別，屬於延伸閱讀。如果尚未學習面向對象，可先理解用途，等學完類別後再回來實作。

#### staticmethod

靜態方法不會自動接收 self 或 cls，適合與類別有關但不依賴實例狀態的工具函式。

~~~python
class MathTools:
    @staticmethod
    def add(a, b):
        return a + b


print(MathTools.add(2, 3))
# 輸出：5
# 說明：呼叫 staticmethod 時不需要建立 MathTools 實例，也不會自動傳入 self。
~~~

#### classmethod

類別方法會自動接收類別對象 cls，常用於替代建構方式或操作類別層級資料。

~~~python
class User:
    count = 0

    @classmethod
    def current_count(cls):
        return cls.count


print(User.current_count())
# 輸出：0
# 說明：current_count 自動收到 User 作為 cls，可讀取類別屬性 count。
~~~

#### property

property 讓方法可以像屬性一樣讀取，適合在不改變外部使用方式的情況下加入計算或驗證。

~~~python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height


rectangle = Rectangle(3, 4)
print(rectangle.area)
# 輸出：12
# 說明：area 的本體是方法，但呼叫時使用 rectangle.area，不需要寫括號。
~~~

---

## 12. 常見 Python 語法糖

語法糖是讓程式更簡潔易讀的寫法，通常可以用較基本的語法表達同一件事。語法糖不代表任何簡寫都更好；可讀性優先。

### 12.1 增量賦值

~~~python
count = 1
count += 2
print(count)
# 輸出：3
# 說明：count += 2 大致等價於 count = count + 2；對可變對象時底層行為可能不同。
~~~

常見形式：-=、*=、/=、//=、%=、**=。

### 12.2 條件表達式

~~~python
score = 85
result = "及格" if score >= 60 else "不及格"
print(result)
# 輸出：及格
# 說明：A if condition else B 等價於簡短的 if/else 賦值，不適合承載複雜邏輯。
~~~

### 12.3 序列解包

~~~python
point = (10, 20)
x, y = point
print(x, y)
# 輸出：10 20
# 說明：左側名稱數量必須和右側元素數量匹配，否則會產生 ValueError。
~~~

使用星號接收剩餘元素：

~~~python
head, *middle, tail = [1, 2, 3, 4, 5]
print(head, middle, tail)
# 輸出：1 [2, 3, 4] 5
# 說明：*middle 收集剩餘元素；同一個解包賦值中只能出現一次。
~~~

### 12.4 交換變數

~~~python
a = 1
b = 2
a, b = b, a
print(a, b)
# 輸出：2 1
# 說明：Python 先建立右側元組，再一次性解包，不需要臨時變數。
~~~

### 12.5 列表生成式

~~~python
squares = [number * number for number in range(5)]
print(squares)
# 輸出：[0, 1, 4, 9, 16]
# 說明：列表生成式等價於建立空列表、迴圈 append；適合簡單轉換。
~~~

帶條件的列表生成式：

~~~python
evens = [number for number in range(10) if number % 2 == 0]
print(evens)
# 輸出：[0, 2, 4, 6, 8]
# 說明：只有滿足 if 條件的元素才會加入結果列表。
~~~

### 12.6 字典與集合生成式

~~~python
lengths = {word: len(word) for word in ["Python", "Go"]}
unique_lengths = {len(word) for word in ["cat", "dog", "bird"]}
print(lengths)
print(unique_lengths)
# 輸出：{'Python': 6, 'Go': 2}，以及 {3, 4}
# 說明：字典生成式產生鍵值對；集合生成式會自動去重，集合列印順序不應依賴。
~~~

### 12.7 生成器表達式

~~~python
numbers = (number * number for number in range(5))
print(next(numbers))
print(next(numbers))
# 輸出：依次為 0、1
# 說明：圓括號建立生成器，不會立即建立完整列表；每次 next() 才產生下一個值。
~~~

### 12.8 any() 和 all()

~~~python
scores = [80, 70, 90]
print(all(score >= 60 for score in scores))
print(any(score < 60 for score in scores))
# 輸出：True、False
# 說明：all() 要求全部為真；any() 只要有一個為真。生成器表達式避免先建立中間列表。
~~~

### 12.9 with 語法糖與上下文管理

~~~python
with open("data.txt", "w", encoding="utf-8") as file:
    file.write("Hello")
# 輸出：data.txt 被寫入 Hello，離開 with 區塊後檔案會自動關閉
# 說明：with 會在進入時取得資源，離開時自動執行清理，等價於 try/finally 的資源管理模式。
~~~

### 12.10 裝飾器語法糖

假設前文的 timer 已定義：

~~~python
@timer
def calculate():
    return 1
# 輸出：定義完成後 calculate 已指向 timer 返回的包裝函式。
# 說明：這段用於展示語法結構；實際輸出取決於 timer 的實作。
~~~

等價於：

~~~python
def calculate():
    return 1


calculate = timer(calculate)
# 輸出：calculate 定義完成後立即被 timer 包裝；呼叫 calculate() 時執行包裝器。
# 說明：@ 語法改變的是寫法，不是底層執行模型。
~~~

---

## 13. 裝飾器的設計檢查清單

每次寫裝飾器，都可以逐項檢查：

- [ ] 是否真的需要裝飾器？若只呼叫一次，普通函式可能更清楚。
- [ ] 裝飾器是否只負責一件橫切功能？
- [ ] 是否使用 wrapper(*args, **kwargs) 支援任意參數？
- [ ] 是否返回原函式結果？
- [ ] 是否讓原函式的例外按預期傳遞？
- [ ] 是否使用 @wraps(func) 保存元資料？
- [ ] 有參裝飾器是否正確分成「配置 → 原函式 → 呼叫」三層？
- [ ] 多個裝飾器的先後順序是否經過設計？
- [ ] 是否避免把敏感資料寫入日誌？
- [ ] 裝飾器是否會意外改變函式的參數、返回值或副作用？

---

## 14. 常見錯誤對照表

| 錯誤 | 常見結果 | 修正方式 |
|---|---|---|
| 忘記 return wrapper | 被裝飾名稱變成 None | 裝飾器最後返回包裝函式 |
| wrapper 沒有 *args、**kwargs | 帶參數函式無法呼叫 | 原樣接收並轉交參數 |
| wrapper 沒有 return result | 原函式結果變成 None | 保存並返回結果 |
| 沒有使用 wraps | 函式名稱和文件字串變成 wrapper | 使用 @wraps(func) |
| 在 wrapper 外呼叫 func() | 裝飾定義時就提前執行原函式 | 把呼叫放在 wrapper 內 |
| 裝飾器疊加順序錯誤 | 日誌、權限、快取行為不符合預期 | 展開成巢狀函式檢查 |
| 無條件攔截所有例外 | 真正錯誤被吞掉 | 只處理可預期例外，必要時重新拋出 |
| 日誌記錄全部參數 | 密碼、令牌等敏感資料洩漏 | 脫敏或不記錄敏感欄位 |

---

## 15. 綜合練習

### 練習 1：執行時間與日誌

寫兩個裝飾器：

1. timer：打印執行時間。
2. log_call：打印函式名稱和返回值。

把兩者疊加到一個計算函式上，分別交換裝飾器順序，觀察輸出差異。

### 練習 2：權限檢查

寫有參裝飾器 require_role("admin")：

- 使用者角色符合時才執行原函式。
- 不符合時返回「權限不足」。
- 不要把密碼寫入日誌。

### 練習 3：重試機制

寫 retry(times=3)：

- 原函式成功時立即返回。
- 發生指定例外時才重試。
- 超過次數後保留最後一次錯誤。

### 練習 4：用語法糖重寫

把以下普通程式改寫成更簡潔但仍易讀的形式：

- 普通 if/else 賦值 → 條件表達式。
- 普通迴圈累積列表 → 列表生成式。
- 臨時變數交換 → 序列解包。
- 手動 try/finally 關閉檔案 → with。
- 手動函式重新賦值 → @decorator。

改寫後要比較可讀性；如果表達式變得過長，應退回普通寫法。

---

## 16. 知識自查

### 基礎理解

1. 函式作為一等對象意味著什麼？
2. 閉包為什麼能記住外層變數？
3. 裝飾器接收什麼、返回什麼？
4. @decorator 展開後等價於哪一行賦值？
5. 裝飾器包裝發生一次，還是每次呼叫都發生？

### 實作理解

6. 為什麼通用 wrapper 要寫成 wrapper(*args, **kwargs)？
7. 為什麼要 return result？
8. functools.wraps 解決什麼問題？
9. 有參裝飾器為什麼需要三層函式？
10. @a、@b、def f 的等價展開是什麼？
11. 哪個裝飾器先進入、哪個裝飾器最後離開？

### 設計理解

12. 什麼功能適合用裝飾器，什麼功能不適合？
13. 如何避免裝飾器吞掉原函式例外？
14. 如何避免日誌裝飾器洩漏敏感資料？
15. 何時列表生成式反而會降低可讀性？
16. with 相比手動 close() 解決了什麼問題？

---

## 17. 最終總結

裝飾器可以濃縮成以下模型：

~~~text
原函式 f
   ↓ 傳入
裝飾器 decorator(f)
   ↓ 返回
包裝函式 wrapper
   ↓ 呼叫時
前置操作 → f(*args, **kwargs) → 後置操作 → 返回結果
~~~

而 @decorator 只是把以下程式碼寫得更清楚：

~~~python
def f():
    pass


f = decorator(f)
# 輸出：f 被替換成 decorator 返回的包裝函式。
# 說明：語法糖改變的是寫法，不是底層執行模型。
~~~

學習重點不在於大量背誦裝飾器模板，而在於能清楚回答三個問題：

1. 我想在原函式前後增加什麼行為？
2. 如何完整保留原函式的參數、返回值和例外？
3. 多個包裝層的責任和執行順序是否清楚？

能清楚回答這三個問題，就具備把裝飾器用在真實程式中的基礎。
