# 🎯 第二階段：FastAPI 骨架 (AutoQA 的進入點)

> 這是負責接收請求、派發任務的系統。

---

## 📚 2.1 FastAPI 基礎與第一步

### 什麼是 FastAPI？

FastAPI 是一個現代、快速的 Python Web 框架，用於建立 API。特點：
- **快速**：效能與 NodeJS、Go 相當
- **簡單**：容易上手
- **自動文件**：內建 Swagger UI
- **型別安全**：完全支援 Python 型別提示

### 安裝

```bash
pip install fastapi uvicorn
```

### 第一個 API

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello, World!"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### 啟動伺服器

```bash
uvicorn main:app --reload
```

### 訪問

- API 文件：http://127.0.0.1:8000/docs
- 替代文件：http://127.0.0.1:8000/redoc

---

## 📚 2.2 路徑與查詢參數 (Path & Query Parameters)

### 路徑參數 (Path Parameters)

網址中的變數：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id, "name": "小明"}

@app.get("/posts/{post_id}")
def get_post(post_id: int):
    return {"post_id": post_id, "title": "文章標題"}
```

訪問：
- `/users/1` → `user_id = 1`
- `/posts/5` → `post_id = 5`

### 查詢參數 (Query Parameters)

網址中 `?` 後面的參數：

```python
from fastapi import FastAPI

app = FastAPI()

# ?limit=10&offset=0
@app.get("/items")
def get_items(limit: int = 10, offset: int = 0):
    return {
        "limit": limit,
        "offset": offset,
        "items": ["item1", "item2", "item3"]
    }

# 可選的查詢參數
@app.get("/search")
def search(q: str = None):
    if q:
        return {"query": q, "results": ["result1", "result2"]}
    return {"results": []}
```

訪問：
- `/items?limit=5` → `limit = 5`
- `/search?q=python` → `q = "python"`

### 混合使用

```python
@app.get("/users/{user_id}/posts")
def get_user_posts(
    user_id: int,  # 路徑參數
    limit: int = 10,  # 查詢參數
    published: bool = True  # 查詢參數
):
    return {
        "user_id": user_id,
        "limit": limit,
        "published": published
    }
```

訪問：`/users/1/posts?limit=5&published=false`

---

## 📚 2.3 請求本體 (Request Body)

### 什麼是 Request Body？

當客戶端需要傳送複雜資料（如 JSON）給伺服器時，使用 Request Body。

### 使用 Pydantic 模型

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    quantity: int = 0

@app.post("/items/")
def create_item(item: Item):
    return {
        "name": item.name,
        "price": item.price,
        "total": item.price * item.quantity
    }
```

### 測試

```bash
curl -X POST "http://127.0.0.1:8000/items/" \
  -H "Content-Type: application/json" \
  -d '{"name": "iPhone", "price": 999.0, "quantity": 2}'
```

### 巢狀物件

```python
from pydantic import BaseModel
from typing import List

class Item(BaseModel):
    name: str
    price: float

class Order(BaseModel):
    user_id: int
    items: List[Item]  # 巢狀列表

@app.post("/orders/")
def create_order(order: Order):
    total = sum(item.price for item in order.items)
    return {"user_id": order.user_id, "total": total}
```

### 多個 Body 參數

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item, priority: bool = False):
    return {
        "item_id": item_id,
        "item": item,
        "priority": priority
    }
```

---

## 📚 2.4 實戰：AutoQA API 範例

根據 AutoQA 專案，這是常見的 API 結構：

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
from typing import List, Optional
from enum import Enum

app = FastAPI(title="AutoQA API")

# ============ 資料模型 ============

class TestStatus(str, Enum):
    PENDING = "pending"
    RUNNING = "running"
    COMPLETED = "completed"
    FAILED = "failed"

class TestCreate(BaseModel):
    name: str
    description: Optional[str] = None
    test_data: dict

class TestRun(BaseModel):
    id: str
    name: str
    status: TestStatus
    progress: float = 0.0
    result: Optional[dict] = None

# ============ API 端點 ============

@app.post("/tests", response_model=TestRun)
async def create_test(test: TestCreate, background_tasks: BackgroundTasks):
    """建立新測試"""
    test_id = "test_123"  # 實際應該生成 UUID
    test_run = TestRun(
        id=test_id,
        name=test.name,
        status=TestStatus.PENDING
    )
    
    # 啟動背景任務執行測試
    background_tasks.add_task(run_test, test_id, test.test_data)
    
    return test_run

@app.get("/tests/{test_id}", response_model=TestRun)
async def get_test_status(test_id: str):
    """取得測試狀態"""
    return TestRun(
        id=test_id,
        name="Example Test",
        status=TestStatus.RUNNING,
        progress=0.5
    )

@app.websocket("/ws/test/{test_id}")
async def websocket_test(websocket, test_id: str):
    """WebSocket 即時狀態更新"""
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            # 發送進度更新
            await websocket.send_json({
                "test_id": test_id,
                "progress": 0.75,
                "status": "running"
            })
    except Exception:
        pass

async def run_test(test_id: str, test_data: dict):
    """背景任務：執行測試"""
    # 這裡會呼叫 Playwright 執行自動化測試
    pass
```

---

## 📝 今日小任務

1. 建立一個 `/users/{user_id}/posts` API，回傳用戶的文章列表
2. 加入查詢參數 `limit` 和 `offset` 進行分頁
3. 使用 POST 建立新文章，傳入 JSON body

---

## 🔗 參考連結

- [FastAPI First Steps](https://fastapi.tiangolo.com/tutorial/first-steps/)
- [FastAPI Path Parameters](https://fastapi.tiangolo.com/tutorial/path-params/)
- [FastAPI Request Body](https://fastapi.tiangolo.com/tutorial/body/)
