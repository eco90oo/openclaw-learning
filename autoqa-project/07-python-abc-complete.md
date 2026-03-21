# 🐍 進階：Python ABC 抽象基底類別完全指南

> 這就是 Java 的 Interface！讓你的程式碼更具擴充性和維護性

---

## 📚 1. 什麼是 ABC？

### 比喻

```
ABC (Abstract Base Class) = 設計圖/契約

比如：
- 設計圖定義了「房子必須有門、窗、屋頂」
- 具體的「別墅」、「公寓」繼承這個設計圖
- 必須實作所有規定的元素
```

### 為什麼要用 ABC？

| 優點 | 說明 |
|------|------|
| **定義規範** | 強制所有子類別實作特定方法 |
| **程式碼重用** | 共用的邏輯放在基底類別 |
| **多型** | 可以用基底類別型別接受所有子類別 |
| **文件作用** | 清楚定義介面規範 |

---

## 📚 2. 基本語法

### 簡單範例

```python
from abc import ABC, abstractmethod

# 1. 定義抽象基底類別
class Animal(ABC):
    
    @abstractmethod
    def speak(self):
        """所有動物都必須會叫"""
        pass
    
    @abstractmethod
    def move(self):
        """所有動物都必須會移動"""
        pass
    
    # 普通方法（可以有預設實作）
    def info(self):
        print(f"這是一個 {self.__class__.__name__}")

# 2. 實作具體類別
class Dog(Animal):
    def speak(self):
        return "汪汪！"
    
    def move(self):
        return "用四條腿跑步"

class Cat(Animal):
    def speak(self):
        return "喵喵！"
    
    def move(self):
        return "用四條腿走路"

# 3. 使用
dog = Dog()
print(dog.speak())  # 汪汪！

# 錯誤示範：不能直接實例化抽象類別
# animal = Animal()  # ❌ 錯誤！

# 錯誤示範：沒有實作所有抽象方法
# class Bird(Animal):
#     def speak(self):
#         return "嘰嘰！"
# bird = Bird()  # ❌ 錯誤！因為 move 沒實作
```

---

## 📚 3. ABC vs Java Interface

### 對照表

| Java | Python ABC |
|------|-----------|
| `interface` | `class XXX(ABC)` |
| `void method()` | `@abstractmethod def method(self)` |
| `implements` | `class XXX(Animal)` |
| 預設方法 (Java 8+) | 普通方法可以直接實作 |

### Java 風格轉換

```java
// Java
public interface Animal {
    void speak();
    void move();
}

public class Dog implements Animal {
    public void speak() { System.out.println("汪汪"); }
    public void move() { System.out.println("跑步"); }
}
```

轉換成 Python：

```python
# Python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass
    
    @abstractmethod
    def move(self):
        pass

class Dog(Animal):
    def speak(self):
        print("汪汪")
    
    def move(self):
        print("跑步")
```

---

## 📚 4. 抽象屬性

```python
from abc import ABC, abstractproperty

class Person(ABC):
    @abstractproperty
    def name(self):
        """必須有名字"""
        pass
    
    @abstractproperty
    def age(self):
        """必須有年齡"""
        pass

class Student(Person):
    @property
    def name(self):
        return "小明"
    
    @property
    def age(self):
        return 20
```

---

## 📚 5. 抽象屬性（新版語法）

Python 3.3+ 可以用 `@property` + `@abstractmethod`：

```python
from abc import ABC, abstractmethod

class Person(ABC):
    @property
    @abstractmethod
    def name(self):
        pass
    
    @property
    @abstractmethod
    def email(self):
        pass
    
    # 普通屬性
    @property
    def display_name(self):
        return f"{self.name} <{self.email}>"

class Employee(Person):
    @property
    def name(self):
        return "John Doe"
    
    @property
    def email(self):
        return "john@example.com"

emp = Employee()
print(emp.display_name)  # John Doe <john@example.com>
```

---

## 📚 6. 抽象類別方法與靜態方法

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @classmethod
    @abstractmethod
    def connect(cls):
        """連線到資料庫"""
        pass
    
    @staticmethod
    @abstractmethod
    def validate_config(config: dict):
        """驗證設定"""
        pass
    
    @abstractmethod
    def query(self, sql: str):
        """執行查詢"""
        pass

class MySQL(Database):
    @classmethod
    def connect(cls):
        return "MySQL 連線建立"
    
    @staticmethod
    def validate_config(config: dict):
        return "host" in config and "port" in config
    
    def query(self, sql: str):
        return f"執行: {sql}"

db = MySQL.connect()
print(db)  # MySQL 連線建立
```

---

## 📚 7. 實際應用：AutoQA 的工具系統

這是 AutoQA 專案的核心設計模式：

```python
from abc import ABC, abstractmethod
from typing import Any, Dict

# ============ 1. 定義抽象基底類別 ============
class BaseTool(ABC):
    """所有自動化工具的基底類別"""
    
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
    async def execute(self, page, params: Dict[str, Any]) -> Dict[str, Any]:
        """執行工具"""
        pass
    
    def validate_params(self, params: Dict[str, Any]) -> bool:
        """驗證參數"""
        return True

# ============ 2. 實作具體工具 ============
class ClickTool(BaseTool):
    @property
    def name(self) -> str:
        return "click"
    
    @property
    def description(self) -> str:
        return "點擊網頁元素"
    
    async def execute(self, page, params: Dict[str, Any]) -> Dict[str, Any]:
        selector = params.get("selector")
        await page.click(selector)
        return {"success": True, "action": "click", "selector": selector}

class FillTool(BaseTool):
    @property
    def name(self) -> str:
        return "fill"
    
    @property
    def description(self) -> str:
        return "填寫輸入框"
    
    async def execute(self, page, params: Dict[str, Any]) -> Dict[str, Any]:
        selector = params.get("selector")
        value = params.get("value")
        await page.fill(selector, value)
        return {"success": True, "action": "fill", "selector": selector, "value": value}

class WaitTool(BaseTool):
    @property
    def name(self) -> str:
        return "wait"
    
    @property
    def description(self) -> str:
        return "等待元素出現"
    
    async def execute(self, page, params: Dict[str, Any]) -> Dict[str, Any]:
        selector = params.get("selector")
        timeout = params.get("timeout", 30000)
        await page.wait_for_selector(selector, timeout=timeout)
        return {"success": True, "action": "wait", "selector": selector}

# ============ 3. 工具工廠 ============
class ToolFactory:
    """工具工廠：根據 action_type 回傳對應的工具"""
    
    def __init__(self):
        self._tools: Dict[str, BaseTool] = {}
    
    def register(self, tool: BaseTool):
        self._tools[tool.name] = tool
    
    def get_tool(self, action_type: str) -> BaseTool:
        if action_type not in self._tools:
            raise ValueError(f"未知的工具類型: {action_type}")
        return self._tools[action_type]
    
    def list_tools(self) -> list:
        return list(self._tools.keys())

# ============ 4. 使用範例 ============
factory = ToolFactory()
factory.register(ClickTool())
factory.register(FillTool())
factory.register(WaitTool())

# LLM 返回的動作
llm_response = {
    "action": "click",
    "selector": "#submit-button"
}

# 自動找到對應的工具並執行
tool = factory.get_tool(llm_response["action"])
result = await tool.execute(page, llm_response)

print(result)  # {'success': True, 'action': 'click', 'selector': '#submit-button'}
```

---

## 📚 8. 為什麼這樣設計？

### 優點

| 優點 | 說明 |
|------|------|
| **新增工具超簡單** | 只要寫一個新類別繼承 BaseTool |
| **不用改主流程** | Agent 邏輯不需要動 |
| **好測試** | 每個工具可以單獨測試 |
| **符合開放/封閉原則** | 對擴展開放，對修改封閉 |

### 示意圖

```
Agent (決策者)
    ↓ 產生 action: "click"
ToolFactory (工廠)
    ↓ 找到 ClickTool
ClickTool (執行者)
    ↓ 執行
Playwright (瀏覽器)
```

---

## 📝 今日小任務

1. 建立一個 `BaseCalculator` 抽象類別
2. 實作 `AddCalculator` 和 `MultiplyCalculator`
3. 建立工廠類別，根據操作符號回傳對應的計算機

---

## 🔗 參考連結

- [Python abc module](https://docs.python.org/3/library/abc.html)
- [PEP 3119](https://peps.python.org/pep-3119/) - ABC 引入原因
