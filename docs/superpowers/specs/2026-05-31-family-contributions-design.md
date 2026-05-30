# 家人投稿功能 — 設計文件

**日期：** 2026-05-31
**狀態：** 已核准，待實作
**目標網站：** https://grandma-zheng.netlify.app · https://furic.github.io/grandma-zheng/

## 一、目的

讓外婆的家人能在傳記網站上：

1. 留下自己的**回憶／留言**（testimony）
2. **建議相片的說明／日期／人物**（caption / detail）
3. **上傳新相片**

所有投稿先進入待審狀態，由站長（Richard）**確認後**才刊登。本版本（v1）採最輕量、零維運的方案，並刻意設計成可平滑升級為日後的自動審核佇列（Tier C / Supabase）。

## 二、限制與背景

- 網站為純靜態單頁（`index.html`），相冊由 JavaScript 動態產生。
- 託管於 **Netlify**，且**已連結 GitHub repo**（push 至 `main` 自動部署）。
- 靜態主機無法自行接收／儲存投稿 → 採用 **Netlify Forms**（內建、免費、無需自建後端）。
- **關鍵技術限制：** Netlify 只在「建置時」掃描 HTML 來偵測表單。由 JS 產生的表單不會被偵測，因此必須在 `index.html` 內放置一個**靜態（可隱藏）的表單**，包含所有欄位名稱。

## 三、方案決定

**單一表單 + 類型選擇器**（而非每種投稿各一表單）。

- 家人在一個「分享」表單中選擇：回憶留言 / 相片說明 / 上傳相片。
- 依選擇顯示／隱藏對應欄位。
- 隱藏欄位 `type` 記錄投稿類型，方便站長分辨。
- 理由：對家人最簡單、站長只需看一個收件匣、程式碼只有一處。

## 四、家人端 UI

### 4.1 「分享您的回憶」區塊
- 位於頁面底部附近，沿用既有暖色（深棕／米白／金）設計 tokens。
- 一句溫馨邀請語 + 表單。

### 4.2 表單欄位
| 欄位 | 類型 | 說明 |
|------|------|------|
| `name` | text（必填） | 稱呼／姓名 |
| `relationship` | text | 與外婆的關係（例：孫女、外甥） |
| `type` | radio（必填） | 回憶留言 / 相片說明 / 上傳相片 |
| `message` | textarea | 回憶內容 或 相片說明建議 |
| `photo` | text | 相片編號（選「相片說明」時顯示；由燈箱帶入時自動填入） |
| `era_hint` | text | 年代提示（選「上傳相片」時顯示，選填） |
| `file` | file | 相片檔（選「上傳相片」時顯示） |
| `bot-field` | hidden honeypot | 防垃圾訊息 |

### 4.3 燈箱整合
- 相片放大檢視中加入小型「📝 建議說明」連結。
- 點擊 → 跳至表單、`type` 設為「相片說明」、`photo` 自動填入該相片檔名、平滑捲動定位。

### 4.4 送出後體驗
- 以 JavaScript（fetch）送出，家人留在原頁。
- 顯示「謝謝您 🌸 待家人確認後刊出」訊息。
- 若瀏覽器停用 JS，退回標準表單送出。

## 五、技術設計

- **偵測表單：** `index.html` 內放置隱藏的靜態 HTML 表單（`name="contribution"`、`data-netlify="true"`、`netlify-honeypot="bot-field"`），列出所有欄位名稱，供 Netlify 建置時偵測。
- **送出：** 可見表單以 `fetch` POST 至 `/`（`Content-Type` 由 `FormData` 自動處理，支援檔案 multipart）。成功後顯示頁內感謝訊息。
- **檔案上傳：** 透過 file 欄位夾帶。Netlify 免費方案可處理小型 JPEG；過大檔案以友善訊息提示（前端先檢查大小，例如 > 8MB 時提醒）。
- **防垃圾：** honeypot 欄位（對人類隱形）。日後若出現垃圾投稿再加 Netlify reCAPTCHA。

## 六、審核流程（v1：人工）

1. 家人送出 → Netlify 寄信至 **fur@richardfu.net**，並記錄於 **Netlify → Forms**。
2. 站長審閱。**核准**方式：
   - 回憶留言 → 文字加入 `index.html` 新增的**「親友的話」**區塊。
   - 相片 → 檔案放入 `Photos/`，並加入相冊陣列對應年代。
   - 說明修正 → 直接套用至該相片 caption。
3. push 至 `main` → 兩站自動更新。
4. 所有內容皆存於 git（可追溯、可還原）。

> Netlify 通知信設定於 Netlify 後台 **Forms → Form notifications**，收件人 `fur@richardfu.net`。

## 七、實作範圍

修改／新增：
- `index.html`
  - 隱藏的 Netlify 偵測表單。
  - 可見的「分享您的回憶」區塊 + 樣式。
  - 燈箱「建議說明」連結 + 帶入邏輯。
  - 送出 / 感謝訊息的 JS（含 honeypot、檔案大小前端檢查、no-JS 退回）。
  - 新增「親友的話」區塊（首則回憶核准前隱藏，作為日後刊登之處）。
- `CONTRIBUTING-setup.md`
  - Netlify 通知信設定步驟。
  - 站長「如何核准投稿」速查表。

**不更動：** 既有相片、時間軸、生平內容。

## 八、Tier C 升級相容性

v1 收集的欄位（name、relationship、type、message、photo ref、file）即為日後 Supabase 資料表所需。升級時僅需把表單目的地由 Netlify Forms 換成 Supabase insert（狀態 `pending`）並加上站長審核頁，無需重新設計欄位或 UI。

## 九、驗證方式

靜態 HTML/JS 表單依賴 Netlify 後端，無法以單元測試覆蓋；採人工驗證：
1. 本機開啟 `index.html` 確認區塊與燈箱連結顯示正常、欄位依類型正確顯示／隱藏。
2. push 後於正式站送出各類型測試投稿，確認 Netlify Forms 收到、通知信寄達 fur@richardfu.net。
3. 確認感謝訊息顯示、no-JS 退回可用。
