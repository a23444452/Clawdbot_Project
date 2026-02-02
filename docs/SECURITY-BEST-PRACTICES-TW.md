# 🔒 OpenClaw 資安最佳實踐指南

> 企業級安全設定與最佳實踐建議

---

## 📋 目錄

- [核心安全功能](#核心安全功能)
- [認證與授權](#認證與授權)
- [網路安全](#網路安全)
- [資料保護](#資料保護)
- [監控與審計](#監控與審計)
- [定期維護](#定期維護)
- [安全檢查清單](#安全檢查清單)

---

## 核心安全功能

### 🔐 1. API Key 加密儲存（已內建）

本專案實作了企業級 API Key 保護機制。

#### 自動加密

所有敏感憑證自動使用 **AES-256-GCM** 加密：

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
        "data": "加密後的資料",
        "iv": "初始向量",
        "authTag": "認證標籤"
      }
    }
  }
}
```

**特點：**
- ✅ **NIST 認證** - AES-256-GCM 是美國政府認證的加密標準
- ✅ **防竄改** - 包含 Authentication Tag 驗證資料完整性
- ✅ **自動執行** - 無需手動設定，儲存時自動加密
- ✅ **系統整合** - 加密金鑰儲存在系統 Keychain

#### 金鑰管理

加密金鑰安全儲存於系統 Keychain：

| 平台 | 儲存位置 | 存取方式 |
|------|----------|----------|
| macOS | Keychain Access | `security` 指令 |
| Linux | libsecret | `secret-tool` 指令 |
| Windows | Credential Manager | Windows API |

**服務識別：**
- Service Name: `openclaw.encryption`
- Account Name: `auth-profiles`

#### 驗證加密狀態

```bash
# 檢查 auth-profiles.json
cat ~/.openclaw/credentials/auth-profiles.json

# 加密後的檔案包含：
# - version: 1
# - algorithm: "aes-256-gcm"
# - data, iv, authTag 欄位

# 如果看到明文 API Key，執行：
openclaw models auth refresh
```

---

### 🙈 2. 自動 Redaction（敏感資訊隱藏）

預設啟用自動隱藏敏感資訊，防止意外洩漏。

#### 支援的格式

| 類型 | 格式範例 | 隱藏後 |
|------|----------|--------|
| OpenAI | `sk-proj-abc123...` | `sk-proj-***` |
| Anthropic | `sk-ant-xyz789...` | `sk-ant-***` |
| GitHub | `ghp_abc123...` | `ghp_***` |
| Slack | `xoxb-123-456...` | `xoxb-***` |
| AWS | `AKIAIOSFODNN7...` | `AKIA***` |
| JWT | `eyJhbGciOiJIUz...` | `eyJ***` |

**總計支援 10+ 種常見格式**

#### 設定選項

編輯 `~/.openclaw/openclaw.json`:

```json
{
  "logging": {
    "redactSensitive": "tools",  // 推薦設定
    "redactPatterns": [
      // 自訂 patterns
      "internal\\.company\\.com",
      "VAULT_TOKEN=[A-Za-z0-9_-]{20,}",
      "api-key:\\s*['\"]([^'\"]{20,})['\"]"
    ]
  }
}
```

**模式說明：**

| 模式 | 說明 | 使用時機 |
|------|------|----------|
| `"tools"` | 在工具輸出和錯誤訊息中隱藏 | ✅ 推薦（預設） |
| `"all"` | 在所有地方隱藏（包含 transcript） | 最大安全性 |
| `"off"` | 關閉自動隱藏 | ⚠️ 僅用於本地除錯 |

#### 測試 Redaction

```bash
# 觸發錯誤訊息查看 redaction 效果
openclaw models auth setup-key --provider openai
# 輸入: sk-test-1234567890abcdef1234567890abcdef

# 查看日誌
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# 應該看到類似：
# [ERROR] Invalid API key: sk-***
# 而非完整的 key
```

---

## 認證與授權

### 🔑 Gateway 認證

#### 方法 1: Token 認證（推薦）

```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "使用密碼產生器產生的 32+ 字元隨機字串"
    }
  }
}
```

**產生強密碼：**
```bash
# macOS/Linux
openssl rand -base64 32

# 或
head -c 32 /dev/urandom | base64
```

#### 方法 2: 設備認證

```json
{
  "gateway": {
    "auth": {
      "mode": "device",
      "allowedDevices": [
        "MacBook-Pro-01",
        "iPhone-12-Pro"
      ]
    }
  }
}
```

**獲取設備 ID：**
```bash
openclaw gateway status | grep "Device ID"
```

#### 方法 3: 雙因素認證

結合 token + device：

```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "your-token",
      "requireDevice": true,
      "allowedDevices": ["trusted-device-1"]
    }
  }
}
```

### 🚪 頻道白名單

**強制執行白名單** - 只有授權的使用者可以與 bot 互動。

#### Telegram

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "allowFrom": [
        "your_username",      // Telegram username
        "123456789"           // 或 Telegram User ID
      ],
      "groups": {
        "policy": "allowlist",
        "allow": [
          "-1001234567890"    // 授權的群組 ID
        ]
      }
    }
  }
}
```

**獲取 Telegram User ID：**
1. 與 [@userinfobot](https://t.me/userinfobot) 對話
2. 發送任意訊息
3. Bot 會回覆你的 User ID

#### WhatsApp

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": [
        "886912345678@s.whatsapp.net"  // 國碼 + 號碼
      ]
    }
  }
}
```

#### Discord

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "allowFrom": [
        "123456789012345678"  // Discord User ID
      ],
      "guilds": {
        "allow": [
          "987654321098765432"  // 授權的伺服器 ID
        ]
      }
    }
  }
}
```

---

## 網路安全

### 🌐 連線模式

#### 模式 1: Loopback Only（最安全）

只允許本機連線：

```json
{
  "gateway": {
    "bind": "loopback",  // 只監聽 127.0.0.1
    "port": 18789
  }
}
```

**適用情境：**
- ✅ Gateway 和客戶端在同一台機器
- ✅ 最高安全性要求
- ❌ 無法從其他設備訪問

#### 模式 2: Tailscale（推薦用於遠端訪問）

安全的點對點連線：

```bash
# 1. 安裝 Tailscale
brew install tailscale  # macOS
sudo apt install tailscale  # Linux

# 2. 登入
sudo tailscale up

# 3. 設定 OpenClaw
openclaw configure
# 選擇 "Enable Tailscale Serve"

# 4. 驗證
openclaw gateway status
```

**設定檔：**
```json
{
  "gateway": {
    "tailscale": {
      "enabled": true,
      "serve": true
    }
  }
}
```

**優點：**
- ✅ 加密連線（WireGuard 協定）
- ✅ 不需要開放防火牆 port
- ✅ 自動 NAT 穿透
- ✅ 跨平台支援

#### 模式 3: Reverse Proxy（進階）

使用 Nginx 或 Caddy 搭配 SSL：

**Nginx 範例：**
```nginx
server {
    listen 443 ssl http2;
    server_name openclaw.yourdomain.com;

    # SSL 設定
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Proxy 設定
    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Caddy 範例（更簡單）：**
```
openclaw.yourdomain.com {
    reverse_proxy 127.0.0.1:18789
}
```

**OpenClaw 設定：**
```json
{
  "gateway": {
    "bind": "loopback",
    "trustedProxies": [
      "127.0.0.1",
      "::1"
    ]
  }
}
```

### 🔥 防火牆設定

#### macOS

```bash
# 檢查防火牆狀態
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# 啟用防火牆
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate on

# 允許 Node.js（OpenClaw）
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/local/bin/node
```

#### Linux (UFW)

```bash
# 啟用 UFW
sudo ufw enable

# 如果使用 loopback，不需要開放 port

# 如果需要遠端訪問（不推薦）
sudo ufw allow from YOUR_IP to any port 18789
```

---

## 資料保護

### 💾 備份策略

#### 重要檔案清單

```bash
~/.openclaw/
├── openclaw.json           # 主設定檔 ⭐
├── credentials/            # 憑證（已加密）⭐
│   ├── auth-profiles.json
│   └── whatsapp-*
├── agents/                 # Agent 資料 ⭐
│   └── main/
│       └── sessions/
└── skills/                 # 自訂技能
```

#### 自動備份腳本

建立 `~/bin/backup-openclaw.sh`:

```bash
#!/bin/bash

BACKUP_DIR=~/openclaw-backups
DATE=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="$BACKUP_DIR/openclaw-$DATE.tar.gz"

# 建立備份目錄
mkdir -p "$BACKUP_DIR"

# 建立備份
tar -czf "$BACKUP_FILE" \
    ~/.openclaw/openclaw.json \
    ~/.openclaw/credentials \
    ~/.openclaw/agents \
    ~/.openclaw/skills

# 保留最近 30 天的備份
find "$BACKUP_DIR" -name "openclaw-*.tar.gz" -mtime +30 -delete

echo "Backup created: $BACKUP_FILE"
```

**設定自動執行（cron）：**
```bash
# 每天凌晨 2 點備份
0 2 * * * ~/bin/backup-openclaw.sh
```

#### 恢復備份

```bash
# 列出備份
ls -lh ~/openclaw-backups/

# 恢復特定備份
tar -xzf ~/openclaw-backups/openclaw-20260202-020000.tar.gz -C ~/
```

### 🗑️ 安全刪除

刪除敏感資料時使用安全抹除：

```bash
# macOS - 使用 srm（需安裝）
brew install srm
srm -v ~/.openclaw/credentials/old-token.json

# Linux - 使用 shred
shred -vfz -n 3 ~/.openclaw/credentials/old-token.json

# 或使用 secure-delete 套件
sudo apt install secure-delete
srm -v ~/.openclaw/credentials/old-token.json
```

### 🔄 金鑰輪替

定期更換 API Keys 和 tokens：

```bash
# 1. 產生新的 API Key（在提供商網站）

# 2. 更新 OpenClaw
openclaw models auth setup-key --provider openai

# 3. 驗證新 key
openclaw models status --probe

# 4. 舊 key 在提供商網站上撤銷

# 5. 安全刪除舊的設定檔
srm ~/.openclaw/credentials/auth-profiles.json.bak
```

**建議輪替頻率：**
- 🔴 **Gateway Token**: 每 90 天
- 🟡 **API Keys**: 每 180 天
- 🟢 **Channel Tokens**: 每年或懷疑洩漏時

---

## 監控與審計

### 📊 日誌監控

#### 啟用詳細日誌

```json
{
  "logging": {
    "level": "info",
    "file": "/var/log/openclaw/openclaw.log",
    "maxFiles": 30,
    "maxSize": "50m"
  }
}
```

#### 監控關鍵事件

建立監控腳本 `~/bin/monitor-openclaw.sh`:

```bash
#!/bin/bash

LOG_FILE="/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log"
ALERT_EMAIL="your-email@example.com"

# 監控登入失敗
FAILED_AUTH=$(grep -c "Auth failed" "$LOG_FILE")
if [ "$FAILED_AUTH" -gt 10 ]; then
    echo "Warning: $FAILED_AUTH failed auth attempts" | mail -s "OpenClaw Security Alert" "$ALERT_EMAIL"
fi

# 監控異常錯誤
ERROR_COUNT=$(grep -c "ERROR" "$LOG_FILE")
if [ "$ERROR_COUNT" -gt 50 ]; then
    echo "Warning: $ERROR_COUNT errors in logs" | mail -s "OpenClaw Error Alert" "$ALERT_EMAIL"
fi

# 監控未授權訪問嘗試
UNAUTHORIZED=$(grep -c "Unauthorized" "$LOG_FILE")
if [ "$UNAUTHORIZED" -gt 5 ]; then
    echo "Warning: $UNAUTHORIZED unauthorized access attempts" | mail -s "OpenClaw Security Alert" "$ALERT_EMAIL"
fi
```

**設定每小時檢查：**
```bash
0 * * * * ~/bin/monitor-openclaw.sh
```

#### 日誌分析工具

```bash
# 查看今天的錯誤
grep ERROR /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# 統計錯誤類型
grep ERROR /tmp/openclaw/openclaw-*.log | cut -d':' -f4 | sort | uniq -c | sort -rn

# 查看登入失敗
grep "Auth failed" /tmp/openclaw/openclaw-*.log

# 查看 API 用量
grep "API request" /tmp/openclaw/openclaw-*.log | wc -l
```

### 🔔 即時告警

使用 `journalctl` (Linux) 或 `log` (macOS) 設定即時告警：

**macOS:**
```bash
# 監控 OpenClaw 日誌
log stream --predicate 'subsystem == "ai.openclaw"' --level error
```

**Linux:**
```bash
# 監控 systemd 服務
journalctl -u openclaw-gateway -f --priority=err
```

---

## 定期維護

### 📅 每週檢查

```bash
# 檢查系統狀態
openclaw status --deep

# 檢查更新
openclaw update check

# 檢查頻道連線
openclaw channels status --probe

# 檢查磁碟空間
df -h ~/.openclaw
```

### 🔄 每月維護

```bash
# 1. 更新 OpenClaw
openclaw update wizard

# 2. 清理舊日誌
find /tmp/openclaw -name "*.log" -mtime +30 -delete

# 3. 檢查設定檔
openclaw doctor

# 4. 驗證備份
ls -lh ~/openclaw-backups/ | tail -5

# 5. 審查存取日誌
grep "Auth" /tmp/openclaw/openclaw-*.log | tail -100

# 6. 更新依賴
cd /path/to/openclaw && pnpm update
```

### 🔐 每季度安全審查

- [ ] 審查所有授權使用者和設備
- [ ] 輪替 Gateway token
- [ ] 檢查並更新白名單
- [ ] 審查日誌中的異常活動
- [ ] 測試備份恢復流程
- [ ] 更新 API Keys（如適用）
- [ ] 檢查加密狀態
- [ ] 審查外掛和技能權限

---

## 安全檢查清單

### ✅ 初始設定

- [ ] Gateway 使用 token 或 device 認證
- [ ] Gateway 綁定 loopback（或使用 Tailscale）
- [ ] 所有頻道都設定了白名單
- [ ] API Keys 已自動加密（檢查檔案格式）
- [ ] Redaction 已啟用（預設開啟）
- [ ] Control UI 強制 HTTPS 或 device 認證
- [ ] 防火牆已設定（如需要）
- [ ] 備份腳本已設定

### ✅ 持續維護

- [ ] 定期檢查日誌（每週）
- [ ] 定期更新系統（每月）
- [ ] 定期審查存取權限（每季）
- [ ] 定期測試備份恢復（每季）
- [ ] 定期輪替憑證（每季/半年）
- [ ] 監控異常活動（持續）

### ✅ 事件回應

當懷疑安全問題時：

1. **立即行動：**
   ```bash
   # 停止 Gateway
   openclaw gateway stop

   # 檢查日誌
   grep -i "auth\|error\|fail" /tmp/openclaw/openclaw-*.log | tail -100
   ```

2. **評估影響：**
   - 檢查哪些憑證可能外洩
   - 檢查異常的 API 呼叫
   - 檢查未授權的訪問嘗試

3. **修復：**
   ```bash
   # 輪替所有憑證
   openclaw models auth refresh

   # 更新 Gateway token
   openclaw configure

   # 重新審查白名單
   vim ~/.openclaw/openclaw.json
   ```

4. **驗證：**
   ```bash
   # 重啟並測試
   openclaw gateway start
   openclaw status --deep
   ```

---

## 進階主題

### 🔒 Zero Trust 架構

實作零信任原則：

```json
{
  "gateway": {
    "auth": {
      "mode": "token",
      "requireDevice": true,
      "tokenRotation": "daily"
    }
  },
  "channels": {
    "*": {
      "requireAuth": true,
      "allowFrom": "allowlist-only"
    }
  },
  "tools": {
    "policy": "deny-by-default",
    "allow": ["read", "grep", "web_search"],
    "elevated": {
      "require": "approval"
    }
  }
}
```

### 🛡️ 多層防禦

1. **網路層**: Tailscale / VPN
2. **應用層**: Gateway Token + Device Auth
3. **頻道層**: 白名單
4. **工具層**: 權限管制
5. **資料層**: 加密儲存
6. **日誌層**: 自動 Redaction

---

## 參考資源

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/controls)
- [OpenClaw 官方安全文件](https://docs.openclaw.ai/gateway/security)

---

**最後更新**: 2026-02-02
**版本**: 1.0.0

---

**🔒 安全是持續的過程，不是一次性的設定！**
