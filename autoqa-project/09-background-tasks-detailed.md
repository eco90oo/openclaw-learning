# 🧩 補齊拼圖 2：背景任務與併發控制完整教學

> 這是 AutoQA 實現「啟動測試後立即回應，同時在背景執行」的核心技術

---

## 📚 1. 為什麼需要背景任務？

### 問題情境

在 AutoQA 中：
```
1. 前端發送 POST /tests（啟動測試）
2. Playwright 開始執行（可能需要 5 分鐘）
3. 如果用同步方式：
   - 前端要等 5 分鐘才收到回應
   - HTTP 連線可能會超時
   - 使用者體驗很差
```

### 解決方案

```
1. 前端發送 POST /tests
2. 後端立即回應（收到請求了）
3. 啟動「背景任務」執行測試
4. 透過 WebSocket 推播進度給前端
5. 前端立即收到回應，顯示「測試已啟動」
```

---

## 📚 2. FastAPI 背景任務

### 基本語法

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def write_log(message: str):
    """背景任務函數"""
    with open("log.txt", "a") as f:
        f.write(message + "\n")

@app.post("/test")
async def run_test(background_tasks: BackgroundTasks):
    """啟動測試"""
    # 把函數加入背景執行
    background_tasks.add_task(write_log, "測試開始")
    
    # 立即回應
    return {"status": "started", "message": "測試已在背景執行"}
```

### 加入多個背景任務

```python
@app.post("/test")
async def run_test(background_tasks: BackgroundTasks):
    # 加入多個任務
    background_tasks.add_task(task1)
    background_tasks.add_task(task2)
    background_tasks.add_task(task3)
    
    return {"status": "started"}
```

### 傳遞參數

```python
def process_test(test_id: str, data: dict):
    """背景任務：處理測試"""
    print(f"處理測試: {test_id}")
    # ... 執行測試

@app.post("/tests")
async def create_test(test_data: dict, background_tasks: BackgroundTasks):
    test_id = "test_123"
    
    # 傳遞參數
    background_tasks.add_task(process_test, test_id, test_data)
    
    return {"test_id": test_id}
```

---

## 📚 3. asyncio 基礎

### 什麼是 asyncio？

asyncio 是 Python 的「非同步 I/O」庫，讓你能夠：
- 同時執行多個任務
- 在等待 I/O 時不阻塞

### 基礎語法

```python
import asyncio

# 定義 async 函數
async def say_hello():
    print("Hello!")
    await asyncio.sleep(1)  # 模擬等待
    print("World!")

# 執行
asyncio.run(say_hello())
```

### async / await

```python
async def fetch_data():
    print("開始獲取資料...")
    await asyncio.sleep(2)  # 模擬網路請求
    print("資料獲取完成!")
    return {"data": "test"}

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())
```

### 並發執行

```python
import asyncio

async def task(name: str, seconds: int):
    print(f"{name} 開始")
    await asyncio.sleep(seconds)
    print(f"{name} 完成")

async def main():
    # 三個任務同時開始
    await asyncio.gather(
        task("任務A", 2),
        task("任務B", 3),
        task("任務C", 1)
    )

asyncio.run(main())
# 輸出：
# 任務A 開始
# 任務B 開始
# 任務C 開始
# 任務C 完成 (1秒)
# 任務A 完成 (2秒)
# 任務B 完成 (3秒)
```

---

## 📚 4. Semaphore（信號量）

### 為什麼需要？

在 AutoQA 中：
- 如果同時有 100 個測試啟動
- 每個測試都會開一個瀏覽器
- 伺服器記憶體會被耗盡 → 當機

### 解決方案：限制並發數

```python
import asyncio

# 限制最多 3 個同時執行
semaphore = asyncio.Semaphore(3)

async def run_test(test_id: str):
    async with semaphore:  # 佔用一個名額
        print(f"開始測試: {test_id}")
        await asyncio.sleep(5)  # 模擬測試
        print(f"完成測試: {test_id}")

async def main():
    # 啟動 10 個測試
    tasks = [run_test(f"test_{i}") for i in range(10)]
    await asyncio.gather(*tasks)

asyncio.run(main())

# 輸出：
# 開始測試: test_0
# 開始測試: test_1
# 開始測試: test_2
# （同時最多 3 個）
# test_0 完成
# 開始測試: test_3
# ...
```

### 在 FastAPI 中使用

```python
import asyncio
from fastapi import FastAPI

app = FastAPI()

# 限制最多 5 個測試同時執行
test_semaphore = asyncio.Semaphore(5)

async def execute_test(test_id: str):
    async with test_semaphore:
        # 執行 Playwright 測試
        print(f"執行測試: {test_id}")
        await asyncio.sleep(5)
        print(f"完成測試: {test_id}")

@app.post("/tests/{test_id}")
async def run_test(test_id: str):
    # 立即回應，不會阻塞
    asyncio.create_task(execute_test(test_id))
    return {"status": "started", "test_id": test_id}
```

---

## 📚 5. Queue（任務佇列）

### 什麼是 Queue？

當任務數量很多時，用「排隊」的方式處理：

```python
import asyncio
from asyncio import Queue

async def worker(worker_id: int, queue: Queue):
    """工作者：從佇列取任務執行"""
    while True:
        # 阻塞直到有新任務
        task = await queue.get()
        
        if task is None:  # 結束訊號
            break
            
        print(f"Worker {worker_id} 處理: {task}")
        await asyncio.sleep(2)  # 模擬處理
        queue.task_done()

async def main():
    queue = Queue()
    
    # 加入任務
    for i in range(10):
        await queue.put(f"task_{i}")
    
    # 加入結束訊號（每個 worker 一個）
    for _ in range(3):
        await queue.put(None)
    
    # 啟動 3 個 worker
    workers = [asyncio.create_task(worker(i, queue)) for i in range(3)]
    
    # 等待所有任務完成
    await queue.join()
    
    # 取消 worker
    for w in workers:
        w.cancel()

asyncio.run(main())
```

### 實際應用：任務排程

```python
import asyncio
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

class TaskQueue:
    def __init__(self, max_workers: int = 3):
        self.queue = asyncio.Queue()
        self.max_workers = max_workers
        self.workers = []
    
    async def start(self):
        """啟動 worker 池"""
        self.workers = [
            asyncio.create_task(self._worker(i))
            for i in range(self.max_workers)
        ]
    
    async def _worker(self, worker_id: int):
        """Worker 迴圈"""
        while True:
            task = await self.queue.get()
            if task is None:
                self.queue.task_done()
                break
            
            await task()
            self.queue.task_done()
    
    async def submit(self, coro):
        """提交任務"""
        await self.queue.put(coro)
    
    async def shutdown(self):
        """關閉 worker"""
        for _ in self.max_workers:
            await self.queue.put(None)
        await asyncio.gather(*self.workers)

# 全域任務佇列
task_queue = TaskQueue(max_workers=3)

@app.on_event("startup")
async def startup():
    await task_queue.start()

@app.on_event("shutdown")
async def shutdown():
    await task_queue.shutdown()

@app.post("/test")
async def run_test(test_id: str, background_tasks: BackgroundTasks):
    async def test_task():
        await asyncio.sleep(5)  # 模擬測試
        print(f"測試完成: {test_id}")
    
    await task_queue.submit(test_task())
    return {"status": "queued", "test_id": test_id}
```

---

## 📚 6. 完整範例：AutoQA 測試系統

### 架構圖

```
[HTTP POST /tests]
       ↓
[立即回應 test_id]
       ↓
[加入任務佇列]
       ↓
[Semaphore 限制並發]
       ↓
[Worker 執行 Playwright]
       ↓
[WebSocket 推播進度]
```

### 完整程式碼

```python
import asyncio
from fastapi import FastAPI, BackgroundTasks, WebSocket
from pydantic import BaseModel
from typing import Dict
import uuid

app = FastAPI()

# ============ 設定 ============
MAX_CONCURRENT_TESTS = 3  # 最多同時 3 個測試
test_semaphore = asyncio.Semaphore(MAX_CONCURRENT_TESTS)

# 儲存 WebSocket 連線
websockets: Dict[str, WebSocket] = {}

# ============ 資料模型 ============
class TestRequest(BaseModel):
    name: str
    steps: list

class TestResponse(BaseModel):
    test_id: str
    status: str

# ============ API ============
@app.post("/tests", response_model=TestResponse)
async def create_test(request: TestRequest, background_tasks: BackgroundTasks):
    """建立測試"""
    test_id = str(uuid.uuid4())[:8]
    
    # 立即回應
    return TestResponse(test_id=test_id, status="started")

@app.websocket("/ws/test/{test_id}")
async def test_websocket(websocket: WebSocket, test_id: str):
    """WebSocket 接收測試進度"""
    await websocket.accept()
    websockets[test_id] = websocket
    
    try:
        while True:
            await websocket.receive_text()
    except:
        websockets.pop(test_id, None)

# ============ 背景任務 ============
async def run_test_in_background(test_id: str, steps: list):
    """在背景執行測試"""
    async with test_semaphore:
        # 發送開始
        await send_progress(test_id, "running", 0)
        
        # 執行每個步驟
        for i, step in enumerate(steps):
            # 執行 Playwright
            result = await execute_playwright_step(step)
            
            # 發送進度
            progress = (i + 1) / len(steps)
            await send_progress(test_id, "running", progress, result)
            
            # 模擬等待
            await asyncio.sleep(1)
        
        # 發送完成
        await send_progress(test_id, "completed", 1.0)

async def send_progress(test_id: str, status: str, progress: float, result: dict = None):
    """發送進度給 WebSocket"""
    if test_id in websockets:
        message = {
            "test_id": test_id,
            "status": status,
            "progress": progress
        }
        if result:
            message["result"] = result
        await websockets[test_id].send_json(message)

async def execute_playwright_step(step: dict):
    """執行單一步驟"""
    # 實際的 Playwright 操作
    await asyncio.sleep(1)  # 模擬
    return {"success": True}

# ============ 使用 ============
# 當有測試請求時：
# asyncio.create_task(run_test_in_background(test_id, steps))
```

---

## 📚 7. 錯誤處理

### Timeout

```python
import asyncio
from fastapi import HTTPException

async def run_test_with_timeout(test_id: str, timeout: int = 300):
    try:
        return await asyncio.wait_for(
            execute_test(test_id),
            timeout=timeout
        )
    except asyncio.TimeoutError:
        raise HTTPException(status_code=408, detail="測試執行超時")
```

### 重試機制

```python
async def run_with_retry(task_func, max_retries: int = 3):
    for attempt in range(max_retries):
        try:
            return await task_func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(2 ** attempt)  # 指數退避
```

---

## 📝 今日小任務

1. 建立一個 FastAPI 伺服器，加入背景任務
2. 實現 Semaphore 限制並發數
3. 建立簡單的任務佇列
4. 實現 WebSocket 進度回報

---

## 🔗 參考連結

- [FastAPI Background Tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)
- [Python asyncio](https://docs.python.org/3/library/asyncio.html)
- [Python Semaphore](https://docs.python.org/3/library/asyncio-sync.html#asyncio.Semaphore)
