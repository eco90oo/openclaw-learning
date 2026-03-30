# 📚 Day 3：JavaScript 基礎語法 — 像說故事一样認識程式邏輯

> **適合對象**：完全沒碰過程式的人  
> **預期收穫**：學會變數、迴圈、函式的概念，能寫出第一段 JavaScript 程式碼  
> **前置知識**：建議先看過 Day 1 HTML 基礎

---

## 1️⃣ 基礎概念：用「餐廳點餐」比喻 JavaScript

### 🎯 什麼是 JavaScript？

**JavaScript = 網頁的靈魂**，讓靜態的 HTML + CSS 開始「會動」。

### 🏪 比喻：餐廳的故事

想象你走進一家餐廳：

```
┌─────────────────────────────────────────────────┐
│                  餐廳比喻                        │
├─────────────────────────────────────────────────┤
│                                                   │
│  HTML = 餐廳的建築物、桌椅、菜單（靜態的）        │
│                                                   │
│  CSS = 裝潢風格、盤子設計、顏色配置（好看的）      │
│                                                   │
│  JavaScript = 服務生、廚師、結帳系統（會動的）    │
│                                                   │
└─────────────────────────────────────────────────┘
```

沒有 JavaScript 的網頁，就像一家沒有服務生的餐廳：
- 菜單擺在那裡（HTML）
- 裝潢很漂亮（CSS）
- 但沒人幫你點餐、沒人幫你做菜 → 餐廳是「死」的

加上 JavaScript 後，網頁就能：
- 對使用者的點擊做出反應（按鈕事件）
- 跟伺服器要資料（fetch API）
- 自動更新畫面（DOM 操作）

### 📝 什麼是「程式」？

**程式 = 一步一步的指令**，就像食譜：
```
食譜（程式）：
1. 拿出雞蛋（宣告變數）
2. 打蛋（執行操作）
3. 加熱（重複動作）
4. 裝盤（輸出結果）
```

### 🎯 為什麼要學 JavaScript？

1. **所有網站都用**：Google、Facebook、YouTube 全部靠 JavaScript 運作
2. **入門門檻低**：不需要安裝任何東西，瀏覽器就能執行
3. **可以做很多事**：前端、後端（Node.js）、手機 App、遊戲

---

## 2️⃣ 核心知識：JavaScript 基本語法

### 第一步：認識「變數」——像替東西貼標籤

**變數 = 儲存資料的盒子**，每個盒子有名字。

#### 生活比喻：變數就像便利貼

```
冰箱裡有很多東西，我們貼上便利貼標記：

🍎 蘋果 → 便利貼寫「水果」
🥚 雞蛋 → 便利貼寫「蛋白質」

便利貼（變數名） → 內容（變數值）
```

#### 怎麼宣告變數？

JavaScript 有三種方式宣告變數：

```javascript
// 方法一：let（現代標準，推薦使用）
// 想像：拿一張便利貼，寫上名字「name」，內容是「小明」
let name = "小明";
// let = 「我要建立一個盒子」
// name = 盒子的名字（便利貼上的字）
// "小明" = 盒子裡裝的內容

// 方法二：const（常數，值不能改變）
// 想像：拿一張便利貼，寫上「PI」，內容是「3.14」，永遠不能改
const PI = 3.14;
// const = 「我要建立一個盒子，裡面的值固定不變」
// PI = 盒子名字
// 3.14 = 固定的值

// 方法三：var（旧時代用法，不推薦）
// 跟 let 很像，但有一些奇怪的行為，現代寫法不建議用
var age = 25;
```

#### 變數的資料類型

```javascript
// 字串（String）→ 一串文字，用引號包起來
let message = "你好，今天天氣很好！";
// 字串就像：「這是一句話」，用雙引號或單引號都可以

// 數字（Number）→ 整數或小數都可以
let score = 98.5;
// 數字就像：「分數、溫度、年齡」，不用加引號

// 布林（Boolean）→ 只有兩個可能：true（真）或 false（假）
let isRaining = false;
// 布林就像：「是/否」的開關
// true = 是，正在下雨
// false = 否，沒在下雨

// 空值（null / undefined）→ 「沒有東西」
let nothing = null;
// null = 「我故意的，現在沒有值」（明確設定為空）
let notSet = undefined;
// undefined = 「根本沒設定過值」（預設狀態）

// 陣列（Array）→ 就像一個多層收納盒，放很多東西
let fruits = ["蘋果", "香蕉", "橘子"];
// 陣列就像：[第一格放蘋果, 第二格放香蕉, 第三格放橘子]
// 用方括號 [] 包起來，項目之間用逗點 , 分隔

// 物件（Object）→ 就像一本記錄簿，記錄很多相關的資訊
let student = {
    name: "小明",      // 名字欄位
    age: 18,           // 年齡欄位
    grade: "A"         // 成績欄位
};
// 物件就像：一本個人資料卡
// 用大括號 {} 包起來，格式是「鍵: 值」
```

---

### 第二步：認識「運算子」——數學與比較

```javascript
// ===== 算術運算子（數學計算） =====

// 加法：+
let sum = 5 + 3;  // sum = 8
// 就像：5 + 3 = 8

// 減法：-
let diff = 10 - 4;  // diff = 6
// 就像：10 - 4 = 6

// 乘法：*
let product = 6 * 7;  // product = 42
// 就像：6 × 7 = 42

// 除法：/
let quotient = 20 / 4;  // quotient = 5
// 就像：20 ÷ 4 = 5

// 取餘數（modulo）：%
let remainder = 17 % 5;  // remainder = 2
// 17 ÷ 5 = 3...2，所以 remainder = 2
// 這個常用來判斷「是否為偶數」：n % 2 === 0


// ===== 比較運算子（比較大小，返回布林值） =====

// 等於（值相等）：==
let isEqual = 5 == "5";  // isEqual = true
// == 會忽略資料類型，「5」和 5 被認為是相同的

// 嚴格等於（值和類型都相等）：===
let isStrictEqual = 5 === "5";  // isStrictEqual = false
// === 會檢查資料類型，5（數字）和 "5"（字串）是不一樣的
// 建議：永遠用 === 而不用 ==

// 不等於：!=
let isNotEqual = 5 != "5";  // isNotEqual = false
// 5 和 "5" 相等，所以「不相等」是 false

// 大於 / 小於：> / <
let isGreater = 10 > 5;  // isGreater = true
let isLess = 3 < 1;     // isLess = false

// 大於等於 / 小於等於：>= / <=
let isGreaterOrEqual = 5 >= 5;  // isGreaterOrEqual = true
```

---

### 第三步：認識「條件判斷」——像十字路口

**條件判斷 = 根據情況做不同的事**。

#### 生活比喻：紅綠燈

```
十字路口（if-else）：
如果（if）紅燈 → 停下來
否則如果（else if）黃燈 → 減速
否則（else）→ 綠燈，前進
```

#### 實際語法

```javascript
// ===== 基本 if-else =====
let temperature = 30;

// 如果溫度大於 35 度，就執行裡面的程式碼
if (temperature > 35) {
    // 這個大括號裡的程式碼，只有在條件成立時才會執行
    console.log("太熱了！要開冷氣！");
    // console.log = 在控制台印出訊息（除錯用）
}
// 如果溫度 <= 35，就跳過上面的 block，檢查下一個條件

// 否則如果（else if）：第二個、第三個...條件
else if (temperature > 25) {
    // 只有當溫度在 26~35 度之間時才執行
    console.log("天氣不錯，很舒服");
}

// 否則（else）：所有條件都不成立時執行
else {
    console.log("天氣有點冷，穿外套");
}

// ===== 多條件：邏輯運算子 =====

// 且（AND）：&& — 兩個條件都要成立
if (temperature > 20 && temperature < 35) {
    // 當溫度在 20~35 度之間（兩邊都要滿足）
    console.log("完美天氣！");
}

// 或（OR）：|| — 至少一個條件成立
if (temperature < 10 || temperature > 40) {
    // 當溫度小於 10 度 OR 大於 40 度（滿足一個就執行）
    console.log("極端天氣！");
}

// 不（NOT）：! — 把布林值反過來
if (!isRaining) {
    // 如果 isRaining 是 false，!isRaining 就是 true
    console.log("沒下雨，出門吧！");
}

// ===== switch：多選一的情況 =====
// 如果有太多 else if，可以改成 switch
let day = "星期一";

switch (day) {
    case "星期一":
        console.log("週一了，打起精神！");
        break;  // break = 離開 switch，不再繼續檢查
    case "星期五":
        console.log("週五行將來！");
        break;
    case "星期六":
    case "星期日":
        console.log("週末，放鬆一下！");
        break;
    default:
        console.log("普通的日子");
        // default = 如果沒有任何 case 匹配，執行這裡
}
```

---

### 第四步：認識「迴圈」——自動重複做事

**迴圈 = 幫你做重複的事情**，不用一直寫相同的程式碼。

#### 生活比喻：複印機

```
想像你要發傳單給 100 個人：

沒有迴圈的做法（痛苦）：
印傳單給第 1 個人
印傳單給第 2 個人
印傳單給第 3 個人
...（重複 100 次）

有迴圈的做法（輕鬆）：
重複 100 次：「印傳單給下一個人」

迴圈 = 自動化的重複
```

#### for 迴圈：知道要跑幾次時用

```javascript
// ===== 基本 for 迴圈 =====
// 想像：倒數 1 到 5

// for (起始; 條件; 每次做完後要做什麼)
for (let i = 1; i <= 5; i++) {
    // let i = 1 → 起始值：從 1 開始
    // i <= 5 → 條件：只要 i <= 5 就繼續
    // i++ → 每次跑完後：把 i 加 1（i++ 等於 i = i + 1）
    
    console.log("倒數：" + i);
    // + 號連接字串和數字，印出：
    // "倒數：1"
    // "倒數：2"
    // "倒數：3"
    // "倒數：4"
    // "倒數：5"
}

// ===== 用迴圈跑陣列 =====
let fruits = ["蘋果", "香蕉", "橘子", "葡萄"];

// 方法一：用 length（長度）配合迴圈
for (let i = 0; i < fruits.length; i++) {
    // i = 0, 1, 2, 3（陣列的索引，0 開始）
    // fruits.length = 4（陣列有 4 個元素）
    
    console.log("我喜歡吃：" + fruits[i]);
    // fruits[0] = "蘋果"
    // fruits[1] = "香蕉"
    // fruits[2] = "橘子"
    // fruits[3] = "葡萄"
}

// 方法二：用 for...of（更簡潔的寫法）=====
// 如果不需要索引，只想要每個元素，用 for...of 更直觀
for (let fruit of fruits) {
    // fruit 會依序變成："蘋果"、"香蕉"、"橘子"、"葡萄"
    console.log("水果：" + fruit);
}
```

#### while 迴圈：不確定要跑幾次時用

```javascript
// ===== while 迴圈 =====
// while = 「當...的時候」

let energy = 3;

// 當 energy > 0 的時候就一直跑
while (energy > 0) {
    // 每次迴圈開始時檢查條件
    console.log("還有 " + energy + " 格能量，持續戰鬥！");
    
    energy--;  // 消耗 1 格能量（energy-- 等於 energy = energy - 1）
    
    // 如果忘記 energy--，迴圈會永遠跑下去（無窮迴圈）！
}
// 當 energy 變成 0 時，條件不再成立，迴圈停止


// ===== do...while：至少執行一次再判斷 =====
// do = 做...（先做再說）
let count = 0;

do {
    console.log("至少會跑一次：" + count);
    count++;
} while (count < 0);
// 這個迴圈會至少印一次，即使 count < 0 不成立
```

#### 迴圈控制：break 和 continue

```javascript
// ===== break：強制停止迴圈 =====
for (let i = 1; i <= 10; i++) {
    if (i === 5) {
        console.log("遇到 5 了，不跑了！");
        break;  // 立刻停止迴圈，跳出
    }
    console.log(i);
}
// 只會印出：1, 2, 3, 4, "遇到 5 了，不跑了！"


// ===== continue：跳過這一次，繼續下一輪 =====
for (let i = 1; i <= 5; i++) {
    if (i === 3) {
        console.log("跳過 3！");
        continue;  // 跳過這次迴圈的剩餘程式碼，繼續下一輪
    }
    console.log("數字：" + i);
}
// 會印出：數字：1, 數字：2, 跳過 3！, 數字：4, 數字：5
```

---

### 第五步：認識「函式」——把常用的動作包裝起來

**函式 = 可重複使用的程式碼區塊**。

#### 生活比喻：微波爐

```
微波爐（函式）：
輸入（參數）：食物
處理：加熱 2 分鐘
輸出（回傳值）：熱好的食物

你不需要知道微波爐怎麼運作，只需要：
1. 把食物放進去（傳入參數）
2. 按開始（呼叫函式）
3. 拿出熱好的食物（取得回傳值）
```

#### 函式的三種寫法

```javascript
// ===== 方法一：function 宣告（傳統）=====
// 這個函式會把兩個數字相加並回傳結果

function 加法(數字一, 數字二) {
    // function = 「我要定義一個函式」
    // 加法 = 函式的名字（可以自己取）
    // (數字一, 數字二) = 參數列表（傳入的資料）
    
    let 結果 = 數字一 + 數字二;
    // 把兩個數字相加，存到結果變數
    
    return 結果;
    // return = 「把這個值回傳出去」，結束函式
}

// 使用這個函式
let answer = 加法(3, 5);
console.log("3 + 5 = " + answer);  // 印出：3 + 5 = 8


// ===== 方法二：箭頭函式（現代推薦）=====
// 箭頭函式是 function 的簡寫，適合短小的函式

// 原本的寫法：
function 乘法(a, b) {
    return a * b;
}

// 改成箭頭函式：
const 乘法 = (a, b) => {
    // const 乘法 = 建立一個常數，值是這個函式
    // (a, b) => 箭頭前面是參數，箭頭後面是函式內容
    return a * b;
};

// 更簡短：如果函式只有一行，可以省略大括號和 return
const 乘法簡寫 = (a, b) => a * b;


// ===== 方法三：匿名函式（沒名字的函式）=====
// 沒有名字的函式，通常用在回調（callback）場景
const 顯示訊息 = function(message) {
    // 這是一個匿名函式，存到變數裡
    console.log(message);
};

顯示訊息("你好！");  // 印出：你好！
```

#### 函式的實際應用

```javascript
// ===== 計算水果價格 =====
function 計算水果总价(水果名稱, 數量, 單價) {
    // 這支函式接收三個參數：水果名稱、數量、單價
    
    let 總價 = 數量 * 單價;
    // 總價 = 數量 × 單價
    
    let 訊息 = "購買 " + 水果名稱 + "：" + 數量 + " 個，共 " + 總價 + " 元";
    // 把結果組合成一個字串
    
    return 訊息;  // 回傳這個訊息
}

// 使用
let 發票 = 計算水果总价("蘋果", 5, 30);
console.log(發票);
// 印出：購買 蘋果：5 個，共 150 元


// ===== 預設參數（參數可以有預設值）=====
function 打招呼(名字 = "陌生人") {
    // 如果呼叫時沒傳名字，就用 "陌生人" 當預設值
    return "你好，" + 名字 + "！";
}

console.log(打招呼());         // 印出：你好，陌生人！
console.log(打招呼("小明"));   // 印出：你好，小明！


// ===== 巢狀函式（函式裡面包函式）=====
function 外部函式() {
    let message = "我是外部的";
    
    function 內部函式() {
        let inner = "我是內部的";
        console.log(message + " 和 " + inner);
        // 內部函式可以讀到外部函式的變數（閉包概念）
    }
    
    內部函式();  // 在外部呼叫內部
}

外部函式();  // 印出：我是外部的 和 我是內部的
```

---

### 附加：常見的 JavaScript 方法

```javascript
// ===== 字串操作 =====
let text = "Hello, World!";

// 取長度
console.log(text.length);  // 13（字串有 13 個字元）

// 轉大寫/小寫
console.log(text.toUpperCase());  // HELLO, WORLD!
console.log(text.toLowerCase());  // hello, world!

// 找字串
console.log(text.includes("World"));  // true（有沒有包含 World）
console.log(text.indexOf("World"));   // 7（World 在第 7 個位置，0 開始）

// 取代
console.log(text.replace("World", "JavaScript"));  // Hello, JavaScript!

// 切開
console.log(text.split(", "));  // ["Hello", "World!"]（用逗點切開成陣列）


// ===== 陣列操作 =====
let numbers = [1, 2, 3, 4, 5];

// 加入/移除元素
numbers.push(6);     // 在最後加入 6 → [1,2,3,4,5,6]
numbers.pop();       // 移除最後一個 → [1,2,3,4,5]
numbers.unshift(0);  // 在最前面加入 0 → [0,1,2,3,4,5]
numbers.shift();     // 移除最前面一個 → [1,2,3,4,5]

// 找元素
console.log(numbers.includes(3));  // true（有沒有 3）
console.log(numbers.indexOf(4));    // 3（第 4 個位置，0 開始）

// 遍歷（對每個元素做什麼）
numbers.forEach(function(num) {
    console.log(num);
});
// 印出：1, 2, 3, 4, 5

// 映射（把每個元素轉換成另一個值）=====
let doubled = numbers.map(function(num) {
    return num * 2;
});
console.log(doubled);  // [2, 4, 6, 8, 10]

// 篩選（只留下符合條件的）=====
let evens = numbers.filter(function(num) {
    return num % 2 === 0;  // 只留下偶數
});
console.log(evens);  // [2, 4]


// ===== 數字操作 =====
let price = 99.567;

// 四捨五入
console.log(Math.round(price));  // 100

// 取到小數點後兩位
console.log(price.toFixed(2));    // "99.57"（字串）

// 亂數
console.log(Math.random());  // 0 到 1 之間的隨機小數
console.log(Math.floor(Math.random() * 10));  // 0 到 9 的隨機整數
```

---

## 3️⃣ 實際操作：動手寫 JavaScript

### 你的第一個 JavaScript 程式

現在我們要把 JavaScript 加入 HTML，讓網頁有互動性：

**Step 1：建立 `js-demo.html`**

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>我的第一個 JavaScript 程式</title>
    
    <!-- 在 head 裡面寫 JavaScript 也可以，但通常放在 body 最下面比較好 -->
</head>
<body>
    <h1>🎉 JavaScript 互動網頁</h1>
    
    <!-- 按鈕：點擊後會觸發 JavaScript -->
    <button onclick="sayHello()">點我打招呼</button>
    
    <!-- 顯示結果的地方 -->
    <p id="output">（這裡會顯示結果）</p>
    
    <hr>
    
    <h2>📊 計算機</h2>
    <input type="number" id="num1" placeholder="第一個數字">
    +
    <input type="number" id="num2" placeholder="第二個數字">
    <button onclick="calculate()">計算</button>
    <p id="calcResult"></p>

    <!-- 👇 這裡開始寫 JavaScript 👇 -->
    <script>
        // ===== 第一個函式：打招呼 =====
        function sayHello() {
            // 這是一個函式，叫 sayHello
            
            let name = prompt("請輸入你的名字：");
            // prompt = 彈出一個輸入框，讓使用者打字
            // 使用者打的內容會存進 name 變數
            
            let greeting = "你好，" + name + "！歡迎來學 JavaScript！";
            // 把字串連接起來
            
            document.getElementById("output").innerText = greeting;
            // document.getElementById("output") = 找到 id="output" 的 HTML 元素
            // .innerText = 改變那個元素裡面的文字
        }
        
        // ===== 第二個函式：簡單計算機 =====
        function calculate() {
            // 從 input 框取得使用者輸入的數字
            let n1 = document.getElementById("num1").value;
            // .value = 取得 input 框裡面的值（都是字串）
            
            let n2 = document.getElementById("num2").value;
            
            // 把字串轉成數字再相加
            let result = parseInt(n1) + parseInt(n2);
            // parseInt = 把字串轉成整數（如果是要小數用 parseFloat）
            
            // 顯示結果
            document.getElementById("calcResult").innerText = "結果是：" + result;
        }
    </script>
</body>
</html>
```

**Step 2：儲存並用瀏覽器打開**

1. 把上面的程式碼複製到新檔案 `js-demo.html`
2. 用瀏覽器打開
3. 點「點我打招呼」按鈕 → 會彈出輸入框
4. 輸入名字 → 會看到個人化問候
5. 嘗試用計算機功能

---

### 實作：簡單的體溫記錄器

```javascript
// ===== 體溫記錄器（完整示範）=====

// 步驟 1：建立陣列存放體溫資料
let 體溫記錄 = [];  // 一開始是空的

// 步驟 2：定義加入記錄的函式
function 加入體溫(溫度) {
    // 傳入一個溫度值，加到記錄陣列中
    
    體溫記錄.push(溫度);  // 把溫度加到陣列最後面
    // push = 「推入」一個元素到陣列末尾
    
    console.log("已記錄體溫：" + 溫度 + " 度");
}

// 步驟 3：定義顯示記錄的函式
function 顯示所有記錄() {
    console.log("===== 體溫記錄 =====");
    
    for (let i = 0; i < 體溫記錄.length; i++) {
        // i = 0, 1, 2... 直到 length-1
        console.log("第 " + (i + 1) + " 次：" + 體溫記錄[i] + " 度");
    }
    
    console.log("===== 共 " + 體溫記錄.length + " 筆 =====");
}

// 步驟 4：定義計算平均的函式
function 計算平均體溫() {
    let 總和 = 0;  // 先把總和設為 0
    
    for (let i = 0; i < 體溫記錄.length; i++) {
        總和 = 總和 + 體溫記錄[i];  // 把所有體溫加起來
    }
    
    let 平均 = 總和 / 體溫記錄.length;  // 總和除以筆數
    console.log("平均體溫：" + 平均.toFixed(1) + " 度");
    // .toFixed(1) = 取到小數點後一位
}

// 步驟 5：實際使用
加入體溫(36.5);  // 記錄 36.5 度
加入體溫(36.8);  // 記錄 36.8 度
加入體溫(37.0);  // 記錄 37.0 度
加入體溫(36.6);  // 記錄 36.6 度

顯示所有記錄();
// 印出：
// 第 1 次：36.5 度
// 第 2 次：36.8 度
// 第 3 次：37.0 度
// 第 4 次：36.6 度

計算平均體溫();
// 印出：平均體溫：36.7 度
```

---

## 4️⃣ 今日小任務

### ✅ 任務清單

- [ ] 建立 `js-demo.html` 檔案，貼上上面的程式碼
- [ ] 用瀏覽器打開，測試「打招呼」功能
- [ ] 測試「計算機」功能，嘗試 10 + 20
- [ ] 新增一個新按鈕，功能是：「點一下，畫面背景變成紅色」
  - 提示：用 `document.body.style.backgroundColor = "red"`
- [ ] 修改上面的「體溫記錄器」程式碼，再加入 3 筆體溫記錄
- [ ] （挑戰題）寫一個函式，能找出體溫記錄中的最高體溫

### 🎯 驗收標準

完成後，你應該能：
1. ✅ 理解「變數」是什麼，能宣告 let / const
2. ✅ 理解「if-else」條件判斷，會用 && 和 ||
3. ✅ 理解「for / while」迴圈，會跑陣列
4. ✅ 理解「函式」是什麼，能定義和呼叫函式
5. ✅ 能用 `<script>` 標籤把 JavaScript 加入 HTML

---

## 📚 下一天預告

**Day 4：DOM 操作 — 如何操控網頁元素**  
學會如何「用 JavaScript 改變 HTML 內容」——新增、刪除、修改網頁上的任何元素！

---

## 💡 小技巧

### console.log 是最好的老師

當你不知道變數的值對不對時，在程式中間加一行：
```javascript
console.log(你懷疑的變數);
```
然後按 F12 開啟開發者工具，看 Console 分頁，就能看到印出的結果。

### 「=」、「==」、「===」是不一樣的！

```javascript
// 一個 = → 賦值（把右邊的東西放進左邊）
let x = 5;  // 把 5 放進 x 這個盒子

// 兩個 == → 鬆散相等（只看值，不看類型）
5 == "5"  // true（會自動轉換類型）

// 三個 === → 嚴格相等（值和類型都要一樣）✅ 推薦
5 === "5" // false（數字和字串不同）
// 永遠用 === 來避免奇怪的 bug！
```

### 常見錯誤提醒

❌ `if (x = 5)` — 把 `=` 和 `==` 搞混了（新手最容易犯的錯）  
❌ 忘記 `let` / `const` 就直接用變數  
❌ `else if` 打成 `else if`（中間有空格，要分開）  
❌ 迴圈忘記遞增/遞減，導致無窮迴圈  
❌ 字串和數字混用沒轉換，導致結果變成 `"10" + 20 = "1020"`

---

*🎉 恭喜你完成 Day 3！JavaScript 是前端開發的核心，把變數、迴圈、函式搞懂，未來學 React 就輕鬆多了！*