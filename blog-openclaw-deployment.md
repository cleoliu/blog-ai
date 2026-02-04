# OpenClaw 部署教學：從踩坑到上岸的完整指南

> 原 Clawdbot、Moltbot，現在叫 OpenClaw——因為每隻太空龍蝦都需要一個響亮的名字

如果你想要一個 24/7 運行的 AI 助理，能透過 Telegram 隨時聊天、幫你處理各種任務，那 OpenClaw 就是你的菜。這篇文章會帶你從零開始部署，包括我踩過的坑和最終的解決方案。

---

## 目錄

1. [OpenClaw 是什麼](#openclaw-是什麼)
2. [建議資源配置](#建議資源配置)
3. [GCP VM vs macOS：我的踩坑經驗](#gcp-vm-vs-macos我的踩坑經驗)
4. [在 UTM macOS 虛擬機上部署](#在-utm-macos-虛擬機上部署)
5. [AI 模型配置：省錢又好用的選擇](#ai-模型配置省錢又好用的選擇)
6. [Telegram 配對完整教學](#telegram-配對完整教學)
7. [常見問題排解](#常見問題排解)

---

## OpenClaw 是什麼

OpenClaw 是一個 AI 助理網關，可以連接 WhatsApp、Telegram、Discord、iMessage 等聊天平台，讓你透過這些平台與 AI 代理互動。

簡單說就是：發訊息給它，它會用 AI 回覆你，還能幫你執行各種任務（查資料、寫程式、控制智慧家居等等）。

```
WhatsApp / Telegram / Discord / iMessage
        │
        ▼
  ┌───────────────────────────┐
  │      OpenClaw Gateway     │
  │     （你的 AI 助理中樞）    │
  └───────────┬───────────────┘
              │
              ├─ Pi agent (AI 代理)
              ├─ Skills (各種技能)
              └─ 你的工作區
```

---

## 建議資源配置

先說結論，不同部署方式的最低配置：

| 部署方式 | CPU | 記憶體 | 硬碟 | 估計月費 |
|---------|-----|-------|------|---------|
| **Linux VPS（推薦新手）** | 2 vCPU | 2GB | 20GB | ~$5-12/月 |
| **macOS VM（UTM）** | 2+ 核心 | 4GB+ | 60GB | 電費 |
| **GCP e2-small** | 2 vCPU | 2GB | 20GB | ~$12/月 |
| **GCP e2-micro** | 2 vCPU (共享) | 1GB | 20GB | 免費方案 |

**重點提醒：**
- 1GB RAM 勉強能跑，但容易 OOM（記憶體不足）
- 如果你需要 iMessage 整合，只能選 macOS
- 如果你需要瀏覽器自動化，本地 macOS 比 VPS 更適合（很多網站會擋資料中心 IP）

---

## GCP VM vs macOS：我的踩坑經驗

### 我最初的選擇：GCP Compute Engine

一開始我選了 GCP，想說雲端 VPS 24/7 運行很穩定。結果發現幾個問題：

**GCP VM 的優點：**
- 真正的 24/7 運行，不怕停電
- 付費就有，不需要額外硬體
- 適合純文字聊天場景

**GCP VM 踩到的坑：**

1. **e2-micro 記憶體太小**
   - 1GB RAM 跑 OpenClaw + Docker 真的很緊
   - 偶爾會 OOM 被 kill
   - 解法：升級到 e2-small（2GB），但月費從免費變 $12

2. **Docker 二進位檔問題**
   - 容器重啟後，運行時安裝的工具都消失
   - 必須在 Dockerfile 裡預先 bake 所有需要的二進位檔
   - 這個坑我踩了好幾次才搞懂

3. **瀏覽器自動化困難**
   - 很多網站會擋資料中心 IP
   - 需要額外設定 headless browser + proxy
   - VPS 上跑 Playwright 比本地麻煩很多

4. **技能功能受限**
   - 很多 macOS 專屬技能無法使用（imsg、memo、Peekaboo 等）
   - 如果你需要 iMessage，GCP 完全沒戲

### 最終選擇：UTM macOS 虛擬機

後來我改用 UTM 在 Apple Silicon Mac 上跑 macOS 虛擬機，效果好很多：

**macOS VM 優點：**
- 完整的 macOS 環境，所有技能都能用
- 本地運行，網路請求來自住宅 IP
- 可以用 iMessage（透過 BlueBubbles）
- 隔離環境，不影響主系統

**macOS VM 缺點：**
- 需要實體 Mac（或 Mac mini 當伺服器）
- 需要保持電腦開機
- 初始設定比 VPS 複雜一點

### 結論：怎麼選？

| 你的需求 | 推薦方案 |
|---------|---------|
| 只要 Telegram 文字聊天 | Linux VPS（便宜穩定） |
| 需要 iMessage | macOS VM（唯一選擇） |
| 需要瀏覽器自動化 | 本地 macOS + node |
| 需要 24/7 + macOS 功能 | Mac mini 或 macOS VM 長期運行 |
| 預算有限 + 簡單需求 | GCP e2-micro 免費方案 |

---

## 在 UTM macOS 虛擬機上部署

UTM 是免費的虛擬化軟體，可以在 Apple Silicon Mac 上跑 macOS 虛擬機。以下是完整步驟：

### 前置需求

- Apple Silicon Mac（M1/M2/M3/M4）
- macOS Sonoma 或更新版本
- 約 60GB 硬碟空間
- 20-30 分鐘時間

### 步驟 1：安裝 UTM

從官網下載安裝：https://mac.getutm.app/

或用 Homebrew：

```bash
brew install --cask utm
```

### 步驟 2：下載 macOS IPSW

去 Apple 官網或用 [Mr. Macintosh](https://mrmacintosh.com/apple-silicon-m1-full-macos-restore-ipsw-firmware-files-database/) 下載 IPSW 檔案。

建議下載 macOS Sonoma 或 Sequoia。

### 步驟 3：在 UTM 建立虛擬機

1. 開啟 UTM → 點「+」新增 → 選「Virtualize」
2. 選「macOS」
3. 選你下載的 IPSW 檔案
4. 配置資源：
   - CPU：2-4 核心（建議 4）
   - RAM：4-8 GB（建議 8）
   - 硬碟：60GB 以上
5. 完成並啟動

### 步驟 4：macOS 初始設定

1. 選語言、地區
2. Apple ID 可以跳過（除非你需要 iMessage）
3. 建立使用者帳號（記住帳號密碼）
4. 完成設定

### 步驟 5：啟用 SSH

1. 開啟「系統設定」→「一般」→「共享」
2. 開啟「遠端登入」（Remote Login）

記下 VM 的 IP 位址（通常是 `192.168.64.x`）

### 步驟 6：SSH 連入並安裝 OpenClaw

```bash
# 從你的主系統 SSH 進 VM
ssh youruser@192.168.64.x

# 確認有 Node.js 22+
node -v

# 如果沒有，先安裝 Homebrew 再裝 Node
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install node

# 安裝 OpenClaw
npm install -g openclaw@latest

# 執行初始設定（會引導你設定 AI 模型）
openclaw onboard --install-daemon
```

### 步驟 7：驗證安裝

```bash
# 檢查狀態
openclaw status

# 檢查服務是否運行
openclaw health
```

如果看到 Gateway 正在運行，恭喜你完成了基礎安裝！

### 步驟 8：讓 VM 背景運行

在 UTM 中，你可以讓 VM 在背景運行。建議：

1. VM 內關閉螢幕保護程式和休眠
2. Mac 主機設定「永不休眠」
3. 需要時可以隨時 SSH 進去操作

---

## AI 模型配置：省錢又好用的選擇

這是重點中的重點。AI 模型的選擇直接影響使用體驗和花費。

### 選擇優先順序（CP 值排序）

#### 1. 免費選項：OpenRouter 免費模型

OpenRouter 提供一些免費模型，適合輕度使用：

```bash
# 設定 OpenRouter
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "你的_OPENROUTER_KEY"
```

設定檔範例（`~/.openclaw/openclaw.json`）：

```json5
{
  "env": {
    "OPENROUTER_API_KEY": "sk-or-..."
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/google/gemma-2-9b-it:free"
      }
    }
  }
}
```

**免費模型推薦：**
- `openrouter/google/gemma-2-9b-it:free` - Google Gemma，品質不錯
- `openrouter/mistralai/mistral-7b-instruct:free` - Mistral，速度快

缺點：免費配額有限，高峰期可能要排隊。

##### 🆕 openrouter/free — 免費模型自動路由

OpenRouter 最近推出了 `openrouter/free` 這個特殊的路由器模型。它會自動從 OpenRouter 上所有可用的免費模型中隨機選擇一個來處理你的請求。

**特點：**
- 完全免費（input/output 都是 $0）
- 智慧選擇：根據你的請求需求（圖片理解、tool calling、structured outputs 等）自動過濾出支援該功能的免費模型
- 支援多模態輸入（text + image），最高 200k context
- 無需手動追蹤哪些模型目前免費

**設定範例：**

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/openrouter/free"
      }
    }
  }
}
```

**目前可能被路由到的免費模型包括：**
- DeepSeek 系列（當免費時）
- Kimi（Moonshot AI）
- StepFun Step 3.5 Flash
- Upstage Solar Pro 3
- LiquidAI LFM 系列
- Arcee AI Trinity
- 以及其他提供免費額度的模型

**使用建議：**
- 適合預算極低、對模型一致性要求不高的場景
- 每次請求可能路由到不同模型，回覆風格會有差異
- 如果需要穩定的模型行為，還是建議指定特定模型

#### 2. 便宜又好用：Venice AI

Venice AI 是隱私優先的 AI 服務，價格合理，模型選擇多。

```bash
# 取得 API Key
# 1. 註冊 venice.ai
# 2. 到 Settings → API Keys → 建立新金鑰

# 設定 OpenClaw
openclaw onboard --auth-choice venice-api-key
```

設定檔範例：

```json5
{
  "env": {
    "VENICE_API_KEY": "vapi_..."
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "venice/llama-3.3-70b"
      }
    }
  }
}
```

**Venice 推薦模型：**

| 用途 | 模型 | 特點 |
|-----|------|-----|
| 日常聊天 | `venice/llama-3.3-70b` | 平衡、全私密 |
| 程式開發 | `venice/qwen3-coder-480b-a35b-instruct` | 程式專用，262k 上下文 |
| 複雜推理 | `venice/deepseek-v3.2` | 強推理能力 |
| 最強品質 | `venice/claude-opus-45` | 透過 Venice 代理使用 Opus |

#### 3. 品質最佳：Anthropic Claude

如果你有 Claude Pro/Max 訂閱，可以直接用：

```bash
# 產生 setup-token
claude setup-token

# 在 OpenClaw 設定
openclaw models auth paste-token --provider anthropic
```

或用 API Key：

```json5
{
  "env": {
    "ANTHROPIC_API_KEY": "sk-ant-..."
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-5"
      }
    }
  }
}
```

#### 4. 本地免費：Ollama

如果你的 Mac 夠力（建議 16GB+ RAM），可以跑本地模型：

```bash
# 安裝 Ollama
brew install ollama

# 下載模型
ollama pull llama3.2

# 啟動服務
ollama serve
```

設定檔：

```json5
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/llama3.2"
      }
    }
  }
}
```

**本地模型優缺點：**
- 優點：完全免費、隱私、無限使用
- 缺點：需要強大硬體、品質可能不如雲端大模型

### 我的建議配置

剛開始：用 **Venice AI + Llama 3.3 70B**，便宜、隱私、品質夠用。

進階需求：搭配 **Claude Opus**（透過 Venice 代理或直接 API），處理複雜任務。

### Qwen 模型配置（特別推薦）

最近發現 Qwen 模型非常出色且實用，特別是在程式碼理解和生成方面表現突出。以下是配置 Qwen 模型的詳細步驟：

#### 步驟 1：註冊 Qwen Portal 帳號

1. 訪問 https://portal.qwen.ai
2. 使用阿里的雲帳號或創建新帳號
3. 確保帳戶有足夠的配額或訂閱服務

#### 步驟 2：取得 Qwen API Key

1. 登入 Qwen Portal 後，前往「個人設定」或「API Keys」
2. 點擊「創建新的 API Key」
3. 複製生成的 API Key 並妥善保存

#### 步驟 3：在 OpenClaw 中配置 Qwen

##### 方法一：使用 CLI 配置

```bash
# 設定 Qwen 認證
openclaw models auth --provider qwen-portal --mode apiKey
# 系統會提示您輸入 API Key
```

##### 方法二：手動編輯設定檔

編輯設定檔：

```bash
nano ~/.openclaw/openclaw.json
```

在 `auth.profiles` 中添加 Qwen 配置：

```json5
{
  "auth": {
    "profiles": {
      "qwen-portal:default": {
        "provider": "qwen-portal",
        "mode": "api-key",
        "apiKey": "your-qwen-api-key-here"
      }
    }
  },
  "models": {
    "providers": {
      "qwen-portal": {
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "apiKey": "qwen-api-key",
        "api": "openai-completions",
        "models": [
          {
            "id": "qwen-max-latest",
            "name": "Qwen Max",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0.005,
              "output": 0.015,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 32768,
            "maxTokens": 8192
          },
          {
            "id": "qwen-plus-latest",
            "name": "Qwen Plus",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0.002,
              "output": 0.006,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 128000,
            "maxTokens": 8192
          },
          {
            "id": "qwen-turbo-latest",
            "name": "Qwen Turbo",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0.0003,
              "output": 0.0006,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 32768,
            "maxTokens": 8192
          },
          {
            "id": "qwen-coder-plus",
            "name": "Qwen Coder Plus",
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0.002,
              "output": 0.006,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 128000,
            "maxTokens": 8192
          },
          {
            "id": "qwen-vl-max",
            "name": "Qwen VL Max",
            "reasoning": true,
            "input": ["text", "image"],
            "cost": {
              "input": 0.005,
              "output": 0.015,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 32768,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "qwen-portal/qwen-plus-latest"
      },
      "models": {
        "qwen-portal/qwen-max-latest": {"alias": "qwen-max"},
        "qwen-portal/qwen-plus-latest": {"alias": "qwen-plus"},
        "qwen-portal/qwen-turbo-latest": {"alias": "qwen-turbo"},
        "qwen-portal/qwen-coder-plus": {"alias": "qwen-coder"},
        "qwen-portal/qwen-vl-max": {"alias": "qwen-vl"}
      }
    }
  }
}
```

#### 步驟 4：Qwen 模型選擇指南

| 用途 | 推薦模型 | 特點 |
|-----|---------|------|
| 日常聊天 | `qwen-portal/qwen-plus-latest` | 平衡效能與成本 |
| 程式開發 | `qwen-portal/qwen-coder-plus` | 專門優化的程式碼模型 |
| 快速回應 | `qwen-portal/qwen-turbo-latest` | 速度最快，適合快速查詢 |
| 複雜推理 | `qwen-portal/qwen-max-latest` | 最強大的推理能力 |
| 圖片分析 | `qwen-portal/qwen-vl-max` | 支援視覺理解 |

#### 步驟 5：測試 Qwen 配置

配置完成後，重啟 OpenClaw：

```bash
openclaw gateway restart
```

然後可以使用以下命令測試：

```bash
# 查看可用模型
openclaw models list

# 切換到 Qwen 模型
/model qwen-plus

# 或使用別名
/model qwen-coder  # 如果你要做程式開發
```

#### Qwen 模型優勢

1. **中文支援極佳**：阿里雲在中文 NLP 方面投入巨大，Qwen 的中文理解能力非常出色
2. **程式碼能力強**：Qwen Coder 系列在程式碼生成和理解方面表現卓越
3. **性價比高**：相比其他大型模型，Qwen 的價格相當具有競爭力
4. **持續更新**：阿里雲團隊持續優化模型，經常推出新版
5. **豐富的功能**：支援視覺理解、長文本處理等多種能力

#### 注意事項

- 確保您的 Qwen 帳戶有足夠的信用額度
- 定期監控用量，避免超出預算
- Qwen 模型在處理中文內容時表現最佳，對於繁體中文支援也很好
- 如果遇到 API 限制問題，可以聯繫阿里雲客服調整配額

使用 Qwen 模型作為主要或備用模型，可以顯著提升中文對話體驗，特別是在程式碼開發和技術問題解答方面。

---

## Telegram 配對完整教學

終於到了連接 Telegram 的部分。這是讓你隨時隨地與 AI 聊天的關鍵。

### 步驟 1：建立 Telegram Bot

1. 開啟 Telegram，搜尋 `@BotFather`
2. 發送 `/newbot`
3. 輸入 Bot 名稱（例如：My OpenClaw Bot）
4. 輸入 Bot 使用者名稱（必須以 `bot` 結尾，例如：`my_openclaw_bot`）
5. BotFather 會給你一個 Token，類似：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`

**保管好這個 Token！**

### 步驟 2：設定 OpenClaw

編輯設定檔：

```bash
nano ~/.openclaw/openclaw.json
```

加入 Telegram 設定：

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456789:ABCdefGHIjklMNOpqrsTUVwxyz",
      "dmPolicy": "pairing"  // 需要配對才能使用
    }
  }
}
```

重啟 Gateway：

```bash
openclaw gateway restart
```

### 步驟 3：配對你的 Telegram

1. 在 Telegram 中，找到你剛建立的 Bot
2. 發送任何訊息（例如 `hello`）
3. Bot 會回覆配對碼，類似：
   ```
   OpenClaw: access not configured.
   Your Telegram user id: 8444190675
   Pairing code: JYWKCCMM
   ```

4. 在 OpenClaw 終端機執行：
   ```bash
   openclaw pairing approve telegram JYWKCCMM
   ```

5. 完成！現在可以開始聊天了

### 步驟 4：群組設定（可選）

如果你想讓 Bot 在群組中回應：

```json5
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "你的_TOKEN",
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["你的_USER_ID"],
      "groups": {
        "*": {
          "requireMention": true  // 需要 @mention 才回應
        }
      }
    }
  }
}
```

**如何取得你的 Telegram User ID：**
- 最簡單：私訊你的 Bot，配對訊息裡就有
- 或者：私訊 `@userinfobot`

### 步驟 5：進階設定

**取消 Privacy Mode（讓 Bot 看到所有群組訊息）：**

1. 找 `@BotFather`
2. 發送 `/setprivacy`
3. 選擇你的 Bot
4. 選擇 `Disable`
5. **重要：** 把 Bot 從群組移除再重新加入，設定才會生效

**常用指令：**
- `/status` - 查看狀態
- `/reset` - 重置對話
- `/model` - 查看/切換模型
- `/reasoning` - 開關推理模式

---

## 常見問題排解

### Q: Gateway 沒有回應

```bash
# 檢查狀態
openclaw status

# 看 log
openclaw logs --follow
```

常見原因：
- 沒有設定 AI 模型
- API Key 無效或過期
- 網路問題

### Q: Telegram Bot 不回訊息

1. 確認 Bot Token 正確
2. 確認已經配對成功
3. 檢查 `dmPolicy` 設定

```bash
# 查看頻道狀態
openclaw channels status
```

### Q: 記憶體不足（OOM）

如果是 VM 或 VPS：
- 增加 RAM（至少 2GB）
- 減少同時運行的服務

如果是本地：
- 關閉不必要的應用程式
- 不要用太大的本地模型

### Q: 如何更新 OpenClaw

```bash
# npm 安裝
npm update -g openclaw

# 重啟服務
openclaw gateway restart
```

---

## 結語

部署 OpenClaw 確實有學習曲線，但一旦設定好，你就有了一個隨時待命的 AI 助理。

我的建議：
1. **新手**：從 Linux VPS 開始，便宜簡單
2. **需要完整功能**：用 macOS VM 或本地 Mac
3. **模型選擇**：Venice AI 是目前 CP 值最高的選項

有問題可以到 [OpenClaw Discord](https://discord.com/invite/clawd) 發問，社群很活躍。

Happy hacking！

---

*最後更新：2026-02-04*

*參考資料：*
- [OpenClaw 官方文件](https://docs.openclaw.ai)
- [GitHub](https://github.com/openclaw/openclaw)
