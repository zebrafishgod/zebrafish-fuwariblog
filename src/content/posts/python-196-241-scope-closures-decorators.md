---
title: Python 第 196–241 集：名稱空間、作用域、閉包與裝飾器
published: 2026-09-08
description: 深入理解 Python 名稱空間、LEGB 作用域、global、nonlocal、閉包、裝飾器與多層裝飾器的執行方式。
tags: [Python, 作用域, 閉包, 裝飾器]
category: Python 學習
draft: false
lang: zh_TW
---

> 本章整理名稱空間、名稱查找、作用域、`global`、`nonlocal`、函式傳遞、閉包、裝飾器、有參裝飾器與裝飾器疊加。

## 1. 本階段學習目標

完成本階段後，應能夠：

- 解釋名稱空間和作用域的用途。
- 掌握 Python 的名稱查找順序 LEGB。
- 區分內置、全域、局部和嵌套作用域。
- 理解函式是對象，可以傳遞、保存和返回。
- 理解閉包如何保存外層函式的狀態。
- 寫出基本裝飾器、有參裝飾器和多層裝飾器。
- 在不修改原函式核心程式碼的情況下增加日誌、計時、權限等功能。

## 2. 知識地圖

| 集數 | 主題 | 核心問題 |
|---:|---|---|
| 196–201 | 名稱空間與名稱查找 | Python 如何找到一個名稱？ |
| 202–208 | 函式嵌套與作用域 | 內層函式如何使用外層資料？ |
| 209–214 | 函式傳遞與應用 | 函式能否像普通資料一樣使用？ |
| 215–216 | 閉包 | 函式如何記住建立時的環境？ |
| 217–233 | 裝飾器基礎與模板 | 如何包裝原函式而增加功能？ |
| 234–241 | 有參裝飾器與疊加 | 如何配置裝飾器並理解執行順序？ |

## 3. 名稱空間

名稱空間可以理解為「名稱到對象的映射表」。變數名、函式名和模組名都需要在某個名稱空間中找到對應的對象。

```python
x = 10
print(x)
```

這裡 `x` 是名稱，`10` 是對象；名稱空間保存了兩者的關係。

### 3.1 主要名稱空間

- **內置名稱空間**：`print`、`len`、`int` 等 Python 內置名稱。
- **全域名稱空間**：模組檔案最外層定義的名稱。
- **局部名稱空間**：函式呼叫時建立，保存該次呼叫的局部變數和參數。
- **嵌套函式作用域**：外層函式中的名稱可被內層函式引用。

### 3.2 LEGB 名稱查找順序

Python 通常按以下順序查找名稱：

```text
L - Local：目前函式局部作用域
E - Enclosing：外層函式作用域
G - Global：模組全域作用域
B - Built-in：內置名稱空間
```

```python
name = "global"


def outer():
    name = "enclosing"

    def inner():
        name = "local"
        print(name)

    inner()


outer()  # local
```

如果目前作用域找不到，就向外層查找；全部找不到時產生 `NameError`。

### 3.3 名稱遮蔽

內層作用域可以定義同名名稱，遮蔽外層名稱，但不會刪除外層名稱：

```python
value = "global"


def show():
    value = "local"
    print(value)


show()       # local
print(value) # global
```

避免把自己的變數命名為 `list`、`str`、`sum` 等內置名稱，以免遮蔽內置功能。

## 4. 作用域、`global` 與 `nonlocal`

### 4.1 局部和全域作用域

函式內賦值通常會建立局部名稱：

```python
count = 0


def increase():
    count = 1       # 新的局部名稱
    print(count)
```

若要在函式內重新綁定全域名稱，需要 `global`：

```python
count = 0


def increase():
    global count
    count += 1
```

過度使用 `global` 會讓資料流不清楚。優先考慮用參數傳入、用返回值傳出。

### 4.2 `nonlocal`

`nonlocal` 用於內層函式重新綁定最近外層函式的局部名稱，不能用來直接修改全域名稱：

```python
def make_counter():
    count = 0

    def counter():
        nonlocal count
        count += 1
        return count

    return counter


counter = make_counter()
print(counter())  # 1
print(counter())  # 2
```

## 5. 函式是一等對象

Python 中函式可以像普通資料一樣：

- 賦值給變數。
- 放進列表或字典。
- 作為另一個函式的參數。
- 作為函式的返回值。

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


operations = [add, multiply]
for operation in operations:
    print(operation(2, 3))
```

把函式傳給另一個函式：

```python
def apply_operation(func, a, b):
    return func(a, b)


print(apply_operation(add, 2, 3))
```

這是理解閉包和裝飾器的前提。

## 6. 閉包

閉包通常由三部分組成：

1. 外層函式。
2. 內層函式。
3. 內層函式引用外層函式的局部變數，且外層函式返回內層函式。

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply


times_two = make_multiplier(2)
print(times_two(5))  # 10
```

雖然 `make_multiplier()` 已經執行完畢，`multiply()` 仍記得 `factor`。這個被保留的外層環境就是閉包的重要特徵。

### 6.1 閉包的用途

- 建立帶有私有狀態的函式。
- 延遲執行某項操作。
- 產生一批行為相似但配置不同的函式。
- 作為裝飾器的基礎。

### 6.2 閉包與 `nonlocal`

若要修改閉包保存的外層變數，需要 `nonlocal`：

```python
def make_counter(start=0):
    count = start

    def count_up():
        nonlocal count
        count += 1
        return count

    return count_up
```

## 7. 裝飾器

裝飾器是一個接收函式、返回新函式的函式。它可以在不修改原函式核心內容的情況下，為函式增加功能。

常見用途：

- 執行日誌。
- 計時。
- 權限檢查。
- 快取。
- 參數驗證。
- 統一錯誤處理。

### 7.1 最基本的裝飾器

```python
def log_call(func):
    def wrapper():
        print("函式開始")
        result = func()
        print("函式結束")
        return result

    return wrapper


@log_call
def say_hello():
    print("Hello")


say_hello()
```

`@log_call` 是語法糖，等價於：

```python
def say_hello():
    print("Hello")


say_hello = log_call(say_hello)
```

### 7.2 支援原函式參數

通用裝飾器通常使用 `*args` 和 `**kwargs`：

```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"開始執行：{func.__name__}")
        result = func(*args, **kwargs)
        print(f"執行完成：{func.__name__}")
        return result

    return wrapper
```

這樣無論原函式有多少位置參數或關鍵字參數，裝飾器都能轉交它們。

### 7.3 裝飾器必須返回結果

若包裝函式忘記 `return result`，原函式的返回值就會遺失：

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)  # 沒有返回
    return wrapper
```

正確寫法：

```python
def good_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result
    return wrapper
```

### 7.4 保留原函式資訊

包裝後的函式可能失去原名稱和文件字串。實務上可使用 `functools.wraps`：

```python
from functools import wraps


def log_call(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("開始")
        return func(*args, **kwargs)

    return wrapper
```

## 8. 有參裝飾器

普通裝飾器接收原函式；有參裝飾器要再多一層函式，先接收配置，再接收原函式。

```python
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
```

等價流程：

```text
repeat(3) → 得到 decorator
 decorator(function) → 得到 wrapper
  呼叫 wrapper() → 重複執行 function()
```

有參裝飾器常用於：

- 設定重試次數。
- 設定日誌等級。
- 指定權限角色。
- 設定快取時間。

## 9. 裝飾器疊加與執行順序

多個裝飾器由上到下書寫，但包裝關係通常是由下到上建立：

```python
@decorator_a
@decorator_b
def work():
    pass
```

等價於：

```python
work = decorator_a(decorator_b(work))
```

呼叫時，外層 `decorator_a` 先進入；若它呼叫內層函式，才會進入 `decorator_b`。因此常見輸出順序是：

```text
A before
B before
原函式
B after
A after
```

設計多層裝飾器時，要確認：

- 每一層是否都轉交 `*args`、`**kwargs`。
- 每一層是否都返回下一層的結果。
- 執行順序是否符合需求。
- 是否需要使用 `wraps` 保留函式資訊。

## 10. 綜合範例：計時裝飾器

```python
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


print(calculate_total(range(100000)))
```

這個例子將「計時」從業務邏輯中分離出來，原函式只負責計算總和。

## 11. 常見錯誤

### 11.1 忘記 `nonlocal`

內層函式要修改外層局部變數時，沒有 `nonlocal` 會被視為建立新的局部名稱，可能出現 `UnboundLocalError`。

### 11.2 函式名稱被遮蔽

不要在函式或模組中把變數命名成 `print`、`sum`、`list` 等，否則可能無法使用對應內置功能。

### 11.3 裝飾器忘記返回包裝函式

裝飾器最後需要 `return wrapper`；否則被裝飾名稱可能變成 `None`。

### 11.4 包裝函式忘記傳參數

通用包裝器應使用 `wrapper(*args, **kwargs)` 並把參數傳給原函式。

### 11.5 包裝函式忘記返回結果

如果原函式有返回值，包裝器通常需要 `return func(...)` 或 `return result`。

### 11.6 過度使用全域變數

`global` 能解決部分問題，但會讓函式依賴隱藏狀態，降低可測試性。優先使用參數、返回值和閉包狀態。

## 12. 知識自查

### 概念題

1. 什麼是名稱空間？
2. LEGB 分別代表什麼？
3. 名稱查找不到時會發生什麼？
4. 內層同名變數為什麼會遮蔽外層變數？
5. `global` 和 `nonlocal` 分別修改哪一層名稱？
6. 函式為什麼可以放進列表或作為參數傳遞？
7. 閉包由哪些條件組成？
8. 裝飾器解決什麼類型的問題？
9. `@decorator` 語法糖等價於什麼？
10. 為什麼通用裝飾器要使用 `*args` 和 `**kwargs`？
11. 為什麼裝飾器通常要返回原函式結果？
12. 兩個裝飾器疊加時，哪個先建立、哪個先進入？

### 動手題

- 寫一個 `make_counter()`，每次呼叫返回遞增數字。
- 寫一個閉包 `make_power(n)`，返回計算 `x ** n` 的函式。
- 寫一個日誌裝飾器，列印函式名稱、參數和返回值。
- 寫一個權限裝飾器，只有角色為 `admin` 時才執行原函式。
- 寫一個有參裝飾器 `retry(times)`，原函式失敗時最多重試指定次數。
- 使用兩個裝飾器分別實現計時和日誌，觀察疊加順序。

## 13. 進入下一階段前的達標標準

- 能畫出簡單程式中的 LEGB 名稱查找路徑。
- 能說明 `global` 和 `nonlocal` 的使用限制。
- 能把函式作為參數傳遞或返回。
- 能手寫一個保存狀態的閉包。
- 能手寫支援任意參數的裝飾器。
- 能解釋裝飾器語法糖和多裝飾器的執行順序。
- 能使用 `functools.wraps` 保留原函式資訊。

本階段的核心轉變是：函式不再只是「一段可重複程式碼」，而是可以被保存、傳遞、包裝和組合的對象。掌握這一點後，下一步學習迭代器、生成器、模組和包會更容易。
