---
title: Claude Code 懶人包 - 重點整理
source: https://github.com/mathruffian-dot/claude-code-lazy-packs
author: 三師爸（mathruffian-dot）
license: MIT
created: 2026-10-08
type: clipping
tags:
  - clippings
  - ClaudeCode
  - Obsidian
  - 第二大腦
  - 懶人包
---

# Claude Code 懶人包 — 重點整理

> [!summary] 一句話
> 每份懶人包是一個 Markdown 檔，**丟給 Claude Code 桌面版，它就照著步驟自動安裝設定**，遇到要手動操作的地方會暫停告訴你。
> 原本是寫給**老師**的系列（Claude 基本功 EP01–EP16），佳盈的簡報推薦用其中的 **#03 接 Obsidian → #04 建系統**。
> 相關：[[AI 剪片＆第二大腦系統 - 重點整理]]

## 📦 懶人包清單
| 編號 | 名稱 | 做什麼 | 跟我有關嗎？ |
| --- | --- | --- | --- |
| 00 | 環境建置 | 安裝 uv 等基礎工具 | 可略過，其他包會自動補裝 |
| 01 | 連接 NotebookLM | 讓 Claude 操控 NotebookLM 產簡報、資訊圖表 | 選配 |
| 02 | 連接 GitHub | 建 repo、把網頁推上線（GitHub Pages） | 之後自己做網頁時才需要 |
| **03** | **建立第二大腦 Obsidian** | 安裝 Obsidian、雲端同步、讓 Claude 能讀寫筆記 | ⭐ **核心** |
| **03+** | **第二大腦設定指南** | 三層資料夾、CLAUDE.md、模板、每週知識重整排程 | ⭐ **核心** |
| 04 | 連接 Supabase | 雲端資料庫（SQL） | 做下單網站時才考慮 |
| 04.5 | 連接 Firebase | 雲端資料庫，作者認為比 Supabase 適合新手 | 做下單網站時才考慮 |
| 05 | 本地 AI Ollama | 在電腦跑免費 AI | 不需要 |
| 06 | Gemini 免費 API | 讓自己做的工具有 AI 能力 | 不需要 |
| 07 | 班級工具工作模式 | 老師專用的專案模式、`/收工` 指令 | 概念可參考，內容是給老師的 |
| 08 | gpt-image-2 生圖 | 用 OpenAI 付費 API 生圖 | 不需要（我是手繪） |

> 注意：GitHub 上的檔名是 `04-第二大腦設定指南.md`，但 README 把它編成 #03+；佳盈簡報說的「#04 建系統」指的就是這份。

## 🧾 怎麼使用
**方式一（最簡單）**：在 Claude Code 貼上
```
這是 Claude Code 懶人包全集 https://github.com/mathruffian-dot/claude-code-lazy-packs
請讀取 repo 內容，列出所有可用的懶人包，問我要裝哪些。
```
**方式二**：下載單一 MD 檔，丟給 Claude Code 桌面版的 Code 分頁執行。

**最低先備條件**
- Claude 帳號 **Pro 方案以上**
- **Claude Code 桌面版**（需要一台 Mac 或 Windows 電腦，iPad／手機不行）
- 網路連線

---

## ⭐ #03 建立第二大腦（Obsidian）

### 會做什麼
讓 Claude Code 能**讀取、搜尋、新增、編輯**你 Obsidian 筆記庫裡的筆記。

### 步驟
1. **環境檢查**：作業系統、Node.js、npx、Google Drive 桌面版、Obsidian
2. **安裝 Obsidian**（手動，到 obsidian.md 下載）
3. **安裝 Google Drive 桌面版**（手動），讓筆記自動同步到雲端
4. **在 Google Drive 建 vault 資料夾**（預設名稱 `secondbrain`）
5. **用 Obsidian 開啟這個資料夾**作為筆記庫
6. **安裝 mcpvault**（讓 Claude 讀寫筆記的工具，不需要 Obsidian 開著、不用裝外掛）
7. **寫入設定檔**（為了保險寫在三個位置）
8. **重啟 Claude Code**，驗證能讀到筆記、能新增測試筆記
9. **建立 CLAUDE.md**（Claude 的「班規」，每次對話自動讀取）
10. **建立第一篇正式筆記**

### 同步方案
| 方案 | 費用 | 說明 |
| --- | --- | --- |
| Google Drive（預設） | 免費 | vault 放在 Google Drive 資料夾裡 |
| Obsidian Sync | 付費約 US$4／月 | 官方同步，手機和電腦最順 |
| iCloud | 免費額度 | 適合 Mac＋iPhone／iPad |

> 差別只在 vault 放在哪，Claude 連接方式完全一樣。

### 常見坑（作者實測）
- Windows 的 Claude Code 可能找不到 Node.js → 需要加 PATH
- 桌面版可能沒有 `claude` 指令 → 改手動寫設定檔
- 設定檔要用**完整路徑**，路徑有中文或空格要注意跳脫
- **避免兩台裝置同時編輯同一篇筆記**（Google Drive 同步衝突）

---

## ⭐ #03+ 第二大腦設定指南（建系統）

### 會做什麼
把空的 Obsidian 筆記庫，變成完整的 AI 知識管理系統。完成後你只要做兩件事：
1. **平時**：看到好文章 → 用 Web Clipper 存到 `Clippings/`
2. **每週**：Claude 自動整理成知識庫

### 1. 先認識你
它會問：科目、年級、回答語言、其他偏好。
> 這是寫給老師的問題，**我要改成回答自己的身分**（見下方「我的版本」）。

### 2. 三層資料夾（靈感來自 Karpathy）
```
vault/
├── Clippings/   ← 輸入：網路上剪下的文章（不修改）
├── 知識庫/      ← 消化：AI 整理的結構化知識（AI 寫，我審核）
├── 創作庫/      ← 輸出：自己的原創作品（我寫的）
├── 每日筆記/    ← 每日紀錄、週計畫、知識重整週報
├── Templates/   ← 筆記模板
└── CLAUDE.md    ← Claude 的班規
```

### 3. CLAUDE.md 的工作規則
- 新增筆記一律加 frontmatter（title、date、tags）
- 依內容自動判斷存到哪個資料夾
- `Clippings/` 是原始資料，**不要修改**
- 知識庫有 `index.md`（目錄）和 `log.md`（操作紀錄），每次更新都要記錄
- 說「幫我新增到筆記」就是存到這個 vault

### 4. 三個模板
- **每日筆記**：今日事項、反思、明日優先事項
- **週計畫**：本週重點、重要事項、每日連結
- **知識庫頁面**：核心概念、與我的連結、內容缺口
> 模板用到 `tp.date.now` 語法，需要 Obsidian 的 **Templater** 外掛才會自動帶入日期。

### 5. Web Clipper
瀏覽器擴充功能，按一下就把網頁存進 Obsidian。設定預設資料夾為 `Clippings`。

### 6. 每週知識重整排程
- 預設**每週日早上 9:17** 自動執行（佳盈改成每週一、四凌晨）
- 六個步驟：
  1. 盤點本週新增或修改的筆記
  2. 消化 Clippings → 知識庫
  3. 創作庫回流，提煉新概念
  4. 健康檢查：可互連的筆記、知識缺口、孤兒頁面
  5. 更新 `index.md`、`log.md`
  6. 產出週報到 `每日筆記/`
- 也可以隨時說「**跑一次知識重整**」手動觸發
> 排程是在自己的電腦上跑，時間到時電腦和 Claude Code 桌面版通常要開著。

### 7. 常用指令
| 你說的話 | Claude 會做的事 |
| --- | --- |
| 「幫我新增到筆記」 | 存到 Obsidian，自動判斷資料夾 |
| 「搜尋筆記有沒有 XXX」 | 搜尋整個 vault |
| 「跑一次知識重整」 | 手動執行完整流程 |
| 「消化 Clippings」 | 只消化新剪藏的文章 |
| 「知識庫 lint」 | 只做健康檢查 |

---

## ✏️ 我的版本：改成手繪接案用

### 資料夾對照
| 懶人包 | 我的用法 |
| --- | --- |
| `Clippings/` | 教學文章、別人的經營分享（例如這篇和佳盈的兩篇） |
| `知識庫/` | AI 整理的經營、行銷、繪圖技巧心得 |
| `創作庫/` | 我的作品說明、貼文文案、精選動態文字 |
| `每日筆記/` | 每日待辦、週復盤 |
| **加一個** `創業庫/` | [[Feiyen 手繪接案經營]]、[[Feiyen 商品表]]、[[Feiyen 需求調查表單設計]]、`委託案/` |

### 我的 CLAUDE.md 草稿
```markdown
# 我的第二大腦 — CLAUDE.md

## 關於我
- 我是 Feiyen，兼職手繪畫家，IG @fei.ru.yen
- 商品：無臉畫圖檔、無框畫、鑰匙圈、生日插旗、生日立牌、紅包袋、春聯
- 這個 vault 是我的接案與創作第二大腦

## 語言偏好
- 所有回應和筆記都用繁體中文

## 筆記庫結構
| 資料夾 | 用途 |
|---|---|
| `Clippings/` | 別人的文章、教學（不修改） |
| `知識庫/` | AI 整理的知識，有 index.md 和 log.md |
| `創作庫/` | 我的作品說明、貼文文案（只有我改） |
| `創業庫/` | 經營規劃、商品表、表單設計 |
| `創業庫/委託案/` | 一案一筆記 |
| `每日筆記/` | 每日待辦、週復盤、知識重整週報 |
| `Templates/` | 模板 |

## 工作規則
- 新增筆記一律加 frontmatter（title、date、tags）
- Clippings 是原始資料，不要修改
- 我的作品和文案只給建議，不直接改寫
- 委託案筆記要記：客人 IG、商品、價格、付款狀態、交件日、修改紀錄
- 價格以 [[Feiyen 商品表]] 為準
```

---

## ⚠️ 開始前要注意
- **需要一台電腦**：Claude Code 桌面版要 Mac 或 Windows。iPad 可以用 Obsidian 看筆記，但不能跑懶人包。
- **同步方式要先決定**：電腦和 iPad 都要看到同一個筆記庫，建議用 Obsidian Sync 或 iCloud（Mac）；Google Drive 在 iPad 上和 Obsidian 搭配較麻煩。
- **懶人包會在電腦上安裝軟體**（Node.js、mcpvault 等），執行前可以先讀過內容，遇到不懂的步驟就先問。
- 這是作者持續更新的版本，執行時的內容可能和這份整理不同。

## ✅ 我的下一步
- [ ] 確認有一台 Mac 或 Windows 電腦，並訂閱 Claude Pro
- [ ] 決定同步方式（Obsidian Sync／iCloud／Google Drive）
- [ ] 執行 #03：安裝 Obsidian 並讓 Claude 連接筆記庫
- [ ] 執行 #03+：建三層結構，回答問題時用上面的「我的版本」
- [ ] 把現有的筆記（Clippings、創業庫）搬進新的筆記庫
- [ ] 試一次「跑一次知識重整」

## 🔗 相關
- [[AI 剪片＆第二大腦系統 - 重點整理]] · [[ga02 AI 陪跑 - 重點整理]] · [[Feiyen 手繪接案經營]]
