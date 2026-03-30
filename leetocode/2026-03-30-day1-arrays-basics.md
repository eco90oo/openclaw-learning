# Day 1：陣列基礎與常見操作 - 詳細時間複雜度分析

## 1️⃣ 基礎概念（用生活比喻）

### 🎯 什麼是陣列？

想象你有個**停車場**，每個車位都有固定的編號：
- 車位 0、車位 1、車位 2、...、車位 n
- 每個車位只能停一輛車
- 你知道車牌號碼（資料），也知道車位編號（索引）

**這就是陣列（Array）**：一塊連續的記憶體空間，每個位置都有固定的「門牌號碼」（索引）。

### 📍 陣列的關鍵特性

| 特性 | 說明 | 生活比喻 |
|------|------|----------|
| **連續記憶體** | 陣列佔用一整塊連續的記憶體空間 | 停車場的一排連續車位 |
| **固定大小** | 建立時決定大小，不能動態擴展 | 停車場有多少車位是固定的 |
| **索引從 0 開始** | 第一個元素索引是 0，不是 1 | 停車場編號從 0 開始（0 號車位是入口） |
| **同質性** | 只能存放相同類型的資料 | 停車場只停轎車，不能停機車 |

### 🆚 陣列 vs 串列（Array vs Linked List）

| 特性 | 陣列 (Array) | 串列 (Linked List) |
|------|-------------|-------------------|
| 記憶體配置 | 一次分配連續空間 | 每個節點獨立配置 |
| 隨機存取 | ✅ O(1) 快速訪問 | ❌ O(n) 需要遍歷 |
| 插入/刪除 | ❌ O(n) 需要搬移 | ✅ O(1) 只需要修改指標 |
| 空間浪費 | 無浪費 | 每個節點需要額外指標空間 |

---

## 2️⃣ 核心知識（演算法解釋）

### ⏰ 時間複雜度完整分析

```python
# ====== 陣列操作時間複雜度 ======

# 1. 存取元素（Access）
#    - 語法：arr[i]
#    - 時間：O(1) ✅
#    - 原因：像停車場知道車位號碼，直接走過去就找到了
arr = [1, 2, 3, 4, 5]
element = arr[2]  # 直接取得索引 2 的元素，耗時固定

# 2. 搜尋元素（Search）
#    - 時間：O(n) 最壞情況
#    - 時間：O(1) 最佳情況（運氣好第一個就是）
#    - 時間：O(n/2) 平均情況
#    - 原因：像在停車場一輛一輛檢查車牌
def linear_search(arr, target):
    """
    線性搜尋：從第一個元素一個一個檢查
    時間複雜度：O(n) - 最壞情況要檢查所有元素
    空間複雜度：O(1) - 只用一個變數
    """
    for i in range(len(arr)):  # 最多執行 n 次
        if arr[i] == target:   # 每次比較是 O(1)
            return i           # 找到就回傳
    return -1                   # 沒找到

# 3. 插入元素（Insert）
#    - 語法：arr.append(x) / arr.insert(i, x)
#    - 末端插入：O(1) amortized（攤平分析）
#    - 中間插入：O(n) - 需要搬移後面所有元素
#    - 原因：在停車場中間插隊停車，後面的車都要往後挪
def insert_middle(arr, index, value):
    """
    在陣列中間插入元素
    時間複雜度：O(n) - 需要搬移 index 以後的所有元素
    空間複雜度：O(1) - 直接修改原陣列
    """
    # 假設要在 index 位置插入 value
    # 1. 從最後一個元素開始，往後搬移一位
    for i in range(len(arr) - 1, index - 1, -1):
        arr[i + 1] = arr[i]  # 往後搬，O(n)
    # 2. 在目標位置放入新值
    arr[index] = value      # O(1)

# 4. 刪除元素（Delete）
#    - 語法：arr.pop() / arr.remove(x) / del arr[i]
#    - 末端刪除：O(1)
#    - 中間刪除：O(n) - 需要往前補位
#    - 原因：停車場中間的車開走後，後面的車都要往前挪
def delete_middle(arr, index):
    """
    刪除陣列中間的元素
    時間複雜度：O(n) - 需要搬移 index 以後的所有元素
    空間複雜度：O(1) - 直接修改原陣列
    """
    # 從 index+1 開始，所有元素往前搬
    for i in range(index, len(arr) - 1):
        arr[i] = arr[i + 1]  # 往前搬，O(n)
    # 最後一個位置設為空（或忽略）
    arr[len(arr) - 1] = None

# 5. 修改元素（Update）
#    - 時間：O(1) ✅
#    - 原因：知道車位號碼，直接換車牌
arr[3] = 100  # 直接修改索引 3 的值，耗時固定
```

### 💾 空間複雜度分析

```python
# ====== 陣列空間複雜度 ======

# 基本陣列
arr = [1, 2, 3, 4, 5]  # 空間：O(n) - 需要 n 個位置

# 二維陣列
matrix = [[1, 2, 3],
          [4, 5, 6]]   # 空間：O(n*m) - n 列 m 行

# 複製陣列
arr_copy = arr[:]      # 空間：O(n) - 需要額外 n 個位置
```

### 🔬 攤平分析（Amortized Analysis）

為什麼 `append()` 末端插入是 O(1) 而不是 O(n)？

```
假設陣列容量是 4，陸續插入 5 個元素：

插入第 1 個：容量 4 → 4，空位直接放 → O(1)
插入第 2 個：容量 4 → 4，空位直接放 → O(1)
插入第 3 個：容量 4 → 4，空位直接放 → O(1)
插入第 4 個：容量 4 → 4，空位直接放 → O(1)
插入第 5 個：容量 4 → 8，申請新空間，複製舊資料 → O(n)

總操作：4×O(1) + 1×O(n) = O(n)
平均每次：O(n)/5 = O(1) 攤平後是常數時間！
```

---

## 3️⃣ 經典題目（LeetCode 實際題目）

### 📌 題目 1：最大子陣列（Maximum Subarray）

> LeetCode 53 - 經典動態規劃入門題

```python
def maxSubArray(nums):
    """
    找出連續子陣列的最大和
    
    範例輸入：[-2, 1, -3, 4, -1, 2, 1, -5, 4]
    範例輸出：6  （子陣列 [4, -1, 2, 1]）
    
    ====== 演算法思路 ======
    使用「卡丹算法」（Kadane's Algorithm）
    核心想法：像滾雪球一樣，如果 current_sum 變成負數，
             就重新開始找新的子陣列
    
    ====== 時間複雜度：O(n) ======
    - 只遍歷一次陣列
    
    ====== 空間複雜度：O(1) ======
    - 只用兩個變數：current_sum 和 max_sum
    """
    
    # 初始化：假設第一個元素是目前的最大值
    # 這也是目前的「雪球」起始點
    current_sum = nums[0]   # 目前的子陣列和
    max_sum = nums[0]       # 記錄至今的最大和
    
    # 從第二個元素開始遍歷（索引 1 到 n-1）
    for i in range(1, len(nums)):
        # 決定策略：要不要把當前元素加入現有子陣列？
        # 策略 A：繼續擴展：current_sum + nums[i]
        # 策略 B：重新開始：nums[i]（放棄之前的負和）
        
        # 數學表達：取兩者最大值
        # 如果 current_sum 是負數，當然重新開始比較好
        current_sum = max(nums[i], current_sum + nums[i])
        
        # 更新最大和（如果現在的 current_sum 比歷史最大還大）
        max_sum = max(max_sum, current_sum)
    
    # 回傳找到的最大子陣列和
    return max_sum


# ====== 測試範例 ======
test_cases = [
    [-2, 1, -3, 4, -1, 2, 1, -5, 4],  # 預期輸出：6
    [1],                               # 預期輸出：1
    [5, 4, -1, 7, 8],                  # 預期輸出：23
]

for test in test_cases:
    result = maxSubArray(test)
    print(f"輸入：{test}")
    print(f"輸出：{result}")
    print("-" * 30)
```

### 📌 題目 2：旋轉矩陣（Rotate Image）

> LeetCode 48 - 矩陣操作經典題

```python
def rotate(matrix):
    """
    將 n×n 矩陣順時針旋轉 90 度
    
    範例輸入：      範例輸出：
    [1,2,3]       [7,4,1]
    [4,5,6]       [8,5,2]
    [7,8,9]       [9,6,3]
    
    ====== 演算法思路 ======
    分兩步驟：
    1. 先沿主對角線翻轉（左上 ↔ 右下）
    2. 再沿垂直中線翻轉（左 ↔ 右）
    
    為什麼這樣做？因為旋轉 90° = 對角線翻轉 + 水平翻轉
    
    ====== 時間複雜度：O(n²) ======
    - 需要訪問矩陣中所有 n² 個元素
    
    ====== 空間複雜度：O(1) ======
    - 原地修改，不需要額外空間
    """
    
    n = len(matrix)  # 矩陣大小
    
    # ====== 第一步：沿主對角線翻轉 ======
    # 對角線元素（i == j）不需要動
    # 我們只需要處理右上三角（i < j）
    for i in range(n):
        for j in range(i + 1, n):  # 只遍歷右上三角
            # 交換 matrix[i][j] 和 matrix[j][i]
            # 就像沿對角線鏡射
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    
    # ====== 第二步：沿垂直中線翻轉 ======
    # 每一列的元素，左右互換
    for i in range(n):
        # 左半邊：j 從 0 到 n//2 - 1
        for j in range(n // 2):
            # 交換左半和右半對應的位置
            left = j                    # 左邊索引
            right = n - 1 - j           # 右邊對應索引
            
            # 執行交換
            matrix[i][left], matrix[i][right] = matrix[i][right], matrix[i][left]


# ====== 測試範例 ======
test_matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print("旋轉前：")
for row in test_matrix:
    print(row)

rotate(test_matrix)

print("\n旋轉後：")
for row in test_matrix:
    print(row)
```

### 📌 題目 3：存在重複元素（Contains Duplicate）

> LeetCode 217 - 簡單但重要的基礎題

```python
def containsDuplicate(nums):
    """
    判斷陣列中是否有重複元素
    
    ====== 方法一：排序後檢查 ======
    時間複雜度：O(n log n) - 因為需要排序
    空間複雜度：O(1) - 原地排序（如果用 sort() 會用 O(n)）
    
    ====== 方法二：雜湊集合 ======
    時間複雜度：O(n) - 查詢和插入都是 O(1)
    空間複雜度：O(n) - 需要額外空間存集合
    
    這裡展示方法二（更高效）
    """
    
    # 建立一個雜湊集合（Hash Set）
    # 集合的特色：裡面的元素不會重複
    seen = set()  # 建立空的雜湊集合
    
    # 遍歷每個數字
    for num in nums:
        # 如果這個數字已經在集合裡，表示找到重複！
        if num in seen:
            return True  # 有重複，回傳 True
        
        # 否則，把這個數字加入集合
        seen.add(num)  # add() 是 O(1)
    
    # 全部檢查完都沒重複
    return False  # 沒有重複，回傳 False


# ====== 測試範例 ======
test_cases = [
    [1, 2, 3, 1],      # True
    [1, 2, 3, 4],      # False
    [1, 1, 1, 3, 3, 4, 3, 2, 4, 2]  # True
]

for test in test_cases:
    result = containsDuplicate(test)
    print(f"輸入：{test}")
    print(f"輸出：{result}")
    print("-" * 30)
```

---

## 4️⃣ 相似題目推薦

| 題目編號 | 題目名稱 | 難度 | 核心概念 |
|----------|----------|------|----------|
| LeetCode 1 | Two Sum | Easy | 雜湊表 + 陣列 |
| LeetCode 26 | Remove Duplicates | Easy | 雙指標技巧 |
| LeetCode 121 | Best Time to Buy Stock | Easy | Kadane 變體 |
| LeetCode 283 | Move Zeroes | Easy | 雙指標應用 |
| LeetCode 88 | Merge Sorted Array | Easy | 從後往前雙指標 |

---

## 5️⃣ 今日小任務

### 🎯 練習題

1. **基礎題**：實現 `reverse_array(arr)` - 反轉陣列
   - 挑戰：嘗試用 O(1) 額外空間原地反轉

2. **中級題**：LeetCode 118 - Pascal's Triangle（楊輝三角）
   - 練習二維陣列的操作

3. **挑戰題**：LeetCode 56 - Merge Intervals
   - 合併重疊區間，綜合應用排序和陣列操作

### 💡 自我檢查清單

- [ ] 能解釋什麼是陣列的連續記憶體特性
- [ ] 能說出陣列存取、搜尋、插入、刪除的時間複雜度
- [ ] 能用白話文解釋「攤平分析」是什麼
- [ ] 能實現基本的線性搜尋
- [ ] 能理解 Kadane's Algorithm 的核心思想

---

## 📚 延伸閱讀

- [Arrays in Python - Real Python](https://realpython.com/python-arrays/)
- [Time Complexity - Big O Cheat Sheet](https://www.bigocheatsheet.com/)

---

*Day 1 完成！明天我們將深入「二分搜尋」和「旋轉陣列」的核心技巧。*
