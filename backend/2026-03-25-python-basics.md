# Python 後端開發（從零開始）- Day 1

## 📚 【今日學習：Python 基礎語法】

> 提示：這是專為程式設計初學者設計的課程。我們會用生活化的比喻，讓抽象的程式概念變得容易理解。請跟著範例動手操作，程式這種技能一定要「寫」才能學會！

---

## 1️⃣ 基礎概念（用生活比喻，越詳細越好）

### 🎯 什麼是程式碼？

**比喻**：程式碼就像是一本「料理食譜」。食譜會告訴你：
- 需要哪些食材（資料）
- 先做什麼、再做什麼（流程控制）
- 最終做出什麼菜（輸出結果）

電腦就像一位很聽話的廚師，你寫什麼食譜，它就照著做。

### 🎯 變數（Variable）= 貼標籤的盒子

**比喻**：想象你家裡有很多箱子（記憶體空間），你在每個箱子上貼一張標籤（變數名），然後把東西放進去。
- 箱子裡的「東西」就是「資料」
- 箱子上的「標籤」就是「變數名稱」

```python
# 就像把蘋果放進標註著 "水果" 的箱子裡
水果 = "蘋果"  # 箱子 "水果" 裡放了一個叫 "蘋果" 的東西
```

### 🎯 資料型態（Data Type）= 盒子的種類

不同的東西要放不同的盒子：
- **字串（str）**：放文字，像信封
- **整數（int）**：放整數，像量杯
- **浮點數（float）**：放小數，像精密測量儀
- **布林（bool）**：放真假，像開關

**比喻**：就像廚房裡有不同的容器：
- 保鮮盒放湯（字串）
- 量杯放毫升數（整數）
- 磅秤放重量（浮點數）
- 燈的開關（布林）

### 🎯 流程控制 = 料理的步驟

**if/else（如果...否則...）**：
就像問朋友「你要吃什麼？」
- 如果他說「我要吃牛肉麵」，就去牛肉麵店
- 否則（else），就去自助餐

**for 迴圈（重複做）**：
就像今天你要洗 10 個碗
- 「對每一個碗，都執行『洗乾淨』這個動作」

**while 迴圈（當...時重複）**：
就像「當鍋子裡的水還沒沸騰，就繼續加熱」

---

## 2️⃣ 語法解說（Python 程式碼，每行都要有中文註解）

### 📌 2.1 變數與基本資料型態

```python
# ========== 變數宣告 ==========
# Python 的特色：不需要先宣告變數類型，直接賦值即可
# 這就像直接拿一個箱子，把東西放進去，箱子會自動變成適合的類型

# 字串（String）- 用引號包住的文字
name = "小明"  # 變數 name 的值是 "小明" 這個文字
print(name)   # 印出：小明

# 整數（Integer）- 沒有小數點的數字
age = 18      # 變數 age 的值是 18
print(age)    # 印出：18

# 浮點數（Float）- 有小數點的數字
height = 175.5  # 變數 height 的值是 175.5
print(height)  # 印出：175.5

# 布林（Boolean）- 只有 True（真）或 False（假）
is_student = True   # 變數 is_student 為真（是學生）
is_tired = False    # 變數 is_tired 為假（不累）
print(is_student)   # 印出：True
print(is_tired)     # 印出：False
```

### 📌 2.2 型態查詢與轉換

```python
# ========== 查詢變數的類型 ==========
# 用 type() 函式可以知道變數是什麼類型
x = 10
y = "hello"
z = 3.14

print(type(x))  # 印出：<class 'int'> → 整數
print(type(y))  # 印出：<class 'str'> → 字串
print(type(z))  # 印出：<class 'float'> → 浮點數

# ========== 型態轉換（ Casting ） ==========
# Python 可以讓不同類型的資料互相轉換

# 字串 → 整數
num_str = "100"      # 這是字串 "100"，不是數字
num_int = int(num_str)  # 用 int() 轉成整數 100
print(num_int + 5)   # 印出：105

# 整數 → 字串
age = 20
age_str = str(age)   # 用 str() 轉成字串 "20"
print("我 " + age_str + " 歲")  # 印出：我 20 歲

# 整數 → 浮點數
a = 5
a_float = float(a)   # 用 float() 轉成 5.0
print(a_float)       # 印出：5.0
```

### 📌 2.3 if / else 條件判斷

```python
# ========== if（如果）/ else（否則）語法 ==========
# 語法結構：
# if 條件:
#     執行這裡（當條件為 True 時）
# else:
#     執行這裡（當條件為 False 時）

# 範例：判斷年齡是否成年
age = 20

if age >= 18:  # 如果 age 大於或等於 18
    print("你是成年人")  # 印出：你成年了
else:          # 否則（年齡小於 18）
    print("你還未成年")  # 印出：你還未成年

# ========== elif（else if 的縮寫）==========
# 當有多個條件要檢查時使用

score = 85

if score >= 90:           # 如果分數 >= 90
    print("優等")
elif score >= 80:         # 否則如果分數 >= 80
    print("甲等")
elif score >= 70:         # 否則如果分數 >= 70
    print("乙等")
elif score >= 60:         # 否則如果分數 >= 60
    print("丙等")
else:                     # 上面都不符合
    print("不及格")

# ========== 邏輯運算子 ==========
# and（且）：兩個條件都要成立
# or（或）：任一個條件成立即可
# not（不）：反轉真假值

age = 25
has_money = True

# and：年齡大於 20 且有錢
if age > 20 and has_money:
    print("可以去旅行")

# or：年齡大於 18 或有錢
if age > 18 or has_money:
    print("可以買東西")

# not：年齡不是 18 歲
if not age == 18:
    print("你不是 18 歲")
```

### 📌 2.4 for 迴圈（重複執行）

```python
# ========== for 迴圈 ==========
# 語法：for 變數 in 序列:
#       執行這裡

# 範例 1：遍歷字串（把每個字逐一處理）
text = "Python"
for char in text:  # char 會依序變成 'P', 'y', 't', 'h', 'o', 'n'
    print(char)    # 印出每個字

# 範例 2：遍歷數字範圍（range 產生一個數字序列）
# range(5) 產生 0, 1, 2, 3, 4（共 5 個數字）
for i in range(5):
    print(f"第 {i+1} 次迴圈")  # f-string：{} 裡面放變數

# 範例 3：遍歷列表（下一章節會詳細教）
fruits = ["蘋果", "香蕉", "橘子"]
for fruit in fruits:  # fruit 會依序變成每個水果
    print(fruit)

# ========== 迴圈控制 ==========
# break：跳出迴圈（不再繼續）
# continue：跳過這一次，繼續下一次

# break 範例：找到目標後停止
numbers = [1, 2, 3, 4, 5]
for n in numbers:
    if n == 3:
        print("找到 3，跳出迴圈")
        break          # 找到後停止，不再繼續
    print(f"檢查中：{n}")

# continue 範例：跳過特定的數
for i in range(5):
    if i == 2:
        continue        # 跳過 i=2 的這一次，不執行下面的 print
    print(f"數字：{i}")
# 輸出：數字：0, 數字：1, 數字：3, 數字：4（跳過 2）
```

### 📌 2.5 while 迴圈（當...時重複）

```python
# ========== while 迴圈 ==========
# 語法：while 條件:
#       執行這裡（當條件為 True 時重複）

# 範例 1：基本 while
count = 0
while count < 3:        # 當 count 小於 3 時重複
    print(f"count = {count}")
    count = count + 1   # 每次加 1
# 輸出：count = 0, count = 1, count = 2

# 範例 2：使用 break 結束迴圈
num = 1
while True:            # 永遠為 True（無限迴圈），靠 break 結束
    print(num)
    if num >= 5:
        print("達到 5，結束")
        break          # 跳出迴圈
    num = num + 1

# 範例 3：使用者輸入驗證（模擬）
# 假設密碼是 "1234"
correct_password = "1234"
input_password = ""

while input_password != correct_password:
    input_password = input("請輸入密碼：")  # 模擬輸入
    if input_password != correct_password:
        print("密碼錯誤，請重試")

print("登入成功！")
```

---

## 3️⃣ 實際操作（一步一步帶著做）

### 🖥️ 步驟 1：安裝 Python（如果還沒安裝）

```bash
# 檢查是否已安裝 Python
python3 --version

# 如果顯示類似 "Python 3.11.0" 表示已安裝
# 如果沒有，請到官網下載：https://www.python.org/
```

### 🖥️ 步驟 2：建立第一個 Python 檔案

```bash
# 建立一個名為 hello.py 的檔案
touch hello.py
```

### 🖥️ 步驟 3：寫入程式碼

```python
# 用編輯器打開 hello.py，寫入以下內容：

# 我的第一支 Python 程式
# 作者：小明
# 日期：2026-03-25

# ====== 基本變數 ======
name = "Python 初學者"  # 名字
age = 1                 # 年齡（歲）
is_learning = True      # 正在學習嗎？（是）

# 印出自我介紹
print(f"大家好，我叫 {name}")
print(f"我學 Python {age} 年了")
print(f"我正在學習：{is_learning}")

# ====== 簡易計算 ======
a = 10
b = 3

print(f"\n===== 簡易計算 =====")
print(f"{a} + {b} = {a + b}")   # 加法
print(f"{a} - {b} = {a - b}")   # 減法
print(f"{a} * {b} = {a * b}")   # 乘法
print(f"{a} / {b} = {a / b}")   # 除法
print(f"{a} // {b} = {a // b}") # 整數除法（取商）
print(f"{a} % {b} = {a % b}")   # 取餘數

# ====== if/else 練習 ======
score = 75

print(f"\n===== 成績評定 =====")
print(f"你的分數：{score}")

if score >= 90:
    print("等級：優等")
elif score >= 80:
    print("等級：甲等")
elif score >= 70:
    print("等級：乙等")
elif score >= 60:
    print("等級：丙等")
else:
    print("等級：需要再加強")

# ====== for 迴圈練習 ======
print(f"\n===== 顯示 1 到 5 =====")
for i in range(1, 6):  # range(1, 6) 產生 1, 2, 3, 4, 5
    print(f"數字：{i}")

# ====== while 迴圈練習 ======
print(f"\n===== 倒數計時 =====")
countdown = 5
while countdown > 0:
    print(f"倒數：{countdown}...")
    countdown = countdown - 1
print("時間到！")
```

### 🖥️ 步驟 4：執行程式

```bash
# 在終端機執行
python3 hello.py
```

**預期輸出**：
```
大家好，我叫 Python 初學者
我學 Python 1 年了
我正在學習：True

===== 簡易計算 =====
10 + 3 = 13
10 - 3 = 7
10 * 3 = 30
10 / 3 = 3.3333333333333335
10 // 3 = 3
10 % 3 = 1

===== 成績評定 =====
你的分數：75
等級：乙等

===== 顯示 1 到 5 =====
數字：1
數字：2
數字：3
數字：4
數字：5

===== 倒數計時 =====
倒數：5...
倒數：4...
倒數：3...
倒數：2...
倒數：1...
時間到！
```

---

## 4️⃣ 常見錯誤提醒

### ❌ 錯誤 1：縮排錯誤（IndentationError）

```python
# 錯誤示範
if age > 18:
print("成年人")  # 這行沒有縮排，會報錯！

# 正確寫法
if age > 18:
    print("成年人")  # 前面要有 4 個空格（或 Tab）
```

**💡 提示**：Python 用縮排來表示程式碼的「層次關係」，這是 Python 的特色！

### ❌ 錯誤 2：變數名稱寫錯（NameError）

```python
name = "小明"
print(nmae)  # 拼錯了！是 name 不是 nmae
```

### ❌ 錯誤 3：型態不匹配（TypeError）

```python
age = 20
text = "我今年" + age + "歲"  # 錯誤！字串 + 整數會報錯

# 正確寫法
text = "我今年" + str(age) + "歲"  # 把 age 轉成字串
# 或
text = f"我今年{age}歲"  # 用 f-string
```

### ❌ 錯誤 4：除以零（ZeroDivisionError）

```python
result = 10 / 0  # 會報錯：除數不能為 0
```

### ❌ 錯誤 5：迴圈忘記終止（Infinite Loop）

```python
# 糟糕的範例：永遠不會停止的迴圈
# count = 0
# while count < 10:
#     print(count)  # 忘記遞增，會永遠執行
```

**💡 提示**：使用 `while` 時要小心，確保條件最終會變成 `False`！

---

## 5️⃣ 今日小任務

### 📝 任務 1：計算BMI（身體質量指數）

BMI = 體重(公斤) / 身高(公尺)²

請寫一個 Python 程式：
1. 宣告兩個變數：`weight`（體重）和 `height`（身高）
2. 計算 BMI
3. 根據 BMI 顯示體重狀態：
   - BMI < 18.5 → 「體重過輕」
   - 18.5 <= BMI < 24 → 「正常」
   - 24 <= BMI < 27 → 「過重」
   - BMI >= 27 → 「肥胖」

**提示**：記得把身高從「公分」轉換成「公尺」再計算！

### 📝 任務 2：印出九九乘法表

用 for 迴圈印出：
```
1 x 1 = 1
1 x 2 = 2
...
9 x 9 = 81
```

**提示**：使用巢狀迴圈（迴圈裡面再包迴圈）

### 📝 任務 3：猜數字遊戲（使用 while）

請寫一個「猜數字」遊戲：
1. 電腦先選一個 1~100 的隨機數字
2. 讓使用者一直猜，直到猜中為止
3. 每次猜測後提示「太大了」或「太小了」

**提示**：需要使用 `random` 模組（明天會教），你可以先試著自己寫寫看！

---

## 📚 今日學習重點回顧

| 概念 | 說明 |
|------|------|
| 變數 | 就像貼標籤的盒子，用來存放資料 |
| 資料型態 | str(字串)、int(整數)、float(浮點數)、bool(布林) |
| if/else | 條件判斷，就像「如果...否則...」 |
| for | 已知次數的重複，用 `range()` |
| while | 未知次數的重複，當條件成立時重複 |
| type() | 查詢變數的類型 |
| str/int/float | 型態轉換函式 |

---

**🎉 恭喜完成 Day 1！明天我們會學習「函式與模組」，敬請期待！**

> 記住：寫程式最重要的就是「動手做」，只看不做是學不會的喔！