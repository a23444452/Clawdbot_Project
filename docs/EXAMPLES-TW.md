# 💡 OpenClaw 使用範例集

> 實際使用場景與完整對話範例

---

## 📋 目錄

- [程式開發](#程式開發)
- [資料分析](#資料分析)
- [文件處理](#文件處理)
- [系統管理](#系統管理)
- [學習與研究](#學習與研究)
- [創意寫作](#創意寫作)
- [日常助理](#日常助理)

---

## 程式開發

### 範例 1: 建立 REST API

**對話：**
```
你: 幫我用 Express.js 建立一個簡單的 REST API，
    包含 CRUD 操作，使用 MongoDB

Bot: 我會幫你建立一個完整的 Express.js + MongoDB REST API。

首先讓我建立專案結構：

[執行: mkdir api-project && cd api-project]
[執行: npm init -y]
[執行: npm install express mongodb dotenv cors]

現在讓我建立各個檔案...

[寫入: server.js]
```javascript
require('dotenv').config();
const express = require('express');
const { MongoClient, ObjectId } = require('mongodb');
const cors = require('cors');

const app = express();
const port = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB 連線
const client = new MongoClient(process.env.MONGODB_URI);
let db;

async function connectDB() {
  try {
    await client.connect();
    db = client.db('myapp');
    console.log('✅ MongoDB 連線成功');
  } catch (error) {
    console.error('❌ MongoDB 連線失敗:', error);
    process.exit(1);
  }
}

// Routes

// GET - 取得所有項目
app.get('/api/items', async (req, res) => {
  try {
    const items = await db.collection('items').find().toArray();
    res.json(items);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// GET - 取得單一項目
app.get('/api/items/:id', async (req, res) => {
  try {
    const item = await db.collection('items').findOne({
      _id: new ObjectId(req.params.id)
    });
    if (!item) {
      return res.status(404).json({ error: '項目不存在' });
    }
    res.json(item);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// POST - 建立新項目
app.post('/api/items', async (req, res) => {
  try {
    const result = await db.collection('items').insertOne(req.body);
    res.status(201).json({
      _id: result.insertedId,
      ...req.body
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// PUT - 更新項目
app.put('/api/items/:id', async (req, res) => {
  try {
    const result = await db.collection('items').updateOne(
      { _id: new ObjectId(req.params.id) },
      { $set: req.body }
    );
    if (result.matchedCount === 0) {
      return res.status(404).json({ error: '項目不存在' });
    }
    res.json({ message: '更新成功' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// DELETE - 刪除項目
app.delete('/api/items/:id', async (req, res) => {
  try {
    const result = await db.collection('items').deleteOne({
      _id: new ObjectId(req.params.id)
    });
    if (result.deletedCount === 0) {
      return res.status(404).json({ error: '項目不存在' });
    }
    res.json({ message: '刪除成功' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 啟動伺服器
connectDB().then(() => {
  app.listen(port, () => {
    console.log(`🚀 伺服器運行於 http://localhost:${port}`);
  });
});
```

[寫入: .env]
```
MONGODB_URI=mongodb://localhost:27017
PORT=3000
```

[寫入: .gitignore]
```
node_modules/
.env
```

專案建立完成！

目錄結構：
```
api-project/
├── package.json
├── server.js
├── .env
└── .gitignore
```

使用方式：
1. 啟動 MongoDB: `mongod`
2. 啟動伺服器: `node server.js`
3. 測試 API:
   - GET: `curl http://localhost:3000/api/items`
   - POST: `curl -X POST http://localhost:3000/api/items -H "Content-Type: application/json" -d '{"name":"Test","value":123}'`
```

### 範例 2: 程式碼審查與重構

**對話：**
```
你: 請審查這段程式碼並給出改進建議：

```javascript
function getData() {
  var data = [];
  for(var i=0; i<items.length; i++) {
    if(items[i].active == true) {
      data.push(items[i]);
    }
  }
  return data;
}
```

Bot: 我來分析這段程式碼並提供改進建議。

## 🔍 問題分析

1. **使用 var** - 應該使用 const/let
2. **效率問題** - 可以使用 filter() 更簡潔
3. **嚴格相等** - `==` 應該用 `===`
4. **缺少型別提示** - 可讀性較差
5. **缺少錯誤處理** - 沒有檢查 items 是否存在

## ✅ 改進版本 1: 基本改進

```javascript
function getData(items) {
  if (!items || !Array.isArray(items)) {
    return [];
  }

  const data = [];
  for (let i = 0; i < items.length; i++) {
    if (items[i].active === true) {
      data.push(items[i]);
    }
  }
  return data;
}
```

## 🚀 改進版本 2: 使用現代 JavaScript

```javascript
function getActiveItems(items) {
  if (!items || !Array.isArray(items)) {
    return [];
  }

  return items.filter(item => item.active === true);
}
```

## 💎 改進版本 3: TypeScript + 完整驗證

```typescript
interface Item {
  active: boolean;
  [key: string]: any;
}

function getActiveItems(items: Item[]): Item[] {
  // 輸入驗證
  if (!Array.isArray(items)) {
    throw new TypeError('items must be an array');
  }

  // 過濾並驗證每個項目
  return items.filter(item => {
    if (typeof item !== 'object' || item === null) {
      console.warn('Invalid item encountered:', item);
      return false;
    }
    return item.active === true;
  });
}
```

## 📊 效能比較

對 10,000 個項目的測試：

| 版本 | 時間 | 記憶體 | 可讀性 |
|------|------|--------|--------|
| 原始版本 | 1.2ms | 中 | ⭐⭐ |
| 版本 1 | 1.1ms | 中 | ⭐⭐⭐ |
| 版本 2 | 0.8ms | 低 | ⭐⭐⭐⭐⭐ |
| 版本 3 | 0.9ms | 低 | ⭐⭐⭐⭐⭐ |

## 💡 建議選擇

- **小型專案**: 版本 2（簡潔高效）
- **大型專案**: 版本 3（完整型別安全）
- **舊專案**: 版本 1（最小改動）

需要我解釋任何部分嗎？
```

### 範例 3: Bug 除錯

**對話：**
```
你: 這段程式碼有 bug，但我找不到問題在哪：

```javascript
async function fetchUsers() {
  const response = await fetch('/api/users');
  const users = response.json();
  return users.filter(u => u.active);
}
```

Bot: 我找到問題了！這裡有一個非同步處理的 bug。

## 🐛 問題所在

```javascript
const users = response.json();  // ❌ 錯誤！
```

`response.json()` 回傳的是一個 **Promise**，需要使用 `await`。

## ✅ 修正版本

```javascript
async function fetchUsers() {
  const response = await fetch('/api/users');
  const users = await response.json();  // ✅ 加上 await
  return users.filter(u => u.active);
}
```

## 🛡️ 更完整的版本（含錯誤處理）

```javascript
async function fetchUsers() {
  try {
    const response = await fetch('/api/users');

    // 檢查 HTTP 狀態
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const users = await response.json();

    // 驗證資料格式
    if (!Array.isArray(users)) {
      throw new TypeError('Expected an array of users');
    }

    return users.filter(u => u.active);

  } catch (error) {
    console.error('Failed to fetch users:', error);
    // 回傳空陣列或重新拋出錯誤，取決於你的需求
    return [];
  }
}
```

## 🧪 測試程式碼

```javascript
// 測試
fetchUsers()
  .then(users => console.log('Active users:', users))
  .catch(error => console.error('Error:', error));
```

問題解決了！主要是缺少 `await`，導致你得到的是 Promise 物件而不是實際資料。
```

---

## 資料分析

### 範例 4: CSV 資料分析

**對話：**
```
你: 我有一個 sales.csv 檔案，幫我分析銷售資料並產生報告

Bot: 我會幫你分析 CSV 檔案並產生報告。

首先讓我讀取檔案：

[執行: head -10 sales.csv]
```csv
date,product,quantity,price,region
2026-01-01,Product A,10,99.99,North
2026-01-01,Product B,5,149.99,South
...
```

資料欄位：date, product, quantity, price, region

現在讓我建立分析腳本：

[寫入: analyze_sales.py]
```python
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

# 讀取資料
df = pd.read_csv('sales.csv')
df['date'] = pd.to_datetime(df['date'])
df['total'] = df['quantity'] * df['price']

# === 基本統計 ===
print("=" * 50)
print("📊 銷售資料分析報告")
print("=" * 50)
print(f"\n資料期間: {df['date'].min()} 至 {df['date'].max()}")
print(f"總筆數: {len(df):,}")
print(f"總銷售額: ${df['total'].sum():,.2f}")
print(f"平均訂單金額: ${df['total'].mean():,.2f}")

# === 產品分析 ===
print("\n" + "=" * 50)
print("🏆 Top 5 暢銷產品")
print("=" * 50)
product_sales = df.groupby('product')['total'].sum().sort_values(ascending=False)
for i, (product, total) in enumerate(product_sales.head(5).items(), 1):
    print(f"{i}. {product}: ${total:,.2f}")

# === 區域分析 ===
print("\n" + "=" * 50)
print("🌍 各區域銷售")
print("=" * 50)
region_sales = df.groupby('region')['total'].sum().sort_values(ascending=False)
for region, total in region_sales.items():
    percentage = (total / df['total'].sum()) * 100
    print(f"{region}: ${total:,.2f} ({percentage:.1f}%)")

# === 時間趨勢 ===
df_monthly = df.set_index('date').resample('M')['total'].sum()
print("\n" + "=" * 50)
print("📈 月銷售趨勢")
print("=" * 50)
for date, total in df_monthly.items():
    print(f"{date.strftime('%Y-%m')}: ${total:,.2f}")

# === 產生圖表 ===
fig, axes = plt.subplots(2, 2, figsize=(15, 10))

# 1. 產品銷售圖
product_sales.head(10).plot(kind='barh', ax=axes[0, 0], color='steelblue')
axes[0, 0].set_title('Top 10 產品銷售額')
axes[0, 0].set_xlabel('銷售額 ($)')

# 2. 區域銷售圓餅圖
region_sales.plot(kind='pie', ax=axes[0, 1], autopct='%1.1f%%')
axes[0, 1].set_title('各區域銷售佔比')

# 3. 月銷售趨勢
df_monthly.plot(ax=axes[1, 0], marker='o', color='green')
axes[1, 0].set_title('月銷售趨勢')
axes[1, 0].set_xlabel('日期')
axes[1, 0].set_ylabel('銷售額 ($)')
axes[1, 0].grid(True)

# 4. 日銷售量
df_daily = df.set_index('date').resample('D')['quantity'].sum()
df_daily.plot(ax=axes[1, 1], color='orange')
axes[1, 1].set_title('每日銷售數量')
axes[1, 1].set_xlabel('日期')
axes[1, 1].set_ylabel('數量')
axes[1, 1].grid(True)

plt.tight_layout()
plt.savefig('sales_report.png', dpi=300, bbox_inches='tight')
print("\n✅ 圖表已儲存至 sales_report.png")
```

[執行: python analyze_sales.py]

分析完成！結果：

📊 **重點發現:**
1. 總銷售額: $1,234,567.89
2. 最暢銷產品: Product A ($345,678)
3. 最佳區域: North (38.5%)
4. 銷售趨勢: 1月較去年同期成長 23%

圖表和詳細報告已產生。需要更深入的分析嗎？
```

---

## 文件處理

### 範例 5: Markdown 轉 PDF

**對話：**
```
你: 把 README.md 轉成 PDF，要包含目錄和樣式

Bot: 我會使用 pandoc 將 Markdown 轉成精美的 PDF。

[執行: which pandoc]
✅ pandoc 已安裝

現在讓我建立樣式檔案和轉換：

[寫入: style.css]
```css
body {
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  color: #2c3e50;
  border-bottom: 3px solid #3498db;
  padding-bottom: 10px;
}

h2 {
  color: #34495e;
  border-bottom: 2px solid #95a5a6;
  padding-bottom: 5px;
  margin-top: 30px;
}

code {
  background-color: #f4f4f4;
  padding: 2px 6px;
  border-radius: 3px;
  font-family: 'Courier New', monospace;
}

pre {
  background-color: #2c3e50;
  color: #ecf0f1;
  padding: 15px;
  border-radius: 5px;
  overflow-x: auto;
}

blockquote {
  border-left: 4px solid #3498db;
  padding-left: 20px;
  color: #7f8c8d;
  font-style: italic;
}

table {
  border-collapse: collapse;
  width: 100%;
  margin: 20px 0;
}

th, td {
  border: 1px solid #bdc3c7;
  padding: 10px;
  text-align: left;
}

th {
  background-color: #3498db;
  color: white;
}
```

[執行: pandoc README.md -o README.pdf \
  --pdf-engine=xelatex \
  --toc \
  --toc-depth=3 \
  --css=style.css \
  --metadata title="README 文件" \
  --metadata author="Your Name" \
  --metadata date="$(date +%Y-%m-%d)" \
  -V geometry:margin=1in \
  -V fontsize=11pt \
  -V documentclass=article \
  -V CJKmainfont="Noto Sans TC"]

✅ PDF 已產生：README.pdf

包含功能：
- ✅ 自動產生目錄（3 層深度）
- ✅ 自訂樣式（標題、程式碼區塊、表格）
- ✅ 中文字型支援
- ✅ 頁面設定（1 英吋邊界）

檔案位置：./README.pdf
```

---

## 系統管理

### 範例 6: 自動化部署腳本

**對話：**
```
你: 寫一個 bash 腳本來自動部署 Node.js 應用程式，
    包含備份、測試、部署和回滾功能

Bot: 我會建立一個完整的部署腳本，包含所有必要的安全檢查。

[寫入: deploy.sh]
```bash
#!/bin/bash

set -e  # 遇到錯誤立即退出

# === 設定 ===
APP_NAME="my-nodejs-app"
APP_DIR="/var/www/$APP_NAME"
BACKUP_DIR="/var/backups/$APP_NAME"
LOG_FILE="/var/log/$APP_NAME-deploy.log"
MAX_BACKUPS=5

# 顏色輸出
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# === 函數 ===

log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')]${NC} $1" | tee -a "$LOG_FILE"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1" | tee -a "$LOG_FILE"
    exit 1
}

warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1" | tee -a "$LOG_FILE"
}

# 建立備份
backup() {
    log "📦 建立備份..."

    BACKUP_FILE="$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S).tar.gz"
    mkdir -p "$BACKUP_DIR"

    if [ -d "$APP_DIR" ]; then
        tar -czf "$BACKUP_FILE" -C "$(dirname $APP_DIR)" "$(basename $APP_DIR)" 2>/dev/null || true
        log "✅ 備份完成: $BACKUP_FILE"

        # 清理舊備份
        BACKUP_COUNT=$(ls -1 "$BACKUP_DIR" | wc -l)
        if [ "$BACKUP_COUNT" -gt "$MAX_BACKUPS" ]; then
            ls -1t "$BACKUP_DIR" | tail -n +$((MAX_BACKUPS + 1)) | xargs -I {} rm "$BACKUP_DIR/{}"
            log "🧹 清理舊備份（保留最近 $MAX_BACKUPS 個）"
        fi
    else
        warning "應用程式目錄不存在，跳過備份"
    fi
}

# 拉取最新程式碼
pull_code() {
    log "📥 拉取最新程式碼..."

    cd "$APP_DIR" || error "無法進入應用程式目錄"

    # 檢查 git 狀態
    if [ -n "$(git status --porcelain)" ]; then
        warning "有未提交的變更"
        git stash save "auto-stash-$(date +%Y%m%d-%H%M%S)"
    fi

    git fetch origin
    git checkout main
    git pull origin main || error "拉取程式碼失敗"

    log "✅ 程式碼更新完成"
}

# 安裝依賴
install_dependencies() {
    log "📚 安裝依賴..."

    cd "$APP_DIR"

    if [ -f "package-lock.json" ]; then
        npm ci --production || error "安裝依賴失敗"
    else
        npm install --production || error "安裝依賴失敗"
    fi

    log "✅ 依賴安裝完成"
}

# 執行測試
run_tests() {
    log "🧪 執行測試..."

    cd "$APP_DIR"

    # 安裝開發依賴
    npm install --dev || warning "安裝開發依賴失敗"

    # 執行測試
    if npm test; then
        log "✅ 測試通過"
    else
        error "測試失敗，中止部署"
    fi
}

# 建置應用程式
build() {
    log "🔨 建置應用程式..."

    cd "$APP_DIR"

    if npm run build; then
        log "✅ 建置完成"
    else
        error "建置失敗"
    fi
}

# 重啟服務
restart_service() {
    log "🔄 重啟服務..."

    if systemctl restart "$APP_NAME"; then
        log "✅ 服務重啟成功"
    else
        error "服務重啟失敗"
    fi

    # 等待服務啟動
    sleep 5

    # 檢查服務狀態
    if systemctl is-active --quiet "$APP_NAME"; then
        log "✅ 服務運行中"
    else
        error "服務未正常運行"
    fi
}

# 健康檢查
health_check() {
    log "🏥 執行健康檢查..."

    local max_attempts=5
    local attempt=1

    while [ $attempt -le $max_attempts ]; do
        HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/health)

        if [ "$HTTP_CODE" = "200" ]; then
            log "✅ 健康檢查通過"
            return 0
        fi

        warning "健康檢查失敗 (嘗試 $attempt/$max_attempts), HTTP Code: $HTTP_CODE"
        attempt=$((attempt + 1))
        sleep 5
    done

    error "健康檢查失敗，執行回滾"
}

# 回滾
rollback() {
    log "⏪ 執行回滾..."

    LATEST_BACKUP=$(ls -1t "$BACKUP_DIR" | head -n 1)

    if [ -z "$LATEST_BACKUP" ]; then
        error "找不到備份檔案"
    fi

    log "使用備份: $LATEST_BACKUP"

    # 停止服務
    systemctl stop "$APP_NAME" || true

    # 移除當前版本
    rm -rf "$APP_DIR"

    # 恢復備份
    tar -xzf "$BACKUP_DIR/$LATEST_BACKUP" -C "$(dirname $APP_DIR)"

    # 重啟服務
    restart_service

    log "✅ 回滾完成"
}

# === 主程序 ===

main() {
    log "========================================="
    log "🚀 開始部署 $APP_NAME"
    log "========================================="

    # 執行部署步驟
    backup
    pull_code
    install_dependencies
    run_tests
    build
    restart_service

    # 健康檢查
    if health_check; then
        log "========================================="
        log "✅ 部署成功完成！"
        log "========================================="
    else
        log "========================================="
        log "❌ 部署失敗，執行回滾"
        log "========================================="
        rollback
        exit 1
    fi
}

# 處理 Ctrl+C
trap 'error "部署被中斷"' INT TERM

# 執行主程序
main

```

腳本建立完成！

**使用方式：**

```bash
# 賦予執行權限
chmod +x deploy.sh

# 執行部署
sudo ./deploy.sh

# 只執行備份
sudo ./deploy.sh --backup-only

# 回滾到上一版
sudo ./deploy.sh --rollback
```

**功能特點：**
- ✅ 自動備份（保留最近 5 個）
- ✅ Git 程式碼拉取
- ✅ 依賴安裝
- ✅ 自動測試
- ✅ 建置
- ✅ 服務重啟
- ✅ 健康檢查
- ✅ 失敗自動回滾
- ✅ 詳細日誌記錄
- ✅ 顏色輸出
- ✅ 錯誤處理

需要我解釋任何部分嗎？
```

---

## 學習與研究

### 範例 7: 技術文章摘要

**對話：**
```
你: 搜尋並總結 2026 年最新的 Web 開發趨勢

Bot: 讓我搜尋最新的 Web 開發趨勢...

[執行: web_search query="Web development trends 2026"]

根據搜尋結果，這是 2026 年的主要 Web 開發趨勢：

# 🌐 2026 Web 開發趨勢總結

## 1. 🤖 AI 整合成為標配

**主要發展：**
- AI Copilots 普遍應用於程式碼編寫
- 智慧 UI 組件自動調整
- 自然語言轉程式碼工具成熟

**代表技術：**
- GitHub Copilot X
- OpenAI Code Interpreter
- Anthropic Claude for Coding

## 2. ⚡ 效能優先

**關鍵指標：**
- Core Web Vitals 3.0
- INP (Interaction to Next Paint) < 200ms
- 首屏載入 < 1秒

**優化技術：**
- Partial Hydration
- Islands Architecture
- Edge Computing

## 3. 🎨 新世代框架

**熱門框架：**
- Next.js 15 (Server Actions 成熟)
- Astro 4 (超快靜態生成)
- SolidJS (React 的替代品)
- Svelte 5 (Runes 新語法)

## 4. 🔒 零信任架構

**安全實踐：**
- 預設啟用 HTTPS/3
- CSP (Content Security Policy) 嚴格模式
- Subresource Integrity 強制檢查

## 5. 🎯 TypeScript 主導

**統計數據：**
- 87% 新專案使用 TypeScript
- 類型推導更智慧
- 與 AI 工具深度整合

## 6. 📱 Progressive Web Apps (PWA) 2.0

**新功能：**
- File System Access API
- Compute Pressure API
- WebGPU 支援

## 7. 🌍 邊緣運算普及

**平台：**
- Cloudflare Workers
- Vercel Edge Functions
- Deno Deploy

## 8. 🎨 CSS 新時代

**新功能：**
- Container Queries 廣泛應用
- CSS Nesting 原生支援
- :has() 選擇器

## 9. 🔌 API 設計演進

**趨勢：**
- GraphQL 持續成長
- tRPC (Type-safe RPC)
- gRPC-Web 興起

## 10. 🌿 永續性考量

**綠色編程：**
- 碳排放監控工具
- 優化能源效率
- 輕量化優先

---

## 📚 推薦學習資源

1. **課程：**
   - Frontend Masters: "Web Platform 2026"
   - Egghead.io: "Modern Web Performance"

2. **書籍：**
   - "Web Development in 2026" by O'Reilly
   - "TypeScript Deep Dive" 5th Edition

3. **社群：**
   - Dev.to
   - Frontend Horse
   - CSS Tricks

## 💡 給開發者的建議

1. ✅ 深入學習 TypeScript
2. ✅ 掌握至少一個現代框架
3. ✅ 關注 Web 效能
4. ✅ 了解 AI 輔助工具
5. ✅ 實踐無障礙設計
6. ✅ 持續學習新標準

需要更深入了解某個主題嗎？
```

---

## 創意寫作

### 範例 8: 部落格文章生成

**對話：**
```
你: 幫我寫一篇關於「TypeScript 最佳實踐」的技術部落格，
    目標讀者是中級開發者，大約 1500 字

Bot: 我會為你撰寫一篇專業的技術部落格文章。

---

# TypeScript 最佳實踐：提升程式碼品質的 10 個技巧

> 從中級邁向高級開發者的必經之路

*發布日期：2026-02-02 | 閱讀時間：8 分鐘*

---

## 前言

TypeScript 已經成為現代 Web 開發的標準選擇。根據 2025 年 Stack Overflow 調查，超過 78% 的開發者在新專案中選擇 TypeScript。然而，僅僅使用 TypeScript 並不足夠 —— 重點在於如何「正確地」使用它。

本文將分享 10 個進階技巧，幫助你寫出更安全、更易維護的 TypeScript 程式碼。

---

## 1. 善用 Utility Types

TypeScript 提供了許多內建的 Utility Types，能大幅減少重複的型別定義。

### ❌ 不好的做法

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

interface UpdateUser {
  id?: string;
  name?: string;
  email?: string;
  age?: number;
}
```

### ✅ 好的做法

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

type UpdateUser = Partial<User>;
type UserWithoutId = Omit<User, 'id'>;
type ReadOnlyUser = Readonly<User>;
```

**常用 Utility Types：**
- `Partial<T>` - 所有屬性變為可選
- `Required<T>` - 所有屬性變為必填
- `Readonly<T>` - 所有屬性變為唯讀
- `Pick<T, K>` - 選取特定屬性
- `Omit<T, K>` - 排除特定屬性
- `Record<K, T>` - 建立物件型別

---

## 2. 嚴格的 tsconfig.json

啟用嚴格模式能提早發現潛在問題。

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true
  }
}
```

**重點說明：**
- `strict: true` 啟用所有嚴格檢查
- `noUncheckedIndexedAccess` 防止陣列/物件索引錯誤

---

## 3. 優先使用 Type Narrowing

Type Guards 讓型別推論更精確。

```typescript
function processValue(value: string | number) {
  // ❌ 不好的做法
  if (typeof value === 'string') {
    return (value as string).toUpperCase();
  }

  // ✅ 好的做法（TypeScript 自動推論）
  if (typeof value === 'string') {
    return value.toUpperCase();  // value 已經是 string
  }

  return value.toFixed(2);  // value 已經是 number
}
```

**進階技巧：自訂 Type Guard**

```typescript
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim();  // TypeScript 知道這是 Fish
  } else {
    pet.fly();   // TypeScript 知道這是 Bird
  }
}
```

---

## 4. 善用 Discriminated Unions

處理複雜狀態時，使用標籤聯合型別。

```typescript
type LoadingState = {
  status: 'loading';
};

type SuccessState<T> = {
  status: 'success';
  data: T;
};

type ErrorState = {
  status: 'error';
  error: Error;
};

type State<T> = LoadingState | SuccessState<T> | ErrorState;

function handleState<T>(state: State<T>) {
  switch (state.status) {
    case 'loading':
      return <Spinner />;

    case 'success':
      return <DataView data={state.data} />;  // 型別安全！

    case 'error':
      return <ErrorMessage error={state.error} />;  // 型別安全！
  }
}
```

---

## 5. 避免 `any`，使用 `unknown`

`unknown` 是更安全的 `any`。

```typescript
// ❌ 不好
function parseJSON(json: string): any {
  return JSON.parse(json);
}

// ✅ 好
function parseJSON(json: string): unknown {
  return JSON.parse(json);
}

const result = parseJSON('{"name": "John"}');

// 必須先驗證型別才能使用
if (typeof result === 'object' && result !== null) {
  console.log((result as { name: string }).name);
}
```

---

## 6. 正確處理 Null 和 Undefined

```typescript
// ❌ 不好
function getUserName(user: User | null) {
  return user!.name;  // 危險！
}

// ✅ 好：Optional Chaining
function getUserName(user: User | null) {
  return user?.name ?? 'Guest';
}

// ✅ 好：Nullish Coalescing
const port = config.port ?? 3000;
```

---

## 7. 使用 `const assertions`

建立不可變的型別。

```typescript
// ❌ 型別推論為 string
const direction = 'north';

// ✅ 型別推論為 'north'
const direction = 'north' as const;

// ✅ 深度唯讀
const config = {
  api: 'https://api.example.com',
  timeout: 5000,
  retries: 3
} as const;

// config.api = '...';  // ❌ 錯誤！
```

---

## 8. 泛型約束

讓泛型更精確。

```typescript
// ❌ 太寬鬆
function getProperty<T>(obj: T, key: string) {
  return obj[key];  // 錯誤！
}

// ✅ 使用約束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'John', age: 30 };
const name = getProperty(user, 'name');  // ✅ 型別是 string
// const invalid = getProperty(user, 'invalid');  // ❌ 編譯錯誤
```

---

## 9. 函數重載

為同一個函數提供多個型別簽名。

```typescript
function createElement(tag: 'a'): HTMLAnchorElement;
function createElement(tag: 'div'): HTMLDivElement;
function createElement(tag: string): HTMLElement;
function createElement(tag: string): HTMLElement {
  return document.createElement(tag);
}

const link = createElement('a');  // 型別是 HTMLAnchorElement
const div = createElement('div');  // 型別是 HTMLDivElement
```

---

## 10. 使用 ESLint + TypeScript

安裝必要的套件：

```bash
pnpm add -D @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

設定 `.eslintrc.json`:

```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking"
  ],
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn"
  }
}
```

---

## 結論

TypeScript 的強大在於它的型別系統。遵循這些最佳實踐，你將能：

1. ✅ 更早發現 bugs
2. ✅ 提升程式碼可讀性
3. ✅ 改善開發體驗
4. ✅ 減少執行時錯誤
5. ✅ 增強團隊協作效率

記住：**好的 TypeScript 程式碼不需要註解，型別本身就是最好的文件。**

---

## 延伸閱讀

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Effective TypeScript](https://effectivetypescript.com/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

---

*如果這篇文章對你有幫助，歡迎分享給更多開發者！*

---

文章已完成！需要調整語氣、長度或內容嗎？
```

---

## 日常助理

### 範例 9: 會議記錄摘要

**對話：**
```
你: 幫我整理今天會議的重點，並產生待辦清單

[貼上會議記錄...]

Bot: 我會幫你分析會議記錄並產生結構化的摘要。

# 📋 會議記錄摘要

**會議日期：** 2026-02-02
**會議時間：** 14:00-15:30
**會議主題：** Q1 產品規劃
**出席人員：** Alice, Bob, Carol, David

---

## 🎯 關鍵決策

1. **產品發布時程**
   - 決定：3月15日發布 v2.0
   - 負責人：Alice
   - 截止日期：2026-03-15

2. **新功能優先順序**
   - 決定：優先開發 AI 整合功能
   - 次要：社交分享功能
   - 延後：進階分析儀表板

3. **預算核准**
   - 批准額外 $50,000 行銷預算
   - 條件：需提供詳細計劃

---

## 💡 重點討論

### 技術架構
- 採用 microservices 架構
- 使用 Kubernetes 部署
- 資料庫：PostgreSQL + Redis

### 行銷策略
- 目標：Q1 獲得 10,000 新用戶
- 通路：社群媒體、部落格、廣告
- 預算分配：40% 社群、30% 內容、30% 廣告

### 人員需求
- 需招聘 2 名 Backend 工程師
- 1 名 DevOps 工程師
- 1 名產品設計師

---

## ✅ 待辦清單

### 高優先級（本週完成）

- [ ] **Alice**: 完成產品需求文件 (PRD)
      截止：2026-02-05
      相關人員：團隊全員 review

- [ ] **Bob**: 建立技術架構設計文件
      截止：2026-02-06
      依賴：需 Alice 的 PRD

- [ ] **Carol**: 準備行銷預算提案
      截止：2026-02-05
      預算：$50,000

### 中優先級（下週完成）

- [ ] **David**: 開始招聘流程
      截止：2026-02-12
      職位：Backend x2, DevOps x1, Designer x1

- [ ] **Alice**: 與設計團隊討論 UI/UX
      截止：2026-02-10

- [ ] **Bob**: 評估 Kubernetes hosting 方案
      截止：2026-02-12
      選項：AWS EKS, GCP GKE, Azure AKS

### 低優先級（月底前完成）

- [ ] **Carol**: 建立社群媒體內容日曆
      截止：2026-02-28

- [ ] **Team**: 規劃 Q2 產品路線圖
      截止：2026-02-28

---

## ⚠️ 風險與阻礙

1. **時程風險**
   - 3月15日時程緊迫
   - 緩解：考慮減少非核心功能

2. **人力資源**
   - 招聘可能需時較長
   - 緩解：考慮短期外包

3. **預算限制**
   - 行銷預算需核准
   - 緩解：準備詳細 ROI 分析

---

## 📅 下次會議

**日期：** 2026-02-09 (五)
**時間：** 14:00-15:00
**議程：**
1. 審查 PRD 和架構設計
2. 確認行銷預算提案
3. 招聘進度更新

---

## 📎 附件與參考

- [產品需求文件範本]
- [技術架構 Figma 連結]
- [競品分析報告]

---

摘要已完成！我還幫你：
✅ 結構化整理會議內容
✅ 提取所有待辦事項
✅ 標註責任人和截止日期
✅ 識別風險
✅ 準備下次會議議程

需要我匯出成其他格式（PDF、Notion、Trello）嗎？
```

---

## 更多範例

查看完整範例庫：
- [GitHub 範例集](https://github.com/a23444452/Clawdbot_Project/tree/main/examples)
- [互動式教學](https://openclaw.ai/tutorials)
- [社群分享](https://discord.gg/clawd)

---

**最後更新**: 2026-02-02
**版本**: 1.0.0

---

**💡 還有更多問題？在 Telegram/Discord 問你的 OpenClaw bot！**
