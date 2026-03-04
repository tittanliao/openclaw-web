# OpenClaw 黃金交易自動化計畫

## 📋 專案概述

透過 OpenClaw 平台，建構 XAUUSD 黃金交易的自動化分析與通知系統，取代現有手動工作流程。

---

## 🖥️ 現況環境

| 項目 | 說明 |
|------|------|
| **硬體** | Mac Mini M4 |
| **平台** | OpenClaw（已安裝並完成 onboard） |
| **Domain** | aholic.cc（Cloudflare DNS 代管） |
| **子網域 - Agent** | agent.aholic.cc → OpenClaw 服務 |
| **子網域 - SSH** | ssh.aholic.cc → SSH 遠端連線 |
| **LLM** | Gemini API（限 Flash 版本），已綁信用卡，90 天 $300 USD 免費額度 |
| **通訊 Channel** | LINE Messaging API（已設定，手機可正常對話） |
| **Web 登入** | Token 登入，新裝置需 SSH 進後台 approve |
| **瀏覽器** | Chrome + OpenClaw 外掛（127.0.0.1），可擷取網站資訊、另存圖檔至 local |
| **System Prompt** | 已透過 Gemini GEM 優化提示詞，作為 OpenClaw 的 system prompt |
| **TradingView** | 已付費 Plus 方案，支援 Webhook Alert |

---

## 🏗️ 整體架構

```
LINE (手機) ←→ OpenClaw ←→ Gemini Flash (LLM)
                  ↓
    ┌─────────────┼─────────────────┐
    ↓             ↓                 ↓
  Brave Search   Chrome (127.0.0.1)  Webhook Listener
  (外部搜尋)     (TradingView擷取)   (TV Alert 接收)
                  ↓
            Local Files (截圖/資料)
```

---

## 🔧 第一階段：基礎能力補齊 — Brave Search 導入

### 目的

讓 OpenClaw 具備外部搜尋能力，為後續所有函式的前置條件。

### 作法

1. 申請 Brave Search API Key
2. 在 OpenClaw 中設定 Brave Search 作為 Tool
3. 驗證 OpenClaw 可透過 LINE 指令觸發搜尋並回傳結果

### 執行細項

- [ ] 前往 [Brave Search API](https://brave.com/search/api/) 註冊帳號
- [ ] 選擇方案（Free 方案：每月 2,000 次查詢，應足夠初期使用）
- [ ] 取得 API Key
- [ ] 在 OpenClaw 後台設定 Brave Search Tool（確認 OpenClaw 的 tool 設定格式）
- [ ] 透過 LINE 測試：「搜尋 XAUUSD 最新消息」驗證搜尋函式
- [ ] 測試搜尋 CFTC 報告連結是否可正確取得

### 推薦 API 廠商

| 廠商 | 方案 | 價格 | 說明 |
|------|------|------|------|
| **Brave Search API**（推薦） | Free | 2,000 次/月免費 | OpenClaw 原生支援，整合最簡單 |
| Brave Search API | Base | $3/1000 次 | 超出免費額度後的付費方案 |
| SerpAPI | 備選 | 100 次/月免費 | 支援 Google Search，但整合較複雜 |
| Tavily | 備選 | 1,000 次/月免費 | 專為 AI Agent 設計的搜尋 API |

---

## 📸 第二階段：TradingView 截圖回傳（目標 1）

### 目的

透過手機 LINE 對話，請 OpenClaw 開啟 TradingView 的 XAUUSD 30 分鐘圖，截圖回傳至 LINE。

### 作法

1. 定義 LINE 指令格式（如：「截圖 XAUUSD 30m」）
2. OpenClaw 透過 Chrome 外掛開啟 TradingView
3. 導航至 XAUUSD 並切換時間週期
4. 截圖儲存至 local file
5. 透過 LINE Messaging API 回傳圖片

### 執行細項

- [ ] 確認 OpenClaw Chrome 外掛是否可操作 TradingView 的 SPA 頁面
- [ ] 確認 TradingView 登入狀態是否能在 Chrome 中持久保持
- [ ] 測試 TradingView URL 格式：`https://www.tradingview.com/chart/?symbol=XAUUSD&interval=30`
- [ ] 定義截圖操作步驟（OpenClaw 的 browser tool 指令序列）：
  1. 開啟 TradingView chart 頁面
  2. 搜尋商品 XAUUSD
  3. 切換時間週期至 30 分鐘
  4. 等待圖表完全載入
  5. 全螢幕截圖或指定區域截圖
  6. 儲存至 local file
- [ ] 確認 OpenClaw LINE channel 是否支援 image message 回傳
- [ ] 若不支援直接圖片回傳，研究替代方案：
  - 上傳圖片至暫存空間（如 Cloudflare R2 / Imgur）
  - 產生臨時連結回傳 LINE
- [ ] 測試端到端流程：LINE 指令 → 截圖 → 回傳

### 推薦 API / 服務

| 服務 | 用途 | 說明 |
|------|------|------|
| **TradingView（現有 Plus）** | 圖表來源 | 已有付費，直接使用 |
| **Cloudflare R2** | 圖片暫存 | 已有 Cloudflare 帳號，免費額度 10GB/月 |
| **Imgur API** | 圖片暫存備選 | 免費，匿名上傳 |
| **LINE Messaging API（現有）** | 圖片回傳 | 支援 image message type |

---

## 📊 第三階段：週報自動化（目標 2）

### 目的

取代每週日手動製作 XAUUSD 週報的工作流程，由 OpenClaw 自動完成資料收集、策略分析、報告產生。

### 現有手動流程（待自動化）

```
1. 手動從 TradingView 匯出 XAUUSD 各時段價格 Excel
2. 手動查詢 CFTC 持倉報告（籌碼狀況）
3. 將資料輸入 Gemini GEM「黃金交易助手」
4. 根據兩個既有策略進行下周策略分析
5. 手動操作 TradingView + Gemini 產生週報
```

### 作法

建構三個模組，由 OpenClaw 協調執行：

#### 模組 A：資料取得

- TradingView XAUUSD 多時段價格擷取（日線、4H、1H、30m）
- CFTC Commitments of Traders 報告資料擷取
- 相關市場資料（DXY 美元指數、美債殖利率等，如策略需要）

#### 模組 B：策略分析

- 將 Gemini GEM「黃金交易助手」的角色定義遷移至 OpenClaw
- 結構化描述兩個既有策略的邏輯與參數
- Gemini Flash 進行多維度分析

#### 模組 C：週報產生與發送

- 定義週報輸出格式模板
- 產生 Markdown 或 HTML 格式週報
- 透過 LINE 推送週報摘要 + 完整報告連結

### 執行細項

#### 資料取得 — TradingView 價格

- [ ] 研究 TradingView Export Chart Data 函式的自動化操作
- [ ] 備選方案：使用第三方 API 取得 XAUUSD 歷史價格
- [ ] 定義需要的時段與 K 線數量
- [ ] 確認資料輸出格式（JSON / CSV）

#### 資料取得 — CFTC 持倉報告

- [ ] CFTC 官網資料來源：`https://www.cftc.gov/dea/futures/deacmxsf.htm`
- [ ] 或使用 Quandl / Nasdaq Data Link API 取得結構化資料
- [ ] 定義需要的持倉欄位（Large Speculators / Commercial / Small Speculators）
- [ ] Brave Search 輔助確認最新報告日期

#### 策略遷移

- [ ] 匯出 Gemini GEM「黃金交易助手」的完整提示詞
- [ ] 匯出兩個策略的完整邏輯描述
- [ ] 轉化為 OpenClaw 可用的 prompt template 或 tool 定義
- [ ] 驗證 Gemini Flash 的分析品質是否足夠（與原 GEM 輸出比較）

#### 週報模板

- [ ] 定義週報結構：
  1. 本週回顧（價格走勢摘要）
  2. CFTC 持倉變化分析
  3. 策略一分析結果
  4. 策略二分析結果
  5. 下周操作建議（多空方向、關鍵價位、進出場條件）
- [ ] 建立 Markdown 模板

#### 排程或觸發方式

- [ ] 方案 A：LINE 手動指令觸發（如：「產生本周週報」）
- [ ] 方案 B：Mac Mini 本機 cron job 排程（每週日固定時間執行）
- [ ] 方案 C：OpenClaw 內建排程函式（確認是否支援）

#### Gemini Flash Token 消耗估算

- [ ] 估算單次週報分析的 input / output token 數量
- [ ] 確認 $300 / 90 天額度是否足夠（週報 + 後續即時分析）

### 推薦 API 廠商

| 廠商 | 用途 | 價格 | 說明 |
|------|------|------|------|
| **Nasdaq Data Link（原 Quandl）** | CFTC 持倉資料 | 免費（基本） | 提供結構化 CFTC COT 資料，API 友善 |
| **Alpha Vantage** | XAUUSD 價格備選 | 免費 25 次/日 | 提供外匯 / 貴金屬價格 API |
| **Twelve Data** | XAUUSD 價格備選 | 免費 800 次/日 | 支援多時段 K 線資料 |
| **Yahoo Finance API（yfinance）** | 價格資料備選 | 免費 | Python 函式庫，資料穩定度中等 |
| **Metals-API** | 貴金屬專用 | 免費 50 次/月 | 專門提供貴金屬即時與歷史價格 |
| **TradingView（現有 Plus）** | 圖表 + 匯出 | 已付費 | 直接從 Chrome 擷取 |

---

## ⚡ 第四階段：Webhook 即時交易訊號（目標 3）

### 目的

TradingView 觸發 Alert 時，透過 Webhook 通知 OpenClaw，OpenClaw 綜合分析後透過 LINE 回傳是否進行交易 ENTRY。

### 作法

```
TradingView Alert (Webhook POST)
        ↓
   agent.aholic.cc/webhook（OpenClaw 接收）
        ↓
   OpenClaw 綜合分析：
   ├── 讀取上周週報結論
   ├── Chrome 擷取當前 XAUUSD K 線截圖
   ├── Brave Search 社群最新討論
   └── Gemini Flash 綜合判斷
        ↓
   LINE 推送交易建議：
   ├── 是否 ENTRY
   ├── 方向（多/空）
   ├── 建議進場價位
   ├── 止損 / 止盈位置
   └── 信心度評分
```

### 執行細項

#### Webhook 接收端設定

- [ ] 確認 OpenClaw 是否原生支援自定義 Webhook endpoint
- [ ] 若不支援，研究替代方案：
  - 在 Mac Mini 上建構輕量 Webhook listener（Node.js / Python Flask）
  - 收到後轉發至 OpenClaw
- [ ] Cloudflare DNS 確認 agent.aholic.cc 路由正確
- [ ] 設定 HTTPS（Cloudflare 應已處理）

#### TradingView Alert 設定

- [ ] 在 TradingView Plus 中建立 XAUUSD Alert
- [ ] Webhook URL 設定為 `https://agent.aholic.cc/webhook`（或 OpenClaw 指定路徑）
- [ ] 定義 Alert Payload 格式（JSON）：
  ```json
  {
    "ticker": "XAUUSD",
    "action": "{{strategy.order.action}}",
    "price": "{{close}}",
    "time": "{{time}}",
    "interval": "{{interval}}",
    "alert_name": "{{alertName}}"
  }
  ```
- [ ] 測試 Webhook 是否成功送達

#### 分析流程定義

- [ ] 定義 OpenClaw 收到 Webhook 後的分析 SOP
- [ ] 建構上周週報的儲存與讀取機制（local file 或資料庫）
- [ ] 定義社群搜尋關鍵字（如："XAUUSD analysis today", "gold trading signal"）
- [ ] 定義 Gemini Flash 的分析 prompt template

#### LINE 推送格式

- [ ] 設計交易建議的 LINE 訊息格式（含 Flex Message 或純文字）
- [ ] 包含「確認執行」與「忽略」的互動按鈕（若 LINE Messaging API 支援）

#### 延遲與可靠性

- [ ] 估算從 Webhook 到 LINE 推送的端到端延遲
- [ ] 設定 timeout 機制（避免 Chrome 截圖卡住）
- [ ] 設定失敗重試機制

#### 風險管理

- [ ] 加入「人工確認」環節，不要全自動下單
- [ ] 設定每日最大 alert 次數限制
- [ ] 記錄所有交易建議的 log

### 推薦 API / 服務

| 服務 | 用途 | 說明 |
|------|------|------|
| **TradingView Webhook（現有 Plus）** | Alert 發送 | Plus 方案已包含 |
| **Cloudflare Tunnel（現有）** | 安全暴露 Webhook endpoint | 已有 agent.aholic.cc |
| **LINE Messaging API（現有）** | 推送通知 | 支援 push message + flex message |
| **Redis（可選）** | 快取週報結論 | 本地安裝，加速讀取 |

---

## 🔬 第五階段：策略回測系統（目標 4）

### 目的

抓取 TradingView 熱門策略（Pine Script），轉換為 Python 版本，利用歷史價格資料進行回測，定期評估可行策略。

### 作法

1. 透過 Brave Search / Chrome 擷取 TradingView 熱門 Pine Script 策略
2. 利用 Gemini Flash 將 Pine Script 轉譯為 Python 程式碼
3. 透過 TradingView Export 或第三方 API 取得歷史價格資料
4. 使用 Python 回測框架執行回測
5. 產生回測報告，透過 LINE 推送結果

### 執行細項

#### 策略擷取

- [ ] 定義 TradingView 熱門策略的篩選條件（商品、評分、使用人數）
- [ ] Chrome 擷取 Pine Script 原始程式碼
- [ ] 建立策略庫管理機制

#### Pine Script → Python 轉譯

- [ ] 設計 Gemini Flash 的轉譯 prompt（含常見 Pine Script 函式對照表）
- [ ] 驗證轉譯品質（人工審查前幾個策略）
- [ ] 建構常用技術指標的 Python 函式庫對照

#### 歷史資料取得

- [ ] TradingView Export Chart Data（Chrome 自動化）
- [ ] 或第三方 API 取得（見下方推薦）
- [ ] 定義回測所需的資料範圍（如：近 2 年、多時段）

#### 回測框架建構

- [ ] 選擇 Python 回測函式庫
- [ ] 在 Mac Mini 上設定 Python 環境
- [ ] 定義回測指標：勝率、盈虧比、最大回撤、Sharpe Ratio
- [ ] 建立標準化回測報告模板

#### 定期執行

- [ ] 排程（每月 / 每兩周）自動抓取新策略並回測
- [ ] 透過 LINE 推送值得關注的策略回測結果

### 推薦 API / 函式庫

| 名稱 | 類型 | 說明 |
|------|------|------|
| **backtesting.py** | Python 回測框架 | 輕量、易用，適合單策略回測 |
| **vectorbt** | Python 回測框架 | 高效能，支援批量回測，適合策略篩選 |
| **TA-Lib（Python）** | 技術指標計算 | 業界標準技術指標函式庫 |
| **pandas-ta** | 技術指標計算備選 | 純 Python 實作，安裝更簡單 |
| **Twelve Data** | 歷史價格 API | 免費 800 次/日，支援多時段 |
| **Polygon.io** | 歷史價格 API | 免費方案支援外匯，資料品質高 |
| **OANDA API** | 歷史價格 + 交易 | 提供免費模擬帳號 API，XAUUSD 資料完整 |

---

## 💡 第六階段：收入擴展發想（目標 5）

> 待前四階段穩定後展開，以下為初步方向

### 可能方向

| 方向 | 說明 | 可行性 |
|------|------|--------|
| **交易訊號訂閱服務** | 將 OpenClaw 產生的交易訊號透過 LINE 群組 / Telegram 提供付費訂閱 | ⭐⭐⭐ |
| **週報訂閱服務** | 將自動化週報包裝為付費內容 | ⭐⭐⭐ |
| **策略回測即服務** | 幫其他交易者回測策略，收取服務費 | ⭐⭐ |
| **OpenClaw 自動化顧問** | 協助其他交易者建構類似系統 | ⭐⭐⭐ |
| **Pine Script → Python 轉譯服務** | 專門提供策略轉譯服務 | ⭐⭐ |
| **交易日誌自動化工具** | 結合 OpenClaw 自動記錄交易並產生績效分析 | ⭐⭐⭐ |
| **多商品擴展** | 將系統擴展至其他商品（白銀、原油、外匯） | ⭐⭐⭐ |

---

## ⚠️ 風險與注意事項

### 技術風險

| 風險 | 影響 | 緩解方案 |
|------|------|----------|
| Gemini Flash 分析品質不足 | 週報 / 交易建議品質下降 | 保留升級 Gemini Pro 的彈性，或對特定任務使用更強模型 |
| TradingView DOM 變更 | Chrome 自動化失效 | 建構多種資料取得管道（API 備選） |
| Mac Mini 休眠 / 斷電 | 服務中斷 | 設定禁止休眠、UPS 不斷電系統 |
| Cloudflare Tunnel 中斷 | Webhook 無法接收 | 設定健康檢查 + 自動重連 |
| 免費額度用盡 | API 呼叫失敗 | 監控用量，提前規劃付費方案 |

### 合規風險

- 交易訊號服務可能涉及金融顧問法規，需確認當地法規要求
- TradingView 策略爬取需注意其服務條款
- 自動化操作 TradingView 可能違反其 ToS，建議以 API 取代

### 成本估算（月度）

| 項目 | 估計成本 |
|------|----------|
| Gemini Flash API（$300/90天免費後） | ~$10-30/月 |
| Brave Search API（超出免費額度） | ~$0-10/月 |
| TradingView Plus（現有） | 已付費 |
| Cloudflare（現有 Free Plan） | $0 |
| 第三方價格 API（Free Tier） | $0 |
| **預估總計** | **$10-40/月** |

---

## 📅 時程建議

| 階段 | 預估時間 | 前置條件 |
|------|----------|----------|
| 第一階段：Brave Search | 1-2 天 | 無 |
| 第二階段：TradingView 截圖 | 3-5 天 | 第一階段完成 |
| 第三階段：週報自動化 | 1-2 周 | 第二階段完成 |
| 第四階段：Webhook 即時訊號 | 1-2 周 | 第三階段完成 |
| 第五階段：策略回測 | 2-3 周 | 第二階段完成（可與第四階段並行） |
| 第六階段：收入擴展 | 持續進行 | 前四階段穩定 |

---

## 📂 專案檔案結構（建議）

```
openclaw/
├── XAUUSD_AUTOMATION_PLAN.md          ← 本文件（主計畫）
├── prompts/
│   ├── gold_assistant_system.md       ← 黃金交易助手 system prompt
│   ├── weekly_report_template.md      ← 週報模板
│   ├── entry_analysis_template.md     ← 交易 ENTRY 分析 prompt
│   └── pine_to_python_template.md     ← Pine Script 轉譯 prompt
├── strategies/
│   ├── strategy_01.md                 ← 策略一定義
│   └── strategy_02.md                 ← 策略二定義
├── reports/
│   └── weekly/                        ← 歷史週報存放
├── backtest/
│   ├── scripts/                       ← 回測 Python 程式碼
│   └── results/                       ← 回測結果
└── config/
    ├── webhook_payload.json           ← TradingView Webhook 格式定義
    └── api_keys.env                   ← API 金鑰（加入 .gitignore）
```

---

## 🔗 相關連結

### 核心服務

- [OpenClaw 文件](https://docs.openclaw.ai/)
- [Brave Search API](https://brave.com/search/api/)
- [LINE Messaging API 文件](https://developers.line.biz/en/docs/messaging-api/)
- [Gemini API 文件](https://ai.google.dev/docs)
- [Cloudflare Tunnel 文件](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)

### 交易資料

- [TradingView Webhooks 文件](https://www.tradingview.com/support/solutions/43000529348/)
- [CFTC Commitments of Traders](https://www.cftc.gov/MarketReports/CommitmentsofTraders/index.htm)
- [Nasdaq Data Link (Quandl) CFTC](https://data.nasdaq.com/data/CFTC)
- [Twelve Data API](https://twelvedata.com/)
- [OANDA API](https://developer.oanda.com/)
- [Polygon.io](https://polygon.io/)
- [Alpha Vantage](https://www.alphavantage.co/)
- [Metals-API](https://metals-api.com/)

### 回測工具

- [backtesting.py](https://kernc.github.io/backtesting.py/)
- [vectorbt](https://vectorbt.dev/)
- [TA-Lib Python](https://ta-lib.github.io/ta-lib-python/)
- [pandas-ta](https://github.com/twopirllc/pandas-ta)

### 圖片儲存

- [Cloudflare R2](https://developers.cloudflare.com/r2/)
- [Imgur API](https://apidocs.imgur.com/)

---

## 📝 使用說明

本文件供 OpenClaw 作為討論與實作的參考文件。建議使用方式：

1. **存檔至專案目錄**：讓 OpenClaw 可透過 local file 讀取
2. **逐階段討論**：每次與 OpenClaw 對話時，指定討論特定階段
3. **勾選進度**：完成項目後更新 checkbox，追蹤進度

### 與 OpenClaw 對話範例

```
「請參考 XAUUSD_AUTOMATION_PLAN.md，我們先進行第一階段 Brave Search 導入，
 請引導我完成所有執行細項。」

「請參考 XAUUSD_AUTOMATION_PLAN.md 第三階段，
 幫我建立 prompts/weekly_report_template.md 的週報模板。」

「請參考 XAUUSD_AUTOMATION_PLAN.md 的專案檔案結構，
 幫我建立所有需要的資料夾與初始檔案。」
```

---

> 最後更新：2026-03-03
> 版本：v1.0