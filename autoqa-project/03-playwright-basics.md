# 🎯 第三階段：Playwright (AutoQA 的自動化手腳)

> 這是負責去網頁上點擊按鈕、輸入文字、檢查結果的核心工具。

---

## 📚 3.1 元素定位器 (Locators)

### 為什麼要學這個？

自動化測試中最常當機的原因就是「**找不到按鈕**」。必須看懂怎麼使用定位器來精準鎖定網頁上的任何元素。

### 安裝 Playwright

```bash
pip install playwright
playwright install
```

### 基礎語法

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    
    # 前往網頁
    page.goto("https://example.com")
    
    # 找到元素並操作
    page.click("button")
    page.fill("input[name='username']", "test")
    
    browser.close()
```

### 定位器語法 (Locators)

#### 1. 按角色定位 (get_by_role) - 推薦

這是最推薦的方式，因為它最接近使用者和輔助技術感知網頁的方式：

```python
# 找到登入按鈕
page.get_by_role("button", name="登入").click()

# 找到標題
page.get_by_role("heading", name="歡迎").click()

# 找到勾選框
page.get_by_role("checkbox", name="記住我").check()
```

#### 2. 按文字定位 (get_by_text)

```python
# 找到包含文字的元素
page.get_by_text("歡迎回來").click()

# 精確匹配
page.get_by_text("登入", exact=True).click()

# 正則表達式
page.get_by_text(re.compile("welcome|歡迎")).click()
```

#### 3. 按標籤定位 (get_by_label)

找到表單控制項，非常適合登入表單：

```python
# 填寫表單
page.get_by_label("使用者名稱").fill("test_user")
page.get_by_label("密碼").fill("password123")
page.get_by_role("button", name="登入").click()
```

#### 4. 按佔位符定位 (get_by_placeholder)

```python
# 找到輸入框
page.get_by_placeholder("請輸入電子郵件").fill("test@example.com")
```

#### 5. 按標題定位 (get_by_title)

```python
# 找到有 title 屬性的元素
page.get_by_title("設定").click()
```

#### 6. 按測試 ID 定位 (get_by_test_id)

最穩定的方式，但需要開發者在 HTML 中加入 `data-testid`：

```python
# HTML: <button data-testid="submit-btn">提交</button>
page.get_by_test_id("submit-btn").click()
```

#### 7. 用 CSS/XPath (不推薦)

```python
# CSS 選擇器
page.locator("#submit-button").click()

# XPath
page.locator("//button[@id='submit']").click()
```

### 實際範例：自動登入

```python
from playwright.sync_api import sync_playwright

def login_example():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        
        # 前往登入頁面
        page.goto("https://example.com/login")
        
        # 填寫表單
        page.get_by_label("使用者名稱").fill("admin")
        page.get_by_label("密碼").fill("password123")
        
        # 點擊登入按鈕
        page.get_by_role("button", name="登入").click()
        
        # 驗證登入成功
        page.wait_for_selector("text=登入成功")
        
        browser.close()
```

---

## 📚 3.2 動作與斷言 (Actions & Assertions)

### 常見動作

```python
# 點擊
page.click("button")
page.dblclick("button")  # 雙擊
page.right_click("button")  # 右鍵

# 輸入文字
page.fill("input[name='q']", "搜尋內容")
page.press("input", "Enter")  # 按下鍵盤

# 勾選
page.check("input[type='checkbox']")
page.uncheck("input[type='checkbox']")

# 下拉選單
page.select_option("select#country", "TW")

# 懸停
page.hover("div.menu")

# 拖曳
page.drag_and_drop("#source", "#target")
```

### 等待元素

```python
# 等待元素出現
page.wait_for_selector("#result")

# 等待元素消失
page.wait_for_selector("#loading", state="hidden")

# 等待導航
page.wait_for_load_state("networkidle")
page.wait_for_url("**/dashboard")
```

### 斷言 (Assertions)

使用 `expect` 進行驗證：

```python
from playwright.sync_api import expect

# 驗證元素可見
expect(page.get_by_role("heading", name="歡迎")).to_be_visible()

# 驗證文字內容
expect(page.locator(".message")).to_have_text("操作成功")

# 驗證輸入框的值
expect(page.get_by_label("Email")).to_have_value("test@example.com")

# 驗證元素存在
expect(page.locator(".error")).to_be_hidden()

# 驗證元素數量
expect(page.locator(".item")).to_have_count(5)
```

### 實戰：自動化測試範例

```python
from playwright.sync_api import sync_playwright, expect

def test_login_flow():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        
        # 1. 前往首頁
        page.goto("https://example.com")
        
        # 2. 點擊登入
        page.get_by_role("link", name="登入").click()
        
        # 3. 填寫帳號密碼
        page.get_by_label("帳號").fill("testuser")
        page.get_by_label("密碼").fill("testpass")
        
        # 4. 點擊登入按鈕
        page.get_by_role("button", name="登入").click()
        
        # 5. 驗證登入成功
        expect(page.get_by_text("歡迎回來")).to_be_visible()
        
        # 6. 驗證 URL 正確
        expect(page).to_have_url("**/dashboard")
        
        browser.close()
```

---

## 📚 3.3 自動等待機制 (Auto-waiting)

### 為什麼要學這個？

Playwright 在點擊按鈕前，會「**自動**」檢查條件（例如：按鈕是否可見、是否被其他元素遮擋）。理解這個機制能幫你省去在程式碼裡寫一堆 `sleep()` 的麻煩。

### 自動等待的條件

Playwright 會自動等待以下條件：

| 條件 | 說明 |
|------|------|
| 可見 (visible) | 元素在頁面上可見 |
| 可操作 (actionable) | 元素已啟用、穩定 |
| 穩定 (stable) | 元素不再動畫 |
| 啟用 (enabled) | 元素不是 disabled |
| 沒有遮擋 | 元素沒有被其他元素覆蓋 |

### 範例：自動等待

```python
# Playwright 會自動等待按鈕變得可點擊
page.get_by_role("button", name="提交").click()

# 不需要寫 sleep！
# 舊的寫法：
time.sleep(2)  # 不要這樣做！
page.click("button")
```

### 手動等待（必要時）

```python
# 等待特定條件
page.wait_for_selector(".loading", state="hidden")  # 等待 loading 消失

# 等待文字出現
page.wait_for_text("完成")

# 等待函數返回 true
page.wait_for_function("() => window.status === 'ready'")
```

### 常見錯誤與解決

```python
# 錯誤：元素被遮擋
# Playwright: Error: Element is obscured by another element

# 解決：先點擊關閉弹窗
page.get_by_role("button", name="關閉").click()
page.get_by_role("button", name="提交").click()

# 錯誤：元素不可見
# Playwright: Error: Element is not visible

# 解決：先滾動到元素
page.locator(".submit-btn").scroll_into_view_if_needed()
page.get_by_role("button", name="提交").click()
```

---

## 📝 今日小任務

1. 使用 Playwright 自動打開 Google 首頁
2. 在搜尋框輸入「Playwright Python」
3. 點擊搜尋按鈕
4. 驗證搜尋結果頁面標題包含「Playwright」

---

## 🔗 參考連結

- [Playwright Locators](https://playwright.dev/python/docs/locators)
- [Playwright Actions](https://playwright.dev/python/docs/writing-tests)
- [Playwright Auto-waiting](https://playwright.dev/python/docs/actionability)
