# 🎯 第四階段：LangGraph (AutoQA 的 AI 決策大腦)

> 這是整個專案最進階的地方，負責決定「當測試遇到錯誤時該怎麼辦」。

---

## 📚 4.1 核心概念與快速起步

### 什麼是 LangGraph？

LangGraph 是 LangChain 生態系統中的一個庫，用於**建立有狀態的、多步驟的 AI Agent**。它讓你能夠建立複雜的 AI 工作流，包含：
- 多個節點（Node）
- 條件分支（Edge）
- 狀態管理（State）

### 核心概念

| 概念 | 說明 |
|------|------|
| **Node（節點）** | 工作流中的每個步驟，如「分析問題」、「執行測試」、「產生報告」 |
| **Edge（邊）** | 節點之間的連接，控制流程走向 |
| **State（狀態）** | 在整個工作流中流動的資料 |

### 安裝

```bash
pip install langgraph
```

### 第一個範例：基礎對話機器人

```python
from langgraph.graph import StateGraph, END

# 1. 定義狀態
class State(TypedDict):
    messages: list

# 2. 定義節點函數
def call_model(state: State):
    response = "你好！我是 AI 助手"
    return {"messages": [response]}

# 3. 建立圖
workflow = StateGraph(State)

# 4. 加入節點
workflow.add_node("model", call_model)

# 5. 設定流程
workflow.set_entry_point("model")
workflow.add_edge("model", END)

# 6. 編譯
app = workflow.compile()

# 7. 執行
result = app.invoke({"messages": []})
print(result)
```

### 加入工具：讓 AI 可以執行動作

```python
from langchain_core.tools import tool

# 定義一個工具
@tool
def calculate(expression: str) -> str:
    """執行數學計算"""
    result = eval(expression)
    return str(result)

# 在節點中使用工具
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-4")
llm_with_tools = llm.bind_tools([calculate])

def agent_node(state: State):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}
```

---

## 📚 4.2 狀態管理 (State Management)

### 什麼是 State？

State 是在整個工作流中流動的資料。它在每個節點之間傳遞，讓 AI 能夠記住之前的對話和操作。

### 定義自訂 State

```python
from typing import TypedDict
from langgraph.graph import StateGraph

# 定義狀態結構
class AgentState(TypedDict):
    # 對話歷史
    messages: list
    
    # 任務相關資料
    task: str
    context: dict
    
    # 執行結果
    result: str
    
    # 錯誤記錄
    errors: list
    
    # 步驟計數
    step: int
```

### 在節點中更新狀態

```python
def analyze_task(state: AgentState):
    task = state["task"]
    
    # 分析任務...
    analysis = f"分析任務: {task}"
    
    # 更新狀態
    return {
        "context": {"analysis": analysis},
        "step": state.get("step", 0) + 1,
        "messages": ["分析完成"]
    }

def execute_task(state: AgentState):
    # 取得上一步的 context
    context = state.get("context", {})
    
    # 執行任務...
    result = "任務執行完成"
    
    return {
        "result": result,
        "step": state.get("step", 0) + 1,
        "messages": ["執行完成"]
    }
```

### 建立狀態圖

```python
from langgraph.graph import StateGraph, END

workflow = StateGraph(AgentState)

# 加入節點
workflow.add_node("analyze", analyze_task)
workflow.add_node("execute", execute_task)

# 設定流程
workflow.set_entry_point("analyze")
workflow.add_edge("analyze", "execute")
workflow.add_edge("execute", END)

# 編譯
app = workflow.compile()

# 執行
initial_state = {
    "messages": [],
    "task": "自動化測試",
    "context": {},
    "result": "",
    "errors": [],
    "step": 0
}

result = app.invoke(initial_state)
print(result)
```

---

## 📚 4.3 工具呼叫 (Tool Calling / Agent Skills)

### 什麼是 Tool Calling？

Tool Calling 讓 LLM 能夠知道要去觸發你的 Playwright 腳本。`@tool` 裝飾器將普通的 Python 函數包裝成 AI 能夠理解並靈活運用的「技能」。

### 定義工具

```python
from langchain_core.tools import tool

@tool
def click_element(selector: str) -> str:
    """點擊網頁元素
    
    Args:
        selector: 元素的 CSS 選擇器或 XPath
    
    Returns:
        執行結果
    """
    # 實際會呼叫 Playwright
    return f"已點擊: {selector}"

@tool
def fill_input(selector: str, value: str) -> str:
    """填寫輸入框
    
    Args:
        selector: 輸入框的選擇器
        value: 要填寫的值
    
    Returns:
        執行結果
    """
    return f"已填寫 {selector}: {value}"

@tool
def get_text(selector: str) -> str:
    """取得元素文字
    
    Args:
        selector: 元素的選擇器
    
    Returns:
        元素的文字內容
    """
    return "取得的文字內容"
```

### 在 Agent 中使用工具

```python
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import ToolNode

# 1. 建立 LLM 並綁定工具
llm = ChatOpenAI(model="gpt-4")
llm_with_tools = llm.bind_tools([click_element, fill_input, get_text])

# 2. 建立工具節點
tools = [click_element, fill_input, get_text]
tool_node = ToolNode(tools)

# 3. 定義 Agent 節點
def should_continue(state: AgentState):
    messages = state["messages"]
    last_message = messages[-1]
    
    # 如果有工具呼叫，就執行工具
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "tools"
    return "end"

def agent(state: AgentState):
    messages = state["messages"]
    response = llm_with_tools.invoke(messages)
    return {"messages": [response]}

# 4. 建立流程圖
workflow = StateGraph(AgentState)

workflow.add_node("agent", agent)
workflow.add_node("tools", tool_node)

workflow.set_entry_point("agent")
workflow.add_conditional_edges(
    "agent",
    should_continue,
    {
        "tools": "tools",
        "end": END
    }
)
workflow.add_edge("tools", "agent")

app = workflow.compile()
```

### 實際應用：AutoQA Agent

```python
# 這是 AutoQA 中 Agent 的核心邏輯

@tool
def run_playwright_action(action_type: str, selector: str, value: str = None):
    """執行 Playwright 自動化動作
    
    Args:
        action_type: 動作類型 (click, fill, check, etc.)
        selector: 元素選擇器
        value: 可選的值 (用於 fill 等動作)
    """
    if action_type == "click":
        # 執行點擊
        page.click(selector)
        return f"點擊成功: {selector}"
    elif action_type == "fill":
        # 填寫輸入
        page.fill(selector, value)
        return f"填寫成功: {selector} = {value}"
    # ... 其他動作
```

---

## 📚 4.4 AutoQA 實際應用

### 完整流程圖

```
[接收測試請求]
      ↓
[分析測試步驟]
      ↓
[執行第一個步驟] → [檢查結果]
      ↓                    ↓
   成功                  失敗
      ↓                    ↓
[執行下一個步驟]    [AI 決策: 重試 / 跳過 / 報告錯誤]
      ↓
   ...
      ↓
[產生測試報告]
```

### 實作範例

```python
from typing import TypedDict
from langgraph.graph import StateGraph, END

class AutoQAState(TypedDict):
    test_id: str
    test_steps: list
    current_step: int
    results: list
    errors: list

def analyze_test(state: AutoQAState):
    """分析測試請求"""
    return {"current_step": 0, "results": []}

def execute_step(state: AutoQAState):
    """執行單一步驟"""
    current = state["current_step"]
    step = state["test_steps"][current]
    
    # 呼叫 Playwright 執行
    result = run_playwright_action(step["action"], step["selector"])
    
    return {
        "results": state["results"] + [result],
        "current_step": current + 1
    }

def check_completion(state: AutoQAState):
    """檢查是否完成"""
    if state["current_step"] >= len(state["test_steps"]):
        return "complete"
    return "continue"

workflow = StateGraph(AutoQAState)

workflow.add_node("analyze", analyze_test)
workflow.add_node("execute", execute_step)

workflow.set_entry_point("analyze")
workflow.add_edge("analyze", "execute")
workflow.add_conditional_edges(
    "execute",
    check_completion,
    {"continue": "execute", "complete": END}
)

app = workflow.compile()
```

---

## 📝 今日小任務

1. 安裝 LangGraph
2. 建立一個簡單的 StateGraph，包含 3 個節點
3. 嘗試加入一個自訂工具
4. 執行並觀察狀態如何在節點之間流動

---

## 🔗 參考連結

- [LangGraph Introduction](https://langchain-ai.github.io/langgraph/tutorials/introduction/)
- [LangGraph State Management](https://langchain-ai.github.io/langgraph/concepts/state/)
- [LangGraph Tool Calling](https://langchain-ai.github.io/langgraph/how-tos/tool-calling/)
