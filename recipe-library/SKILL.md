---
name: recipe-library
description: 用 Notion 管理個人食譜庫：從 YouTube 烹飪影片整理食譜加入、依手邊材料搭配菜色、給可照做的食譜卡。提到食譜、煮什麼、貼烹飪影片連結時使用。
---

# 食譜庫

一律用**繁體中文**回覆，語氣簡潔。

## 需要的連接

- **Notion**：食譜存在使用者自己的 Notion 資料庫。沒連上時，請使用者到 claude.ai 的 Customize > Connectors 連接 Notion。
- **內建瀏覽器**（Claude 桌面版）：用來讀 YouTube 影片的說明欄和字幕。沒有的話，請使用者貼上說明欄文字或口述重點。

## 第一次使用：找到或建立食譜庫

1. 用 `notion-search` 搜尋「食譜庫」。找到就 `notion-fetch` 它，取得 data source（`collection://...`）和最新 schema。
2. 找不到就建立一個資料庫，標題「🍳 我的食譜庫」，schema：

```
CREATE TABLE (
  "菜名" TITLE,
  "類別" SELECT('主菜':red, '配菜':green, '湯':blue, '早午餐':yellow, '主食':orange, '甜點':purple, '醬料':brown),
  "主要食材" MULTI_SELECT('豬五花':pink, '梅花肉':pink, '豬里肌':pink, '絞肉':pink, '牛腱':red, '牛排':red, '雞胸':yellow, '雞腿':yellow, '魚':blue, '雞蛋':yellow),
  "時間（分鐘）" NUMBER,
  "難度" SELECT('簡單':green, '中等':yellow, '複雜':red),
  "需提前準備" CHECKBOX,
  "事前準備" RICH_TEXT,
  "份數" NUMBER,
  "做過" CHECKBOX,
  "影片" URL,
  "頻道" RICH_TEXT
)
```

   建好後再加一個依「類別」分組的看板檢視（board，`GROUP BY "類別"`），並把資料庫連結給使用者。
3. 第一次也順便問使用者：家裡有哪些廚具（氣炸鍋、電子鍋、烤箱…）、常在哪裡買菜。之後推薦菜色時要考慮這些。

## 資料庫屬性怎麼填

| 屬性 | 說明 |
|---|---|
| 菜名 | |
| 類別 | 主菜／配菜／湯／早午餐／主食／甜點／醬料 |
| 主要食材 | 標籤，用來篩選和比對材料。沒有的選項先用 `notion-update-data-source` 加（ALTER COLUMN 要列出全部舊選項＋新選項） |
| 時間（分鐘） | 影片沒說就自己估，並在備註寫明是估的 |
| 難度 | 簡單／中等／複雜 |
| 需提前準備 | 要先醃、退冰、前一天做就勾 |
| 事前準備 | 寫清楚要提前多久、做什麼 |
| 份數 | 不知道就留空 |
| 做過 | 使用者說煮過了就勾 |
| 影片 | 影片中段的菜加 `&t=秒數` |
| 頻道 | |

### 頁面內容格式

寫內容前先讀 `notion://docs/notion-flavored-markdown-spec`。

```
## 食材
（具體份量）

## 調味料

## 步驟
1. ...

## 備註
（份量缺漏、哪些是估的、資料來源可信度、技巧重點）

## 影片
<video source="YouTube 網址">影片標題</video>
```

使用者煮完回報的心得、失敗原因，用 `<callout icon="🔧" color="yellow_bg">實作心得...</callout>` 加在備註下，並勾「做過」。

## 從 YouTube 影片加入食譜

用內建瀏覽器打開影片，依序嘗試，拿到內容就停：

1. **說明欄**：`#movie_player.getPlayerResponse().videoDetails.shortDescription`，很多影片直接寫了食材份量和步驟。
2. **字幕記錄面板**：展開說明欄，點 `ytd-video-description-transcript-section-renderer button`，等幾秒後讀 `ytd-transcript-segment-renderer` / `transcript-segment-view-model` 的文字。最可靠的逐字稿。
3. **字幕檔**：攔截 XHR/fetch 中含 `timedtext` 的回應，再切換字幕軌（`setOption('captions','track',{languageCode})`）。自動翻譯的字幕要註明「細節可能有出入」。
4. **畫面燒錄字幕／字卡**：把影片設成 1200x1000 視窗，用 canvas 對 `<video>` 每 2–3 秒 seek 一次，裁下字幕那一條，疊成一張長圖放在畫面上，再截圖閱讀（先截全畫面確認字幕高度；有廣告先跳過；截圖若沒更新，先 screenshot 再 zoom）。食譜字卡用全畫面放大讀。做完把視窗設回 desktop。
5. 都拿不到就告訴使用者，請他口述重點，**不要自己編食譜**。

Shorts 網址改用 `watch?v=<id>` 開。

### 寫入規則

- **一律直接寫中文字，不要用 \u 跳脫碼拼字**，很容易拼成形近的錯字（例如「醃」變「醜」、「濾」變「濫」）。
- 建立後 fetch 頁面讀回，檢查錯字。
- 回報時說明：做法重點、哪些份量是影片給的、哪些是估的、資料來源（說明欄／字幕記錄／燒錄字幕）。

## 依手邊材料搭配菜色

當使用者說「我有 X、Y、Z」：
1. 用 `notion-query-data-sources` 讀整個資料庫（必要時 fetch 頁面看完整食材）。
2. 分成：材料齊全可以做的、只差一兩樣的（列出缺什麼）。
3. 給一餐的組合（主菜＋配菜／湯），考慮總時間、爐具是否衝突、「需提前準備」。
4. 使用者只是列出有的材料，不代表全部都要用。
5. 食譜庫沒有合適的，就用常識推薦新菜，問要不要找影片加進來。

## 使用者說「我要煮 X」

- 從 Notion 讀那一頁，用食譜卡（recipe_display）給可照做的食譜，等待、燉煮、烤的步驟都加計時器。
- 影片沒給的份量和時間由你建議，在卡片備註標明「這是我估的」。
- 名稱模糊時，對到最接近的一筆，回覆時說明你的判斷。
- 煮完請使用者回報，把心得補進頁面、勾「做過」。

## 找影片

用 YouTube 搜尋結果頁抓標題、頻道、觀看數、長度，依觀看數排序列成表格。使用者問訂閱數時，fetch 各影片 watch 頁用 regex 讀 `subscriberCountText`。說明觀看數高不代表好吃，也指出哪幾支最貼近使用者的食材。

## 回答烹飪問題

要具體：給比例、溫度、時間，並說明原因。實用技巧優先，例如瘦肉怎麼不乾柴（鹽水醃泡、優格醃）、低溫烤後煎等。
