# 🦞 OpenClaw 完整教學指南

[![GitHub](https://img.shields.io/badge/GitHub-Clawdbot_Project-blue?style=for-the-badge&logo=github)](https://github.com/a23444452/Clawdbot_Project)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.1.29-green?style=for-the-badge)](https://openclaw.ai)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

> **個人 AI 助理完整部署指南** - 從零開始建立、設定、優化到實際使用

---

## 📖 目錄

- [專案簡介](#專案簡介)
- [系統需求](#系統需求)
- [快速開始](#快速開始)
- [完整安裝步驟](#完整安裝步驟)
- [詳細設定指南](#詳細設定指南)
- [資安優化建議](#資安優化建議)
- [頻道設定](#頻道設定)
- [進階功能](#進階功能)
- [使用範例](#使用範例)
- [常見問題](#常見問題)
- [疑難排解](#疑難排解)

---

## 專案簡介

**OpenClaw** 是一個可在個人設備上運行的 AI 助理系統。本專案基於 OpenClaw 官方版本，並加入了企業級安全功能。

### ✨ 主要特色

- 🔐 **企業級資安防護** - API Key 自動加密、敏感資訊隱藏
- 💬 **多頻道支援** - Telegram、WhatsApp、Slack、Discord 等
- 🎯 **完全自主控制** - 在你自己的設備上運行
- 🚀 **高度可客製化** - 支援自訂工具、技能和工作流程
- 🌐 **隱私優先** - 所有資料都在本地處理

### 🔒 本專案獨有的安全功能

1. **API Key 加密儲存**
   - 使用 AES-256-GCM 加密演算法
   - 整合系統 Keychain（macOS/Linux/Windows）
   - 自動加密所有敏感憑證

2. **自動 Redaction（資訊隱藏）**
   - 預設啟用，無需手動設定
   - 自動隱藏 10+ 種 API key/token 格式
   - 防止錯誤訊息外洩敏感資訊

3. **向後相容**
   - 支援既有的明文設定檔
   - 平滑升級，無需重新設定

---

## 系統需求

### 最低需求

- **作業系統**: macOS 10.15+, Linux, Windows 10+ (WSL2)
- **Node.js**: v22.0.0 或更高版本
- **記憶體**: 4GB RAM（建議 8GB+）
- **硬碟空間**: 2GB 可用空間
- **網路**: 穩定的網際網路連線

### 推薦配置

- **作業系統**: macOS 14+ (Sonoma)
- **Node.js**: v25.5.0
- **記憶體**: 16GB RAM
- **處理器**: Apple Silicon (M1/M2/M3) 或 Intel i5 以上
- **網路**: 100Mbps+ 寬頻

---

## 快速開始

### 方法一：一鍵安裝（推薦）

```bash
# 使用 npm
npm install -g openclaw@latest

# 或使用 pnpm（更快）
pnpm add -g openclaw@latest

# 啟動設定精靈
openclaw onboard
```

### 方法二：從原始碼安裝（開發者）

```bash
# 1. Clone 本專案
git clone https://github.com/a23444452/Clawdbot_Project.git
cd Clawdbot_Project

# 2. 安裝依賴
pnpm install

# 3. 建置專案
pnpm build

# 4. 啟動 Gateway
pnpm openclaw gateway run
```

---

## 完整安裝步驟

### 步驟 1: 安裝 Node.js

**macOS (使用 Homebrew):**
```bash
# 安裝 Homebrew（如果尚未安裝）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安裝 Node.js
brew install node@25
```

**Linux (Ubuntu/Debian):**
```bash
# 使用 NodeSource
curl -fsSL https://deb.nodesource.com/setup_25.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Windows:**
1. 下載並安裝 [Node.js v25](https://nodejs.org/)
2. 安裝 [WSL2](https://docs.microsoft.com/windows/wsl/install)
3. 在 WSL2 中執行 Linux 安裝步驟

### 步驟 2: 安裝套件管理器

```bash
# 安裝 pnpm（推薦，速度最快）
npm install -g pnpm@latest

# 驗證安裝
pnpm --version
```

### 步驟 3: 安裝 OpenClaw

```bash
# 全域安裝
pnpm add -g openclaw@latest

# 驗證安裝
openclaw --version
# 預期輸出: 2026.1.29 或更新版本
```

### 步驟 4: 執行設定精靈

```bash
openclaw onboard
```

設定精靈會引導你完成：
1. ✅ Gateway 設定
2. ✅ 工作目錄設定
3. ✅ AI 模型選擇與認證
4. ✅ 頻道設定（Telegram、WhatsApp 等）
5. ✅ 技能與工具設定

---

## 詳細設定指南

### Gateway 設定

Gateway 是 OpenClaw 的核心服務，負責處理所有通訊和 AI 請求。

#### 基本設定

編輯 `~/.openclaw/openclaw.json`:

```json
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your-secure-token-here"
    }
  }
}
```

#### 啟動 Gateway

```bash
# 手動啟動
openclaw gateway run

# 背景執行
openclaw gateway run --bind loopback --port 18789 > /tmp/openclaw-gateway.log 2>&1 &

# 檢查狀態
openclaw gateway status
```

#### Gateway 服務管理

**macOS (LaunchAgent):**
```bash
# 安裝系統服務
openclaw gateway install

# 啟動服務
openclaw gateway start

# 停止服務
openclaw gateway stop

# 查看日誌
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

**Linux (systemd):**
```bash
# 安裝系統服務
openclaw gateway install

# 啟動服務
sudo systemctl start openclaw-gateway

# 開機自動啟動
sudo systemctl enable openclaw-gateway

# 查看狀態
sudo systemctl status openclaw-gateway
```

### AI 模型設定

#### 支援的提供商

| 提供商 | 認證方式 | 推薦模型 |
|--------|----------|----------|
| Anthropic | OAuth (Pro/Max) | claude-opus-4.5 |
| OpenAI | OAuth / API Key | gpt-4-turbo |
| Google | API Key | gemini-pro |
| Azure OpenAI | API Key | gpt-4 |

#### 設定 Anthropic (推薦)

```bash
# 使用 OAuth 登入
openclaw models auth login --provider anthropic

# 設定預設模型
openclaw models set claude-opus-4-5

# 驗證設定
openclaw models status
```

#### 設定 OpenAI

```bash
# 使用 OAuth 登入
openclaw models auth login --provider openai

# 或使用 API Key
openclaw models auth setup-key --provider openai

# 設定預設模型
openclaw models set gpt-4-turbo
```

#### 模型切換與 Fallback

編輯 `~/.openclaw/openclaw.json`:

```json
{
  "agents": {
    "main": {
      "model": "claude-opus-4.5",
      "fallbacks": [
        "claude-sonnet-4.5",
        "gpt-4-turbo"
      ]
    }
  }
}
```

---

## 資安優化建議

### 🔐 1. API Key 保護（已內建）

本專案已實作企業級 API Key 保護機制：

#### 自動加密儲存

所有 API Keys 和 OAuth Tokens 會自動使用 AES-256-GCM 加密：

```json
{
  "version": 1,
  "profiles": {
    "openai-main": {
      "type": "api_key",
      "provider": "openai",
      "key": {
        "version": 1,
        "algorithm": "aes-256-gcm",
        "data": "encrypted-data-here",
        "iv": "initialization-vector",
        "authTag": "authentication-tag"
      }
    }
  }
}
```

加密金鑰安全儲存於系統 Keychain：
- **macOS**: Keychain Access
- **Linux**: libsecret
- **Windows**: Credential Manager

#### 自動 Redaction

預設啟用敏感資訊自動隱藏：

```json
{
  "logging": {
    "redactSensitive": "tools",  // "tools" | "all" | "off"
    "redactPatterns": [
      "internal\\.company\\.com",
      "VAULT_TOKEN=[A-Za-z0-9_-]{20,}"
    ]
  }
}
```

支援的格式：
- ✅ OpenAI keys (`sk-*`, `sk-proj-*`)
- ✅ Anthropic keys (`sk-ant-*`)
- ✅ GitHub tokens (`ghp_*`, `gho_*`, `ghs_*`)
- ✅ Slack tokens (`xoxb-*`, `xoxp-*`, `xoxa-*`)
- ✅ AWS credentials
- ✅ OAuth tokens
- ✅ JWT tokens
- ✅ 自訂 patterns

### 🔒 2. Gateway 安全設定

#### Token 認證

```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "使用強密碼產生器產生 32+ 字元的隨機 token"
    }
  }
}
```

#### 設備配對認證

```json
{
  "gateway": {
    "auth": {
      "mode": "device",
      "allowedDevices": [
        "device-id-1",
        "device-id-2"
      ]
    }
  }
}
```

#### Control UI 安全

```json
{
  "gateway": {
    "controlUi": {
      "allowInsecureAuth": false,  // 強制 HTTPS
      "requireDeviceIdentity": true
    }
  }
}
```

### 🛡️ 3. 網路安全

#### 僅本地訪問（最安全）

```json
{
  "gateway": {
    "bind": "loopback"  // 只允許 127.0.0.1 連線
  }
}
```

#### 使用 Tailscale（遠端安全訪問）

```bash
# 1. 安裝 Tailscale
brew install tailscale  # macOS
sudo apt install tailscale  # Linux

# 2. 設定 OpenClaw
openclaw configure
# 選擇 "Enable Tailscale Serve"

# 3. 驗證
openclaw gateway status
```

#### Reverse Proxy（進階）

使用 nginx 或 Caddy 搭配 SSL：

```nginx
server {
    listen 443 ssl http2;
    server_name openclaw.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 🔑 4. 最佳實踐

1. **定期更新**
   ```bash
   # 檢查更新
   openclaw update check

   # 更新到最新版
   openclaw update wizard
   ```

2. **啟用日誌 Redaction**
   ```json
   {
     "logging": {
       "redactSensitive": "tools"  // 預設已啟用
     }
   }
   ```

3. **限制工具存取**
   ```json
   {
     "tools": {
       "allow": ["read", "write", "grep", "exec"],
       "deny": ["rm", "sudo"]
     }
   }
   ```

4. **設定 Webhook 驗證**
   ```json
   {
     "channels": {
       "telegram": {
         "webhook": {
           "verify": true,
           "secret": "your-webhook-secret"
         }
       }
     }
   }
   ```

---

## 頻道設定

### Telegram Bot

#### 步驟 1: 建立 Bot

1. 與 [@BotFather](https://t.me/BotFather) 對話
2. 發送 `/newbot`
3. 設定 bot 名稱和 username
4. 複製獲得的 Bot Token

#### 步驟 2: 設定 OpenClaw

```bash
# 互動式設定
openclaw channels setup telegram

# 或手動編輯設定檔
```

`~/.openclaw/openclaw.json`:
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": [
        "your-telegram-username"
      ]
    }
  }
}
```

#### 步驟 3: 啟動並測試

```bash
# 重啟 Gateway
openclaw gateway restart

# 檢查頻道狀態
openclaw channels status

# 向你的 bot 發送訊息測試
```

### WhatsApp (Web)

#### 步驟 1: 準備工作

- 確保有 WhatsApp 帳號
- 手機需要能連上網路

#### 步驟 2: 登入 WhatsApp

```bash
# 啟動登入流程
openclaw channels login whatsapp

# 掃描 QR Code
# 使用 WhatsApp 手機 App: 設定 → 連結的裝置 → 掃描 QR Code
```

#### 步驟 3: 設定白名單

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": [
        "886912345678@s.whatsapp.net"  // 你的電話號碼
      ]
    }
  }
}
```

### Discord Bot

#### 步驟 1: 建立 Discord Application

1. 前往 [Discord Developer Portal](https://discord.com/developers/applications)
2. 點選 "New Application"
3. 進入 "Bot" 頁面，點選 "Add Bot"
4. 複製 Bot Token

#### 步驟 2: 設定權限

在 "OAuth2" → "URL Generator" 中勾選：
- ✅ bot
- ✅ applications.commands

Bot Permissions:
- ✅ Read Messages/View Channels
- ✅ Send Messages
- ✅ Send Messages in Threads
- ✅ Embed Links
- ✅ Attach Files
- ✅ Read Message History
- ✅ Use Slash Commands

#### 步驟 3: 設定 OpenClaw

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_DISCORD_BOT_TOKEN",
      "clientId": "YOUR_CLIENT_ID",
      "allowFrom": [
        "your-discord-user-id"
      ]
    }
  }
}
```

#### 步驟 4: 邀請 Bot

使用生成的 OAuth2 URL 邀請 bot 到你的伺服器。

---

## 進階功能

### 自訂技能 (Skills)

Skills 是 OpenClaw 的擴充功能系統。

#### 查看可用技能

```bash
openclaw skills list
```

#### 安裝技能

```bash
# 安裝特定技能
openclaw skills install <skill-name>

# 範例：安裝 PDF 技能
openclaw skills install pdf
```

#### 建立自訂技能

在 `~/.openclaw/skills/` 建立新的 Markdown 檔案：

```markdown
---
name: my-custom-skill
description: 我的自訂技能
---

# 技能說明

這個技能會執行特定的任務...

## 使用方式

1. 步驟一
2. 步驟二
3. 步驟三
```

### 沙箱環境

OpenClaw 支援在隔離的 Docker 容器中執行程式碼：

```json
{
  "sandbox": {
    "enabled": true,
    "image": "openclaw/sandbox:latest",
    "workspaceAccess": "rw"  // "ro" | "rw"
  }
}
```

### 記憶體搜尋

啟用向量搜尋來增強 AI 的記憶能力：

```bash
# 安裝記憶體外掛
openclaw plugins install memory-core

# 索引工作目錄
openclaw memory index

# 查詢
openclaw memory search "關鍵字"
```

### 多代理設定

設定多個獨立的 AI 代理：

```json
{
  "agents": {
    "main": {
      "model": "claude-opus-4.5",
      "systemPrompt": "你是一個通用助理"
    },
    "coder": {
      "model": "claude-sonnet-4.5",
      "systemPrompt": "你是一個專業程式設計師"
    },
    "writer": {
      "model": "gpt-4-turbo",
      "systemPrompt": "你是一個專業文案撰寫者"
    }
  }
}
```

使用方式：
```bash
# 切換代理
/agent coder

# 或在 Telegram/Discord 中
@YourBot /agent writer
```

---

## 使用範例

### 範例 1: 程式碼協助

**Telegram 對話：**
```
你：幫我寫一個 Python 函數，計算費氏數列

Bot：我會幫你寫一個費氏數列函數：

```python
def fibonacci(n):
    """
    計算費氏數列的第 n 項

    Args:
        n: 項數 (從 0 開始)

    Returns:
        第 n 項的費氏數
    """
    if n <= 1:
        return n

    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b

    return b

# 測試
for i in range(10):
    print(f"F({i}) = {fibonacci(i)}")
```

這個函數使用迭代方式，時間複雜度為 O(n)，空間複雜度為 O(1)。
```

### 範例 2: 檔案操作

```
你：列出當前目錄下所有 Python 檔案

Bot：我會使用 grep 工具搜尋：
[執行工具: glob pattern="*.py"]

找到以下 Python 檔案：
1. main.py
2. config.py
3. utils.py
4. tests/test_main.py
5. tests/test_utils.py

共 5 個檔案。
```

### 範例 3: 網頁搜尋

```
你：最新的 TypeScript 5.0 有什麼新功能？

Bot：讓我搜尋最新資訊...
[執行工具: web_search query="TypeScript 5.0 new features 2026"]

根據搜尋結果，TypeScript 5.0 的主要新功能包括：

1. **Decorators** - 正式支援 Stage 3 裝飾器語法
2. **const Type Parameters** - 更精確的型別推論
3. **Supporting Multiple Configuration Files** - 支援 extends 陣列
4. **Speed Improvements** - 編譯速度提升 10-20%
5. **Better Enum Support** - 改進列舉型別的處理

更多詳細資訊可以參考：
- [官方公告](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/)
- [Release Notes](https://github.com/microsoft/TypeScript/releases/tag/v5.0.0)
```

### 範例 4: 圖片理解

```
你：[上傳圖片] 這張圖片是什麼？

Bot：這張圖片顯示一隻橘色的虎斑貓躺在陽光灑落的窗台上。貓咪看起來很放鬆，
眼睛微微瞇起，似乎正在享受溫暖的陽光。背景可以看到一些植物和窗簾。

整體氛圍溫馨舒適，是典型的貓咪午後悠閒時光。
```

### 範例 5: 程式碼審查

```
你：請審查這段程式碼：
```python
def process_data(data):
    result = []
    for item in data:
        if item > 0:
            result.append(item * 2)
    return result
```

Bot：這段程式碼可以運作，但有幾個可以改進的地方：

**建議改進：**

1. **使用列表推導式** - 更簡潔高效
2. **加入型別提示** - 提升可讀性
3. **加入文件字串** - 說明函數用途

**改進後的版本：**

```python
def process_data(data: list[int | float]) -> list[int | float]:
    """
    處理數值列表，將正數乘以 2

    Args:
        data: 包含數值的列表

    Returns:
        處理後的列表，只包含原本為正數的元素（乘以 2）
    """
    return [item * 2 for item in data if item > 0]
```

**改進說明：**
- ✅ 程式碼更簡潔（1 行 vs 4 行）
- ✅ 加入了型別提示
- ✅ 加入了詳細的文件字串
- ✅ 效能略有提升（列表推導式在 Python 中通常更快）
```

### 範例 6: 多步驟任務

```
你：幫我建立一個新的 Node.js 專案，包含 TypeScript 和 ESLint

Bot：我會幫你建立一個完整的 Node.js + TypeScript 專案。讓我逐步執行：

[執行工具: exec command="mkdir my-project && cd my-project"]
✅ 建立專案目錄

[執行工具: exec command="npm init -y"]
✅ 初始化 package.json

[執行工具: exec command="npm install -D typescript @types/node eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin"]
✅ 安裝開發依賴

[執行工具: write path="tsconfig.json"]
✅ 建立 TypeScript 設定檔

[執行工具: write path=".eslintrc.json"]
✅ 建立 ESLint 設定檔

[執行工具: write path="src/index.ts"]
✅ 建立主程式檔案

專案建立完成！目錄結構：
```
my-project/
├── package.json
├── tsconfig.json
├── .eslintrc.json
├── node_modules/
└── src/
    └── index.ts
```

你可以使用以下指令：
- `npm run build` - 編譯 TypeScript
- `npm run lint` - 執行 ESLint
- `npm start` - 執行程式
```

---

## 常見問題

### Q1: Gateway 無法啟動怎麼辦？

**A:** 檢查以下項目：

```bash
# 1. 檢查 port 是否被佔用
lsof -i :18789

# 2. 檢查 Gateway 狀態
openclaw gateway status

# 3. 查看日誌
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# 4. 重啟 Gateway
openclaw gateway restart
```

### Q2: Bot 沒有回應訊息？

**A:** 檢查步驟：

1. **確認 Gateway 運行中**
   ```bash
   openclaw gateway status
   ```

2. **檢查頻道狀態**
   ```bash
   openclaw channels status
   ```

3. **確認白名單設定**
   ```bash
   openclaw config get channels.telegram.allowFrom
   ```

4. **查看錯誤日誌**
   ```bash
   tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log | grep ERROR
   ```

### Q3: 如何切換 AI 模型？

**A:** 有多種方式：

```bash
# 方法 1: CLI 指令
openclaw models set claude-opus-4.5

# 方法 2: 在對話中切換（Telegram/Discord）
/model claude-sonnet-4.5

# 方法 3: 編輯設定檔
vim ~/.openclaw/openclaw.json
# 修改 agents.main.model
```

### Q4: 如何備份設定和資料？

**A:** 備份重要檔案：

```bash
# 建立備份目錄
mkdir ~/openclaw-backup

# 備份設定檔
cp ~/.openclaw/openclaw.json ~/openclaw-backup/

# 備份憑證（已加密）
cp -r ~/.openclaw/credentials ~/openclaw-backup/

# 備份 Sessions
cp -r ~/.openclaw/agents ~/openclaw-backup/

# 打包備份
tar -czf openclaw-backup-$(date +%Y%m%d).tar.gz ~/openclaw-backup
```

### Q5: API 用量太高怎麼辦？

**A:** 優化建議：

1. **使用更便宜的模型**
   ```json
   {
     "agents": {
       "main": {
         "model": "claude-sonnet-4.5"  // 比 Opus 便宜
       }
     }
   }
   ```

2. **啟用 Compaction（上下文壓縮）**
   ```json
   {
     "agents": {
       "main": {
         "compaction": {
           "enabled": true,
           "maxTokens": 100000
         }
       }
     }
   }
   ```

3. **限制歷史訊息數量**
   ```json
   {
     "agents": {
       "main": {
         "maxHistoryMessages": 20
       }
     }
   }
   ```

### Q6: 如何確認 API Key 已加密？

**A:** 檢查方式：

```bash
# 1. 查看 auth-profiles.json
cat ~/.openclaw/credentials/auth-profiles.json

# 應該看到類似這樣的結構（加密後）：
# {
#   "version": 1,
#   "profiles": {
#     "openai-main": {
#       "key": {
#         "version": 1,
#         "algorithm": "aes-256-gcm",
#         "data": "encrypted...",
#         "iv": "...",
#         "authTag": "..."
#       }
#     }
#   }
# }

# 2. 如果看到明文 API Key，執行：
openclaw models auth refresh
```

### Q7: macOS Keychain 不斷彈出授權視窗？

**A:** 解決方式：

```bash
# 1. 解鎖 Login Keychain
security unlock-keychain ~/Library/Keychains/login.keychain-db

# 2. 設定 OpenClaw 可自動存取
security set-key-partition-list -S apple-tool:,apple: -s -k YOUR_PASSWORD ~/Library/Keychains/login.keychain-db

# 3. 重新儲存憑證
openclaw models auth refresh
```

---

## 疑難排解

### 問題 1: Node.js 版本過舊

**症狀：**
```
Error: OpenClaw requires Node.js v22 or higher
```

**解決方式：**
```bash
# macOS
brew install node@25

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_25.x | sudo -E bash -
sudo apt-get install -y nodejs

# 驗證
node --version  # 應該是 v25.x.x
```

### 問題 2: pnpm 指令找不到

**症狀：**
```
command not found: pnpm
```

**解決方式：**
```bash
# 安裝 pnpm
npm install -g pnpm@latest

# 或使用 standalone installer
curl -fsSL https://get.pnpm.io/install.sh | sh -

# 重新載入 shell 設定
source ~/.zshrc  # 或 ~/.bashrc
```

### 問題 3: Port 被佔用

**症狀：**
```
Error: listen EADDRINUSE: address already in use :::18789
```

**解決方式：**
```bash
# 1. 找出佔用 port 的進程
lsof -i :18789

# 2. 終止該進程
kill -9 <PID>

# 3. 或使用不同的 port
openclaw gateway run --port 18790
```

### 問題 4: Telegram Bot 權限不足

**症狀：**
Bot 無法讀取訊息或發送訊息

**解決方式：**
1. 與 [@BotFather](https://t.me/BotFather) 對話
2. 發送 `/mybots`
3. 選擇你的 bot
4. 進入 "Bot Settings" → "Group Privacy"
5. 設定為 **DISABLED** (讓 bot 可以讀取所有訊息)

### 問題 5: WhatsApp 登入失敗

**症狀：**
QR Code 掃描後斷線或無法連接

**解決方式：**
```bash
# 1. 清除舊的 session
rm -rf ~/.openclaw/credentials/whatsapp-*

# 2. 確保 WhatsApp 手機 App 已更新到最新版

# 3. 重新登入
openclaw channels login whatsapp

# 4. 如果仍然失敗，嘗試使用不同的網路
# (有時候 ISP 會封鎖 WhatsApp Web 連線)
```

### 問題 6: Docker 沙箱無法啟動

**症狀：**
```
Error: Cannot connect to the Docker daemon
```

**解決方式：**
```bash
# macOS - 安裝 Docker Desktop
brew install --cask docker

# 啟動 Docker Desktop
open /Applications/Docker.app

# Linux - 安裝 Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 啟動 Docker 服務
sudo systemctl start docker

# 加入 docker 群組（避免需要 sudo）
sudo usermod -aG docker $USER
```

### 問題 7: 記憶體搜尋無法使用

**症狀：**
```
Memory search unavailable
```

**解決方式：**
```bash
# 1. 安裝記憶體外掛
openclaw plugins install memory-core

# 2. 設定向量資料庫
openclaw configure
# 選擇 "Memory Search" → "Enable" → "Configure"

# 3. 建立索引
openclaw memory index

# 4. 驗證
openclaw memory status
```

---

## 進階主題

更多進階設定和使用方式，請參考：

- [官方文件](https://docs.openclaw.ai)
- [API 參考](https://docs.openclaw.ai/api)
- [外掛開發](https://docs.openclaw.ai/plugins)
- [貢獻指南](https://github.com/openclaw/openclaw/blob/main/CONTRIBUTING.md)

---

## 參與貢獻

歡迎提交 Issue 和 Pull Request！

**本專案獨有的安全功能開發:**
- Phase 1: API Key Redaction ✅
- Phase 2: Encryption Storage ✅
- Phase 3: Documentation ✅

查看完整開發歷程：[PROGRESS.md](PROGRESS.md)

---

## 授權

本專案基於 [MIT License](LICENSE)

---

## 相關連結

- **專案首頁**: https://openclaw.ai
- **官方文件**: https://docs.openclaw.ai
- **GitHub**: https://github.com/a23444452/Clawdbot_Project
- **Discord 社群**: https://discord.gg/clawd

---

## 致謝

- OpenClaw 官方團隊
- 所有貢獻者
- Claude AI (Anthropic)

---

**最後更新**: 2026-02-02
**版本**: 2026.1.29 + Security Enhancement

---

**🦞 享受你的個人 AI 助理！**
