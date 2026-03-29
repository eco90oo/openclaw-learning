# 📚 【今日學習：自動化測試 - Day 5】

## 🎯 今日主題：Mock 進階（patch, Mock object）- 深入實戰

---

## 1️⃣ 基礎概念（用生活比喻，越詳細越好）

### 📖 什麼是 Patch？

**比喻：特工「冒充」計畫執行者**

想象一下這個場景：

你是一個電影導演，正在拍摄一场「炸毁大樓」的戏。但你不可能、也不應該真的去炸毁一栋大楼。你會找一個「特技演員」來代替專業爆破師。這樣你可以控制「爆炸」的时机、效果，甚至让他「听你话」。

在自動化測試中，`unittest.mock.patch` 就是這個概念：

- 它會「临时替换」系统中的某个函数或对象
- 讓你在測試中「完全控制」這個替身的行為
- 測試結束後，它会自动「恢复」，不影響真正的程式碼

### 🎯 為什麼要用 Patch？

**問題情境**（昨天的 Mock 基礎無法解決）：

假設我們要測試這樣的程式碼：

```python
def send_email(to, subject, body):
    # 這是一個「外部服務」，在我們的測試環境無法真的發送
    import smtplib
    smtp_server = smtplib.SMTP('smtp.gmail.com', 587)
    smtp_server.sendmail(from_addr, to, message)
    smtp_server.quit()

def register_user(username, email):
    # 註冊成功後發送驗證郵件
    save_to_database(username, email)
    send_email(email, "歡迎註冊", "請驗證您的帳號")
    return {"status": "success"}
```

**問題**：

- 如果直接測試 `register_user`，會真的嘗試連接 SMTP 伺服器
- 測試環境沒有網路、沒有真的帳號密碼 → 測試失敗
- 這不叫「單元測試」，這叫「集成測試」！

**解決方案**：

- 我們要「patch」`send_email` 這個函數
- 讓它变成一個「假」的 send_email
- 我們可以控制它是「成功」還是「失敗」
- 我們可以驗證它「有沒有被調用」
- 我們可以檢查它「收到了什麼參數」

---

## 2️⃣ 核心知識（每行程式碼都要有中文註解！）

### 🛠️ Patch 的三種使用方式

#### 方式一：裝飾器方式（最推薦）

```python
# 引入 unittest.mock 中的 patch 裝飾器
# patch 會自動將「假」的物件注入到函數的參數中
from unittest.mock import patch

# 定義一個「假」的 send_email 函數
# 這是我們自己的測試輔助函數，不是 Mock 物件
def mock_send_email_success(to, subject, body):
    """這是一個「假」的郵件發送函數，永遠成功"""
    print(f"📧 [模擬] 發送郵件到: {to}")
    return True

@patch('myapp.services.send_email', mock_send_email_success)  # @patch 會把 send_email 替換成我們的 mock
def test_register_user_success():
    """測試用戶註冊成功發送驗證郵件"""
    
    # 呼叫要測試的函數
    result = register_user("john", "john@example.com")
    
    # 驗證結果
    assert result["status"] == "success"
```

#### 方式二：上下文管理器（with 語句）

```python
from unittest.mock import patch

def test_register_user_with_context_manager():
    """使用 with 語句的 patch 方式"""
    
    # 這裡開始「patch」
    # patch 會在 with 區塊內生效，離開後自動恢復
    with patch('myapp.services.send_email') as mock_send_email:
        # 設置「假」的 send_email 的回傳值
        # 當真的调用 send_email 時，會返回這個值
        mock_send_email.return_value = True
        
        # 呼叫要測試的函數
        result = register_user("john", "john@example.com")
        
        # 驗證結果
        assert result["status"] == "success"
        
        # 驗證 send_email 有被調用
        # assert_called_once() 是 Mock 物件的專屬方法
        assert mock_send_email.called
        
        # 驗證 send_email 的調用次數
        # call_count 代表被調用了多少次
        assert mock_send_email.call_count == 1
        
        # 檢查參數是否正確
        # call_args[0] 是位置參數，call_args[1] 是關鍵字參數
        call_args = mock_send_email.call_args
        assert call_args[0][0] == "john@example.com"  # to
        assert call_args[0][1] == "歡迎註冊"         # subject
```

#### 方式三：手動調用（start/stop）

```python
from unittest.mock import patch

def test_register_user_manual():
    """手動控制 patch 的開始和結束"""
    
    # 開始 patch
    # 類似「按下錄音鍵」
    mock_email = patch('myapp.services.send_email')
    mock_email.start()
    
    try:
        # 執行測試
        result = register_user("john", "john@example.com")
        assert result["status"] == "success"
    
    finally:
        # 結束 patch
        # 類似「放開錄音鍵」，恢復原始函數
        mock_email.stop()
```

### 🎭 Mock 物件的各種方法

```python
from unittest.mock import Mock, MagicMock, call

# 創建一個「萬能」的 Mock 物件
# MagicMock 是 Mock 的進階版，自動處理很多特殊情況
mock_obj = MagicMock()

# 1. 設置回傳值
mock_obj.return_value = "我是回傳值"
result = mock_obj()  # result = "我是回傳值"

# 2. 設置回傳值為異常（模擬錯誤）
mock_obj.side_effect = ValueError("這是一個錯誤")
try:
    mock_obj()  # 會拋出 ValueError
except ValueError as e:
    print(f"捕獲到錯誤: {e}")

# 3. 不同的調用返回不同的值（模擬序列）
def custom_side_effect(arg):
    """自定義的回應邏輯"""
    if arg == "valid":
        return "有效的結果"
    else:
        raise ValueError("無效的參數")

mock_obj.side_effect = custom_side_effect

result = mock_obj("valid")  # result = "有效的結論"
try:
    mock_obj("invalid")  # 會拋出 ValueError
except ValueError:
    print("錯誤被正確拋出")

# 4. 驗證調用次數
mock_obj.reset_mock()  # 重置 Mock 物件的狀態（調用次數歸零）

mock_obj()
mock_obj()
mock_obj()

assert mock_obj.call_count == 3  # 驗證被調用了 3 次

# 5. 驗證特定的調用
mock_func = MagicMock()
mock_func("hello", name="world")

# 檢查最後一次調用的參數
assert mock_func.call_args == call("hello", name="world")

# 6. 檢查是否被調用過
mock_func2 = MagicMock()
assert not mock_func2.called  # 還沒被調用

mock_func2()
assert mock_func2.called  # 已經被調用了
```

### 🎯 Patch 的目標寫法（重要！）

**口號：永遠 patch「被使用的位置」，不是「被定義的位置」！**

```python
# 假設程式碼結構如下：
# myapp/
#   __init__.py
#   services.py        # <- send_email 在這裡定義
#   user_manager.py   # <- register_user 在這裡使用 send_email
#   main.py

# 錯誤的 patch 方式：
@patch('myapp.services.send_email')  # 這是「定義的位置」❌
def test_wrong_way():
    from myapp import user_manager
    # 這樣不會生效，因為 send_email 沒有被正確替換
    pass

# 正確的 patch 方式：
@patch('myapp.user_manager.send_email')  # 這是「使用/引用」的位置✅
def test_right_way():
    # 這樣才能正確替換 register_user 中的 send_email
    pass

# 另一個例子：
# 如果 register_user 這樣寫：
# from myapp.services import send_email
# def register_user(username, email):
#     send_email(email, ...)

# 這��� patch 目標應該是 'myapp.user_manager.send_email'
# 因為它是在 user_manager.py 中被 import 和使用的
```

---

## 3️⃣ 實際操作（一步一步帶著做）

### 📁 專案結構

```
testing_practice/
├── myapp/
│   ├── __init__.py
│   ├── services.py         # 郵件服務
│   ├── user_manager.py   # 用戶管理
│   └── database.py       # 資料庫模擬
├── tests/
│   ├── __init__.py
│   └── test_user_manager.py  # 用戶管理測試
└── requirements.txt
```

### 📝 Step 1: 創建測試專案

```bash
# 創建目錄結構
mkdir -p testing_practice/myapp testing_practice/tests
cd testing_practice
```

### 📝 Step 2: 建立應用程式碼

建立 `myapp/services.py`（外部服務模擬）：

```python
# myapp/services.py
"""這是應用程式的外部服務模擬"""

import smtplib
from email.mime.text import MIMEText

def send_email(to, subject, body):
    """
    發送郵件的函數
    
    參數:
        to: 收件人郵件地址
        subject: 郵件主題
        body: 郵件內容
    
    返回:
        bool: 發送成功返回 True
    """
    # 這是一個「真的」郵件發送函數，需要 SMTP 伺服器
    # 在測試環境中，我們不會真的發送郵件
    from_addr = "noreply@example.com"
    
    msg = MIMEText(body, 'plain', 'utf-8')
    msg['Subject'] = subject
    msg['From'] = from_addr
    msg['To'] = to
    
    # 真的連接 SMTP 伺服器發送郵件
    # 這在測試環境會失敗！
    smtp_server = smtplib.SMTP('smtp.gmail.com', 587)
    smtp_server.starttls()
    smtp_server.login(from_addr, 'password')
    smtp_server.sendmail(from_addr, [to], msg.as_string())
    smtp_server.quit()
    
    return True
```

建立 `myapp/database.py`（資料庫模擬）：

```python
# myapp/database.py
"""這是資料庫的模擬操作"""

# 用一個簡單的字典模擬資料庫
_database = {}

def save_to_database(username, email):
    """
    儲存用戶到資料庫
    
    參數:
        username: 用戶名
        email: 電子郵件
    
    返回:
        dict: 儲存結果
    """
    if username in _database:
        return {"status": "error", "message": "用戶已存在"}
    
    _database[username] = email
    return {"status": "success"}

def get_user(username):
    """
    從資料庫獲取用戶
    
    參數:
        username: 用戶名
    
    返回:
        dict: 用戶資料
    """
    if username in _database:
        return {"username": username, "email": _database[username]}
    return None
```

建立 `myapp/user_manager.py`（用戶管理）：

```python
# myapp/user_manager.py
"""用戶管理模組"""

from myapp import services
from myapp import database

def register_user(username, email):
    """
    註冊新用戶
    
    參數:
        username: 用戶名（長度 3-20 字元）
        email: 有效的電子郵件地址
    
    返回:
        dict: 註冊結果
    """
    # 驗證用戶名
    if len(username) < 3 or len(username) > 20:
        return {"status": "error", "message": "用戶名長度需在 3-20 字元之間"}
    
    # 驗證電子郵件格式
    if '@' not in email:
        return {"status": "error", "message": "無效的電子郵件格式"}
    
    # 儲存到資料庫
    db_result = database.save_to_database(username, email)
    if db_result["status"] == "error":
        return db_result
    
    # 發送驗證郵件（這裡調用外部服務！）
    # 如果 SMTP 伺服器不可用，整��測��會失敗
    try:
        services.send_email(email, "歡迎註冊", f"親愛的 {username}，感謝您註冊！")
    except Exception as e:
        # 如果郵件發送失敗，回滾資料庫
        print(f"郵件發送失敗: {e}")
        return {"status": "error", "message": "郵件發送失敗"}
    
    return {"status": "success", "message": "註冊成功"}

def login_user(username):
    """
    用戶登入
    
    參數:
        username: 用戶名
    
    返回:
        dict: 登入結果
    """
    # 從資料庫獲取用戶
    user = database.get_user(username)
    
    if user is None:
        return {"status": "error", "message": "用戶不存在"}
    
    return {"status": "success", "user": user}
```

建立 `myapp/__init__.py`：

```python
# myapp/__init__.py
"""這是應用程式的入口點"""
```

### 📝 Step 3: 建立測試檔案

建立 `tests/__init__.py`：

```python
# tests/__init__.py
"""測試套件"""
```

建立 `tests/test_user_manager.py`（完整的 Mock 進階測試）：

```python
# tests/test_user_manager.py
"""
用戶管理模組的單元測試

這個測試檔案展示如何使用 Mock 和 Patch 來：
1. 隔離測試（不依賴外部 SMTP 服務）
2. 驗證函數被正確調用
3. 模擬各種錯誤場景
"""

# 引入 pytest（測試框架）
import pytest

# 引入 unittest.mock（Mock 工具）
from unittest.mock import patch, MagicMock, call

# 引入要測試的模組
from myapp.user_manager import register_user, login_user


# ============ 測試案例 1：基本 patch 使用（裝飾器方式）============
# 這是最推薦的方式，程式碼最乾淨

@patch('myapp.user_manager.services')  # patch 整個 services 模組
def test_register_user_success_decorator(mock_services):
    """
    測試用戶註冊成功（使用裝飾器方式的 patch）
    
    這個測試展示了：
    1. 如何使用 @patch 裝飾器替換整個模組
    2. 如何設置 mock 的回傳值
    3. 如何驗證結果
    """
    # 設置 mock_services.send_email 的回傳值為 True（表示成功）
    # 這樣當 register_user 調用 services.send_email 時，會返回 True
    mock_services.send_email.return_value = True
    
    # 呼叫要測試的 register_user 函數
    result = register_user("testuser", "test@example.com")
    
    # 驗證註冊成功
    assert result["status"] == "success"  # 狀態應該是成功
    assert result["message"] == "註冊成功"  # 訊息應該是註冊成功


# ============ 測試案例 2：使用 with 語句的 patch（上下文管理器）============
# 適合需要精確控制 patch 範圍的情況

@patch('myapp.user_manager.services.send_email')  # 只 patch send_email 函數
def test_register_user_with_context(mock_send_email):
    """
    測試用戶註冊成功（使用 with 語句的方式）
    
    這個測試展示了：
    1. 如何只 patch 特定的函數
    2. 如何驗證函數被調用
    3. 如何檢查調用的參數
    """
    # 設置 mock_send_email 的回傳值為 True
    mock_send_email.return_value = True
    
    # 呼叫 register_user
    result = register_user("alice", "alice@example.com")
    
    # 驗證結果
    assert result["status"] == "success"
    
    # 驗證 send_email 被調用了
    # .called 是 MagicMock 的屬性，檢查函數是否被調用過
    assert mock_send_email.called
    
    # 驗證 send_email 被調用了一次
    assert mock_send_email.call_count == 1
    
    # 驗證調用的參數
    # .call_args 包含最近一次調用的參數
    # call_args[0] 是位置參數 (args)
    # call_args[1] 是關鍵字參數 (kwargs)
    call_args = mock_send_email.call_args
    assert call_args[0][0] == "alice@example.com"  # 第一個參數是 email
    assert call_args[0][1] == "歡迎註冊"            # 第二個參數是 subject


# ============ 測試案例 3：模擬郵件發送失敗 =============
# 這展示了如何測試錯誤場景

@patch('myapp.user_manager.services.send_email')
def test_register_user_email_failed(mock_send_email):
    """
    測試郵件發送失敗的情況
    
    這個測試展示了：
    1. 如何模擬外部服務失敗
    2. 如何測試錯誤處理邏輯
    3. 如何確保測試隔離
    """
    # 設置 mock_send_email 拋出異常（模擬 SMTP 連接失敗）
    # side_effect 可以是一個 exception，會在調用時拋出
    mock_send_email.side_effect = ConnectionError("無法連接 SMTP 伺服器")
    
    # 嘗試註冊用戶
    result = register_user("bob", "bob@example.com")
    
    # 驗證錯誤處理
    assert result["status"] == "error"           # 應該返回錯誤狀態
    assert "郵件發送失敗" in result["message"]    # 錯誤訊息應該提到郵件


# ============ 測試案例 4：使用不同的參數測試多次 =============
# 使用 parametrize 可以減少重複程式碼

@patch('myapp.user_manager.services.send_email')
@pytest.mark.parametrize("username,email,expected", [
    ("user1", "user1@example.com", "success"),
    ("user2", "user2@example.com", "success"),
    ("ab", "user3@example.com", "error"),          # 用戶名太短
])
def test_register_user_multiple(mock_send_email, username, email, expected):
    """
    測試多種用戶名和郵件組合
    """
    # 每次測試前重置 mock，確保乾淨的狀態
    mock_send_email.return_value = True
    mock_send_email.reset_mock()
    
    result = register_user(username, email)
    
    assert result["status"] == expected


# ============ 測試案例 5：驗證調用參數的複雜場景 =============
# 展示如何驗證複雜的調用記錄

@patch('myapp.user_manager.services.send_email')
def test_email_call_args_detailed(mock_send_email):
    """
    測試郵件調用的詳細參數驗證
    """
    mock_send_email.return_value = True
    
    result = register_user("charlie", "charlie@example.com")
    
    # 驗證詳細的調用記錄
    assert mock_send_email.called
    
    # 檢查單獨的調用屬性
    args, kwargs = mock_send_email.call_args
    
    # args[0] 是第一個位置參數（to）
    assert args[0] == "charlie@example.com"
    
    # args[1] 是第二個位置參數（subject）
    assert args[1] == "歡迎註冊"
    
    # args[2] 是第三個位置參數（body）
    assert "charlie" in args[2]


# ============ 測試案例 6：處理多個 mock 物件 =============
# 展示如何同時 patch 多個函數

@patch('myapp.user_manager.services.send_email')
@patch('myapp.user_manager.database.save_to_database')
def test_multiple_mocks(mock_db, mock_email):
    """
    同時測試多個 mock 物件
    """
    # 分別設置不同的 mock
    mock_db.return_value = {"status": "success"}
    mock_email.return_value = True
    
    result = register_user("david", "david@example.com")
    
    assert result["status"] == "success"
    assert mock_db.called
    assert mock_email.called


# ============ 測試案例 7：使用 side_effect 實現複雜邏輯 =============
# 展示如何模擬序列行為

@patch('myapp.user_manager.services.send_email')
def test_side_effect_sequence(mock_send_email):
    """
    測試使用 side_effect 模擬序列行為
    
    side_effect 可以是一個可迭代物件，每次調用返回下一個值
    """
    # 設置 side_effect 為一個列表，每次調用會依次返回列表中的值
    mock_send_email.side_effect = [True, False, True, False]
    
    # 第一次調用 - 成功
    result1 = register_user("user1", "user1@example.com")
    assert result1["status"] == "success"
    
    # 第二次調用 - 失敗（side_effect 的下一個值）
    mock_send_email.side_effect = [False]  # 重置為新的序列
    result2 = register_user("user2", "user2@example.com")
    assert result2["status"] == "error"


# ============ 測試案例 8：驗證登入功能（依賴資料庫）============
# 展示如何測試依賴其他模組的函數

@patch('myapp.user_manager.database')
def test_login_user_success(mock_database):
    """
    測試用戶登入成功
    """
    # 設置 mock 返回用戶資料
    mock_database.get_user.return_value = {
        "username": "testuser",
        "email": "test@example.com"
    }
    
    result = login_user("testuser")
    
    assert result["status"] == "success"
    assert result["user"]["username"] == "testuser"


# ============ 執行測試 =============
# 執行方式：
# pytest tests/test_user_manager.py -v

# 預期輸出：
# ========================= test session starts =========================
# collected 8 items
# 
# test_user_manager.py::test_register_user_success_decorator PASSED
# test_user_manager.py::test_register_user_with_context PASSED
# test_user_manager.py::test_register_user_email_failed PASSED
# test_user_manager.py::test_register_user_multiple PASSED
# test_user_manager.py::test_email_call_args_detailed PASSED
# test_user_manager.py::test_multiple_mocks PASSED
# test_user_manager.py::test_side_effect_sequence PASSED
# test_user_manager.py::test_login_user_success PASSED
# 
# ========================= 8 passed in 0.05s =========================
```

### 📝 Step 4: 執行測試

```bash
# 安裝依賴
pip install pytest

# 執行測試
pytest tests/test_user_manager.py -v
```

---

## 4️⃣ 今日小任務

### 🎯 任務目標

完成以下練習，深入理解 Mock 和 Patch：

1. **必做任務**：
   - 在自己的專案中使用 `@patch` 裝飾器隔離一個外部 API 調用
   - 驗證該外部函數被調用了正確的次數
   - 驗證該外部函數收到了正確的參數

2. **進階任務**：
   - 使用 `side_effect` 模擬一個會失敗的外部服務
   - 使用多個 `@patch` 同時隔離多個依賴

3. **挑戰任務**：
   - 找一個真實的開源專案，分析它們的測試如何隔離依賴
   - 嘗試為該專案添加一個新的測試案例

### 💡 提示

- Patch 的目標寫法：`@patch('模組路徑.函數名')` 中的路徑是「使用的地方」，不是「定義的地方」
- 記得使用 `mock.reset_mock()` 重置 Mock 物件，避免測試之間相互影響
- 使用 `-v` 參數查看詳細的測試輸出

---

## 📚 今日學習重點總結

| 概念 | 說明 |
|------|------|
| `@patch` 裝飾器 | 自動將 Mock 物件注入到函數參數 |
| `with patch()` | 上下文管理器方式，精確控制範圍 |
| `patch.start()/stop()` | 手動控制 patch 的開始和結束 |
| `.return_value` | 設置 Mock 物件的回傳值 |
| `.side_effect` | 設置 Mock 物件的副作用（異常或序列） |
| `.called` | 檢查函數是否被調用 |
| `.call_count` | 檢查函數被調用的次數 |
| `.call_args` | 檢查函數被調用時的參數 |

---

## 📖 下一天預告

**Day 6: Spy 與行為驗證**

- 什麼是 Spy？（監視你的程式碼）
- 如何驗證函數被調用的順序
- 如何驗證複雜的行為互動

---

## 🔗 相關��源

- [Python unittest.mock 官方文檔](https://docs.python.org/3/library unittest.mock.html)
- [Pytest Mock 插件文檔](https://pypi.org/project/pytest-mock/)

---

> ⚠️ 提醒：這個教學內容已同步推送到 GitHub，請確保你已經閱讀並實際練習！