# 📚 【今日學習：Day 8 - 交易基礎（Transaction）是什麼？）】

> 學習日期：2026-03-30
> 適合對象：學過 SQL 基礎，想了解「資料庫怎麼保證操作不會出錯」的人
> 前置知識：Day 1~3（資料庫基礎、SQL語法、表格設計）

---

## 1️⃣ 基礎概念（用生活比喻，越詳細越好）

### 🎯 什麼是 Transaction（交易）？

想像你今天去銀行臨櫃轉帳，你要做兩件事：
1. **從你的帳戶扣款**（-5000 元）
2. **把 5000 元加到對方帳戶**（+5000 元）

現在問你一個關鍵問題：**如果第一步完成後，銀行突然斷電了怎麼辦？**

你的 5000 元已經被扣掉了，但對方沒收到。這筆錢去哪了？**凭空消失了！**

> 這種「兩件事必須**同時成功或同時失敗**」的場景，就是 Transaction 要解決的核心問題。

### 🏦 用餐廳點菜比喻 Transaction

想像你在一家高檔餐廳點餐：
- **沒有 Transaction 的情況**：你點了牛排和紅酒，服務員先把牛排送到廚房，然後突然離職了，紅酒沒人處理。你等了30分鐘，最後只來了牛排，沒有紅酒，而且帳單還收了你紅酒的錢。
- **有 Transaction 的情況**：服務員把所有菜單同時交給廚房，**要嘛全部做完一起上桌，要嘛整單取消重新來**。這就是 Transaction 的精神——「一榮俱榮，一損俱損」。

### 📊 Transaction 的四大保證（ACID）

Transaction 提供四個保障，我們用「轉帳」場景一一說明：

| 特性 | 餐廳比喻 | 資料庫說法 |
|------|---------|-----------|
| **A**tomic（原子性） | 要嘛全部菜做完，要嘛全部不做 | 一組 SQL 視為**一個不可分割的整體** |
| **C**onsistency（一致性） | 最後桌子上的菜必須合理（不能沒付錢就有菜） | 交易前後，資料庫狀態**必須合法** |
| **I**solation（隔離性） | 隔壁桌的菜不會影響你的上菜順序 | 併發執行時，每筆交易**隔開執行** |
| **D**urability（持久性） | 廚房做完的菜，即使著火了也有單據證明 | 完成的交易結果，**永久保存**不會消失 |

### 🔑 為什麼要有 Transaction？

日常生活中的例子：
- **轉帳**：扣款和存款必須同時成功
- **網購下單**：庫存扣減、訂單建立、扣款，三件事必須同時完成
- **發文+扣積分**：論壇發文章同時扣你的積分

如果沒有 Transaction，系統在操作到一半時當機，你就會得到「半調子」的資料錯誤。這種錯誤在金融系統中是**災難性**的。

---

## 2️⃣ SQL 語法詳解（每行都有中文註解）

### 傳統語法：手動控制 Transaction

在 MySQL / PostgreSQL 中，預設每一條 SQL 語句都是**自動 commit（自動確認）**的。如果你想自己控制一組操作的邊界，有兩種方式：

#### 方式一：手動 BEGIN + COMMIT / ROLLBACK

```sql
-- ==========================================
-- 開始一筆交易（就像打開一個新的工作區域）
-- ==========================================
BEGIN;

-- 第一步：從 A 帳戶扣款
UPDATE accounts
   SET balance = balance - 5000    -- 余額扣掉 5000
 WHERE account_id = 1;              -- 條件：A 帳戶

-- 第二步：給 B 帳戶存款
UPDATE accounts
   SET balance = balance + 5000    -- 余額加上 5000
 WHERE account_id = 2;             -- 條件：B 帳戶

-- 一切都順利，確認這筆交易（把剛才的改動永久寫入硬碟）
COMMIT;
```

```sql
-- 如果中途發現問題，想要全部取消（還原到 BEGIN 之前的狀態）
-- ROLLBACK;
-- 執行後，這兩條 UPDATE 的改動全部消失，資料庫恢復原狀
```

#### 方式二：用 START TRANSACTION（功能相同，語法更明確）

```sql
-- ==========================================
-- START TRANSACTION 是 BEGIN 的現代語法
-- 兩者在多數資料庫中等價
-- ==========================================
START TRANSACTION;

-- 扣庫存（假設商品 ID=100 的庫存-1）
UPDATE products
   SET stock = stock - 1           -- 庫存數量減少 1
 WHERE product_id = 100            -- 目標商品
   AND stock >= 1;                 -- 確保有足夠庫存才能扣

-- 建立訂單記錄
INSERT INTO orders (user_id, product_id, quantity, created_at)
VALUES (42, 100, 1, NOW());        -- 插入一筆新訂單

-- 建立付款記錄
INSERT INTO payments (order_id, amount, status, created_at)
VALUES (LAST_INSERT_ID(), 299, 'pending', NOW());  -- LAST_INSERT_ID() = 上一個 INSERT 的 ID

-- 確認交易
COMMIT;
```

### ⚠️ 失敗時的 ROLLBACK 實戰

```sql
-- ==========================================
-- 示範：當交易中途失敗，如何撤銷
-- ==========================================
START TRANSACTION;

-- 第一步：庫存扣減（假設商品不夠）
UPDATE products
   SET stock = stock - 10
 WHERE product_id = 50;

-- 第二步：建立訂單（假設使用者ID 不存在，會觸發錯誤）
INSERT INTO orders (user_id, product_id, quantity)
VALUES (99999, 50, 10);  -- user_id=99999 不存在 → 發生錯誤！

-- ★ 發生錯誤時，要手動ROLLBACK
-- MySQL 的机制是：錯誤發生後，這筆交易會進入「回滾狀態」
-- 我們手動執行 ROLLBACK 來完全撤銷
ROLLBACK;

-- 驗證：products.stock 沒有被改變，orders 沒有新資料
```

### 🐍 Python 實戰：交易在程式碼中的用法

```python
import pymysql

# 建立資料庫連線（關閉自動commit）
conn = pymysql.connect(
    host='localhost',
    user='root',
    password='your_password',
    database='shop_db',
    autocommit=False  # ★ 重要：關閉自動commit，由我們手動控制
)

try:
    with conn.cursor() as cursor:
        # ===== 開始交易（隱含 BEGIN）=====
        
        # 第一步：檢查庫存是否足夠
        cursor.execute(
            "SELECT stock FROM products WHERE product_id = %s FOR UPDATE",
            (100,)
        )
        product = cursor.fetchone()
        
        if product is None:
            raise Exception("商品不存在")  # 手動拋出錯誤觸發 ROLLBACK
        
        if product[0] < 5:
            raise Exception("庫存不足，無法下單")
        
        # 第二步：扣庫存
        cursor.execute(
            "UPDATE products SET stock = stock - 5 WHERE product_id = %s",
            (100,)
        )
        
        # 第三步：建立訂單
        cursor.execute(
            "INSERT INTO orders (user_id, product_id, quantity) VALUES (%s, %s, %s)",
            (42, 100, 5)
        )
        order_id = cursor.lastrowid  # 取得新建訂單的ID
        
        # 第四步：建立付款記錄
        cursor.execute(
            "INSERT INTO payments (order_id, amount, status) VALUES (%s, %s, %s)",
            (order_id, 1495, 'pending')
        )
        
        # ===== 全部成功，手動提交 =====
        conn.commit()  # ★ 這行執行後，所有改動才會永久寫入硬碟
        print(f"✅ 訂單成立！訂單編號：{order_id}")
        
except Exception as e:
    # ===== 任何一步失敗，撤銷整筆交易 =====
    conn.rollback()  # ★ 立刻撤銷 BEGIN 以來的所有改動
    print(f"❌ 交易失敗：{e}")
    
finally:
    conn.close()  # 關閉連線
```

### 🔑 Transaction 的生命週期（流程圖）

```
┌─────────────────────────────────────────────────┐
│                   BEGIN                         │
│                      ↓                          │
│              ┌─────────────────┐                 │
│              │  執行 SQL 1     │ ← 第一次修改   │
│              └────────┬────────┘                 │
│                       ↓                          │
│              ┌─────────────────┐                 │
│              │  執行 SQL 2     │ ← 第二次修改   │
│              └────────┬────────┘                 │
│                       ↓                          │
│         ┌─────────────┴─────────────┐           │
│         ↓                           ↓           │
│   【成功】COMMIT               【失敗】ROLLBACK  │
│   把改動寫入硬碟              還原到BEGIN前狀態   │
└─────────────────────────────────────────────────┘
```

---

## 3️⃣ 實際操作（一步一步帶著做）

### 環境準備：用 Docker 起一個 MySQL

```bash
# 啟動一個 MySQL 容器（學習用，不需要任何設定）
docker run -d \
  --name mysql-learn \
  -e MYSQL_ROOT_PASSWORD=learn123 \
  -e MYSQL_DATABASE=shop_db \
  -p 3306:3306 \
  mysql:8.0
```

### Step 1：建立測試資料庫

```sql
-- ==========================================
-- 建立練習用的資料庫和表格
-- ==========================================

-- 確保當前在正確的資料庫
USE shop_db;

-- 建立帳戶資料表（餘額代表我們有多少錢）
CREATE TABLE accounts (
    id      INT AUTO_INCREMENT PRIMARY KEY,  -- 帳戶唯一ID
    name    VARCHAR(50)   NOT NULL,          -- 帳戶持有人名稱
    balance DECIMAL(15,2) NOT NULL DEFAULT 0  -- 帳戶餘額（最多15位數，小數點後2位）
);

-- 插入測試資料：兩個帳戶
INSERT INTO accounts (name, balance) VALUES
    ('小明', 10000.00),   -- 小明有 10000 元
    ('小華',  5000.00);   -- 小華有 5000 元

-- 驗證：確認初始狀態
SELECT * FROM accounts;
-- 結果：
-- id=1, name=小明, balance=10000.00
-- id=2, name=小華, balance=5000.00
```

### Step 2：正常執行一筆轉帳交易

```sql
-- ==========================================
-- 小明轉 3000 元給小華（正常成功的情況）
-- ==========================================

START TRANSACTION;  -- 開始交易

-- 第一步：小明的帳戶扣款
UPDATE accounts
   SET balance = balance - 3000
 WHERE id = 1;

-- 第二步：小華的帳戶存款
UPDATE accounts
   SET balance = balance + 3000
 WHERE id = 2;

COMMIT;  -- 確認交易

-- 驗證結果
SELECT * FROM accounts;
-- 小明：10000 → 7000
-- 小華：5000 → 8000
```

### Step 3：模擬交易失敗（庫存不足的情況）

```sql
-- ==========================================
-- 建立商品資料表（測試庫存不足的場景）
-- ==========================================

CREATE TABLE products (
    id       INT AUTO_INCREMENT PRIMARY KEY,
    name     VARCHAR(100) NOT NULL,
    stock    INT         NOT NULL DEFAULT 0   -- 庫存數量
);

INSERT INTO products (name, stock) VALUES
    ('iPhone 16', 2),      -- 只有2庫存
    ('MacBook Pro', 5);    -- 有5庫存

-- ==========================================
-- 嘗試下單 10 台的交易（庫存不夠，應該全部失敗）
-- ==========================================

START TRANSACTION;

-- 第一步：扣庫存（從2庫存扣10，雖然變成-8但SQL語法上執行成功）
UPDATE products
   SET stock = stock - 10    -- 庫存 2 - 10 = -8（數學上不合理）
 WHERE id = 1;

-- 第二步：建立訂單
INSERT INTO orders (product_id, quantity)
VALUES (1, 10);

COMMIT;

-- 檢查結果
SELECT * FROM products;
-- 糟糕！iPhone 16 的庫存變成 -8（不合理）
-- 如果一開始就用 Transaction + 業務邏輯檢查，可以在扣庫前先驗證
```

### Step 4：加入業務邏輯檢查的正確做法

```sql
-- ==========================================
-- 正確的庫存扣減交易（加入事前檢查）
-- ==========================================

START TRANSACTION;

-- 第一步：先查詢現有庫存（加 FOR UPDATE 鎖住這列，防止其他人同時讀寫）
SELECT stock FROM products WHERE id = 1 FOR UPDATE;

-- 第二步：業務邏輯檢查（在程式碼中檢查）
-- 如果庫存 < 需要的數量，ROLLBACK 並退出
-- （以下 SQL 只是示範，實際上會在 Python/JS 等語言中判斷）
UPDATE products
   SET stock = stock - 10
 WHERE id = 1
   AND stock >= 10;  -- ★ 額外條件：庫存必須 >= 10 才執行

-- 檢查影響行數（如果為0表示條件不滿足）
-- SHOW ERRORS;

ROLLBACK;  -- 這裡只是測試，正常流程中如果影響行數=0就ROLLBACK

-- ===== 正確的流程（在Python中執行）=====
-- if cursor.rowcount == 0:
--     conn.rollback()
--     print("庫存不足")
-- else:
--     conn.commit()
```

---

## 4️⃣ 常見錯誤 / 雷點提醒

### ❌ 雷點 1：忘記 commit，導致資料「卡住」

```sql
-- 很多人忘記最後要 COMMIT
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- 忘記了 COMMIT 或 ROLLBACK
-- 結果：在 MySQL 中，預設會自動 ROLLBACK 等待中的交易（8小時超時）
-- 但在 PostgreSQL 中，這筆交易會一直占用鎖，直到你手動處理！
-- 解決：永遠搭配 try/except 或確認最後有 commit/rollback
```

### ❌ 雷點 2：巢狀交易的陷阱

```sql
-- 很多初學者以為可以這樣做：
BEGIN;  -- 外層交易
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
BEGIN;  -- 錯誤！MySQL 不支援巢狀交易，這行會自動 COMMIT 上一筆！
-- 這時候第一筆 UPDATE 已經被提交了，無法回滾！
```

### ❌ 雷點 3：自動提交（autocommit）沒有關閉

```sql
-- MySQL 預設每一條語句都自動 commit
-- 如果你執行：
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- 自動COMMIT！
-- 這筆修改已經永久寫入，無法 ROLLBACK 了！

-- 正確做法：先關閉自動提交
SET autocommit = 0;  -- 關閉自動提交
-- 或者用 START TRANSACTION，它會自動暫時禁用 autocommit
```

### ❌ 雷點 4：SELECT 也會鎖住資料？

```sql
-- 在 Transaction 中，普通的 SELECT 會讀到舊資料（快照讀）
-- 但加了 FOR UPDATE 的 SELECT 會鎖住那一列
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- 這時另一個交易嘗試 UPDATE 或 SELECT...FOR UPDATE 同一列，就會等待（卡住）
-- 很多人不知道這點，誤以為「只是讀取」不會影響其他人
```

### ❌ 雷點 5：長交易阻塞整個系統

```sql
-- 範例：交易時間太長
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- 然後你去做其他事情，忘了還有交易在進行...
-- 這段時間內，其他人要讀寫同一筆記錄都會被阻塞
-- 解決：交易要**快進快出**，不要在交易中做網路請求或複雜計算
```

---

## 5️⃣ 今日小任務

### 🎯 任務：完成一個「扣款+日誌」的完整 Transaction

使用你熟悉的程式語言（Python/Node.js/Go 皆可），實作以下功能：

1. **建立資料表**：
   ```sql
   CREATE TABLE wallet (
       user_id INT PRIMARY KEY,
       balance DECIMAL(15,2) NOT NULL DEFAULT 0
   );
   CREATE TABLE wallet_log (
       id        INT AUTO_INCREMENT PRIMARY KEY,
       user_id   INT NOT NULL,
       amount    DECIMAL(15,2) NOT NULL,
       type      ENUM('deposit', 'withdraw') NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   INSERT INTO wallet (user_id, balance) VALUES (1, 5000);
   ```

2. **實作提款函式**（用 Transaction）：
   - 檢查餘額是否足夠
   - 扣款
   - 寫入日誌（記錄這筆操作）
   - 如果中途失敗，回滾並回傳錯誤訊息

3. **測試以下情境**：
   - ✅ 正常提款 1000 元 → 成功，餘額變 4000
   - ✅ 提款 6000 元（餘額不足）→ 失敗，餘額仍是 4000
   - ✅ 在交易中斷言失敗（如：網路錯誤）→ 回滾，資料不變

4. **把程式碼 commit 到 GitHub**，檔案命名：`2026-03-30-Day8-Transaction.md`，放在同一個章節目錄下。

---

## 📚 今日重點速記

```
Transaction（交易）＝ 一組「綁在一起」的資料庫操作

BEGIN          → 開始一筆交易
COMMIT         → 確認所有改動，永久寫入
ROLLBACK       → 撤銷所有改動，回到 BEGIN 前的狀態

核心精神：
- 原子性：全部成功 或 全部失敗
- 一致性：交易前後資料都合法
- 應用場景：轉帳、庫存扣減、下單、扣款
```

---

## 🔜 明日預習：Day 9 - ACID 特性深入探討

明天我們會更深入地探討 ACID 四個特性的**底層原理**：
- 原子性怎麼實現？（Redo Log、Undo Log）
- 一致性的「合法」是誰說了算？
- 隔離性如何影響併發讀寫？
- 持久性與 Write-Ahead Log 的關係

> 📖 預習教材：建議先看「隔離層級」章節（Day 12），會更容易理解。

---

*資料庫設計與實戰系列 - Day 8 of 30*