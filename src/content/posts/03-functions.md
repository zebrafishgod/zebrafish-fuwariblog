---
title: Python 第 168-195 集：函數/函式
published: 2026-09-11
updated: 2026-09-15
description: 函數
tags: [Python, 基礎篇]
category: Python 學習
draft: false
lang: zh_TW
---

# 第三章　函式：替一段工作取名字（第 168–195 集）

當新增學生、驗證分數、統計平均分的程式開始重複，應該把它們拆成有名字的小工作。本章解決的是：「如何讓一段程式可以重複使用，並清楚知道它需要什麼、產生什麼？」

前置知識：判斷、迴圈、列表、字典、字串拆分。學習路線是：定義與呼叫 → 返回值 → 參數繫結 → 預設值 → 收集與解包 → 拆分一個完整流程。先不學裝飾器。

每個 Python 程式碼框都可單獨執行。先預測畫面輸出，再執行；特別分清「畫面印出了什麼」與「呼叫者拿到什麼」。

## 1. 函式是有名字的一組步驟

### 1.1 定義是準備，呼叫才執行

```python
def greet():
    print("你好，Python！")


print("先準備")
greet()
print("已完成")
# 輸出：先準備；你好，Python！；已完成。
# 說明：def 建立函式物件並繫結名稱；執行 greet() 才進入函式本體。
```

逐步追蹤：

1. Python 執行到 `def`，建立函式，讓名稱 `greet` 指向它；不執行本體裡的 `print()`。
2. 顯示「先準備」。
3. 遇到 `greet()`，進入函式本體，顯示問候。
4. 函式結束，回到呼叫之後的位置，顯示「已完成」。

`greet` 是函式物件，`greet()` 是呼叫動作。空函式可以暫用 `pass` 佔位。函式本體的縮排決定哪些步驟屬於函式。

### 1.2 參數讓同一份步驟處理不同資料

```python
def greet_user(name):
    print(f"你好，{name}！")


greet_user("小明")
greet_user("小華")
# 輸出：你好，小明！；你好，小華！
# 說明：每次呼叫會將該次傳入的文字繫結到 name，函式本體只需寫一次。
```

定義括號裡的 `name` 叫**形參**，是等待資料的名稱；呼叫括號裡的 `"小明"` 叫**實參**，是這次提供的資料。形參不是「所有呼叫共用一個全域變數」，每次呼叫都有自己的局部繫結。

**練習：**寫 `show_product(name, stock)`，顯示商品名稱與庫存。**自查簡答：**兩個形參在 `def` 中，兩個實參在呼叫時提供；只定義不呼叫不會顯示商品。

## 2. `print()` 顯示資料，`return` 交還資料

### 2.1 計算結果要能接著使用

```python
def add(a, b):
    return a + b


result = add(3, 5)
print(result)
print(result * 2)
# 輸出：8；16。
# 說明：return 將 8 交還呼叫者；result 保存它，之後還可以參與計算。
```

追蹤 `result = add(3, 5)`：先呼叫 `add` → 形參為 `a=3`、`b=5` → 算出 8 → `return 8` → 呼叫表達式得到 8 → 將 8 賦值給 `result`。`return` 本身不會在終端機顯示結果。

```python
def show_total(a, b):
    print(a + b)


value = show_total(3, 5)
print(value)
# 輸出：8；None。
# 說明：函式顯示了 8，但沒有 return 值；執行完畢時隱式返回 None。
```

這是初學者最容易混淆的一點：**看見 8，不代表呼叫者得到 8。**沒有 `return`，或只寫 `return`，都會返回 `None`。`None` 是「沒有具體結果」的特殊值，不是字串 `"None"`，也不是數字 0。

### 2.2 `return` 立即結束本次函式呼叫

```python
def divide(a, b):
    if b == 0:
        return None
    return a / b


print(divide(10, 2))
print(divide(10, 0))
# 輸出：5.0；None。
# 說明：除數為 0 時先返回，不執行除法；本例約定 None 表示無法計算。
```

返回之後同一路徑上的後續步驟不再執行。`return` 結束函式；`break` 只跳出最近一層迴圈；`continue` 只進入下一次迴圈。不要把它們混為一談。

### 2.3 多個結果其實是一個元組

```python
def analyze(scores):
    if not scores:
        return None
    return min(scores), max(scores), sum(scores) / len(scores)


result = analyze([70, 80, 90])
print(result)
minimum, maximum, average = result
print(minimum, maximum, average)
print(analyze([]))
# 輸出：(70, 90, 80.0)；70 90 80.0；None。
# 說明：逗號將三個結果打包為元組；左側三個名稱再解包；空列表有明確返回規則。
```

呼叫者應先判斷結果是否為 `None`，再解包，不能把空資料的結果當成三個值。常用 `result is None` 判斷這個特殊值。

**練習：**寫 `find_max_min(numbers)`，非空返回最大與最小值，空列表返回 `None`。**自查簡答：**要兩個結果仍只使用一次 `return maximum, minimum`；分兩行 `return`，第二行不會被執行。

## 3. 參數是如何對上的？

### 3.1 位置與關鍵字是兩種傳入方式

```python
def introduce(name, age):
    return f"{name} 今年 {age} 歲"


print(introduce("小明", 18))
print(introduce(age=18, name="小明"))
print(introduce("小明", age=18))
# 輸出：三次都是 小明 今年 18 歲。
# 說明：位置實參依順序繫結，關鍵字實參依名稱繫結；這些形參都允許兩種方式。
```

一般呼叫時，先寫普通位置實參，再寫關鍵字實參。關鍵字名稱必須對得上形參，除非函式另外使用 `**kwargs` 收集它。

以下捕捉兩種錯誤，方便觀察而不中斷例子：

```python
def introduce(name, age):
    return f"{name} 今年 {age} 歲"


try:
    introduce("小明", name="小華", age=18)
except TypeError:
    print("name 被提供了兩次")
try:
    introduce("小明")
except TypeError:
    print("缺少 age")
# 輸出：name 被提供了兩次；缺少 age。
# 說明：第一個位置已繫結 name，不能再給同名關鍵字；必要參數也不能漏掉。
```

### 3.2 預設值是「沒提供時用什麼」

```python
def greet(name, message="歡迎你"):
    return f"{message}，{name}！"


print(greet("小明"))
print(greet("小華", "早安"))
print(greet("小明", message=""))
# 輸出：歡迎你，小明！；早安，小華！；，小明！
# 說明：只有沒傳 message 時才使用預設值；明確傳空字串仍是有效的傳入值。
```

對一般可按位置傳入的形參，沒有預設值的放前，有預設值的放後。此規則不應簡化成「任何必要參數都只能在所有預設參數前面」；僅關鍵字參數有不同規則，下一小節補充。

### 3.3 閱讀補充：`*` 之後只能用關鍵字

```python
def make_label(name, *, prefix="商品"):
    return f"{prefix}：{name}"


print(make_label("筆記本", prefix="新品"))
# 輸出：新品：筆記本。
# 說明：星號之後的 prefix 是僅限關鍵字參數，不能以第二個普通位置實參傳入。
```

若看見 `def f(a, /, b)`，斜線之前的 `a` 只能按位置傳入；`*` 之後只能按關鍵字傳入。初學先能讀懂，不必為每個函式加入這些限制。

**自查簡答：**形參指函式定義中的名稱；位置／關鍵字描述呼叫時如何給值。不能把「有預設值」誤當成「只能用關鍵字」。

## 4. 預設值只在定義時建立一次

這一節解決「明明呼叫兩次，為什麼第二次混入第一次的資料？」

```python
def add_item(item, items=[]):
    items.append(item)
    return items


print(add_item("A"))
print(add_item("B"))
# 輸出：['A']；['A', 'B']。
# 說明：這是刻意展示的錯誤設計；預設列表在定義時建立一次，多次省略 items 會共享它。
```

不是所有預設參數都有問題；問題在於你修改了跨呼叫共享的可變預設物件。希望每次預設使用新列表，可用 `None` 表示尚未提供。

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items


print(add_item("A"))
print(add_item("B"))
existing = ["C"]
print(add_item("D", existing))
print(existing)
# 輸出：['A']；['B']；['C', 'D']；['C', 'D']。
# 說明：省略時每次建新列表；明確傳入 existing 時，本例按約定修改呼叫者的列表。
```

追蹤：第一次 `items` 為 `None`，建立新列表加 A；第二次也從 `None` 開始，所以得到另一個列表。第三次收到 `existing` 的引用，沒有新建，會改動它。

不要隨便寫成 `items = items or []`，那會把呼叫者刻意提供的空列表也替換掉。`is None` 才精確對應「未提供」的約定。

**練習：**把預設字典 `options={}` 改成每次獨立建立。**自查簡答：**預設設為 `None`，函式內 `if options is None: options = {}`。

## 5. 傳入列表後，函式能不能改到外面？

Python 呼叫函式時，形參會繫結到實參所代表的物件。不是把整個列表複製一份，也不是一律「只能傳值／只能傳址」的簡單二分。

```python
def append_score(scores):
    scores.append(90)


def replace_scores(scores):
    scores = [100]
    return scores


original = [80]
append_score(original)
print(original)
new_scores = replace_scores(original)
print(original, new_scores)
# 輸出：[80, 90]；[80, 90] [100]。
# 說明：append 修改共同列表；scores = [100] 只重新繫結函式內的名稱。
```

若函式按設計要修改傳入的列表或字典，名稱和說明要讓呼叫者知道。若想保持原資料，可先複製，並確認淺拷貝是否足夠。

## 6. `*args` 與 `**kwargs`：收集未被接走的參數

### 6.1 定義時 `*args` 收集位置實參

```python
def total(*numbers):
    print(numbers)
    result = 0
    for number in numbers:
        result += number
    return result


print(total(1, 2, 3))
print(total())
# 輸出：(1, 2, 3)；6；()；0。
# 說明：星號將位置實參收集為元組；名稱不必是 args，numbers 在本例更有意義。
```

`args` 是慣用名稱，不是關鍵字；真正控制收集的是前面的 `*`。

### 6.2 `**kwargs` 收集關鍵字實參

```python
def build_student(name, score, **extra):
    student = {"name": name, "score": score}
    student.update(extra)
    return student


print(build_student("小明", 88, city="台北", club="程式社"))
# 輸出：{'name': '小明', 'score': 88, 'city': '台北', 'club': '程式社'}。
# 說明：name、score 先由普通形參接收，其餘關鍵字收集成 extra 字典。
```

可變長參數適合數量真的不固定的場合。若函式只需要商品 ID 與數量，直接寫兩個明確形參通常更好讀。

### 6.3 混用時，按類別接收

```python
def describe(title, *items, separator=" / ", **options):
    print(title)
    print(separator.join(items))
    print(options)


describe("購物車", "鉛筆", "筆記本", separator="、", color="green")
# 輸出：購物車；鉛筆、筆記本；{'color': 'green'}。
# 說明：title 先接第一項，items 收集其餘位置項，separator 僅限關鍵字，options 收集剩餘關鍵字。
```

追蹤不能只數括號裡有幾個值，還要看它們是位置實參或關鍵字實參、是否已被具名形參接走。

## 7. 呼叫時 `*`、`**` 是展開，不是收集

```python
def add(a, b, c):
    return a + b + c


numbers = [1, 2, 3]
options = {"a": 1, "b": 2, "c": 3}
print(add(*numbers))
print(add(**options))
# 輸出：6；6。
# 說明：呼叫中的 * 展開為位置實參；** 展開字典為關鍵字實參。
```

| 出現位置 | `*` | `**` |
|---|---|---|
| 函式定義 | 收集多個位置實參為元組 | 收集多個關鍵字實參為字典 |
| 函式呼叫 | 將可迭代資料展成位置實參 | 將映射展成關鍵字實參 |

`add(numbers)` 只傳了一個列表，不等於 `add(*numbers)`。`**options` 的鍵必須是字串，且最終參數要符合被呼叫函式的規則；與已提供的參數重複仍會出錯。

**練習：**用 `values=[2,3]` 呼叫 `power(base, exponent)`。**自查簡答：**`power(*values)`，等同 `power(2, 3)`；星號不會把函式自動改成接受任意參數。

## 8. 函式設計：一個名稱對應一項主要責任

一個函式「只有一項主要責任」，不代表只能有一行；而是使用者能用一句話說清它做什麼。例如：把文字轉成分數列表、分析分數、顯示分析結果。這三件事變更的理由不同，分開後更容易驗證。

先寫三項約定：

| 函式 | 輸入 | 返回與副作用 |
|---|---|---|
| `parse_scores(text)` | 空白分隔的整數文字 | 合法返回列表，錯誤返回 `None`；不列印 |
| `analyze_scores(scores)` | 分數列表 | 非空返回三項元組，空列表返回 `None` |
| `show_analysis(result)` | 統計結果或 `None` | 顯示結果，隱式返回 `None` |

```python
def parse_scores(text):
    scores = []
    for item in text.split():
        try:
            score = int(item)
        except ValueError:
            return None
        if not 0 <= score <= 100:
            return None
        scores.append(score)
    return scores


def analyze_scores(scores):
    if not scores:
        return None
    return min(scores), max(scores), sum(scores) / len(scores)


def show_analysis(result):
    if result is None:
        print("沒有分數可以統計")
        return
    minimum, maximum, average = result
    print(f"最低分：{minimum}")
    print(f"最高分：{maximum}")
    print(f"平均分：{average:.2f}")


text = "70 80 90"
scores = parse_scores(text)
if scores is None:
    print("請輸入 0 到 100 的整數分數")
else:
    show_analysis(analyze_scores(scores))
# 輸出：最低分：70；最高分：90；平均分：80.00。
# 說明：資料依序通過轉換、分析、顯示；None 表示非法輸入，空列表表示沒有輸入分數。
```

此處 `try/except ValueError` 只為辨識整數轉換失敗，後續例外章會深入。先用固定文字讓例子可重跑，再將 `text = "70 80 90"` 改成 `text = input("請輸入分數：")`。這樣輸入方式改變時，分析函式不用重寫。

用以下輸入自己追蹤一次：

| `text` | 解析結果 | 後續行為 |
|---|---|---|
| `"70 80 90"` | `[70, 80, 90]` | 正常分析 |
| `""` | `[]` | 顯示沒有分數 |
| `"80 abc"` | `None` | 顯示輸入錯誤 |
| `"101"` | `None` | 顯示超範圍的輸入錯誤 |

若把輸入、轉換、計算、列印和檔案保存全塞進一個函式，要測一次平均值就得走完全部流程。分開後，`analyze_scores([70, 80, 90])` 可直接驗證。

## 9. 常見錯誤查閱表

| 現象 | 原因 | 修正方向 |
|---|---|---|
| 定義了卻沒輸出 | 沒呼叫 | 分清 `func` 和 `func()` |
| 印出正確值，接到 `None` | 只有 `print` 沒 `return` | 計算函式返回結果 |
| `TypeError` 缺少參數 | 呼叫不符合形參 | 依位置和名稱畫繫結表 |
| 重複提供同一參數 | 位置已提供，又寫關鍵字 | 移除重複 |
| 第二次混入第一次資料 | 修改了可變預設物件 | `None` 作預設，呼叫內新建 |
| 改傳入列表影響外部 | 修改共享物件 | 明確約定副作用，必要時複製 |
| 函式有時返回值、有時沒有 | 分支漏了 `return` | 檢查所有分支的返回規則 |
| 空列表統計出錯 | `min`、`max`、除法沒有空資料規則 | 先處理空列表 |

## 10. 章末練習與自查簡答

1. 寫 `calculate_discount(price_cents, rate=0.9)`，返回折後金額。先約定小數如何處理；不要在函式內列印。這只是參數與返回值練習，完整電商作業會使用整數分作金額單位。
2. 寫 `find_max_min(*numbers)`；沒有數字時返回 `None`，否則返回最大值和最小值。
3. 寫 `build_student(name, score, **extra)`，返回字典，並說明新增欄位如何處理。
4. 將上一章學生管理草稿拆成「查詢學生」「新增學生」「計算平均分」「顯示選單」等函式。每個函式寫一句輸入、返回值與副作用說明。
5. 將猜數字遊戲拆成「判斷本次猜測」「進行一局」「顯示結果」。判斷函式接收答案和猜測，返回偏大、偏小或正確，不自行讀取輸入。

自查簡答：

- 定義和呼叫差在哪裡？定義建立函式物件；呼叫才執行本體。
- `return` 後還會執行同一路徑後面的程式嗎？不會；它結束本次呼叫。
- `print()` 能替代 `return` 嗎？不能；顯示和交還資料是不同事情。
- `*args`、`**kwargs` 在函式內是什麼？元組、字典；名稱可以更換。
- 預設值何時建立？執行函式定義時，不是每次呼叫時。
- 50 行函式一定不好嗎？不能只看行數；看責任、資料流和分支是否清楚。
- 怎樣才算能進下一章？能不看範例寫函式，追蹤參數如何對應，清楚區分返回值與副作用。

下一章將回答：「函式內的名稱到哪裡找？函式能否像資料一樣保存、傳遞，甚至包裝另一個函式？」
