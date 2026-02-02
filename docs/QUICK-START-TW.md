# ⚡ OpenClaw 快速開始指南

> 5 分鐘內啟動你的個人 AI 助理

---

## 🎯 系統需求

- ✅ Node.js v22+
- ✅ macOS 10.15+ / Linux / Windows 10+ (WSL2)
- ✅ 4GB+ RAM
- ✅ 穩定網路連線

---

## 📦 安裝步驟

### 步驟 1: 安裝 OpenClaw (1 分鐘)

```bash
# 使用 pnpm（推薦）
pnpm add -g openclaw@latest

# 或使用 npm
npm install -g openclaw@latest

# 驗證安裝
openclaw --version
```

### 步驟 2: 執行設定精靈 (2 分鐘)

```bash
openclaw onboard
```

精靈會引導你完成：
1. Gateway 設定
2. AI 模型選擇
3. 頻道設定（Telegram/WhatsApp等）

### 步驟 3: 啟動 Gateway (30 秒)

```bash
# 啟動
openclaw gateway run

# 或背景執行
openclaw gateway start
```

### 步驟 4: 驗證運作 (30 秒)

```bash
# 檢查狀態
openclaw status

# 檢查頻道
openclaw channels status
```

---

## 🤖 設定你的第一個 Bot (Telegram 範例)

### 1. 建立 Telegram Bot

與 [@BotFather](https://t.me/BotFather) 對話：
```
你: /newbot
BotFather: Alright, a new bot. How are we going to call it?
你: My Personal AI
BotFather: Good. Now let's choose a username...
你: MyPersonalAI_bot
BotFather: Done! ... Use this token to access HTTP API:
         1234567890:ABCDEFGHIJKLMNOP...
```

### 2. 設定 OpenClaw

```bash
# 互動式設定
openclaw channels setup telegram

# 輸入 Bot Token
# 輸入你的 Telegram username（用於白名單）
```

### 3. 測試

在 Telegram 中：
```
你: /start
Bot: 你好！我是你的個人 AI 助理。

你: 你好，請自我介紹
Bot: 我是基於 Claude 的 AI 助理，運行在你的 OpenClaw 系統上...
```

---

## ⚙️ 基本設定

### 切換 AI 模型

```bash
# 查看可用模型
openclaw models list

# 設定預設模型
openclaw models set claude-opus-4.5
```

### 設定白名單

編輯 `~/.openclaw/openclaw.json`:
```json
{
  "channels": {
    "telegram": {
      "allowFrom": [
        "your-username"
      ]
    }
  }
}
```

### 查看日誌

```bash
# 即時查看
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# 查看錯誤
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log | grep ERROR
```

---

## 🔧 常用指令

### Gateway 管理

```bash
# 啟動
openclaw gateway start

# 停止
openclaw gateway stop

# 重啟
openclaw gateway restart

# 狀態
openclaw gateway status
```

### 頻道管理

```bash
# 查看所有頻道狀態
openclaw channels status

# 設定新頻道
openclaw channels setup <channel-name>

# 登入頻道（WhatsApp/Discord）
openclaw channels login <channel-name>
```

### 模型管理

```bash
# 列出所有模型
openclaw models list

# 查看當前設定
openclaw models status

# 登入提供商（OAuth）
openclaw models auth login --provider anthropic

# 設定 API Key
openclaw models auth setup-key --provider openai
```

---

## 💡 快速範例

### 範例 1: 程式碼生成

```
你: 寫一個 TypeScript 函數來排序陣列

Bot:
```typescript
function sortArray<T>(arr: T[], compareFn?: (a: T, b: T) => number): T[] {
  return [...arr].sort(compareFn);
}

// 使用範例
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];
const sorted = sortArray(numbers, (a, b) => a - b);
console.log(sorted); // [1, 1, 2, 3, 4, 5, 6, 9]
```
```

### 範例 2: 檔案搜尋

```
你: 找出所有包含 "TODO" 的 TypeScript 檔案

Bot: [使用 grep 工具搜尋...]
找到 15 個檔案包含 "TODO":
1. src/main.ts (3 處)
2. src/utils.ts (1 處)
...
```

### 範例 3: 網頁搜尋

```
你: TypeScript 5.0 有什麼新功能？

Bot: [執行網頁搜尋...]
TypeScript 5.0 的主要新功能：
1. Decorators 正式支援
2. const Type Parameters
3. 效能改進...
```

---

## 🔒 安全檢查清單

設定完成後，請確認：

- [ ] Gateway 使用 token 或 device 認證
- [ ] 頻道已設定白名單（allowFrom）
- [ ] API Keys 已自動加密（檢查 ~/.openclaw/credentials/）
- [ ] Redaction 已啟用（預設開啟）
- [ ] 只在需要時才啟用遠端訪問

---

## 📚 下一步

- 📖 閱讀完整教學：[README-TW.md](../README-TW.md)
- 🔐 查看資安最佳實踐：[SECURITY-BEST-PRACTICES-TW.md](./SECURITY-BEST-PRACTICES-TW.md)
- 💡 瀏覽使用範例：[EXAMPLES-TW.md](./EXAMPLES-TW.md)
- 🛠️ 客製化設定：[官方文件](https://docs.openclaw.ai)

---

## ❓ 遇到問題？

### Gateway 無法啟動
```bash
# 檢查 port 是否被佔用
lsof -i :18789

# 查看錯誤日誌
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

### Bot 沒有回應
```bash
# 檢查 Gateway 狀態
openclaw gateway status

# 檢查頻道狀態
openclaw channels status

# 重啟 Gateway
openclaw gateway restart
```

### 更多協助

- [疑難排解](../README-TW.md#疑難排解)
- [常見問題](../README-TW.md#常見問題)
- [Discord 社群](https://discord.gg/clawd)

---

**🎉 恭喜！你的個人 AI 助理已準備就緒！**
