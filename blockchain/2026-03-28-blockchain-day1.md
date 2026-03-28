# 📚 【今日學習：區塊鏈與 DeFi - Day 1】

## 1️⃣ 基礎概念（用生活比喻，越詳細越好）

### 區塊鏈到底是什麼？用「村莊帳本」告訴你

想象一個小村莊裡，大家常常互相借東西、買東西。傳統做法是找一個 **村長（中心化機構）** 來記帳：
- 🏦 銀行幫你存錢
- 🏛️ 政府幫你確認身份
- 📜 律師幫你寫合約

**問題來了**：如果村長生病了、喝醉酒、或者作弊偷偷改帳本怎麼辦？

**區塊鏈的革命性想法**：「既然大家都不信任某個人，那我們讓每個人都有一本帳本！」

### 🔗 區塊鏈的核心比喻

| 傳統中心化 | 區塊鏈（去中心化）|
|-----------|-----------------|
| 村長一個人記帳 | 全村每個人都有帳本副本 |
| 村長說了算 | 大多數人說了算 |
| 村長下班就無法交易 | 24 小時随时可交易 |
| 村長跑路=帳本消失 | 有人跑路不影响帳本 |

### 什麼是「區塊(Block)」？

**區塊 = 一張記帳紙**
- 這張紙上記錄了這段時間內所有的交易
- 紙張有大小限制，大約每秒能處理幾筆到幾十筆交易（比特幣 ~7筆/秒，以太坊 ~15-30筆/秒）

### 什麼是「鏈(Chain)」？

**鏈 = 訂在一起的記帳紙**
- 每張新紙都會蓋上舊紙的「指紋」（Hash）
- 如果有人想偷偷改舊帳本，就會讓指紋對不上
- 就像多米諾骨牌，一張倒下就會影響後面所有張

---

## 2️⃣ 核心知識（詳細教學，包含公式與計算）

### 比特幣 Bitcoin 的誕生 (2008)

**歷史背景**：
- 2008年，全球金融海嘯，銀行系統信任崩潰
- 2008年10月31日，一位匿名人物「**中本聰**」(Satoshi Nakamoto) 發表論文《Bitcoin: A Peer-to-Peer Electronic Cash System》
- 2009年1月3日，比特幣創世區塊（Genesis Block）誕生

**比特幣的核心創新**：
```
區塊鏈 = 分散式帳本 + 共識機制 + 加密學
```

**比特幣解決的問題**：
1. **雙花問題（Double Spending）**：如何防止同一筆錢花兩次？
2. **第三方信任**：不需要銀行也能轉帳
3. **總量固定**：只有 2100 萬枚，永不通膨

### 以太坊 Ethereum 的誕生 (2015)

** Vitalik Buterin 的願望**：
- 比特幣只能「轉帳」，無法「執行程式」
- 2013年，Vitalik 提出以太坊白皮書
- 2015年7月30日，以太坊主網上線

**以太坊的核心創新**：
```
以太坊 = 可程式化的區塊鏈（World Computer）
```

| 特性 | 比特幣 | 以太坊 |
|-----|-------|--------|
| 定位 | 數位黃金 | 分散式電腦 |
| 用途 | 價值儲存 | 智能合約 |
| 單位 | BTC | ETH |
| 供應量 | 2100萬枚（固定）| 無限供應 |
| 圖靈完整性 | 否 | 是 |

---

## 3️⃣ 實際操作（程式碼範例，每行都要有中文註解）

### 安裝 Python Web3 庫，連接區塊鏈測試網

```bash
# 安裝 Web3.py 庫
pip install web3

# 安裝 eth-abis 套件（用於解析合約 ABI）
pip install eth-abi
```

### 使用 Web3.py 連接以太坊測試網，取得區塊鏈基本資訊

```python
# 引入 Web3 類別，用於與以太坊區塊鏈互動
from web3 import Web3

# 建立 RPC 連線，連接到 Sepolia 測試網（免費公開節點）
# 選擇免費的公共 RPC URL，速度較慢但可用於測試
w3 = Web3(Web3.HTTPProvider('https://rpc.sepolia.org'))

# 檢查區塊鏈連線是否成功
# 顯示是否已連線
is_connected = w3.is_connected()
print(f"區塊鏈連線狀態: {'已連線' if is_connected else '連線失敗'}")

# 取得最新區塊編號
# 區塊編號會隨著每筆交易增加
latest_block = w3.eth.block_number
print(f"最新區塊編號: {latest_block}")

# 取得當前以太坊 ETH 的價格（以美元計價）
# 這是從交易所取得的即時價格
eth_price_usd = w3.eth.gas_price  # 這是 Gas 價格，不是 ETH 價格！
print(f"当前 Gas 價格 (Wei): {eth_price_usd}")
print(f"轉換為 ETH: {w3.from_wei(eth_price_usd, 'ether')} ETH")

# 取得區塊鏈網路版本
# 用於確認連接的網路類型
chain_id = w3.eth.chain_id
print(f"區塊鏈網路 ID: {chain_id}")
# 1 = Ethereum Mainnet
# 11155111 = Sepolia Testnet
# 8453 = Base Mainnet
```

### 比特幣概念驗證：用 Python 模擬簡易區塊

```python
# 引入 hashlib 庫，用於計算 SHA256 哈希值
import hashlib
import time

# 定義一個簡化的「區塊」類別
class Block:
    def __init__(self, index, timestamp, data, previous_hash):
        """
        初始化區塊
        - index: 區塊編號（第幾個區塊）
        - timestamp: 區塊創建時間
        - data: 區塊內記錄的資料（交易等）
        - previous_hash: 前一個區塊的哈希值（用於形成鏈）
        """
        self.index = index          # 區塊編號
        self.timestamp = timestamp # 創建時間戳記
        self.data = data         # 區塊資料
        self.previous_hash = previous_hash  # 前一個區塊的哈希
        self.hash = self.calculate_hash()  # 自己區塊的哈希
    
    def calculate_hash(self):
        """
        計算區塊的 SHA256 哈希值
        將所有區塊資訊串接後雜湊，產生指紋
        """
        # 將所有資料轉為字串並串接
        block_string = str(self.index) + str(self.timestamp) + str(self.data) + str(self.previous_hash)
        # 使用 SHA256 雜湊演算法
        return hashlib.sha256(block_string.encode()).hexdigest()

# 定義區塊鏈類別
class Blockchain:
    def __init__(self):
        """
        初始化區塊鏈
        創建第一個區塊（創世區塊）
        """
        # 創建創世區塊（第一個區塊，之前沒有雜湊所以設為 0）
        self.chain = [self.create_genesis_block()]
    
    def create_genesis_block(self):
        """
        創建創世區塊（Genesis Block）
        比特幣第一個區塊是中本聰挖出的，稱為創世區塊
        """
        return Block(0, time.time(), "創世區塊: 區塊鏈時代開始！", "0")
    
    def get_latest_block(self):
        """
        取得最新的區塊
        用於新增區塊時參考
        """
        return self.chain[-1]
    
    def add_block(self, new_block):
        """
        新增區塊到鏈上
        - 首先設定新區塊的 previous_hash 為上一個區塊的 hash
        - 然後將新區塊加入鏈中
        """
        new_block.previous_hash = self.get_latest_block().hash
        new_block.hash = new_block.calculate_hash()
        self.chain.append(new_block)
    
    def is_chain_valid(self):
        """
        驗證區塊鏈是否被篡改
        檢查每個區塊的 hash 與 previous_hash 是否正確連接
        """
        # 從第二個區塊開始檢查（跳過創世區塊）
        for i in range(1, len(self.chain)):
            current_block = self.chain[i]
            previous_block = self.chain[i-1]
            
            # 檢查區塊的 hash 是否正確（如果被改過會對不上）
            if current_block.hash != current_block.calculate_hash():
                print(f"第 {i} 區塊哈希不正確！")
                return False
            
            # 檢查鏈的連接是否正確（previous_hash 必須對上前一個區塊的 hash）
            if current_block.previous_hash != previous_block.hash:
                print(f"第 {i} 區塊的 previous_hash 與第 {i-1} 區塊的 hash 不匹配���")
                return False
        
        return True

# === 測試區塊鏈 ===
print("=== 建立測試區塊鏈 ===")

# 創建新的區塊鏈
my_blockchain = Blockchain()

# 新增第一個區塊
block1 = Block(1, time.time(), "小明轉給小紅 1 BTC", my_blockchain.get_latest_block().hash)
my_blockchain.add_block(block1)

# 新增第二個區塊
block2 = Block(2, time.time(), "小紅轉給小華 0.5 ETH", my_blockchain.get_latest_block().hash)
my_blockchain.add_block(block2)

# 新增第三個區塊
block3 = Block(3, time.time(), "小華轉給小明 100 USDT", my_blockchain.get_latest_block().hash)
my_blockchain.add_block(block3)

# 打印所有區塊
print("\n=== 區塊鏈內容 ===")
for block in my_blockchain.chain:
    print(f"區塊 #{block.index}")
    print(f"  時間戳記: {block.timestamp}")
    print(f"  資料: {block.data}")
    print(f"  前一個區塊哈希: {block.previous_hash[:16]}...")  # 只顯示前16個字元
    print(f"  本區塊哈希: {block.hash[:16]}...")
    print()

# 驗證區塊鏈是否有效
print(f"區塊鏈有效性: {'有效 ✓' if my_blockchain.is_chain_valid() else '無效 ✗'}")

# 嘗試篡改區塊鏈（模擬攻擊）
print("\n=== 模擬篡改攻擊 ===")
my_blockchain.chain[1].data = "小明轉給小紅 9999 BTC"  # 修改資料
my_blockchain.is_chain_valid()  # 驗證會失敗
```

---

## 4️⃣ 常見錯誤提醒

### ❌ 錯誤一：混淆比特幣與以太坊
- 比特幣是「數位黃金」，主要用途是價值儲存
- 以太坊是「世界電腦」，可以執行程式（智能合約）
- 以太坊的 ETH 是「燃料」，用來支付 Gas 費用

### ❌ 錯誤二：以為區塊鏈完全匿名
- 區塊鏈是「假匿名」（Pseudo-anonymous）
- 地址雖然不是實名，但所有交易都公開可查
- 如果知道某地址是你的，，就可以追蹤所有交易

### ❌ 錯誤三：以為區塊鏈免費
- 轉帳需要支付 Gas 費用
- 網路擁塞時，Gas 費用會暴漲
- 2021年高峰期，一筆 USDT 轉帳可能收到幾百美元 Gas

---

## 5️⃣ 今日小任務

### 🎯 今天的任務

1. **理解概念**：用自己的話向朋友解釋「什麼是區塊鏈」，使用「村莊帳本」比喻

2. **動手練習**：修改上面的 Python 程式碼，新增兩個新的區塊，然後執行驗證

3. **思考問題**：為什麼比特幣要叫「區塊鏈」？「區塊」和「鏈」分別代表什麼意思？

4. **下集預告**：Day 2 將介紹「非對稱加密」，也就是公鑰、私鑰、數位簽章的原理

---

### 📚 課後練習答案

**思考問題解答**：
- 「區塊」就像一張記帳紙，上面記錄了一段時間內的所有交易
- 「鏈」就像將這些記帳紙訂在一起，每張紙都蓋上前一張的指紋，形成無法篡改的連續記錄

明天 Day 2 見！