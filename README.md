[README.md](https://github.com/user-attachments/files/32154484/README.md)
# 象棋開局影片搜尋（第一階段）

分成兩部分：
- `frontend/index.html`：靜態網頁，部署到 GitHub Pages（跟象棋開局譜同做法）
- `backend/`：Python 後端，部署到 Render 或 Railway（免費方案即可）

---

## 一、申請 YouTube Data API 金鑰

1. 到 https://console.cloud.google.com 建立一個專案（或用現有的）
2. 左側選單「API 和服務」→「程式庫」，搜尋 **YouTube Data API v3**，點「啟用」
3. 「憑證」→「建立憑證」→「API 金鑰」，複製產生的金鑰
4. 免費額度：每天 10,000 單位，一次搜尋約耗 100 單位，等於每天約可搜尋 100 次，超過會回傳 quotaExceeded 錯誤

## 二、取得你自己頻道的 Channel ID

1. 到你的 YouTube 頻道頁面 → 「關於」
2. 「分享頻道」→「複製頻道 ID」，會是 `UC` 開頭的一串英數字
   （這串我沒辦法幫你查證，需要你自己複製，避免錯誤資料）

## 三、部署後端（以 Render 為例）

1. 把 `backend/` 資料夾推到一個新的 GitHub repo（例如 `xiangqi-video-search`）
2. 到 https://render.com 註冊/登入 → New → Web Service → 連接該 repo
3. 設定：
   - Root Directory: `backend`
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
4. 環境變數（Environment）：
   - `YOUTUBE_API_KEY` = 你剛申請的金鑰
   - `CHANNEL_ID` = 你的頻道 ID
   - `ALLOWED_ORIGIN` = 之後你的 GitHub Pages 網址（例如 `https://labr999.github.io`），先不填也可以
5. 部署完成後會拿到一個網址，例如 `https://xiangqi-video-search.onrender.com`
6. 用瀏覽器打開 `該網址/api/health`，看到 `{"ok":true,"youtube_key_configured":true}` 就代表成功

> Render 免費方案閒置一段時間會休眠，第一次呼叫會慢個十幾秒喚醒，屬正常現象。
> Railway 步驟大同小異，差別在介面位置。

## 四、部署前端

1. 把 `frontend/index.html` 上傳到一個新的 GitHub repo，開啟 GitHub Pages（跟象棋開局譜的做法一樣）
2. 打開網站 → 底部「設定」分頁 → 填入後端網址（例如 `https://xiangqi-video-search.onrender.com`，**不要加結尾斜線**）→ 儲存
3. 回到「搜尋」分頁即可開始搜尋

## 檔案結構
```
xiangqi-video-search/
├── frontend/
│   └── index.html          # 搜尋介面（GitHub Pages）
├── backend/
│   ├── main.py              # FastAPI 後端
│   └── requirements.txt
└── README.md
```

## 目前功能（第一階段）
- 搜尋自己頻道 / 全 YouTube 的影片（關鍵字搜尋）
- 手動將影片標記為某個開局分類（先手／後手 + 細分類）
- 已標記影片可依開局分類篩選瀏覽
- 標記資料存在後端 SQLite（`tags.db`），不會因為你換手機而消失

## 尚未做的部分（第二、三階段）
- 自動用棋盤影像辨識（V2.9 pipeline）判斷開局，取代手動標記
- 辨識信心值標示、辨識失敗時的人工複核流程
- 辨識結果回饋修正開局樹

## 已知限制
- YouTube API 每日額度有限，全站搜尋量大時可能需要快取或申請提高額度
- Render/Railway 免費方案的資料庫（SQLite 檔案）在服務重新部署時可能會被清空，
  正式上線前建議改用 Render 的付費 Persistent Disk 或外部資料庫（如 Supabase 免費方案）
