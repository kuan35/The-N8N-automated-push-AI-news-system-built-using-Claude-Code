#  N8N Automated AI News Digest System

> 每天自動抓取 AI 自動化與 AI 語音最新新聞，由 GPT-4o 摘要整理後，以精美黑白 HTML 電子報寄送至你的信箱。
>
> Built with [Claude Code](https://claude.ai/code) + n8n + Tavily + OpenAI

---

##  功能介紹

- **自動排程**：每天早上 9:00（台灣時間 UTC+8）自動執行
- **雙主題搜尋**：同時搜尋「AI 自動化」與「AI 語音」兩大主題的最新新聞
- **AI 摘要**：使用 GPT-4o 將新聞整理為繁體中文摘要
- **精美電子報**：黑白極簡風格 HTML，適合各主流郵件客戶端
- **自動寄信**：透過 Gmail SMTP 發送至指定信箱

---

##  工作流程架構

```
每日 09:00 AM (UTC+8)
        │
        ├──► Tavily 搜尋「AI 自動化」新聞 ──┐
        │                                    ├──► Merge 合併結果
        └──► Tavily 搜尋「AI 語音」新聞 ────┘
                                                    │
                                             整理文章資料 (JS)
                                             去重 + 組建請求
                                                    │
                                          GPT-4o 生成繁中摘要
                                                    │
                                          生成黑白 HTML 電子報
                                                    │
                                          Gmail SMTP 寄送信箱
```

---

##  建置教學

### 前置需求

| 工具 | 說明 | 取得方式 |
|------|------|---------|
| n8n | 工作流程自動化平台 | [n8n.io](https://n8n.io) |
| Tavily API Key | AI 新聞搜尋 | [app.tavily.com](https://app.tavily.com) |
| OpenAI API Key | GPT-4o 摘要生成 | [platform.openai.com](https://platform.openai.com) |
| Gmail 帳號 | 電子報寄送 | 需開啟兩步驟驗證 |

---

### Step 1：安裝並啟動 n8n

**方式一：使用 npx（最快速）**
```bash
npx n8n
```

**方式二：使用 Docker**
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

啟動後前往 `http://localhost:5678` 完成帳號設定。

---

### Step 2：取得 API Keys

#### Tavily API Key（免費）
1. 前往 [app.tavily.com](https://app.tavily.com) 註冊帳號
2. 登入後點選左側 **API Keys**
3. 點 **+ New API Key**，輸入名稱後複製
4. 格式：`tvly-xxxxxxxxxxxxxxxx`
5. 免費方案：每月 **1,000 次**搜尋

#### OpenAI API Key
1. 前往 [platform.openai.com](https://platform.openai.com) 登入
2. 右上角 → **API Keys** → **+ Create new secret key**
3. 複製 Key（格式：`sk-proj-xxxxxxxx`）
4. 確認帳號有足夠 Credits（GPT-4o 每次約 $0.01–0.05）

---

### Step 3：設定 Gmail 應用程式密碼

> Gmail SMTP 需要「應用程式密碼」，而非你的登入密碼

1. 前往 [myaccount.google.com](https://myaccount.google.com) → **安全性**
2. 開啟**兩步驟驗證**（必須先開啟）
3. 返回安全性頁面，搜尋**應用程式密碼**
4. 輸入名稱（如 `n8n`）→ 點**建立**
5. 複製產生的 **16 位數密碼**（格式：`xxxx xxxx xxxx xxxx`）

---

### Step 4：在 n8n 建立 Gmail SMTP Credential

1. 開啟 n8n → 左側選單 **Credentials**
2. 點右上角 **+ Add Credential** → 搜尋 **SMTP**
3. 填入以下資訊：

| 欄位 | 值 |
|------|-----|
| Host | `smtp.gmail.com` |
| Port | `465` |
| SSL/TLS | 開啟 ✅ |
| User | 你的 Gmail（`yourname@gmail.com`）|
| Password | Step 3 取得的 16 位應用程式密碼 |

4. 點 **Save** 儲存，記下 Credential 名稱

---

### Step 5：匯入 Workflow

1. 開啟 n8n → 左上角 **+** → **Import from file**
2. 選擇本專案的 `ai-news-digest.json`
3. Workflow 自動匯入，名稱為「AI 新聞日報」

---

### Step 6：設定 API Keys

開啟匯入的 Workflow，依序設定以下節點：

#### 搜尋 AI 自動化新聞 & 搜尋 AI 語音新聞
- 點開節點 → 找到 **Body** 欄位
- 將 `api_key` 值換成你的 Tavily API Key

#### GPT-4 生成摘要
- 點開節點 → **Headers** → **Authorization**
- 確認值為 `Bearer YOUR_OPENAI_API_KEY`

#### 寄送郵件
- **Credential to connect with** → 選擇 Step 4 建立的 SMTP Credential
- **From Email** → 填入你的 Gmail
- **To Email** → 填入收件信箱（可填同一個 Gmail）

---

### Step 7：測試與啟用

1. 點右下角 **Execute Workflow** 手動測試一次
2. 確認信箱收到 AI 新聞電子報
3. 測試成功後，點右上角 **Publish** 啟用排程

---

##  電子報預覽

```
┌─────────────────────────────────────────┐
│  DAILY DIGEST                           │
│  AI 新聞日報                             │
│  2026/03/15  ·  過去 24 小時精選         │
└─────────────────────────────────────────┘

▌ AI 自動化
┌─────────────────────────────────────────┐
│ 文章標題（可點擊連結）                    │
│ GPT-4o 生成的 2-3 行繁體中文摘要...       │
│ 閱讀完整報導 →                           │
└─────────────────────────────────────────┘

▌ AI 語音
┌─────────────────────────────────────────┐
│ 文章標題（可點擊連結）                    │
│ GPT-4o 生成的 2-3 行繁體中文摘要...       │
│ 閱讀完整報導 →                           │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  TODAY'S INSIGHT                        │
│  今日重點觀察                            │
│  GPT-4o 綜合分析（100字以內）...          │
└─────────────────────────────────────────┘
```

---

## ⚙️ 自訂設定

### 修改排程時間
點開「每日排程 9AM」節點 → 修改 Cron 表達式：
```
0 1 * * *   → 每天 09:00 台灣時間（UTC+8 = UTC+1）
0 23 * * *  → 每天 07:00 台灣時間
0 10 * * *  → 每天 18:00 台灣時間
```

### 修改搜尋關鍵字
點開 Tavily 節點 → 修改 `query` 欄位，例如：
- `"AI agent latest news"` → AI Agent 相關新聞
- `"LLM benchmark 2026"` → 大型語言模型最新評測

### 修改搜尋筆數
修改 Tavily 節點的 `max_results`（預設 8，最大 20）

---

## 📦 專案結構

```
n8n-demo/
├── ai-news-digest.json   # n8n workflow 定義檔
├── .env.example          # API Key 設定範本
├── .gitignore            # 排除敏感檔案
├── CLAUDE.md             # Claude Code 專案說明
└── README.md             # 本文件
```

---

##  使用技術

| 技術 | 用途 |
|------|------|
| [n8n](https://n8n.io) | 工作流程自動化平台 |
| [Tavily API](https://tavily.com) | 即時 AI 新聞搜尋 |
| [OpenAI GPT-4o](https://openai.com) | 新聞摘要與分析 |
| Gmail SMTP | 電子報寄送 |

---
