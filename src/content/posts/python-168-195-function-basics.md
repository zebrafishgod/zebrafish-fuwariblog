---
title: Python 第 168–195 集：函式基礎與參數設計
published: 2026-09-07
description: 從函式定義與呼叫開始，掌握返回值、位置與關鍵字參數、預設參數、可變長參數及函式設計原則。
tags: [Python, 函式, 參數設計, 基礎語法]
category: Python 學習
draft: false
lang: zh_TW
---

> 本章整理函式引入、定義與呼叫、參數、返回值、可變長參數和函式設計原則。

## 1. 學習目標

完成本階段後，應能夠：

- 使用 `def` 定義和呼叫函式。
- 區分形參與實參、位置參數與關鍵字參數。
- 使用預設參數、`*args`、`**kwargs`。
- 理解 `return` 與 `print()` 的區別。
- 把重複流程拆成責任清楚的函式。

## 2. 函式基本概念

函式把一組有明確責任的步驟命名並封裝起來，使程式可以重複使用、拆分問題並降低耦合。

```python
def greet():
    """顯示問候語。"""
    print("Hello, Python!")


greet()
```

- `def` 用於定義函式。
- 函式定義不會立即執行函式體。
- 呼叫函式時才會執行其中的程式碼。
- 函式體必須縮排。

無參函式不需要外部資料；有參函式透過參數接收資料：

```python
def greet_user(name):
    print(f"你好，{name}！")


greet_user("小明")
```

## 3. 返回值與 `return`

`return` 有兩個作用：把結果交還給呼叫者，並立即結束目前函式。

```python
def add(a, b):
    return a + b


result = add(3, 5)
print(result)
```

沒有 `return` 時，函式返回 `None`：

```python
def show_message():
    print("完成")


value = show_message()
print(value)  # None
```

`print()` 是顯示資料，`return` 是交還資料。計算類函式通常應返回結果，顯示類函式才直接列印。

函式可以提前返回：

```python
def divide(a, b):
    if b == 0:
        return None
    return a / b
```

多個返回值會被打包成元組：

```python
def analyze(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)


minimum, maximum, average = analyze([70, 80, 90])
```

## 4. 形參、實參與參數種類

```python
def power(base, exponent):  # base、exponent 是形參
    return base ** exponent


power(2, 3)                 # 2、3 是實參
```

- 形參：定義函式時括號中的名稱。
- 實參：呼叫函式時實際傳入的資料。

### 4.1 位置參數

```python
def introduce(name, age):
    print(f"{name} 今年 {age} 歲")


introduce("小明", 18)
```

按照形參順序傳值，順序錯誤可能造成語意錯誤。

### 4.2 關鍵字參數

```python
introduce(age=18, name="小明")
```

明確寫出參數名稱，可提升可讀性；位置參數必須放在關鍵字參數前面。

### 4.3 預設參數

```python
def greet(name, message="歡迎你"):
    print(f"{message}，{name}！")


greet("小明")
greet("小華", "早安")
```

必要參數放前面，預設參數放後面。避免直接用列表或字典作為預設值，因為多次呼叫可能共享同一個可變物件：

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

## 5. 可變長參數

`*args` 收集多個位置參數，函式內是元組：

```python
def total(*args):
    result = 0
    for number in args:
        result += number
    return result


print(total(1, 2, 3))
```

`**kwargs` 收集多個關鍵字參數，函式內是字典：

```python
def show_profile(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")


show_profile(name="小明", age=18)
```

普通參數、`*args`、`**kwargs` 可以混用：

```python
def describe(title, *items, **options):
    print(title)
    print(items)
    print(options)
```

呼叫時也能解包：

```python
def add(a, b, c):
    return a + b + c


numbers = [1, 2, 3]
print(add(*numbers))

options = {"a": 1, "b": 2, "c": 3}
print(add(**options))
```

## 6. 函式設計原則

### 單一責任

一個函式最好只完成一項主要任務：

```python
def calculate_average(scores):
    return sum(scores) / len(scores)
```

不要讓同一函式同時負責輸入、複雜計算、檔案寫入和大量列印。

### 輸入與輸出清楚

```python
def is_passed(score, passing_score=60):
    return score >= passing_score
```

看到函式名稱和參數，就應能預測它的功能。盡量透過參數傳入資料，減少對全域變數的依賴。

### 函式呼叫流程

```text
呼叫函式 → 建立本次局部執行環境 → 形參繫結實參 → 執行函式體 → return 或執行完畢 → 返回結果
```

## 7. 綜合範例

```python
def read_scores():
    text = input("請輸入分數：")
    return [int(item) for item in text.split()]


def analyze_scores(scores):
    if not scores:
        return None
    return min(scores), max(scores), sum(scores) / len(scores)


scores = read_scores()
result = analyze_scores(scores)

if result is None:
    print("沒有輸入分數")
else:
    minimum, maximum, average = result
    print(f"最低分：{minimum}")
    print(f"最高分：{maximum}")
    print(f"平均分：{average:.2f}")
```

此例把輸入和分析分離，並明確處理空列表。

## 8. 常見錯誤

- `greet` 只是取得函式對象，`greet()` 才是呼叫。
- 把 `print()` 當成 `return`，會導致呼叫者得到 `None`。
- 參數數量、順序或名稱不匹配會產生 `TypeError`。
- 可變預設參數可能跨呼叫保留資料。
- 函式過長或責任混雜時，應重新拆分。

## 9. 知識自查

1. 函式定義與函式呼叫有什麼區別？
2. 形參和實參分別出現在哪裡？
3. `print()` 和 `return` 有什麼不同？
4. 沒有 `return` 時返回什麼？
5. `return` 執行後後面的程式碼會執行嗎？
6. 位置參數、關鍵字參數、預設參數如何選擇？
7. `*args` 和 `**kwargs` 在函式內分別是什麼類型？
8. 為什麼不建議使用列表作為預設參數？
9. 如何把一個 50 行以上的程序拆成多個函式？

## 10. 練習

- 寫 `calculate_discount(price, rate=0.9)`，返回折後價。
- 寫 `find_max_min(*numbers)`，返回最大值和最小值，空輸入時返回 `None`。
- 寫 `build_student(name, score, **extra)`，返回學生字典。
- 將猜數字遊戲拆成「判斷猜測」「執行一局」「顯示結果」三個函式。
- 將學生管理器的平均分、最高分、查詢功能分別抽成函式。

## 11. 進入第 196 集前的達標標準

- 能不看範例寫出有參和無參函式。
- 能正確使用位置、關鍵字和預設參數。
- 能判斷函式應該列印還是返回。
- 能使用 `*args`、`**kwargs` 完成合理場景。
- 能把單文件程序拆成責任清楚的函式。
- 能說明每個函式的輸入、處理和輸出。
