# DevOps 實戰 Day 1：Docker 基礎與容器化概念

## 📚 【今日學習：DevOps 實戰】

---

### 1️⃣ 基礎概念（用生活比喻，越詳細越好）

#### 什麼是容器化？（用餐廳廚房來比喻）

想象一下，你開一家餐廳：

**傳統方式（虛擬機）**：
- 每道菜都需要一個獨立的廚房空間
- 廚師A做義大利麵有自己的爐具、刀具、調味料
- 廚師B做壽司也有完全獨立的設備
- 結果：空間浪費、資源閒置、啟動慢

**容器化方式（Docker）**：
- 大家共用一個大廚房（共用作業系統核心）
- 每個廚師帶自己的「工具箱」（依賴庫、配置文件）
- 需要做菜？拿起工具箱就能開工
- 用完，清洗乾淨工具箱，別人馬上能用

**簡單說**：容器就是「打包好的應用程式 + 运行环境」，讓軟體在哪裡都能跑。

#### Docker 是什麼？

**官方定義**：一個開放平台，用於開發、交付、運行應用程式。

**比喻**：Docker 就像是「軟體的貨櫃箱」標準。

就像貨櫃改變了物流業（全世界都用同一種箱子），Docker 改變了軟體部署（標準化、可移植）。

#### 為什麼要用 Docker？

| 傳統問題 | Docker 解決方案 |
|---------|----------------|
| 「在我電腦上能跑啊！」 | 環境一致，Ship it! |
| 部署超慢 | 秒級啟動 |
| 資源浪費 | 輕量虛擬化 |
| 難以擴展 | 隨時可以複制 N 份 |

---

### 2️⃣ 核心知識（每個指令/語法都要有中文註解！）

#### Docker 三大核心概念

**1. Image（映像檔）**
- 類似「食譜」或「模板」
- 唯讀（不能修改）
- 包含：作業系統、軟體、依賴、配置
- 範例：`nginx:latest`、`python:3.9`、`mysql:8.0`

**2. Container（容器）**
- Image 的「實例」/「運行中的副本」
- 可讀寫
- 就像：用食譜做出來的「一道菜」
- 容器刪除後，改動不會保存（除非用 Volume）

**3. Registry（倉庫）**
- 存放 Image 的地方
- 最有名：Docker Hub（公開免費）
- 企業用：AWS ECR、Azure ACR、私有倉庫

#### Docker 架構（Client-Server 架構）

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │ ───► │   Daemon    │ ───► │  Registry   │
│  (CLI/IDE)  │      │ (Docker引擎)│      │ (存放Image) │
└─────────────┘      └─────────────┘      └─────────────┘
```

- **Docker Client**：你下指令的地方（docker 命令）
- **Docker Daemon**：背景運行的服務，負責實際操作
- **Docker Registry**：存放 Image 的仓库

#### 重要術語

| 術語 | 解釋 |
|-----|------|
| Dockerfile | 用來构建 Image 的「配方檔案」 |
| Layer | 映像檔的分層結構，每個指令一層 |
| Namespace | 隔離機制（隔離 PID、網路、檔案系統等） |
| Cgroup | 資源限制（CPU、記憶體） |
| UnionFS | 檔案系統聯合，實現分層疊加 |

---

### 3️⃣ 實際操作（一步一步帶著做）

#### Step 1：安裝 Docker（Ubuntu/Debian）

```bash
# 更新套件庫
# sudo：管理員權限
# apt：Ubuntu 的套件管理工具
# update：刷新套件清單
sudo apt update

# 安裝必要的前置套件
# curl：用於下載檔案
# ca-certificates：HTTPS 憑證
# gnupg：加密工具
sudo apt install -y curl ca-certificates gnupg lsb-release

# 添加 Docker 官方 GPG 金鑰
# -o：輸出檔案
# -D：解碼
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 新增 Docker 官方 repository（軟體源）
# 「deb [arch=$(dpkg --print-architecture) signed-by=...]」：指定架構和信任金鑰
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 再次更新套件庫（這次會包含 Docker）
sudo apt update

# 安裝 Docker Engine（社區版）
# -y：自動確認安裝
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 將當前使用者加入 docker 群組（這樣不用每次都打 sudo）
# $USER：當前登入的使用者名稱
sudo usermod -aG docker $USER

# 重新登入後生效（或執行以下命令立即生效）
newgrp docker
```

#### Step 2：驗證 Docker 安裝

```bash
# 查看 Docker 版本資訊
# version：顯示版本
docker version

# 執行 hello-world 映像檔測試
# run：運行一個容器
# hello-world：測試用的小映像檔
docker run hello-world
```

**預期輸出會看到**：
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

#### Step 3：理解 Docker 運作原理

```bash
# 查看 Docker 系統資訊
# info：顯示詳細資訊（容器數、映像檔數、磁碟使用等）
docker info

# 列出所有容器（包括停止的）
# ps -a：process status all
# -a：all，包含已停止的
docker ps -a

# 列出本地端的映像檔
# images：列出所有映像檔
docker images
```

#### Step 4：概念實驗（從生活角度理解）

```bash
# 假設你要開一家餐廳，先看看有什麼「食譜」可用
# search：在 Docker Hub 搜尋映像檔
docker search nginx

# 拉取一個映像檔（相當於「買食譜」）
# pull：下載映像檔到本地
docker pull nginx:latest

# 再次查看本地映像檔，應該有 nginx 了
docker images
```

---

### 4️⃣ 今日小任務

#### ✅ 必做任務

1. **完成 Docker 安裝**：在本地或 VM 上安裝 Docker
2. **執行測試**：運行 `docker run hello-world` 確認安裝成功
3. **探索指令**：執行 `docker --help` 了解有哪些指令可用

#### 📝 選做挑戰

4. **搜尋映像檔**：嘗試 `docker search python`，看看有哪些 Python 版本
5. **拉取映像檔**：嘗試拉取 `docker pull alpine`（一個超輕量的 Linux 映像檔）

#### 💡 今日重點回顧

- ✅ Docker = 軟體的「貨櫃箱」標準
- ✅ Image = 食譜/模板（唯讀）
- ✅ Container = 用食譜做出來的菜（可運行）
- ✅ Registry = 存放食譜的倉庫（最有名是 Docker Hub）

---

### 📚 預告

**Day 2**：我們會深入講解 Docker 常用指令（run, build, ps, exec），並實際操作運行容器！

---

*DevOps 學習路上，每天進步一點點！🚀*