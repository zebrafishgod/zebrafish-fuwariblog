---
title: Python 第 277-308 集：模組與包
published: 2026-09-16
updated: 2026-09-18
description: 模組與包
tags: [Python, 基礎篇]
category: Python 學習
draft: false
lang: zh_TW
---

# 第 6 章｜模組與包：把程式整理成能找到、能重用的檔案

> 對應既有學習路線的 277–308 節主題，並把後面第 358 節的包補充按主題併入。這是依課程目錄重組的講義，並非逐集逐字稿。示例使用 Python 3；本章不需要類別知識。

## 1. 本章解決什麼問題？需要哪些前置知識？

如果所有商品資料、價格計算、使用者輸入都放在同一個檔案，程式會逐漸難找、難修改。本章教你把工作拆成多個檔案，並回答：

1. `import` 到底做了什麼？它不是單純把另一個檔案文字貼進來。
2. `import module` 和 `from module import name` 有什麼差別？
3. 為什麼有些檔案能直接執行，有些卻必須用 `python -m ...`？
4. 為什麼會找不到模組、重複執行頂層程式碼，或發生循環導入？

前置知識：函式、作用域、串列與字典、檔案和資料夾。類型提示在 [第 5 章](05-iterators-generators-algorithms.md) 已介紹；看到 `price: int` 時，記得它是提示，不是自動驗證。

這章附有實際可執行的 [module_demo 小項目](examples/module_demo/README.md)。你可以先執行一次，再按下文逐步拆解。所有例子都在本機運行，不需外部服務。

## 2. 先建立直覺

### 2.1 模組像一個有自己名字表的工具抽屜

對本章的自訂程式來說，一個 `.py` 檔通常就是一個模組，例如 `pricing.py`。模組也可能來自 Python 內建功能或其他載入方式，所以不要把所有模組都限定成普通文字檔。

模組有自己的**名稱空間（namespace）**：它的變數、常數、函式名字都存放在自己的環境裡。假設你寫 `import pricing`，目前程式取得一個名為 `pricing` 的模組參考；透過 `pricing.line_total(...)` 便可使用它的函式。

你可以把 `pricing.line_total` 理解為：「到 pricing 這個抽屜，找 line_total 這件工具」。不同模組可有同名函式，不一定互相衝突。

### 2.2 包／套件是用來組織模組的層級

本章使用帶有 `__init__.py` 的**一般包（regular package）**。例如 `module_demo` 資料夾裡放 `catalog.py`、`pricing.py`，完整名稱分別是 `module_demo.catalog`、`module_demo.pricing`。

| 名稱 | 本章例子 | 用途 |
|---|---|---|
| 模組 | `pricing.py` | 保存一組相關函式或資料 |
| 包／套件 | `module_demo/` | 把相關模組組成有層級的單位 |
| 包初始化檔 | `__init__.py` | 在首次導入一般包時執行，通常保持簡短 |
| 包入口檔 | `__main__.py` | 執行 `python -m module_demo` 時啟動 |

Python 也支援不含 `__init__.py` 的 namespace package；「資料夾沒有 `__init__.py` 就永遠不能導入」並不準確。初學時先使用一般包，目錄關係最清楚。

### 2.3 導入是載入與綁定，不是複製全部程式碼

第一次用某個模組名稱導入時，Python 會查找、建立模組物件並執行它的頂層程式碼。這些「頂層程式碼」包括函式定義、變數賦值，也包括沒有放在函式內的 `print()` 或 `input()`。

同一個 Python 行程內，再用相同完整名稱導入，通常會從 `sys.modules` 快取取得已有模組。模組已完成的頂層程式碼一般不會再執行一次。

因此，把 `input()`、啟動選單、讀寫正式資料等動作隨意放在模組頂層，會讓別人一導入你的工具，就被迫啟動整個程式。

## 3. 最小可跑例：先學導入，再看多檔案項目

### 3.1 導入標準庫：不用先安裝

標準庫是隨 Python 提供的模組。以數學模組為例：

```python
# 輸出：
# 3.0
# 4.0
# 說明：import 綁定模組名稱；from import 可以綁定指定函式並取別名。
import math
from math import sqrt as square_root

print(math.sqrt(9))
print(square_root(16))
```

`math.sqrt` 和 `square_root` 在這裡指向同一個平方根函式；別名只是目前檔案採用的名稱。導入時不加 `.py`，也不寫檔案系統路徑。

| 寫法 | 目前檔案得到什麼名字？ | 使用方式 |
|---|---|---|
| `import math` | `math` | `math.sqrt(9)` |
| `import math as m` | `m` | `m.sqrt(9)` |
| `from math import sqrt` | `sqrt` | `sqrt(9)` |
| `from math import sqrt as root` | `root` | `root(9)` |

`from module import *` 會把一批名字帶進當前作用域，容易看不出來源或覆蓋已有名字。學習和項目中優先明確列出需要的名字。

### 3.2 先把附帶的小項目跑起來

目錄結構如下，所有檔案已隨筆記提供：

```text
examples/
└─ module_demo/
   ├─ __init__.py
   ├─ __main__.py
   ├─ catalog.py
   ├─ pricing.py
   ├─ entry_demo.py
   ├─ load_demo.py
   ├─ binding_demo.py
   └─ README.md
```

在 Windows PowerShell 執行：

```powershell
Set-Location 'D:\GPT projec1\outputs\python-foundations\examples'
python -m module_demo
```

如果你把筆記搬到其他地方，第一行改成自己 `examples` 資料夾的路徑。第二行的 `module_demo` 是包名稱，沒有 `.py`，也沒有斜線。

預期輸出：

```text
商品：筆
數量：3
小計：15.00 CNY
```

這裡只演示商品查詢與價格計算，還沒有互動選單、庫存、訂單或付款流程。不要把這個演示誤當作後面電商作業的完整答案。

### 3.3 檔案一：`module_demo/catalog.py` 負責資料

來源檔案：[catalog.py](examples/module_demo/catalog.py)。下面是完整內容：

```python
# 輸出：無。
# 說明：保存教學商品資料；價格單位是分，不是元。
CURRENCY = "CNY"
PRODUCTS = {
    "P001": {"name": "筆", "price_cents": 500},
    "P002": {"name": "書", "price_cents": 3000},
}


def get_product(product_id):
    return PRODUCTS.get(product_id)
```

`get_product()` 交回商品字典；找不到時，`dict.get()` 預設交回 `None`。它不負責印出商品，也不要求使用者輸入。

本例直接交回原字典的參考，外部若修改內容，也會改到模組保存的資料；它不是自動複製的安全副本。這裡只讀取，後續要讓其他檔案修改資料時，應把修改入口和規則設計清楚。

### 3.4 檔案二：`module_demo/pricing.py` 負責計算

來源檔案：[pricing.py](examples/module_demo/pricing.py)。下面是完整內容：

```python
# 輸出：無。
# 說明：只計算與格式化，不讀取使用者輸入；前提是非負整數金額及數量。
def line_total(unit_price_cents: int, quantity: int) -> int:
    return unit_price_cents * quantity


def format_money(cents: int) -> str:
    yuan = cents // 100
    fen = cents % 100
    return f"{yuan}.{fen:02d}"
```

為避免直接用二進位浮點數表示金額，本例把 `5.00 元` 存成 `500 分`。`// 100` 取得整數元，`% 100` 取得餘下的分；`:02d` 表示整數顯示至少兩位，不足時在前面補 `0`。

`line_total()` 交回整數分，`format_money()` 交回格式化字串，兩個函式本身都不顯示輸出。這份格式化函式以非負金額為前提，不是通用的金融格式化工具。類型提示也不會阻止負數、字串等不符合預期的輸入，互動程式需要在呼叫前驗證。

### 3.5 檔案三：`__init__.py` 描述這個包

來源檔案：[__init__.py](examples/module_demo/__init__.py)。下面是完整內容：

```python
# 輸出：無。
# 說明：一般套件的初始化檔；保持簡短，不啟動互動選單。
APP_NAME = "教學商品查詢"
__all__ = ["APP_NAME"]
```

`__init__.py` 可以是空的。本例多放一個包層級常數，讓你知道 `import module_demo` 後可使用 `module_demo.APP_NAME`。

`__all__` 主要用來說明 `from module_demo import *` 要帶出的名字；它不是存取權限或安全限制，也不表示所有子模組都自動載入。即使沒有把 `catalog` 寫進這個清單，仍可明確導入 `module_demo.catalog`。

### 3.6 檔案四：`__main__.py` 把工作串起來

來源檔案：[__main__.py](examples/module_demo/__main__.py)。**必須放在上述包結構內，並從 `examples` 執行 `python -m module_demo`。** 不要把本區塊單獨貼到任意 `.py` 檔後直接執行，因為它使用包內的相對導入。

```python
# 輸出：
# 商品：筆
# 數量：3
# 小計：15.00 CNY
# 說明：從 examples 目錄執行 python -m module_demo，才有套件執行脈絡。
from .catalog import CURRENCY, get_product
from .pricing import format_money, line_total


def main():
    product = get_product("P001")
    quantity = 3
    subtotal = line_total(product["price_cents"], quantity)
    print(f"商品：{product['name']}")
    print(f"數量：{quantity}")
    print(f"小計：{format_money(subtotal)} {CURRENCY}")


if __name__ == "__main__":
    main()
```

入口函式決定先查哪個商品、購買多少、最後顯示什麼。`P001` 是本例確定存在的固定資料；若改成使用者輸入，就必須先處理找不到商品的 `None`，不能直接取 `product["price_cents"]`。

現在你已經把工作分開了：資料模組不知道畫面怎麼顯示，計算模組不知道使用者如何輸入，而入口負責安排流程。

### 3.7 `__name__`：現在是入口，還是被別人導入？

來源檔案：[entry_demo.py](examples/module_demo/entry_demo.py)。這是獨立的入口觀察例子：

```python
# 輸出：
# 你好，小明
# 目前模組名：__main__
# 說明：這是用 python -m module_demo.entry_demo 執行時的輸出；導入時不顯示。
def greet(name):
    return f"你好，{name}"


def main():
    print(greet("小明"))
    print(f"目前模組名：{__name__}")


if __name__ == "__main__":
    main()
```

在 `examples` 資料夾執行 `python -m module_demo.entry_demo`，會出現註解中的兩行輸出。它作為本次程式入口執行，所以 `__name__` 是字串 `"__main__"`。

再開一個新 Python 行程，從 `examples` 執行以下片段。可以把片段存成 `examples/check_entry.py`：

```python
# 輸出：
# module_demo.entry_demo
# 你好，小華
# 說明：先前的 main() 不會因導入而被呼叫；greet() 仍然可以使用。
from module_demo import entry_demo

print(entry_demo.__name__)
print(entry_demo.greet("小華"))
```

導入時仍會執行頂層的函式定義與 `if` 判斷，只是條件不成立，因此不執行 `main()`。不要理解成「寫了入口保護，整個檔案在導入時就完全不執行」。

`__main__.py` 是檔名，而 `__name__ == "__main__"` 是執行時的條件，兩者用途不同。一般模組，例如 `entry_demo.py`，也可以在作為入口時得到 `__name__ == "__main__"`。

### 3.8 首次執行與 `sys.modules` 快取

來源檔案：[load_demo.py](examples/module_demo/load_demo.py)：

```python
# 輸出：
# load_demo：頂層執行
# 說明：為觀察首次導入特意加入頂層 print；正式工具模組通常避免這樣做。
print("load_demo：頂層執行")
VALUE = 10
```

從 `examples` 在**新行程**執行下面的片段：

```python
# 輸出：
# load_demo：頂層執行
# True
# True
# True
# 說明：相同完整模組名在同一行程中復用模組；del 只刪掉目前的一個名字。
import sys
from module_demo import load_demo
import module_demo.load_demo as second_name

print(load_demo is second_name)
print(load_demo is sys.modules["module_demo.load_demo"])

del load_demo
import module_demo.load_demo as third_name
print(third_name is second_name)
```

`is` 比較兩個名字是否指向同一個物件。雖然寫了三次導入，頂層訊息只顯示一次。

`del load_demo` 只移除目前作用域的名字；`second_name` 和 `sys.modules` 仍保有參考，所以不等於把模組卸載，也不保證立即回收名稱空間。物件能否回收，要看是否還被其他地方引用。

快取是**每個 Python 行程自己的**，不是把程式執行結果永久存到磁碟。重新啟動程式，頂層程式碼會重新執行。`__pycache__` 下的 `.pyc` 是位元組碼快取，和 `sys.modules` 中目前這次執行的模組物件不是同一回事。

快取也以模組名稱為鍵。若把同一來源檔案當 `__main__` 執行，又用一般模組名導入，可能得到兩次執行與不同狀態。初學時不要用修改 `sys.modules` 或反覆 reload 的方式處理設計問題；先維持清楚、一致的入口。

### 3.9 名稱空間：函式去哪裡找全域變數？

來源檔案：[binding_demo.py](examples/module_demo/binding_demo.py)：

```python
# 輸出：無。
# 說明：用來觀察模組名稱、全域查找及 from 導入後的綁定關係。
PRICE = 10
TAGS = ["new"]


def get_price():
    return PRICE
```

從 `examples` 執行以下片段：

```python
# 輸出：
# 999
# 10
# 說明：get_price() 的全域名稱查找使用它定義時所在的模組。
from module_demo import binding_demo

PRICE = 999
print(PRICE)
print(binding_demo.get_price())
```

呼叫者的 `PRICE = 999` 不會改變 `binding_demo.PRICE`。這正是模組名稱空間的好處：不同檔案可以有相同名稱，而不會只因名字一樣就混在一起。

### 3.10 `from ... import ...` 不會建立「永遠同步的變數」

仍使用 `binding_demo.py`，從 `examples` 的**新行程**執行：

```python
# 輸出：
# 10
# 20
# ['new', 'sale']
# True
# 說明：from 綁定當時的物件；重新綁定整數與修改同一串列，是不同操作。
from module_demo import binding_demo
from module_demo.binding_demo import PRICE, TAGS

binding_demo.PRICE = 20
binding_demo.TAGS.append("sale")

print(PRICE)
print(binding_demo.PRICE)
print(TAGS)
print(TAGS is binding_demo.TAGS)
```

分開看這兩條線：

- `PRICE` 最初和模組的 `PRICE` 都指向整數 `10`。把 `binding_demo.PRICE` 重新指向 `20`，不會自動重新綁定目前檔案的 `PRICE`。
- `TAGS` 和 `binding_demo.TAGS` 都指向同一個串列。`append()` 修改的是共同指向的物件，所以從兩邊都能看到新增項目。

因此，`from ... import ...` 不是深拷貝，也不是每次讀取時自動去模組查詢。若想清楚地讀取「模組目前的狀態」，保留模組名稱並使用 `module.attribute` 往往更直觀。

### 3.11 絕對導入與相對導入

本章包內的同一個工具，可以用以下方式指定：

| 寫法 | 如何解讀？ | 放在哪裡合適？ |
|---|---|---|
| `from module_demo.pricing import line_total` | 從可查找到的頂層包名開始 | 包外或包內，前提是能找到頂層包 |
| `from .pricing import line_total` | 從目前包的 `pricing` 子模組取得 | 包內，且執行時具有包脈絡 |
| `from ..pricing import line_total` | 先往上一層包，再找 `pricing` | 更深一層的子包內 |

相對導入的「相對」是相對於**包名稱**，不是相對於終端目前所在資料夾，也不是任意的硬碟相對路徑。不能用多個點越過頂層包。

如果直接執行 `python module_demo/__main__.py`，檔案會被當成獨立腳本，通常沒有這裡需要的包脈絡，`.catalog` 就會報「attempted relative import with no known parent package」。從 `examples` 用 `python -m module_demo`，Python 按包名稱啟動，便能理解相對導入。

`-m` 不是自動修復所有導入錯誤；前提仍然是目前的搜尋路徑能找到 `module_demo`。也不要把檔名寫成 `python -m module_demo.py`，因為 `-m` 後面接模組名稱，不是檔名。

### 3.12 `sys.path`：Python 到哪裡找檔案型模組？

一般情況下，Python 先查看已載入模組的快取；若需要載入，再透過導入機制查找。對本章的檔案型模組，`sys.path` 是重要的搜尋位置清單，但內建模組或特殊載入器不全都靠普通檔案路徑。

常見的 `sys.path` 來源包括：

- 腳本所在資料夾；使用 `python -m ...` 或互動模式時，通常包含目前工作資料夾。
- `PYTHONPATH` 環境變數提供的位置（如果有設定）。
- Python 標準庫的位置。
- 目前 Python 環境安裝第三方套件的位置，例如 `site-packages`。

具體順序和內容可能受虛擬環境、啟動選項或環境設定影響。最常見的初學錯誤，是把「目前終端資料夾」「正在執行的檔案所在資料夾」「專案頂層」當成永遠相同的位置。

從 `examples` 執行以下片段，檢查實際載入的檔案是否來自預期示例。這裡的 `__file__` 是檔案型模組的來源路徑；不保證所有種類的模組都有它。

```python
# 輸出：
# True
# True
# 說明：Path 處理檔案路徑；此例檢查模組來源，不顯示使用者電腦的完整路徑。
from pathlib import Path
from module_demo import pricing

source_file = Path(pricing.__file__)
print(source_file.name == "pricing.py")
print(source_file.parent.name == "module_demo")
```

`pathlib` 是標準庫；本例只把路徑轉成物件，讀取 `.name`（檔名）與 `.parent.name`（上一層資料夾名），不建立或修改檔案。

`sys.path` 是可修改串列，`sys.path.insert(0, 某個資料夾的字串路徑)` 能讓目前行程優先查找該位置。若要找到 `module_demo`，應加入**包含 module_demo 的上一層資料夾**，而非只加入包內某個子模組所在位置。它不會替你安裝套件，也不會永久改變其他行程。

對這份小項目，從正確的 `examples` 目錄以 `-m` 啟動即可。不要把自己電腦的絕對路徑散落到每個檔案；搬到另一台電腦就可能失效，加入過度優先的路徑也可能導入錯誤的同名模組。

常見例子是把自己的檔案取名為 `json.py`、`random.py` 或 `math.py`，與標準庫名稱衝突；程式可能載入自己的檔案。報錯時可檢查實際來源。也要避免建立 `module_demo.py` 與 `module_demo/` 同時存在於搜尋位置，讓自己難以判斷要載入哪個。

### 3.13 循環導入：兩個模組互相等待尚未定義的名字

假設 `a.py` 第一行就向 `b.py` 要一個名字，而 `b.py` 第一行又向 `a.py` 要稍後才會定義的名字。Python 不會等兩個檔案都執行完才處理導入；此時其中一個模組可能仍處於「只初始化了一部分」的狀態。

以下是**不要照此設計**的示意；程式行保留在註解中，不會真的觸發導入錯誤：

```python
# 輸出：無。
# 說明：以下是兩個檔案的錯誤結構示意，全部保持註解，沒有執行。
# a.py:
# from b import b_value
# a_value = 10
#
# b.py:
# from a import a_value
# b_value = 20
```

從導入 `a` 開始追蹤：

1. Python 開始執行 `a.py`，但尚未執行 `a_value = 10`。
2. 第一行要求載入 `b`，因此開始執行 `b.py`。
3. `b.py` 要求從 `a` 取得 `a_value`，可是 `a` 還沒定義這個名字。
4. 因而可能看到帶有「partially initialized module」的 `ImportError`。

常用修正是把雙方共用的資料或函式移到第三個模組，例如 `common.py`，讓 `a` 和 `b` 都依賴它；或把流程安排集中到入口模組，工具模組只提供資料和函式，不反向導入入口。

把導入移到函式內，有時能延後時間，解除特定循環；但若互相呼叫的設計仍有問題，延後並不是根治。也不是任何形式的互相導入一定報錯，關鍵在於讀取名字時它是否已建立。初學階段以單向、容易追蹤的依賴關係為目標。

### 3.14 軟體目錄規範：先按用途分，不必一次建滿

目前七個 `.py` 檔中，`entry_demo`、`load_demo`、`binding_demo` 都只是教學觀察工具。真正商品示例只需要 `__init__.py`、`__main__.py`、`catalog.py`、`pricing.py` 四個檔案。

之後把程式擴展成可保存商品的本機工具，可以採用這種結構。這是未來整理方式的示意，以下目錄並未在附帶項目內全部建立：

```text
shop_project/
├─ README.md              如何啟動、功能、限制
├─ data/                  商品與訂單等資料檔案
├─ tests/                 核對程式行為的測試
└─ shop/
   ├─ __init__.py
   ├─ __main__.py         入口與操作流程
   ├─ catalog.py          商品查詢、增刪改規則
   ├─ pricing.py          純金額計算
   ├─ storage.py          讀檔、存檔
   └─ settings.py         共用設定
```

在 `shop_project` 目錄執行 `python -m shop`。隨著程式成長再建立需要的部分，不要求一開始就有所有檔案。你應該能回答「這個檔案解決哪一類問題」，而不只是照抄目錄名稱。

幾個分工判斷：

- 想改商品排序規則，主要找 `catalog.py`；想改金額計算，找 `pricing.py`。
- 想把 JSON 檔案換成另一種保存方式，盡量集中修改 `storage.py`，不要讓每個選單選項都直接讀寫檔案。
- 共用設定可放 `settings.py`，設定模組不應反過來啟動入口。
- `data/` 是程式使用或產生的資料，不是需要 `import` 的 Python 程式碼；檔案路徑與模組搜尋是兩件事。
- 讀寫資料時，單用相對檔名通常以「目前工作目錄」為基準，並不會自動跟著程式檔案走。正式加入儲存功能時，要明確指定資料根目錄，相關路徑工具在後續標準庫章節練習。

先讓一個小功能可跑，再整理其職責和依賴，是比盲目增加目錄更容易檢查的方式。模組化的成果應是：別人能找到功能、工具能重用、改一處不需要到處複製同樣程式。

## 4. 執行追蹤：從命令到顯示結果

### 4.1 `python -m module_demo` 的流程

| 步驟 | 發生什麼 | 此時是否有螢幕輸出？ |
|---|---|---|
| 1 | 從搜尋位置找到 `module_demo` 包 | 無 |
| 2 | 首次載入包，執行 `__init__.py` | 無，只定義名字 |
| 3 | 把 `module_demo/__main__.py` 作為入口執行 | 尚無 |
| 4 | 相對導入 `catalog` 和 `pricing` | 無，只建立資料和函式 |
| 5 | 定義 `main()`，判斷入口條件成立 | 尚無 |
| 6 | 呼叫 `main()`，取出 `P001` | 無，取得字典 |
| 7 | `line_total(500, 3)` 交回 `1500` | 無，得到整數分 |
| 8 | 前兩次 `print()` | 顯示商品、數量 |
| 9 | 第三次 `print()` 的參數先求值，其中 `format_money(1500)` 交回 `"15.00"` | 取得字串，尚未顯示小計 |
| 10 | 第三次 `print()` 使用組合好的字串 | 顯示小計 |

函式定義會建立函式，但不等於呼叫函式。這個區分正好接上前章的 `return`：計算結果先交回，直到外層選擇 `print()` 才顯示。

### 4.2 依賴方向

```text
__main__.py（安排流程與顯示）
   ├─→ catalog.py（商品資料）
   └─→ pricing.py（金額計算）
```

`catalog.py` 和 `pricing.py` 不必反過來導入 `__main__.py`。這個例子很小，拆檔的目的是建立職責和依賴的直覺；不是規定每個函式都要獨立一個檔案。

## 5. 常見錯誤與排查順序

| 情況 | 先檢查什麼？ | 常見修正 |
|---|---|---|
| `ModuleNotFoundError` | 名字拼寫、包是否存在、啟動位置、使用哪個 Python 環境 | 回到包上一層，用正確的模組名和 `-m` 啟動 |
| 相對導入沒有父包 | 是否直接執行了包內檔案 | 按完整包名用 `-m` 執行 |
| `ImportError: cannot import name ...` | 目標名字是否存在、是否尚未定義、是否循環導入 | 核對名字與依賴方向，而不是先亂改搜尋路徑 |
| 一導入就跳出輸入提示 | `input()` 或選單是否在頂層 | 放進 `main()`，以入口條件控制呼叫 |
| 模組內容和預期不同 | 是否同名遮蔽、來源路徑是否正確 | 核對 `__file__`，重新命名衝突檔案 |
| 改了檔案，舊互動工作階段沒變 | 模組是否已被快取 | 初學先重新啟動程式再驗證，不把重複 import 當成重載 |
| `del module` 後再導入沒有重新執行 | `sys.modules` 或其他名字是否仍保留物件 | 理解名稱刪除與物件生命週期不同 |
| `from ... import PRICE` 沒同步更新 | 是修改共享物件，還是重新綁定模組屬性 | 需要目前狀態時使用 `module.PRICE` |
| 寫了 `__all__` 仍可導入其他名字 | 把匯出清單誤當成權限控制 | `__all__` 不是禁止存取的安全機制 |

讀錯誤時先看最後一行的種類與訊息，再往上找自己寫的檔案。不要看到「找不到」就立刻安裝套件；自己寫的模組根本不需要去網路安裝。例外處理與測試補充見 [bridge-errors-testing.md](bridge-errors-testing.md)。

### 5.1 寫模組時的實用規則

- 名字簡短、描述用途，使用小寫與底線，例如 `order_service.py`。
- 導入通常放在檔案前部；把標準庫、第三方套件、自己項目的導入分組，方便看依賴。
- 定義可重用的函式時，避免一導入就啟動選單或修改正式資料。
- 明確說明金額、時間、數量的單位及輸入前提，別只靠名稱猜。
- `__init__.py` 保持簡短。只有需要方便使用者導入時，才考慮清楚地重新匯出工具。
- 不用 `from ... import *` 隱藏來源，也不要把內建或常用標準庫名稱拿來當自己的檔名。

未來項目變大時，可以再分出 `tests/`、`data/` 和項目說明文件。包的頂層應清楚可找到；不要為了看起來專業就套用自己還無法解釋的多層目錄。

## 6. 練習：把功能放對地方

### 練習 A｜預測導入結果

不執行程式，先回答：第一次導入 `module_demo.load_demo`、第二次導入、重開一個新 Python 行程再導入，各會印幾次「頂層執行」？然後在終端驗證。

### 練習 B｜新增商品

只在 `catalog.py` 加入 `P003`：名稱「杯子」，價格 `2000` 分。建立一個練習入口，顯示購買兩個杯子的小計。驗收輸出應包含「杯子」「2」「40.00 CNY」。原來的價格計算函式不需要為這件商品另寫一份。

### 練習 C｜增加一個計算工具

在 `pricing.py` 加入 `total_with_shipping(subtotal_cents, shipping_cents)`，交回商品小計加運費，仍以分為單位。為它寫類型提示，並把前提寫在註解中。驗收：`1500` 加 `600` 得到整數 `2100`，格式化後為字串 `"21.00"`。

只測算術時不需要啟動商品選單。請從另一個練習檔案導入並呼叫它，證明這個模組真的可重用。

### 練習 D｜入口保護

把一個「詢問姓名、印出問候」的小程式整理成 `greet(name)`、`main()` 和入口條件。驗收：直接執行會詢問姓名；被另一個檔案導入時不詢問姓名，但能呼叫 `greet("小明")` 取得字串。

### 練習 E｜導入方式

在包內解釋 `from .pricing import line_total` 的一個點代表什麼。再從包外用絕對導入呼叫同一個函式。寫下兩種情境應使用的啟動命令；不要靠到處添加絕對 `sys.path` 才成功。

### 練習 F｜設計依賴

假設 `users.py` 和 `orders.py` 都要讀取同一份運費設定。畫出檔案依賴方向，說明應把設定放在哪裡，避免這兩個模組因為共用設定互相導入。此題可以先用文字或箭頭回答，不要求新增業務功能。

## 7. 自查與簡答

| 自查問題 | 簡答 |
|---|---|
| `import pricing` 是把所有函式複製進目前檔案嗎？ | 不是。通常載入或取得模組物件，並綁定 `pricing` 這個名字。 |
| 模組的頂層 `print()` 什麼時候執行？ | 首次載入時；同一行程相同名字的重複導入通常用快取。作為腳本重新啟動又是新的一次執行。 |
| 導入函式後，它的全域變數在哪裡找？ | 在定義該函式的模組全域名稱空間，而非任意呼叫者的同名變數。 |
| `from module import value` 是深拷貝嗎？ | 不是。它在目前作用域綁定當時取得的物件。 |
| `__name__` 等於檔名嗎？ | 不總是。導入時通常是模組完整名稱；作為入口執行時是 `"__main__"`。 |
| `__init__.py` 和 `__main__.py` 有何不同？ | 前者用於一般包初始化，後者提供 `python -m 包名` 的入口。 |
| 相對導入跟著目前工作目錄走嗎？ | 不是。它根據模組所屬包來解析。 |
| `sys.path` 要放包資料夾，還是它的上一層？ | 要能搜尋到頂層包，通常放包含它的上一層。 |
| 刪除當前 `module` 名字，會立即卸載模組嗎？ | 不會。快取與其他物件仍可能持有參考。 |
| `sys.modules` 和 `__pycache__` 一樣嗎？ | 不一樣。前者是目前行程內的模組快取；後者通常保存編譯後的位元組碼檔案。 |
| 循環導入一定報錯嗎？ | 不一定；問題常是模組尚未初始化完成，就被索取尚未定義的名字。應優先整理依賴方向。 |
| `-m` 後面寫 `module_demo.py` 嗎？ | 不寫。應是可導入的模組／包名稱，例如 `module_demo`。 |

練習核對：A 在同一行程第一回印 `1` 次、第二回新增 `0` 次，重開行程後再印 `1` 次；C 應以 `return subtotal_cents + shipping_cents` 交回數字；F 可使用 `settings.py` 保存設定，讓 `users.py`、`orders.py` 都單向導入它。

本章完成的最低標準：能從正確位置啟動附帶項目、說清楚四個主要檔案的分工、理解導入快取與名稱綁定，並能把自己的小程式拆成資料、計算和入口。後續增加功能時，先維持這種可解釋的結構，再逐步擴大。
