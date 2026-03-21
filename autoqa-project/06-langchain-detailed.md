# 🔧 進階：LangChain 與 LangGraph 完整教學

> LangChain vs LangGraph vs Deep Agents 詳解

---

## 📚 1. LangChain vs LangGraph vs Deep Agents

### 這三者是什麼？

| 框架 | 用途 | 適合誰 |
|------|------|--------|
| **LangChain** | 快速建立 Agent | 初學者，想要快速上手 |
| **LangGraph** | 低階流程控制 | 需要高度客製化的工作流 |
| **Deep Agents** | 現成的強大 Agent | 想要完整功能的產品 |

### 簡單理解

```
LangChain → 簡單易用，10 行程式碼就能建立 Agent
LangGraph → 更底層，可自訂複雜的流程控制
Deep Agents → 完整功能，開箱即用
```

---

## 📚 2. LangChain 快速上手

### 安裝

```bash
pip install langchain langchain-openai
```

### 建立第一個 Agent

```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI

# 1. 定義工具
def get_weather(city: str) -> str:
    """取得城市天氣"""
    return f"{city}的天氣是晴天！"

def get_time() -> str:
    """取得現在時間"""
    from datetime import datetime
    return datetime.now().strftime("%H:%M")

# 2. 建立 Agent
agent = create_agent(
    model="gpt-4",
    tools=[get_weather, get_time],
    system_prompt="你是個有用的助手"
)

# 3. 執行
result = agent.invoke({
    "messages": [{"role": "user", "content": "台北的天氣如何？"}]
})

print(result)
```

### LangChain 的核心優勢

| 優勢 | 說明 |
|------|------|
| **標準化介面** | 統一的模型呼叫方式，輕鬆切換供應商 |
| **容易上手** | 10 行程式碼就能建立 Agent |
| **靈活性高** | 可自由組合各種元件 |

---

## 📚 3. 為什麼 LangChain Agents 基於 LangGraph？

LangChain 的 Agent 實際上是用 LangGraph 構建的，所以繼承了：

| 功能 | 說明 |
|------|------|
| **持久化** | 記住對話歷史 |
| **串流** | 即時輸出回應 |
| **人類介入** | 允許人類干預執行過程 |
| **可復原** | 從錯誤中恢復 |

---

## 📚 4. 什麼時候該用 LangGraph？

當你需要：

1. **複雜的工作流**
   - 多個決策點
   - 條件分支
   - 迴圈

2. **高度客製化**
   - 自訂節點行為
   - 自訂流程控制

3. **確定性 + 智能混合**
   - 部分步驟需要 AI 判斷
   - 部分步驟是固定的

---

## 📚 5. 實際範例：用 LangChain vs LangGraph

### 用 LangChain（簡單）

```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI

agent = create_agent(
    model="gpt-4",
    tools=[my_tool],
    system_prompt="你是助手"
)
```

### 用 LangGraph（彈性）

```python
from langgraph.graph import StateGraph, END

# 完全控制流程
workflow = StateGraph(State)
workflow.add_node("think", think_node)
workflow.add_node("act", act_node)
workflow.add_edge("think", "act")
```

---

## 📚 6. 監控與調試：LangSmith

LangSmith 是 LangChain 的監控工具：

```bash
pip install langsmith
export LANGCHAIN_TRACING=true
export LANGCHAIN_API_KEY="your-api-key"
```

功能：
- 追蹤請求
- 調試 Agent 行為
- 評估輸出
- 視覺化流程

---

## 📚 7. 完整範例：AI 客服機器人

```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI

# 1. 定義工具
def search_order(order_id: str) -> str:
    """查詢訂單"""
    return f"訂單 {order_id} 狀態：已出貨"

def refund(order_id: str) -> str:
    """退貨處理"""
    return f"已受理退貨申請，訂單 {order_id}"

# 2. 建立 Agent
agent = create_agent(
    model="gpt-4",
    tools=[search_order, refund],
    system_prompt="""你是專業的客服助手。
    當客戶詢問訂單時，使用 search_order 工具查詢。
    當客戶要求退貨時，使用 refund 工具處理。
    """
)

# 3. 對話
response = agent.invoke({
    "messages": [{"role": "user", "content": "我想查詢訂單 #12345"}]
})
```

---

## 📝 今日小任務

1. 安裝 LangChain
2. 建立一個簡單的 Agent
3. 加入 2-3 個自訂工具
4. 嘗試與 Agent 對話

---

## 🔗 參考連結

- [LangChain Documentation](https://python.langchain.com/docs/)
- [LangGraph Overview](https://langchain-ai.github.io/langgraph/)
- [LangSmith](https://smith.langchain.com/)
