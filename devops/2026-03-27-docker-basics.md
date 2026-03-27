# Day 1: Docker 基礎與容器化概念 - 詳細原理教學

## 📚 【今日學習：DevOps 實戰】

> 今天是 DevOps 學習的第一天，我們要從「容器化」的核心概念開始。這個概念看似抽象，但其實你每天都在接觸類似的概念！

---

## 1️⃣ 基礎概念（用生活比喻，越詳細越好）

### 🐳 什麼是 Docker？用生活比喻來解釋

**比喻：集裝箱改變世界物流**

想像一下：
- 古代運輸货物時，你需要不同的箱子（木箱、布袋、陶罐）
- 不同國家的碼頭工人要學會處理每一種箱子
- 如果箱子尺寸不合，船就裝不滿，浪費空間

**然後出現了「集裝箱」這個革命性發明：**
- 統一尺寸：無論裡面放的是衣服、電子產品還是食品
- 統一處理方式：全世界都用同一種起重機
- 隔離保護：集裝箱內的東西不會互相影響
- 快速裝卸：幾小時就能裝滿一艘貨輪

**Docker 就是軟體世界的「集裝箱」！**

```
傳統軟體部署（像古代物流）：
┌─────────────┐    ┌─────────────┐
│  伺服器 A    │    │  伺服器 B    │
│ Python 3.8  │    │ Python 3.9   │
│ Django 3.2  │    │ Django 3.1   │
│ PostgreSQL 12│    │ MySQL 8.0   │
│ Redis 6.0   │    │ Redis 5.0    │
└─────────────┘    └─────────────┘
每次部署都是一個「特異性」系統，環境不一致讓人崩潰

Docker 容器化（像現代集裝箱）：
┌─────────────┐    ┌─────────────┐
│  容器 A      │    │  容器 B      │
│ ┌─────────┐ │    │ ┌─────────┐ │
│ │App Image│ │    │ │App Image│ │
│ └─────────┘ │    │ └─────────┘ │
└─────────────┘    └─────────────┘
相同的「映像檔」，可以在任何地方運行
```

### 🔬 容器 vs 虛擬機：深入比較

| 特性 | 傳統虛擬機 (VM) | Docker 容器 |
|------|----------------|-------------|
| **啟動時間** | 分鐘級 | 秒級 |
| **資源占用** | 完整作業系統，動輒 GB | 共享核心，MB 級 |
| **隔離程度** | 硬體層隔離，強 | 行程層隔離，適中 |
| **映像檔大小** | GB 級 | MB 級 |
| **適合場景** | 完整系統、需要強隔離 | 微服務、輕量部署 |

**比喻：**
- VM 像是你買了一整棟房子（獨立空間，但昂貴）
- Container 像是住在飯店的套房（共用設施，但便利）

### 🏗️ Docker 架構：三大核心元件

```
┌─────────────────────────────────────────────────────┐
│                    Docker 主機 (Host)                │
│  ┌───────────────────────────────────────────────┐  │
│  │              Docker Engine (引擎)              │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────┐│  │
│  │  │   Client   │  │   Daemon    │  │ Registry││  │
│  │  │  (客戶端)   │──│  (守護進程)  │  │ (映像檔││  │
│  │  └─────────────┘  └─────────────┘  │ 倉庫)  ││  │
│  │                        │          └─────────┘│  │
│  │                        ▼                      │  │
│  │              ┌─────────────────┐              │  │
│  │              │  Container 容器  │              │  │
│  │              └─────────────────┘              │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
         ▲                ▲                ▲
         │                │                │
    使用者操作       後台運行         存放映像檔
    (CLI/API)      (常駐程式)       (Docker Hub)
```

**三大元件介紹：**

1. **Docker Client (客戶端)** 
   - 你輸入指令的地方（docker build, docker run）
   - 像是餐廳的「服務生」，接收你的命令

2. **Docker Daemon (守護進程)**
   - 在背景運行的程式，負責實際操作
   - 像是餐廳的「廚師」，執行具體工作

3. **Docker Registry (映像檔倉庫)**
   - 存放「映像檔」的地方
   - 像是餐廳的「食材倉庫」
   - 預設使用 Docker Hub (公共倉庫)

---

## 2️⃣ 核心知識：Docker 安裝與基本指令

### 🛠️ 安裝 Docker（Linux 環境）

```bash
# 這是標準的 Ubuntu/Debian 安裝流程
# 1. 更新套件庫（確保獲取最新版本）
sudo apt-get update

# 2. 安裝必備套件（允許 apt 透過 HTTPS 使用 repository）
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# 3. 新增 Docker 的官方 GPG 金鑰（驗證套件真實性）
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. 設定 Docker 官方 repository（從正式來源取得 Docker）
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 再次更新套件庫（包含新加入的 Docker repository）
sudo apt-get update

# 6. 安裝 Docker Engine（、社群版）
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 7. 驗證 Docker 是否正常運作（會顯示版本資訊）
sudo docker --version
```

### 🧪 第一個 Docker 指令：Hello World

```bash
# 執行一個簡單的測試容器
# 這個指令會：
# 1. 首先在本機尋找 hello-world 映像檔
# 2. 如果找不到，自動從 Docker Hub 下載
# 3. 在容器中運行 hello-world 程式
# 4. 運行完畢後自動停止容器
docker run hello-world
```

**輸出範例：**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

### 📋 常用基本指令詳解

```bash
# ========== 映像檔相關 ==========

# 列出本機所有映像檔
# 顯示：映像檔名稱、標籤、ID、建立時間、大小
docker images
# 或是
docker image ls

# 下載映像檔（從 Registry 拉到本機）
# docker pull 映像檔名稱:標籤
# 如果不指定標籤，預設是 "latest"
docker pull ubuntu:22.04
docker pull nginx:latest

# 刪除映像檔
# docker rmi 映像檔ID 或 名稱:標籤
docker rmi ubuntu:22.04
docker rmi abc123def456

# 搜尋映像檔（在 Docker Hub 中搜尋關鍵字）
# 會顯示符合的映像檔名稱、描述、星級
docker search nginx


# ========== 容器相關 ==========

# 列出正在運行的容器
# -a: 顯示所有容器（包括已停止的）
# -q: 只顯示容器 ID
docker ps
docker ps -a
docker ps -aq

# 啟動/停止/刪除容器
# -i: 互動模式（保持 STDIN 開啟）
# -t: 分配一個虛擬終端（TTY）
docker start 容器ID
docker stop 容器ID
docker restart 容器ID

# 刪除已停止的容器
docker rm 容器ID

# 強制刪除執行中的容器（先停止再刪除）
docker rm -f 容器ID

# 進入運行中的容器（類似 ssh 登入）
# 讓你可以像在另一台機器上一樣操作容器內部
docker exec -it 容器ID /bin/bash

# 查看容器日誌
# -f: 即時追蹤日誌（像 tail -f）
# --tail 100: 只顯示最後 100 行
docker logs 容器ID
docker logs -f 容器ID
docker logs --tail 100 容器ID


# ========== 系統資訊 ==========

# 查看 Docker 系統資訊（磁碟使用、容器數量等）
docker system df

# 清理未使用的資源（映像檔、容器、網路）
docker system prune

# 強制清理（包括未使用的映像檔）
docker system prune -a
```

---

## 3️⃣ 實際操作（一步一步帶著做）

### 🚀 練習 1：運行第一個互動式容器

```bash
# 啟動一個 Ubuntu 容器，並進入互動模式
# -it 參數讓你可以和容器互動
# /bin/bash 讓我們可以輸入指令
docker run -it ubuntu /bin/bash

# 在容器內，你可以嘗試這些指令：
# 查看作業系統版本
cat /etc/os-release

# 查看主機名稱
hostname

# 離開容器（回到主機）
exit
```

**操作流程圖：**
```
主機終端機                      容器內部
┌──────────────┐            ┌──────────────┐
│docker run   │ ───────>  │ Ubuntu 22.04 │
│   -it       │            │              │
│ ubuntu      │            │  /bin/bash   │
└──────────────┘            └──────────────┘
       │                          │
       │                          ▼
       │                   輸入指令操作
       │                   exit 離開
       ▼
  容器自動停止
```

### 🚀 練習 2：運行 Nginx 網頁伺服器

```bash
# 在背景運行一個 Nginx 容器
# -d: detach mode（後台運行，不佔用終端機）
# -p 8080:80: 將主機的 8080 埠映射到容器的 80 埠
docker run -d -p 8080:80 nginx

# 檢查容器是否正在運行
docker ps

# 用瀏覽器或 curl 測試
curl http://localhost:8080

# 查看 Nginx 容器的日誌
docker logs 容器ID

# 停止並刪除容器
docker stop 容器ID
docker rm 容器ID
```

**埠映射解釋：**
```
主機 (Host)              容器 (Container)
┌─────────────┐         ┌─────────────┐
│  Port 8080  │ ──────> │   Port 80   │
│   (訪問用)  │         │  (Nginx 監聽)│
└─────────────┘         └─────────────┘
訪問 localhost:8080 會被轉發到容器的 80 埠
```

### 🚀 練習 3：建立簡單的自訂映像檔

首先，創建一個簡單的 Python 應用：

```python
# 創建工作目錄
mkdir -p ~/docker-practice
cd ~/docker-practice

# 創建 Python 應用程式
cat > app.py << 'EOF'
# app.py - 一個簡單的 HTTP 伺服器
from http.server import HTTPServer, SimpleHTTPRequestHandler

class MyHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<h1>Hello from Docker!</h1><p>My first container</p>')

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8000), MyHandler)
    print("Server running on http://localhost:8000")
    server.serve_forever()
EOF

# 創建 Dockerfile
cat > Dockerfile << 'EOF'
# 使用官方 Python 映像檔作為基礎
# 這會從 Docker Hub 下載 Python 3.11
FROM python:3.11-slim

# 設定工作目錄
# 所有後續指令都會在這個目錄下執行
WORKDIR /app

# 將當前目錄的檔案複製到容器的 /app 目錄
COPY . .

# 暴露應用使用的埠號（純文件說明，不會實際做任何事）
EXPOSE 8000

# 啟動容器時執行的指令
# 啟動 Python 腳本
CMD ["python", "app.py"]
EOF

# 構建映像檔
# -t: 給映像檔取個名字和標籤
# .: 使用當前目錄的 Dockerfile
docker build -t my-python-app .

# 運行容器
# -p 8000:8000: 映射埠號
docker run -d -p 8000:8000 my-python-app

# 測試
curl http://localhost:8000

# 清理
docker stop 容器ID
docker rm 容器ID
docker rmi my-python-app
```

---

## 4️⃣ 今日小任務

### 🎯 必做任務

1. **確認 Docker 環境**
   ```bash
   # 檢查 Docker 版本
   docker --version
   
   # 檢查 Docker 是否正常運作
   docker run hello-world
   ```

2. **運行一個 Nginx 容器**
   ```bash
   # 在背景運行 Nginx，映射到 80 埠
   docker run -d -p 80:80 nginx
   
   # 用瀏覽器或 curl 訪問 http://localhost
   curl http://localhost
   ```

3. **體驗互動式容器**
   ```bash
   # 進入 Ubuntu 容器，體驗「在另一個系統中操作」的感覺
   docker run -it ubuntu /bin/bash
   
   # 在容器內執行這些指令看看：
   # apt update && apt install -y cowsay
   # cowsay "Hello DevOps!"
   # exit
   ```

### 🎯 選做挑戰任務

嘗試修改上面的 Python 範例：
- 把顯示的文字改成你的名字
- 重新 build 並運行
- 確認修改有生效

---

## 📝 今日重點回顧

| 概念 | 比喻 | 關鍵指令 |
|------|------|----------|
| Docker 映像檔 | 集裝箱的「設計圖」 | `docker build`, `docker pull` |
| Docker 容器 | 實際裝載運行的「集裝箱」 | `docker run`, `docker ps` |
| Docker Client | 餐廳服務生 | `docker` 指令 |
| Docker Daemon | 餐廚師 | 背景運行的 dockerd |
| Registry | 食材倉庫 | Docker Hub |

---

## 🔜 明天預告

**Day 2: Docker 指令詳解（run, build, ps, exec）- 詳細範例**

明天我們會深入講解每個常用指令的各種參數，讓你能夠更精確地控制容器行為！

> 🚀 持續學習就是最大的競爭力！明天見！