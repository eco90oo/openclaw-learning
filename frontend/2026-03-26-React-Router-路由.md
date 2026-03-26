# 📚 【今日學習：React Router 路由 - 讓網站像有導航系統的超級商場】

## 1️⃣ 基礎概念（用生活比喻，越詳細越好）

### 🏪 比喻：商場的導航系統

想象你走進一個巨大的購物商場，這個商場有很多層：
- 1樓：美食街
- 2樓：服飾店
- 3樓：電影院
- 4樓：遊戲中心

商場的**導航系統**就像網站的「路由」(Router)：
- 當你按下「2樓服飾店」按鈕，導航系統帶你到 2樓
- 當你按下「3樓電影院」按鈕，導航系統帶你到 3樓
- **重點**：你不需要自己走路找，導航系統會幫你決定要去哪裡

### 🌐 在網站中的運作方式

在網頁應用中，路由做的事情就是：
```
網址 (URL)  →  決定要顯示什麼頁面/元件
```

例如：
| 網址 | 應該顯示什麼 |
|------|-------------|
| `/` | 首頁（Home） |
| `/about` | 關於我們 |
| `/products` | 商品列表 |
| `/products/1` | 商品詳情（ID=1） |
| `/cart` | 購物車 |

### 🔄 傳統網頁 vs 現代 SPA 路由

**傳統網頁**（老方法）：
- 每次點擊連結，瀏覽器都要「重新載入整個網頁」
- 就像每次去商場不同樓層，都要先走出商場，再重新走進去

**現代 SPA 路由**（新方法 - 我們要學的）：
- 點擊連結，只更新部分畫面（就像商場內的電扶梯）
- 網址改變，但**不需要重新載入整個網頁**
- 速度快、體驗順暢

---

## 2️⃣ 核心知識（每行程式碼都要有中文註解！）

### 安裝 React Router

首先，在你的 React 專案資料夾中執行：
```bash
# 使用 npm 安裝 React Router
npm install react-router-dom

# 或者如果你使用 yarn
yarn add react-router-dom
```

### 第一步：設定 Router（就像設置商場的導覽地圖）

```jsx
// App.js - 這是我們應用程式的入口點
import React from 'react';
// 引入 React Router 的核心元件
// BrowserRouter 就像是整個導航系統的總控制器
import { BrowserRouter } from 'react-router-dom';

// 引入我們的頁面元件（稍後會創建這些）
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ProductsPage from './pages/ProductsPage';
import ProductDetailPage from './pages/ProductDetailPage';
import CartPage from './pages/CartPage';
import NotFoundPage from './pages/NotFoundPage';

// 引入路由相關元件
// Routes 是路由規則的容器
// Route 是每一條具體的路由規則
import { Routes, Route } from 'react-router-dom';

function App() {
  return (
    /* 
      BrowserRouter 是整個應用程式的根元件
      它會追蹤網址的變化，並決定哪個頁面要顯示
      就像商場的中央控制系統，隨時知道顧客在哪個樓層
    */
    <BrowserRouter>
      <div className="app">
        {/* 這裡是網站導航列（菜單） */}
        <nav>
          <a href="/">首頁</a>
          <a href="/about">關於我們</a>
          <a href="/products">商品列表</a>
          <a href="/cart">購物車</a>
        </nav>

        {/* 
          Routes 元件是「路由規則」的容器
          它會檢查目前網址，決定要顯示哪個頁面
        */}
        <Routes>
          {/* 
            Route 是每一條具體的路由規則
            path="/about" 的意思是：當網址是 /about 時
            element=<AboutPage /> 的意思是：顯示 AboutPage 這個元件
          */}
          
          {/* 首頁 route */}
          <Route 
            path="/"  // 網址路徑（根目錄）
            element={<HomePage />}  // 要顯示的元件
          />
          
          {/* 關於我們 page */}
          <Route 
            path="/about" 
            element={<AboutPage />} 
          />
          
          {/* 商品列表 page */}
          <Route 
            path="/products" 
            element={<ProductsPage />} 
          />
          
          {/* 
            動態路由 - 這是進階功能！
            :id 是「參數」，可以代表任何數字或文字
            例如：/products/1、/products/abc 都會匹配這個規則
          */}
          <Route 
            path="/products/:id" 
            element={<ProductDetailPage />} 
          />
          
          {/* 購物車 page */}
          <Route 
            path="/cart" 
            element={<CartPage />} 
          />
          
          {/* 
            萬用路由（Wildcard）
            * 代表匹配任何路徑
            這通常放在最後，作為「找不到頁面」的 fallback
          */}
          <Route 
            path="*" 
            element={<NotFoundPage />} 
          />
        </Routes>
      </div>
    </BrowserRouter>
  );
}

// 匯出 App 元件，讓其他檔案可以使用它
export default App;
```

### 第二步：使用 Link 元件（比 a 標籤更好！）

```jsx
// Navigation.js - 網站導航列元件

import React from 'react';
// 引入 Link 元件，這是 React Router 提供的
// 它讓使用者可以在不同頁面之間切換，但不會重新載入網頁
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav style={{ padding: '20px', backgroundColor: '#333' }}>
      {/* 
        Link 元件就像 a 標籤，但有以下不同：
        1. 不會觸發網頁重新載入
        2. 會更新瀏覽器的網址
        3. React Router 會偵測網址變化，自動切換頁面
        
        to="/" 表示連結到首頁
      */}
      <Link 
        to="/" 
        style={{ color: 'white', marginRight: '20px' }}
      >
        🏠 首頁
      </Link>

      <Link 
        to="/about" 
        style={{ color: 'white', marginRight: '20px' }}
      >
        ℹ️ 關於我們
      </Link>

      <Link 
        to="/products" 
        style={{ color: 'white', marginRight: '20px' }}
      >
        🛒 商品列表
      </Link>

      <Link 
        to="/cart" 
        style={{ color: 'white' }}
      >
        🛒 購物車
      </Link>
    </nav>
  );
}

export default Navigation;
```

### 第三步：讀取網址參數（動態路由的核心！）

```jsx
// ProductDetailPage.js - 商品詳情頁面

import React from 'react';
// 引入 useParams Hook，用來取得網址中的參數
// 例如：/products/123 中的 123
import { useParams } from 'react-router-dom';

function ProductDetailPage() {
  // 
  // useParams() 回傳一個物件，包含所有網址參數
  // 如果網址是 /products/abc，那麼 useParams() 會回傳 { id: 'abc' }
  //
  const params = useParams();
  
  // 從 params 物件中取出 id 參數
  // 這個 id 就是網址中的 :id 部分
  const productId = params.id;

  return (
    <div style={{ padding: '20px' }}>
      {/* 顯示商品 ID */}
      <h1>📦 商品詳情頁面</h1>
      <p>您正在查看商品 ID: <strong>{productId}</strong></p>
      
      {/* 這裡你可以根據 productId 去資料庫抓取商品詳細資料 */}
      <div style={{ marginTop: '20px', padding: '20px', border: '1px solid #ccc' }}>
        <h2>商品名稱：範例商品 {productId}</h2>
        <p>價格：$999</p>
        <p>庫存：充足</p>
        <button>加入購物車</button>
      </div>
    </div>
  );
}

export default ProductDetailPage;
```

### 第四步：巢狀路由（商場的樓層結構）

```jsx
// App.js - 展示巢狀路由的用法

import React from 'react';
import { BrowserRouter, Routes, Route, Outlet } from 'react-router-dom';

// 首先創建一個「巢狀佈局」元件
// Outlet 是「插槽」的意思，會自動填入子路由的內容
function DashboardLayout() {
  return (
    <div>
      {/* 這是側邊欄，永遠會顯示 */}
      <aside style={{ width: '200px', background: '#f0f0f0', padding: '20px' }}>
        <h3>📊 控制台</h3>
        <nav>
          <a href="/dashboard">總覽</a><br/>
          <a href="/dashboard/analytics">分析</a><br/>
          <a href="/dashboard/settings">設定</a>
        </nav>
      </aside>
      
      {/* 
        Outlet 是關鍵！
        當訪問 /dashboard/analytics 時，
        Analytics 元件會被渲染在這裡
      */}
      <main style={{ flex: 1, padding: '20px' }}>
        <Outlet />
      </main>
    </div>
  );
}

// 總覽頁面
function DashboardHome() {
  return <h2>📈 總覽儀表板</h2>;
}

// 分析頁面
function Analytics() {
  return <h2>📊 分析報告</h2>;
}

// 設定頁面
function Settings() {
  return <h2>⚙️ 系統設定</h2>;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* 
          巢狀路由的寫法：
          父 Route 的 element 是 DashboardLayout
          子 Route 寫在父 Route 裡面
        */}
        <Route path="/" element={<DashboardLayout />}>
          {/* 這是 /dashboard，會顯示 DashboardHome */}
          <Route index element={<DashboardHome />} />
          
          {/* 這是 /dashboard/analytics */}
          <Route path="analytics" element={<Analytics />} />
          
          {/* 這是 /dashboard/settings */}
          <Route path="settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

---

## 3️⃣ 實際操作（一步一步帶著做）

### 步驟 1：建立 React 專案

```bash
# 使用 create-react-app 建立新專案
npx create-react-app my-shop

# 進入專案資料夾
cd my-shop

# 安裝 React Router
npm install react-router-dom
```

### 步驟 2：建立頁面檔案

在 `src` 資料夾中建立 `pages` 資料夾，然後建立以下檔案：

```jsx
// src/pages/HomePage.js
import React from 'react';

function HomePage() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>🏠 歡迎來到我的商店！</h1>
      <p>這是首頁，點擊上方連結開始購物吧！</p>
    </div>
  );
}

export default HomePage;
```

```jsx
// src/pages/AboutPage.js
import React from 'react';

function AboutPage() {
  return (
    <div style={{ padding: '20px' }}>
      <h1>ℹ️ 關於我們</h1>
      <p>我們成立於 2020 年，致力於提供最好的線上購物體驗！</p>
    </div>
  );
}

export default AboutPage;
```

```jsx
// src/pages/ProductsPage.js
import React from 'react';
import { Link } from 'react-router-dom';

function ProductsPage() {
  // 模擬商品資料
  const products = [
    { id: 1, name: 'iPhone 15', price: 999 },
    { id: 2, name: 'MacBook Pro', price: 1999 },
    { id: 3, name: 'AirPods Pro', price: 249 },
  ];

  return (
    <div style={{ padding: '20px' }}>
      <h1>🛒 商品列表</h1>
      <div style={{ display: 'flex', gap: '20px' }}>
        {products.map(product => (
          <div key={product.id} style={{ border: '1px solid #ccc', padding: '10px' }}>
            <h3>{product.name}</h3>
            <p>價格：${product.price}</p>
            {/* 
              Link to={`/products/${product.id}`} 
              這會生成連結如：/products/1、/products/2 等
            */}
            <Link to={`/products/${product.id}`}>
              查看詳情
            </Link>
          </div>
        ))}
      </div>
    </div>
  );
}

export default ProductsPage;
```

### 步驟 3：更新 App.js

```jsx
// src/App.js
import React from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// 引入頁面元件
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ProductsPage from './pages/ProductsPage';
import ProductDetailPage from './pages/ProductDetailPage';

function App() {
  return (
    <BrowserRouter>
      {/* 導航列 - 每個頁面都會顯示這個 */}
      <nav style={{ background: '#333', padding: '15px' }}>
        <Link to="/" style={{ color: 'white', marginRight: '15px' }}>首頁</Link>
        <Link to="/about" style={{ color: 'white', marginRight: '15px' }}>關於我們</Link>
        <Link to="/products" style={{ color: 'white' }}>商品列表</Link>
      </nav>

      {/* 路由規則 */}
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="/products" element={<ProductsPage />} />
        {/* 動態路由：:id 可以匹配任何值 */}
        <Route path="/products/:id" element={<ProductDetailPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### 步驟 4：啟動並測試

```bash
# 啟動開發伺服器
npm start
```

現在打開瀏覽器訪問 `http://localhost:3000`，你會看到：
1. 點擊「首頁」→ 顯示 HomePage
2. 點擊「關於我們」→ 顯示 AboutPage
3. 點擊「商品列表」→ 顯示 ProductsPage
4. 點擊「查看詳情」→ 顯示 ProductDetailPage，並看到 URL 變成 `/products/1`

---

## 4️⃣ 今日小任務

### 🎯 任務 1：建立導航系統
 создайте 一個包含至少 4 個頁面的 React 應用程式，使用 React Router 連接它們。

### 🎯 任務 2：實作動態路由
在商品詳情頁面，使用 `useParams` 取得商品 ID，並顯示「您正在查看商品 #ID」。

### 🎯 任務 3：巢狀路由挑戰
嘗試建立一個「後台系統」介面，有側邊欄和主要內容區域，使用巢狀路由實現。

### 📝 驗收標準
- [ ] 安裝並正確引入 react-router-dom
- [ ] 使用 BrowserRouter 包住應用程式
- [ ] 至少定義 4 條路由規則
- [ ] 使用 Link 而非 a 標籤進行導航
- [ ] 能夠使用 useParams 取得網址參數
- [ ] 巢狀路由正確運作

---

## 📚 延伸學習資源

### 官方文檔
- [React Router 官方文檔](https://reactrouter.com/) - 英文，但有很好的範例

### 影片教學（英文）
- [React Router v6 Tutorial](https://www.youtube.com/watch?v=0c6zn9bhYno) - 詳細的 React Router 教學

### 實作範例
- 嘗試建立一個部落格系統，包含：
  - 首頁 (/): 顯示文章列表
  - 文章頁面 (/post/:id): 顯示單篇文章內容
  - 關於 (/about): 關於作者資訊
  - 聯絡 (/contact): 聯絡表單

---

## 💡 重點回顧

| 概念 | 說明 |
|------|------|
| BrowserRouter | 路由系統的根容器，追蹤網址變化 |
| Routes | 路由規則的容器 |
| Route | 單一路由規則（path + element） |
| Link | 單頁應用中的連結，不會重新載入網頁 |
| useParams | 取得網址中的動態參數 |
| Outlet | 巢狀路由中的「插槽」，填入子路由內容 |

記住：路由就像是網站的 GPS 導航系統，告訴瀏覽器要去哪裡、要顯示什麼！🗺️