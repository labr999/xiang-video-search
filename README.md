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

## 三、申請 Google 登入用的 OAuth 憑證（播放清單搜尋要用）

跟申請 API 金鑰在同一個 Google Cloud 專案裡設定：

1. 左側選單「API 和服務」→「OAuth 同意畫面」，User Type 選 **External**，填基本資料（App name、你的信箱）即可，Scopes 那步可以先跳過，測試使用者（Test users）加入自己的 Google 帳號信箱
2. 左側「憑證」→「+ 建立憑證」→「OAuth 用戶端 ID」
3. 應用程式類型選 **Web應用程式**
4. 「已授權的重新導向 URI」先留空白，等 Render 部署好拿到網址後回來補上（見下面步驟六）
5. 建立後會拿到 **用戶端ID（Client ID）** 和 **用戶端密鑰（Client secret）**，先複製記下來

> 因為還在「測試中」狀態，只有你加入的測試使用者信箱能登入，這對個人使用完全足夠，不需要送 Google 審核。

## 四、部署後端（以 Render 為例）

1. 把 `backend/` 資料夾推到一個新的 GitHub repo（例如 `xiangqi-video-search`）
2. 到 https://render.com 註冊/登入 → New → Web Service → 連接該 repo
3. 設定：
   - Root Directory: `backend`
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
4. 環境變數（Environment）：
   - `YOUTUBE_API_KEY` = 你剛申請的金鑰
   - `CHANNEL_ID` = 你的頻道 ID
   - `GOOGLE_CLIENT_ID` = 上一步拿到的用戶端 ID
   - `GOOGLE_CLIENT_SECRET` = 上一步拿到的用戶端密鑰
   - `GOOGLE_REDIRECT_URI` = 先隨便填 `https://placeholder.onrender.com/auth/google/callback`，等下一步拿到正式網址後要回來改
   - `ALLOWED_ORIGIN` = 之後你的 GitHub Pages 網址（例如 `https://labr999.github.io`），先不填也可以
5. 部署完成後會拿到一個網址，例如 `https://xiangqi-video-search.onrender.com`
6. 用瀏覽器打開 `該網址/api/health`，看到 `{"ok":true,"youtube_key_configured":true}` 就代表基本設定成功

## 五、回頭補上正確的 Redirect URI（兩邊都要改）

1. 把 Render 的 `GOOGLE_REDIRECT_URI` 環境變數改成正式網址，例如：
   `https://xiangqi-video-search.onrender.com/auth/google/callback`
2. 回 Google Cloud Console →「憑證」→ 點剛剛建立的 OAuth 用戶端 ID → 「已授權的重新導向 URI」新增同一個網址，儲存
3. 這兩邊的網址必須**一模一樣**（包含 https、有沒有結尾斜線都要一致），否則登入會失敗

> Render 免費方案閒置一段時間會休眠，第一次呼叫會慢個十幾秒喚醒，屬正常現象。
> Railway 步驟大同小異，差別在介面位置。

## 六、部署前端

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
- 搜尋自己頻道 / 全 YouTube / 自己播放清單裡的影片（關鍵字搜尋）
- 播放清單搜尋：用 Google 登入授權後，按「同步播放清單」把清單內容抓下來快取，
  之後搜尋不用再打 YouTube API，也不受配額限制
- 手動將影片標記為某個開局分類（先手／後手 + 細分類）
- 已標記影片可依開局分類篩選瀏覽
- 標記資料存在後端 SQLite（`tags.db`），不會因為你換手機而消失

## 播放清單搜尋怎麼用
1. 「⚙️設定」分頁 → 按「使用 Google 登入」→ 用你的 Google 帳號登入並同意權限
2. 登入成功後回到設定頁，按「同步播放清單」，等它抓完（清單多、影片多會久一點）
3. 之後每次在播放清單裡新增/整理影片後，記得回設定頁再按一次「同步播放清單」更新快取
4. 「🔍搜尋」分頁切到「播放清單」，就能搜尋你收藏在各個播放清單裡的影片了

## 尚未做的部分（第二、三階段）
- 自動用棋盤影像辨識（V2.9 pipeline）判斷開局，取代手動標記
- 辨識信心值標示、辨識失敗時的人工複核流程
- 辨識結果回饋修正開局樹

## 已知限制
- YouTube API 每日額度有限，全站搜尋量大時可能需要快取或申請提高額度
- Render/Railway 免費方案的資料庫（SQLite 檔案）在服務重新部署時可能會被清空，
  正式上線前建議改用 Render 的付費 Persistent Disk 或外部資料庫（如 Supabase 免費方案）
