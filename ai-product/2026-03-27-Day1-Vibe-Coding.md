# 📚 【Day 1：Vibe Coding 開發革命 - AI 時代開發方式與傳統開發的差異】

---

## 1️⃣ 基礎概念（用生活比喻）

### 🧠 什麼是 Vibe Coding？

想象一下：
- **傳統開發** = 你去米其林餐廳廚房，從頭學切菜、學火候、學擺盤，十年後才能端出一道菜
- **Vibe Coding** = 你直接走進廚房，跟主廚說「我要一道吃起來像初戀的料理」，他幫你完成，你只負責端上桌

**Vibe Coding（氛圍編程）** = 用自然語言描述你想要什麼，AI 幫你把想法變成代碼。

> 這個詞是 Andrej Karpathy（特斯拉前 AI 總監）提出來的，意思是「不要寫代碼，而是用感覺（vibe）來創造產品」。

### 🔄 傳統開發 vs Vibe Coding

| 面向 | 傳統開發 | Vibe Coding |
|------|----------|-------------|
| **输入** | 程式碼語法 | 自然語言（中文/英文） |
| **主力** | 工程師 | 每個人都能做 |
| **速度** | 幾週/幾個月 | 幾小時/幾天 |
| **迭代** | 瀑布式（需求→設計→開發→測試） | 對話式（說→改→說→改） |
| **门槛** | 懂 Syntax、Debug、CI/CD | 會打字就行 |
| **品質** | 穩定但慢 | 快但需要人把關 |

---

## 2️⃣ 核心知識

### 🛠️ Vibe Coding 的四大支柱

#### ① LLM（大型語言模型）
- 像是你的**虛擬工程師**，24小時待命
- 代表：Claude、ChatGPT、Cursor、Windsurf
- 原理：輸入文字 → 預測下一個最可能的字 → 組合成完整代碼

#### ② Agent 框架
- 讓 AI 能**自己動手**做完整任務（不只是給答案）
- 代表：LangChain、AutoGPT、Claude Code、Replit Agent
- 能力：自動開檔案、搜尋資料、跑測試、deploy

#### ③ 低代碼/無代碼平台
- **視覺化幫手**，讓 AI 生成的不是純文字，而是可直接用的 UI
- 代表：v0、Bolt.new、Framer AI、Linear AI

#### ④ 雲端開發環境
- **不必本地安裝**，打開瀏覽器就能開發
- 代表：GitHub Codespaces、Replit、Windsurf Cloud

### 📊 開發流程的質變

```
傳統流程（ weeks）：
需求 → 技術規格 → UI設計 → 後端API → 前端串接 → 測試 → 上線

Vibe Coding（ hours）：
我想要這個 → AI 生成 → 我看看 → 再調整 → 上線
```

---

## 3️⃣ 實際操作/程式碼範例

### 🎯 目標：讓你體驗「說一句話，AI 帮你生出一個網站」

這裡用 **v0 (v0.dev)** 來示範，這是 Vercel 出的 AI 生成 UI 工具。

#### Step 1：打開 v0.dev

```
https://v0.dev
```

#### Step 2：輸入你的需求（中文也可以！）

```
我要一個簡約的個人作品集網站
- 頂部要有導航列：關於、作品、聯絡
- Hero 區塊放我的名字和標語
- 作品區塊用卡片展示 3 個專案
- 聯絡區塊放 Email 連結
- 風格：深色模式、字體用 Inter
```

#### Step 3：AI 會生成什麼？

- ✅ 完整的 React + Tailwind 代碼
- ✅ 可互動的預覽畫面
- ✅ 可以直接 export 到 CodeSandbox 或複製代碼

#### 🎁 更強的：Bolt.new（一次完成更多）

```
https://bolt.new

需求（直接複製）：
「我要一個任務管理應用，包含：
1. 新增任務（標題、截止日期、優先級）
2. 勾選完成/未完成
3. 刪除任務
4. 用 localStorage 存資料
5. 用深色主題」
```

Bolt.new 會帮你生成完整的前後端（前端 + 簡易後端），甚至可以一鍵部署到 Netlify！

### 💻 如果你想用程式碼的方式（用 Cursor）

Cursor = AI 強化版 VS Code

1. **下載 Cursor**：https://cursor.sh
2. **打開新專案**，選擇 "AI Chat"
3. **輸入指令**：

```
Create a simple React app that shows a todo list with add, delete, and toggle complete features. Use Tailwind CSS for styling.
```

4. **看 AI 表演**：它會自動幫你建立檔案結構、寫代碼、安裝套件

---

## 4️⃣ 今日小任務

### ✅ 你的挑戰：生成你的第一個 AI 網站

1. 打開 **https://v0.dev** 或 **https://bolt.new**
2. 用中文描述你要的網站（可以是你個人網站、作品集、創業 idea）
3. **截圖**你的 AI 生成結果
4. **複製**生成的代碼到你的電腦

### 🎓 驗收標準
- [ ] 你成功用 AI 生成了一個網頁
- [ ] 你能說出「傳統開發」和「Vibe Coding」的差別
- [ ] 你能解釋為什麼「會打字」就能做產品

### 📝 進階挑戰（選做）
如果你會一點 HTML/CSS，试着用 Cursor 或 Windsurf 叫 AI 修改一點東西，体验「对话式改代码」的爽感！

---

## 📎 參考資源

- Andrej Karpathy 談 Vibe Coding：https://x.com/karpathy/status/1886195180482394102
- v0.dev：https://v0.dev
- Bolt.new：https://bolt.new
- Cursor：https://cursor.sh
- Windsurf：https://windsurf.ai

---

*Day 1 完成 ✅ | 明天 Day 2：AI 時代的競品分析與市場挖掘*