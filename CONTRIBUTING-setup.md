# 家人投稿 — 設定與審核說明

網站底部的「分享您的回憶」表單，讓家人可投稿：**回憶留言**、**相片說明建議**、**上傳相片**。
投稿透過 **Netlify Forms** 收集，不會自動刊出 — 由站長確認後手動加入網站。

## 一次性設定（在 Netlify 後台）

表單會在下次部署時被 Netlify 自動偵測（表單名稱：`contribution`）。設定通知信：

1. 登入 Netlify → 選擇 `grandma-zheng` 網站。
2. **Forms** → 確認出現名為 **contribution** 的表單（首次需等一次部署完成）。
3. **Forms → Settings and notifications → Form notifications → Add notification → Email notification**。
4. 收件人填 **fur@richardfu.net**，儲存。

> 若 Forms 頁面看不到表單：到 **Site configuration → Forms** 確認 *Form detection* 已開啟，然後重新部署一次（push 任何修改即可）。

## 投稿欄位

| 欄位 | 內容 |
|------|------|
| name | 稱呼／姓名 |
| relationship | 與外婆的關係 |
| type | 回憶留言 / 相片說明 / 上傳相片 |
| message | 回憶內容 / 說明建議 / 相片描述 |
| photo | 相片編號（相片說明時；由燈箱帶入會自動填） |
| era_hint | 年代提示（上傳相片時） |
| file | 相片檔（上傳相片時，< 8MB） |

## 如何審核並刊出

收到通知信後，在 Netlify Forms 看到投稿內容，依類型處理：

### 回憶留言 → 加到「親友的話」
1. 開 `index.html`，找到 `<section id="testimonies" ...>`。
2. **移除** 該 section 的 `style="display:none"`（只需第一次）。
3. 在 `<div id="testimonies-list">` 內加入一則：
   ```html
   <div class="memory">
     <p class="memory-text">投稿的回憶內容……</p>
     <p class="memory-author">姓名（與外婆的關係）</p>
   </div>
   ```

### 相片說明 → 更新該相片 caption
1. 在 `index.html` 的 `eras` 陣列找到該相片（例 `["54.jpg",""]`）。
2. 把第二個值改成說明：`["54.jpg","婆羅浮屠，約1997年"]`。

### 上傳相片 → 加入相冊
1. 在 Netlify Forms 該筆投稿下載附件相片。
2. 命名後放入 `Photos/`（沿用編號規則：澳門早年 5xx、澳門晚年 8xx、近年 10xx）。
3. 在 `eras` 對應年代的 `photos` 陣列加入 `["新檔名.jpg","說明"]`。

### 套用變更
```
git add -A && git commit -m "刊出家人投稿：…" && git push
```
推送後 Netlify 與 GitHub Pages 會自動更新。

## 防垃圾訊息

表單已含隱形 honeypot 欄位（`bot-field`）。若日後出現垃圾投稿，可在 Netlify
**Forms → Settings → Spam filtering** 啟用 reCAPTCHA。

## 日後升級（Tier C，自動審核佇列）

目前為手動審核。投稿欄位（name / relationship / type / message / photo / file）已對齊
日後 Supabase 資料表，升級時把表單目的地改成 Supabase（狀態 `pending`）並加上站長審核頁即可，
無需重做欄位或版面。詳見 `docs/superpowers/specs/2026-05-31-family-contributions-design.md`。
