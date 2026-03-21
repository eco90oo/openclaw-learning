# 🧩 補齊拼圖 1：WebSocket 雙向通訊完整教學

> 這是 AutoQA 專案中實現「即時進度回報」的關鍵技術

---

## 📚 1. 為什麼 HTTP 不夠用？

### HTTP 的限制

```
HTTP 請求流程：
1. 客戶端發請求 → 2. 伺服器處理 → 3. 伺服器回應 → 4. 連線結束

問題：
- 客戶端只能「問」才能得到回應
- 伺服器不能「主動」推播資料
- 如果處理時間很長，客戶端只能一直等
```

### 實際問題

在 AutoQA 中：
- 啟動一個測試可能需要 30 秒到 5 分鐘
- 如果用 HTTP，客戶端要一直等待回應
- 使用者體驗很差

### 解決方案：WebSocket

WebSocket 建立「持續連線的通道」：
- 連線建立後不會斷開
- 伺服器可以隨時主動發訊息給客戶端
- 客戶端也可以隨時發訊息給伺服器

---

## 📚 2. WebSocket 基礎語法

### 伺服器端（FastAPI）

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws/test/{test_id}")
async def websocket_endpoint(websocket: WebSocket, test_id: str):
    """
    WebSocket 端點
    test_id: 測試的 ID
    """
    # 1. 接受連線
    await websocket.accept()
    print(f"客戶端連線: test_id={test_id}")
    
    try:
        while True:
            # 2. 接收客戶端訊息
            data = await websocket.receive_text()
            print(f"收到: {data}")
            
            # 3. 發送訊息給客戶端
            await websocket.send_text(f"收到: {data}")
            
    except WebSocketDisconnect:
        print(f"客戶端斷線: test_id={test_id}")
```

### 客戶端（JavaScript）

```javascript
// 建立 WebSocket 連線
const ws = new WebSocket("ws://localhost:8000/ws/test/123");

// 連線打開時
ws.onopen = () => {
    console.log("WebSocket 連線成功");
    ws.send(JSON.stringify({ type: "start" }));
};

// 收到訊息時
ws.onmessage = (event) => {
    console.log("收到:", event.data);
};

// 發生錯誤時
ws.onerror = (error) => {
    console.error("錯誤:", error);
};

// 連線關閉時
ws.onclose = () => {
    console.log("連線結束");
};
```

---

## 📚 3. JSON 格式通訊

### 伺服器發送結構化資料

```python
import json

@app.websocket("/ws/test/{test_id}")
async def test_websocket(websocket: WebSocket, test_id: str):
    await websocket.accept()
    
    try:
        # 發送 JSON 格式的訊息
        await websocket.send_json({
            "type": "status",
            "status": "running",
            "progress": 0.0,
            "message": "測試開始"
        })
        
        # 模擬測試進度
        for i in range(1, 11):
            await websocket.send_json({
                "type": "progress",
                "progress": i * 10,
                "message": f"執行步驟 {i}/10"
            })
            await asyncio.sleep(1)  # 模擬執行時間
        
        # 測試完成
        await websocket.send_json({
            "type": "complete",
            "status": "success",
            "result": {"passed": 10, "failed": 0}
        })
        
    except WebSocketDisconnect:
        pass
```

### 客戶端接收並處理

```javascript
ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    
    switch(data.type) {
        case "status":
            console.log("狀態:", data.status);
            break;
        case "progress":
            updateProgressBar(data.progress);
            console.log("進度:", data.message);
            break;
        case "complete":
            console.log("測試完成!", data.result);
            break;
    }
};
```

---

## 📚 4. 多客戶端管理

### 伺服器管理多個連線

```python
from fastapi import FastAPI, WebSocket
from typing import Dict, List
import asyncio

app = FastAPI()

# 儲存所有連線
# 格式: {test_id: [WebSocket1, WebSocket2, ...]}
connections: Dict[str, List[WebSocket]] = {}

@app.websocket("/ws/test/{test_id}")
async def test_websocket(websocket: WebSocket, test_id: str):
    await websocket.accept()
    
    # 加入連線列表
    if test_id not in connections:
        connections[test_id] = []
    connections[test_id].append(websocket)
    
    try:
        while True:
            data = await websocket.receive_text()
            # 廣播給所有訂閱這個測試的客戶端
            for ws in connections[test_id]:
                if ws != websocket:  # 不發給自己
                    await ws.send_text(data)
                    
    except WebSocketDisconnect:
        # 移除連線
        if websocket in connections[test_id]:
            connections[test_id].remove(websocket)
```

### 廣播功能

```python
async def broadcast_to_test(test_id: str, message: dict):
    """廣播訊息給所有訂閱某測試的客戶端"""
    if test_id not in connections:
        return
    
    disconnected = []
    for ws in connections[test_id]:
        try:
            await ws.send_json(message)
        except:
            disconnected.append(ws)
    
    # 清理斷線的客戶端
    for ws in disconnected:
        connections[test_id].remove(ws)
```

---

## 📚 5. 實際應用：AutoQA 測試進度

### 完整的流程

```python
from fastapi import FastAPI, WebSocket, BackgroundTasks
from pydantic import BaseModel
import asyncio

app = FastAPI()

# 連線管理
test_connections: Dict[str, WebSocket] = {}

class TestRequest(BaseModel):
    name: str
    steps: list

@app.post("/tests")
async def create_test(request: TestRequest, background_tasks: BackgroundTasks):
    """建立測試並立即回應"""
    test_id = f"test_{len(test_connections) + 1}"
    
    # 啟動背景任務執行測試
    background_tasks.add_task(run_test, test_id, request.steps)
    
    return {"test_id": test_id, "status": "started"}

@app.websocket("/ws/test/{test_id}")
async def test_status(websocket: WebSocket, test_id: str):
    """WebSocket 接收測試進度"""
    await websocket.accept()
    test_connections[test_id] = websocket
    
    try:
        while True:
            await websocket.receive_text()  # 保持連線
    except:
        test_connections.pop(test_id, None)

async def run_test(test_id: str, steps: list):
    """背景任務：執行測試"""
    # 通知開始
    await test_connections[test_id].send_json({
        "type": "status",
        "status": "running",
        "progress": 0
    })
    
    # 執行每個步驟
    for i, step in enumerate(steps):
        # 執行 Playwright
        result = await execute_playwright_step(step)
        
        # 發送進度
        await test_connections[test_id].send_json({
            "type": "progress",
            "progress": (i + 1) / len(steps) * 100,
            "step": step,
            "result": result
        })
    
    # 完成
    await test_connections[test_id].send_json({
        "type": "complete",
        "status": "success"
    })

async def execute_playwright_step(step):
    """執行單一步驟"""
    # 實際的 Playwright 操作
    pass
```

---

## 📚 6. 前端實作：React/Vue 範例

### React

```jsx
import { useState, useEffect } from 'react';

function TestProgress({ testId }) {
    const [status, setStatus] = useState('idle');
    const [progress, setProgress] = useState(0);
    const [results, setResults] = useState(null);
    
    useEffect(() => {
        const ws = new WebSocket(`ws://localhost:8000/ws/test/${testId}`);
        
        ws.onopen = () => {
            console.log('連線成功');
        };
        
        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            
            switch(data.type) {
                case 'status':
                    setStatus(data.status);
                    break;
                case 'progress':
                    setProgress(data.progress);
                    break;
                case 'complete':
                    setStatus('complete');
                    setResults(data.result);
                    break;
            }
        };
        
        return () => ws.close();
    }, [testId]);
    
    return (
        <div>
            <p>狀態: {status}</p>
            <progress value={progress} max="100">{progress}%</progress>
        </div>
    );
}
```

---

## 📚 7. 常見問題與解決

### 1. 連線中斷

```python
# 加入心跳機制
async def keep_alive(websocket: WebSocket):
    while True:
        try:
            await websocket.send_json({"type": "ping"})
            await asyncio.sleep(30)  # 每 30 秒 ping 一次
        except:
            break
```

### 2. 重新連線

```javascript
// 客戶端自動重連
function connect() {
    ws = new WebSocket(url);
    ws.onclose = () => {
        setTimeout(connect, 3000);  // 3 秒後重連
    };
}
```

### 3. 安全性

```python
from fastapi import WebSocket, Query

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket, token: str = Query(None)):
    # 驗證 token
    if not verify_token(token):
        await websocket.close(code=4003)
        return
    
    await websocket.accept()
    # 繼續處理...
```

---

## 📝 今日小任務

1. 建立一個 FastAPI WebSocket 伺服器
2. 實現客戶端連線和基本訊息傳遞
3. 實現進度回報功能
4. 嘗試實現斷線重連

---

## 🔗 參考連結

- [FastAPI WebSocket](https://fastapi.tiangolo.com/advanced/websockets/)
- [MDN WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
