# pyscript-test
最近看到一個新的 Open source project 叫做 [PyScript](https://pyscript.net/)，是一個能在 HTML 中直接撰寫 Python 程式，當使用者開啟該檔案時，就會在背後直接開始執行 Python

感覺運用場景可以很廣，像是直接進行資料視覺化、產出 Dashboard 或是利用爬蟲直接顯示有興趣的網頁的結果

本 repo 主要是測試並記錄 PyScript 試用的過程與感想

---

## 不管怎樣都要先來個 Hello world

<img src="images/helloWorld.png" width="600">

```html
<html>
  <head>
    <link rel="stylesheet" href="https://pyscript.net/alpha/pyscript.css" />
    <script defer src="https://pyscript.net/alpha/pyscript.js"></script>
  </head>
  <body> <py-script> print('Hello, World!') </py-script> </body>
</html>
```

- 開網頁直到顯示出 Hello World 之間，會有明顯有感的延遲時間

---

## 來點簡單的數學計算

<img src="images/math.png" width="600">

```html
<html>

<head>
    <link rel="stylesheet" href="https://pyscript.net/alpha/pyscript.css" />
    <script defer src="https://pyscript.net/alpha/pyscript.js"></script>
</head>

<body>
    <py-script>
print("Let's compute π:")
def wallis(n):
    pi = 2
    for i in range(1, n):
        pi *= 4 * i ** 2 / (4 * i ** 2 - 1)
    return pi

pi = wallis(100000)
s = f"π is approximately {pi:.3f}"
print(s)
    </py-script>
</body>

</html>
```
- 看起來延遲時間是難以避免的了
- 發現沒辦法進行格式上的排版，若把 python 往後 indent，就會出現 error，對於我這種排版狂人看了是有點刺眼啊（笑

---

## 讓 Python 與 HTML Tags 溝通
- 利用 `pyscript.write()` 這個方法來將計算結果傳入指定的 `id`  tag 中

<img src="images/writeToTags.png" width="600">

```html
<html>
    <head>
      <link rel="stylesheet" href="https://pyscript.net/alpha/pyscript.css" />
      <script defer src="https://pyscript.net/alpha/pyscript.js"></script>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" crossorigin="anonymous">
    </head>

  <body>
    <b><p>Today is <u><label id='today'></label></u></p></b>
    <br>
    <div id="pi" class="alert alert-primary"></div>
    <py-script>
import datetime as dt
pyscript.write('today', dt.date.today().strftime('%A %B %d, %Y'))

def wallis(n):
    pi = 2
    for i in range(1,n):
        pi *= 4 * i ** 2 / (4 * i ** 2 - 1)
    return pi

pi = wallis(100000)
pyscript.write('pi', f'π is approximately {pi:.3f}')
    </py-script>
  </body>
</html>
```

- 這部分終於開始有種「哦～好像有點實用！」的感覺出現了

---

## 試著呈現「視覺化圖表」

<img src="images/libraryPlot.png" width="600">

```html
<html>
    <head>
      <link rel="stylesheet" href="https://pyscript.net/alpha/pyscript.css" />
      <script defer src="https://pyscript.net/alpha/pyscript.js"></script>

      <py-env>
        - numpy
        - matplotlib
      </py-env>

    </head>

  <body>

    <h1>Let's plot random numbers</h1>
    <div id="plot"></div>
    <py-script output="plot">
import matplotlib.pyplot as plt
import numpy as np

x = np.random.randn(1000)
y = np.random.randn(1000)

fig, ax = plt.subplots()
ax.scatter(x, y)
fig
    </py-script>
  </body>
</html>
```
- 嗯...跑有點久
- 需要使用 `<py-env>` 來指名需要另外 import 進來的 Python Library (如上方程式碼)
- 另外也可以匯入 `.whl` 檔案，範例如下
    ```html
    <py-env>
    - './static/wheels/travertino-0.1.3-py3-none-any.whl'
    </py-env>
    ```
- 如果沒有你想要的，也可以去 [Pyodide](https://github.com/pyodide/pyodide) 提出 PR 或 issue ([pyodide packages 列表](https://github.com/pyodide/pyodide/tree/main/packages))

---

## 匯入自己定義的 py 檔

<img src="images/importLocalFile.png" width="600">

```html
<html>
    <head>
      <link rel="stylesheet" href="https://pyscript.net/alpha/pyscript.css" />
      <script defer src="https://pyscript.net/alpha/pyscript.js"></script>
      <py-env>
        - numpy
        - matplotlib
        - paths:
          - ./data.py
      </py-env>
    </head>

  <body>
    <h1>Let's plot random numbers</h1>
    <div id="plot"></div>
    <py-script output="plot">
import matplotlib.pyplot as plt
from data import make_x_and_y

x, y = make_x_and_y(n=1000)

fig, ax = plt.subplots()
ax.scatter(x, y)
fig
    </py-script>
  </body>
</html>
```

- 這個範例在自己本機上測試時會無法執行，因為在 web server 上會有找不到該檔案的問題
- 解決方法是
  1. 先使用 `python3 -m http.server` 開啟一個 Local Server
  2. 前往 http://localhost:8000/
  3. 於畫面中點選開啟 HTML 檔就可以看到順利執行的圖了

## 其他有趣應用
### 直接匯入 py 檔並執行
```html
<py-script src="./test.py"></py-script>
```
### 互動式 Python（類似 Jupyter notebook 功能）
![py_repl](images/py_repl.png)
```html
<py-repl id="my-py-repl" auto-generate="true"></py-repl>
```
- auto-generate="true": 按下 shift enter 執行時會自動產生下一行新的 cell


### Mario
https://github.com/pyscript/pyscript/tree/main/pyscriptjs/examples/mario
![mario](images/mario.png)

## 個人感想

### 等待時間過長

- 在測試的過程，一直困擾我的就是執行速度
  
- 雖然說這樣開網頁很方便，但這個等待的時間總覺得不是網頁該有的狀況...

### 可能需要更貼心的提示

- 另一個問題是，在執行時可能還是需要一個「執行中」之類的提示
  
- 不然等待的時候都不知道程式是已經掛了還是還沒跑出來
- 在發生 Error 時如果能直接顯示在畫面上更好
- 現在的情況就需要另外寫 `print` 或是 `try` 去抓錯誤 

### Dubug 上的優化

- 因為 PyScript 是在 HTML 中寫 Python，所以就算我用 VSCode 編輯，也很難做 debug

- 在 HTML 中不會有 python syntax highlight，也沒辦法套用 .py 相關的 extensions
- 考量到這個問題，若要開發的話我可能會先在別的 .py 檔裡面先編輯好
- 再利用 `<py-env>` 去匯入寫好的 method/functions 可能會比較方便

### 總結

- 目前個人覺得 PyScript 可能還僅限於個人專案使用，或是公司內部分享程式碼的成果的時候使用，感覺還有點難運用到產品上
  
- 現階段 Python 的 Web Application 可能還是要選擇像是 Django 或 Flask 之類的網頁框架
  
- 畢竟網頁的使用者不太可能願意等待過長的執行時間