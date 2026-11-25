# 🍳 食譜庫技能（recipe-library）

Claude 的技能：把 YouTube 烹飪影片整理成食譜，存進你自己的 Notion 食譜庫；告訴它手邊有什麼材料，它會幫你搭配菜色；說「我要煮 X」就給你一張附計時器的食譜卡。

## 安裝

1. 下載這個專案裡的 `recipe-library` 資料夾（或按 **Code → Download ZIP**）。
2. 把 `recipe-library` 資料夾壓縮成 zip（裡面要有 `SKILL.md`）。
3. 到 claude.ai → **Customize → Skills**，上傳這個 zip。
4. 到 **Customize → Connectors** 連接 **Notion**。
5. 想讓 Claude 讀 YouTube 影片字幕，建議用 Claude 桌面版（內建瀏覽器）。

## 怎麼用

- 貼一個烹飪影片連結：「幫我記錄這道菜」
- 「我有雞胸肉、優格、番茄，可以煮什麼？」
- 「我要煮滷牛腱」

第一次用時，Claude 會在你的 Notion 找「食譜庫」，找不到就幫你建一個新的「🍳 我的食譜庫」資料庫和依類別的看板。
