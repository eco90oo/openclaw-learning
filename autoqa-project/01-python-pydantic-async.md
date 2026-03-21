# 🎯 第一階段：Python 核心與 Pydantic (AutoQA 的地基)

> 這是 AutoQA 專案的基礎設施，AI 生成的程式碼之所以像天書，通常是因為對這兩個概念不熟。

---

## 📚 1.1 Python 型別提示 (Type Hints)

### 為什麼要學這個？

Python 型別提示是讓程式碼具備「**自動補全**」功能，並且是讓 FastAPI 正常運作的核心基礎。

### 基礎語法

```python
# 變數型別提示
name: str = "小明"
age: int = 25
price: float = 99.9
is_active: bool = True

# 函式型別提示
def greet(name: str) -> str:
    return f"Hello, {name}!"

def add(a: int, b: int) -> int:
    return a + b
```

### 常見型別

| 型別 | 說明 | 範例 |
|------|------|------|
| `str` | 字串 | `"Hello"` |
| `int` | 整數 | `42` |
| `float` | 浮點數 | `3.14` |
| `bool` | 布林值 | `True` |
| `list` | 列表 | `[1, 2, 3]` |
| `dict` | 字典 | `{"key": "value"}` |
| `Optional` | 可選值 | `Optional[str]` = `str \| None` |
| `Union` | 聯合型別 | `Union[int, str]` = `int \| str` |

### 進階：巢狀型別

```python
from typing import List, Dict, Optional

# 複雜的資料結構
users: List[Dict[str, str]] = [
    {"name": "小明", "email": "test@example.com"},
    {"name": "小華", "email": "test2@example.com"}
]

# 可選值
result: Optional[str] = None  # 相當於 result: str | None
```

### 在 FastAPI 中的應用

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int) -> dict:
    return {"user_id": user_id, "name": "小明"}

@app.post("/items/")
def create_item(name: str, price: float) -> dict:
    return {"name": name, "price": price}
```

---

## 📚 1.2 Pydantic 資料模型 (Models)

### 為什麼要學這個？

不管是前端傳來的 JSON，還是給 LangGraph Agent 處理的「狀態 (State)」，全部都是仰賴 **Pydantic 進行嚴格的資料驗證**。

### 什麼是 Pydantic？

Pydantic 是一個 Python 資料驗證庫，可以：
1. 自動驗證資料型別
2. 自動轉換資料（例如字串 `"123"` → 數字 `123`）
3. 產生 JSON Schema（API 文件自動生成）

### 基礎用法

```python
from pydantic import BaseModel

# 定義一個模型
class User(BaseModel):
    id: int
    name: str
    email: str
    age: Optional[int] = None  # 可選欄位，有預設值

# 建立實例
user = User(id=1, name="小明", email="test@example.com")
print(user)
# 輸出: User(id=1, name='小明', email='test@example.com', age=None)

# 從 JSON 建立
data = {
    "id": 2,
    "name": "小華",
    "email": "test2@example.com",
    "age": 25
}
user2 = User(**data)
```

### 自動類型轉換

```python
# Pydantic 會自動轉換類型
user = User(id="123", name="小明", email="test@example.com")
print(user.id)  # 輸出: 123 (自動從 str 轉成 int)
```

### 巢狀模型

```python
from pydantic import BaseModel
from typing import List

class Address(BaseModel):
    city: str
    country: str

class User(BaseModel):
    id: int
    name: str
    addresses: List[Address]  # 巢狀結構

# 建立
user = User(
    id=1,
    name="小明",
    addresses=[
        {"city": "台北", "country": "台灣"},
        {"city": "東京", "country": "日本"}
    ]
)
```

### Field 客製化

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    id: int = Field(..., description="用戶 ID")
    name: str = Field(..., min_length=1, max_length=100)
    email: str = Field(..., regex=r"^[\w\.-]+@[\w\.-]+\.\w+$")
    age: int = Field(default=0, ge=0, le=150)  # ge=最小值, le=最大值
```

### 驗證錯誤

```python
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    id: int
    name: str

try:
    user = User(id="abc", name="小明")  # id 應該是 int
except ValidationError as e:
    print(e)
    # 輸出驗證錯誤訊息
```

### 在 FastAPI 中使用

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: EmailStr  # 自動驗證 email 格式
    password: str

@app.post("/users/")
def create_user(user: UserCreate):
    return {"username": user.username, "email": user.email}
```

---

## 📚 1.3 非同步機制 (Async / Await)

### 為什麼要學這個？

因為 AutoQA 需要「呼叫 AI API」和「操控瀏覽器」，這兩件事都需要**等待時間**。必須搞懂何時要宣告 `async def`，何時要在函數呼叫前面加上 `await`。

### 比喻：買漢堡

想像你去速食店買漢堡：

**同步（傳統）方式**：
- 你站在櫃台等廚師做好
- 在等待期間，你什麼都不能做
- 浪費時間

**非同步（async/await）方式**：
- 你點餐後得到一個號碼牌
- 你可以去座位上聊天、滑手機
- 號碼牌響了再去取餐
- **效率更高！**

### Python 中的 async

```python
import asyncio

# 定義一個非同步函數
async def get_data():
    # 模擬等待（例如 API 呼叫）
    await asyncio.sleep(1)  # 等待 1 秒
    return "資料回來了！"

# 執行非同步函數
async def main():
    result = await get_data()
    print(result)

# 啟動
asyncio.run(main())
```

### 何時用 async / await？

| 情況 | 做法 |
|------|------|
| 呼叫 AI API | `async def` + `await` |
| 操控瀏覽器 (Playwright) | `async def` + `await` |
| 讀寫檔案 | `async def` + `await aiofiles` |
| 資料庫操作 | `async def` + `await` (需要 async 驅動) |
| 簡單計算 | 用普通 `def` 即可 |

### 在 FastAPI 中使用

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

# 普通端點
@app.get("/normal")
def normal_endpoint():
    return {"message": "同步端點"}

# 非同步端點
@app.get("/async")
async def async_endpoint():
    # 模擬等待
    await asyncio.sleep(1)
    return {"message": "非同步端點"}
```

### 重要原則

1. **有 `await` 就要用 `async def`**
2. **普通函數也能呼叫 async 函數**（會阻塞）
3. **不要在 async 函數裡用 `time.sleep()`**（會阻塞），要用 `await asyncio.sleep()`

---

## 📝 今日小任務

1. 建立一個 Pydantic 模型，包含：姓名、年齡、Email
2. 嘗試用 FastAPI 建立一個簡單的 GET API
3. 嘗試用 async / await 寫一個模擬 API 呼叫的函數

---

## 🔗 參考連結

- [Python Type Hints](https://fastapi.tiangolo.com/python-types/)
- [Pydantic Models](https://docs.pydantic.dev/latest/concepts/models/)
- [FastAPI Async](https://fastapi.tiangolo.com/async/)
