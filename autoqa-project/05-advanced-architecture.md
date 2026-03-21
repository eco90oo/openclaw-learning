# 🧩 補齊拼圖：AutoQA 核心架構解析

> 除了四大核心，還有三個進階的系統設計拼圖必須掌握，才能看懂專案的全貌。

---

## 📚 補齊拼圖 1：WebSocket 雙向通訊

### 為什麼你需要懂這個？

測試執行時間通常很長，不可能用傳統的 HTTP Request 等到測試跑完才回傳結果。因此前端會先打一個 API 啟動測試，然後立刻透過 WebSocket 連線，像看直播一樣接收 step、status、complete 的即時串流狀態。

### 什麼是 WebSocket？

HTTP 是「一問一答」的模式：
- 客戶端發請求 → 伺服器回應 → 連線結束

WebSocket 是「建立後不斷開的水管」：
- 建立連線後，雙方可以隨時互相傳訊息
- 伺服器可以主動推播資料給客戶端

### 比喻

| HTTP | WebSocket |
|------|----------|
| 打電話（說完就掛）| 對講機（一直開著）|
| 客戶端問 → 伺服器答 | 雙方隨時可講話 |
| 伺服器不能主動打給客戶 | 伺服器可以主動推播 |

### FastAPI WebSocket 語法

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws/test/{test_id}")
async def websocket_test(websocket: WebSocket, test_id: str):
    # 1. 接受連線
    await websocket.accept()
    
    try:
        while True:
            # 2. 接收客戶端訊息
            data = await websocket.receive_text()
            
            # 3. 發送訊息給客戶端
            await websocket.send_json({
                "type": "status",
                "test_id": test_id,
                "progress": 0.5,
                "message": "執行中..."
            })
            
    except WebSocketDisconnect:
        print(f"客戶端斷線: {test_id}")
```

### AutoQA 中的應用

```python
from fastapi import FastAPI, WebSocket
from typing import Dict
import json

app = FastAPI()

# 儲存所有連線
active_connections: Dict[str, WebSocket] = {}

@app.websocket("/ws/test/{test_id}")
async def test_websocket(websocket: WebSocket, test_id: str):
    await websocket.accept()
    active_connections[test_id] = websocket
    
    try:
        while True:
            # 接收客戶端訊息
            data = await websocket.receive_text()
            message = json.loads(data)
            
            # 處理訊息
            if message.get("type") == "start":
                # 開始執行測試
                await websocket.send_json({
                    "type": "status",
                    "status": "running",
                    "progress": 0
                })
                
    except Exception:
        pass
    finally:
        # 清理連線
        active_connections.pop(test_id, None)

# 背景任務中呼叫這個函數來推播
async def send_progress(test_id: str, progress: float, message: str):
    if test_id in active_connections:
        await active_connections[test_id].send_json({
            "type": "progress",
            "progress": progress,
            "message": message
        })
```

### 前端連線範例（JavaScript）

```javascript
// 建立 WebSocket 連線
const ws = new WebSocket("ws://localhost:8000/ws/test/123");

ws.onopen = () => {
    console.log("連線成功");
    ws.send(JSON.stringify({ type: "start" }));
};

ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log("收到:", data);
    
    if (data.type === "progress") {
        console.log(`進度: ${data.progress * 100}%`);
    }
};

ws.onerror = (error) => {
    console.error("錯誤:", error);
};

ws.onclose = () => {
    console.log("連線結束");
};
```

---

## 📚 補齊拼圖 2：背景任務與併發控制

### 為什麼你需要懂這個？

當前端送出 POST /tests，後端會立刻回傳 test_id，但此時 Playwright 才正要啟動。這代表 run_test() 是一個「被丟到背景去跑」的任務。

此外，TaskService 有做 semaphore（信號量）控制併發，這是為了避免同時開啟太多瀏覽器實例把伺服器資源耗盡弄掛。

### FastAPI 背景任務

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

def run_test(task_id: str):
    """背景任務函數"""
    print(f"開始執行測試: {task_id}")
    # 這會在背景執行
    # ... 執行 Playwright 測試
    print(f"測試完成: {task_id}")

@app.post("/tests/")
async def create_test(background_tasks: BackgroundTasks):
    task_id = "test_123"
    
    # 把任務加入背景執行
    background_tasks.add_task(run_test, task_id)
    
    # 立即回傳，不會等待測試完成
    return {"task_id": task_id, "status": "started"}
```

### Python asyncio 控制併發

#### Semaphore（信號量）

Semaphore 控制同時可以有多少個任務執行：

```python
import asyncio
from typing import Dict

# 建立信號量，限制最多 5 個同時執行
semaphore = asyncio.Semaphore(5)

async def run_test(test_id: str):
    async with semaphore:
        print(f"開始測試: {test_id}")
        # 執行 Playwright 測試
        await asyncio.sleep(3)  # 模擬測試
        print(f"完成測試: {test_id}")

async def main():
    tasks = [run_test(f"test_{i}") for i in range(20)]
    await asyncio.gather(*tasks)

asyncio.run(main())
```

#### Queue（佇列）

用於任務排隊：

```python
import asyncio
from asyncio import Queue

async def worker(worker_id: int, queue: Queue):
    while True:
        task = await queue.get()
        print(f"Worker {worker_id} 處理: {task}")
        await asyncio.sleep(2)  # 模擬處理
        queue.task_done()

async def main():
    queue = Queue()
    
    # 加入任務
    for i in range(10):
        await queue.put(f"task_{i}")
    
    # 啟動 3 個 worker
    workers = [asyncio.create_task(worker(i, queue)) for i in range(3)]
    
    await queue.join()
    
    # 取消 worker
    for w in workers:
        w.cancel()

asyncio.run(main())
```

### AutoQA 中的應用

```python
import asyncio
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel

app = FastAPI()

# 限制最多 3 個測試同時執行
test_semaphore = asyncio.Semaphore(3)

class TestTask(BaseModel):
    test_id: str
    test_data: dict

async def execute_test(test_id: str, test_data: dict):
    async with test_semaphore:
        # 執行 Playwright 測試
        print(f"開始執行測試: {test_id}")
        await asyncio.sleep(5)  # 模擬測試
        print(f"完成測試: {test_id}")

@app.post("/tests/")
async def create_test(task: TestTask, background_tasks: BackgroundTasks):
    background_tasks.add_task(execute_test, task.test_id, task.test_data)
    return {"test_id": task.test_id, "status": "started"}
```

---

## 📚 補齊拼圖 3：抽象基底類別與策略模式

### 為什麼你需要懂這個？

文件中有一段極度核心的設計：AgentService._execute_action 透過 PlaywrightService 去呼叫 src/tools/* 裡的工具。這意味著 LLM 只負責產出一個有 action_type: "click" 的 JSON，而你的 Python 程式會「**自動找到對應的 ClickTool 去執行**」。

這就是**策略模式 (Strategy Pattern)** 的應用！

### 什麼是 ABC？

ABC (Abstract Base Class) 是 Python 用來定義「介面」的工具。在 Java 中你有 Interface，在 Python 中你可以用 ABC。

### 比喻

```
抽象基底類別 (BaseTool) = 契約/規範
   ↓
每個具體工具 (ClickTool, FillTool) = 實作
```

### 實作

```python
from abc import ABC, abstractmethod
from typing import Any

# 1. 定義抽象基底類別
class BaseTool(ABC):
    """所有工具的基底類別"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """工具名稱"""
        pass
    
    @property
    @abstractmethod
    def description(self) -> str:
        """工具描述"""
        pass
    
    @abstractmethod
    async def execute(self, **kwargs) -> Any:
        """執行工具"""
        pass

# 2. 實作具體工具
class ClickTool(BaseTool):
    @property
    def name(self) -> str:
        return "click"
    
    @property
    def description(self) -> str:
        return "點擊網頁元素"
    
    async def execute(self, selector: str, **kwargs) -> dict:
        # 實際呼叫 Playwright
        await page.click(selector)
        return {"success": True, "action": "click", "selector": selector}

class FillTool(BaseTool):
    @property
    def name(self) -> str:
        return "fill"
    
    @property
    def description(self) -> str:
        return "填寫輸入框"
    
    async def execute(self, selector: str, value: str, **kwargs) -> dict:
        await page.fill(selector, value)
        return {"success": True, "action": "fill", "selector": selector, "value": value}

# 3. 工具工廠
class ToolFactory:
    def __init__(self):
        self.tools: dict[str, BaseTool] = {}
    
    def register(self, tool: BaseTool):
        self.tools[tool.name] = tool
    
    def get_tool(self, name: str) -> BaseTool:
        if name not in self.tools:
            raise ValueError(f"Tool {name} not found")
        return self.tools[name]

# 4. 使用
factory = ToolFactory()
factory.register(ClickTool())
factory.register(FillTool())

# LLM 返回 {"action": "click", "selector": "#submit"}
action = "click"
tool = factory.get_tool(action)
result = await tool.execute(selector="#submit")
```

### 為什麼這樣設計？

| 優點 | 說明 |
|------|------|
| **易擴充** | 新增工具只需要實作 BaseTool，不用改 Agent |
| **易維護** | 每個工具邏輯分開 |
| **符合開關原則** | 對擴展開放，對修改封閉 |
| **好測試** | 每個工具可以單獨測試 |

### AutoQA 中的應用

```python
# src/tools/base.py
from abc import ABC, abstractmethod

class BaseTool(ABC):
    @abstractmethod
    async def execute(self, page, params: dict) -> dict:
        pass

# src/tools/click.py
class ClickTool(BaseTool):
    async def execute(self, page, params: dict) -> dict:
        selector = params.get("selector")
        await page.click(selector)
        return {"success": True}

# src/tools/fill.py
class FillTool(BaseTool):
    async def execute(self, page, params: dict) -> dict:
        selector = params.get("selector")
        value = params.get("value")
        await page.fill(selector, value)
        return {"success": True}

# src/tools/factory.py
class ToolFactory:
    def __init__(self):
        self.tools = {
            "click": ClickTool(),
            "fill": FillTool(),
        }
    
    def get_tool(self, action_type: str):
        return self.tools.get(action_type)

# 使用
factory = ToolFactory()
tool = factory.get_tool("click")
await tool.execute(page, {"selector": "#button"})
```

---

## 📝 今日小任務

1. 嘗試建立一個 WebSocket 伺服器
2. 了解 Semaphore 的使用場景
3. 實作一個簡單的 BaseTool 抽象類別

---

## 🔗 參考連結

- [FastAPI WebSocket](https://fastapi.tiangolo.com/advanced/websockets/)
- [Python Background Tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)
- [Python asyncio](https://docs.python.org/3/library/asyncio-task.html)
- [Python ABC](https://docs.python.org/3/library/abc.html)
