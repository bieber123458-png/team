# 佩柔團隊10天上手計劃

新人社群銷售團隊的「10天解鎖式教學系統」：新人每天完成一項任務、送出回饋，上級審核通過才解鎖下一天；每天內容額外用一組3位數PIN碼保護，PIN由上級決定何時公佈（例如當天上課時才公佈）。系統本身分兩個介面：**新人端**（看課程、送出回饋）與**審核台**（上級登入、核可/退回、看PIN碼、備份資料）。

> 這份 README 是為「接手開發」寫的：說明這個專案在做什麼、技術怎麼選、資料夾怎麼擺、要改東西該去哪個檔案。如果你是另一個 AI 或工程師接手這個專案，請先讀完這份文件再動手。

---

## 1. 技術棚疊（Tech Stack）

- **後端**：Node.js + [Express 4](https://expressjs.com/)（純 CommonJS，沒有用 TypeScript、沒有框架化的 MVC 結構，就是一個 `server/index.js` 掛路由）
- **檔案上傳**：[Multer](https://github.com/expressjs/multer)（新人回傳作業時可附一張截圖，存在本機磁碟）
- **資料儲存**：**沒有用資料庫**，用一個本地 JSON 檔案 `data/db.json` 當簡易資料庫（見第5節的重要限制）
- **前端**：**沒有用任何前端框架**（沒有React/Vue），純手寫 HTML + Vanilla JavaScript + 純CSS，瀏覽器直接載入 `<script>` 執行，沒有打包/編譯步驟
- **環境變數**：[dotenv](https://github.com/motdotla/dotenv)，讀取 `.env` 檔案
- **部署平台**：[Render](https://render.com)（Web Service，免費方案），設定檔是根目錄的 `render.yaml`

因為前端沒有框架、沒有 bundler，**這個專案沒有「build」步驟**——`public/` 底下的 HTML/CSS/JS 檔案就是瀏覽器實際執行的檔案，改了存檔重新整理頁面就會生效，不需要編譯。

---

## 2. 專案資料夾結構

```
.
├── render.yaml                 # Render 部署設定（build/start command）
├── package.json / package-lock.json
├── .env.example                 # 環境變數範例（複製成 .env 使用）
├── .gitignore
│
├── server/                      # ===== 後端 =====
│   ├── index.js                 # 入口：建立 Express app、掛載所有路由與靜態檔案路徑、啟動監聽
│   ├── db.js                    # 簡易JSON檔案資料庫：readDb() / writeDb() / genId()
│   ├── adminAuth.js              # 審核台權限中介層：檢查 x-admin-key header 是否等於 ADMIN_KEY
│   ├── tasks.js                  # ★ 10天課程內容本體：每天的標題、PIN碼、頁面內容區塊、送出提示文字
│   └── routes/
│       └── api.js                # 所有 REST API 路由（新人建立身份、PIN解鎖、送出回饋、審核、備份匯出/匯入）
│
├── public/                      # ===== 前端（直接被 express.static 原樣送出） =====
│   ├── index.html                 # 新人端頁面骨架（姓名輸入畫面 + 任務列表容器）
│   ├── app.js                     # ★ 新人端邏輯：呼叫API、渲染每天內容、送出回饋表單、圖片壓縮
│   ├── style.css                  # 全站共用樣式（新人端 + 審核台都用這份）
│   ├── review/
│   │   ├── index.html              # 審核台頁面骨架（金鑰輸入畫面 + 審核列表容器）
│   │   └── review.js               # ★ 審核台邏輯：登入、列出夥伴進度、核可/退回、刪除、備份匯出/匯入
│   └── assets/                     # 課程用的所有圖片與示範影片（詳見下方）
│
└── data/                          # ===== 執行期資料（不會進 git，詳見第5節） =====
    ├── db.json                     # 實際資料庫檔案（trainees / submissions / pinUnlocks），執行時自動建立
    └── uploads/                    # 新人上傳的截圖，依 traineeId 分資料夾存放
```

### `public/assets/` 裡的素材分類

| 類別 | 檔名規則 | 用途 |
|---|---|---|
| 構圖/字體教學截圖 | `comp-*.png`, `fonts-recommend.png`, `camera-app.png`, `design-apps.png` | Day 3「簡單的限動美感」用圖 |
| 色系對比、文案結構範例 | `color-tabu-example.jpg`, `structure-example.jpg` | Day 3/4 用圖 |
| MBA系統截圖 | `mba-system.png`, `grid-settings.png` | Day 1、Day 3 用圖 |
| Day5 示範影片 | `day5-value.mp4`, `day5-goods.mp4`, `day5-restaurant.mp4`, `day5-work.mp4` | 動態分享範例（有聲音） |
| Day5 文字心得範例截圖 | `day5-value-example.jpg`, `day5-shipping-example.jpg`, `day5-reason-example.jpg`, `day5-failure-example.jpg` | 框架A/B的真實範例 |
| Day6 九宮格定位圖 | `day6-nine-grid.jpg` | Day 6 結尾總覽圖 |
| Day7 產品參考圖 | `day7-jerosse-product.jpg` | ChatGPT出圖教學的參考產品照 |
| Day8 纖體班報名素材 | `day8-slimclass-1/2/3.jpg` | 報名流程、陪跑班/陪伴班資訊圖 |
| Day9 官方價格表 | `day9-price-chart.jpg` | 銷售話術段落用的官方零售價圖 |
| 網站封面/App圖示 | `cover-team.jpg` | 新人端登入頁最上方封面圖，也是手機「加入主畫面」的圖示 |

> 所有圖片都已經是最終版本、直接被程式引用，**不需要也不應該重新命名**，否則 `server/tasks.js` 裡的 `src` 路徑會對不上。

---

## 3. 安裝與啟動

```bash
# 1. 安裝依賴
npm install

# 2. 設定環境變數
cp .env.example .env
# 打開 .env，把 ADMIN_KEY 改成你自己的審核台密碼（預設是 changeme）

# 3. 啟動伺服器（沒有build步驟，直接跑）
npm start
# 等同於: node server/index.js
```

啟動後：
- 新人端：`http://localhost:3000/`
- 審核台：`http://localhost:3000/review`（需要輸入 `.env` 裡的 `ADMIN_KEY`）

`npm run dev` 跟 `npm start` 目前是同一個指令（都是 `node server/index.js`），沒有用 nodemon 之類的自動重啟工具，改完程式碼要手動重啟伺服器才會生效。

### 沒有「build」指令

`package.json` 的 `scripts` 只有 `start` 和 `dev`，**沒有 `build`**。因為前端是純靜態 HTML/JS/CSS，不需要編譯打包，Render 的 `buildCommand` 設定也只是 `npm install`。

---

## 4. 主要頁面與功能說明

### 4.1 新人端（`public/index.html` + `public/app.js`）

- **進入**：第一次打開輸入姓名，用瀏覽器 `localStorage` 記住身份（沒有密碼登入機制）
- **解鎖邏輯**（兩道鎖，邏輯在後端 `server/routes/api.js` 的 `traineeView()` / `unlockedSeqCount()`）：
  1. **順序鎖**：前一天的回傳必須被上級「核可」，才會顯示這一天的「輸入PIN」畫面
  2. **PIN鎖**：就算順序解鎖了，還要輸入當天的3位數PIN碼才看得到完整內容（PIN寫在 `server/tasks.js`，由上級自行決定何時口頭告知新人）
- **送出回饋**：填寫「完成內容」+「這堂課學會了什麼」，可選擇附一張圖片（前端會先用 Canvas 壓縮圖片再上傳，壓縮邏輯在 `app.js` 的 `resizeImageToDataUrl()` 附近，目的是避免手機原圖太大）
- 每天的內容排版是由 `server/tasks.js` 裡宣告的「區塊（blocks）」陣列描述，`app.js` 裡的 `renderBlock()` 函式（約在檔案中段）負責把每種區塊類型轉成HTML，支援的區塊類型：

  | type | 用途 |
  |---|---|
  | `subhead` | 小標題 |
  | `p` | 一般段落（`lead:true` 會放大強調） |
  | `list` | 條列清單 |
  | `voicelist` | 一條條語音泡泡樣式的短句（常用來放「心裡話」或客人訊息範例） |
  | `qa` | 一問一答區塊 |
  | `quote` | 引言框（支援 `\n` 換行） |
  | `links` | 外部連結列表（會開新分頁） |
  | `fillBlank` | 提示詞＋範例的兩欄對照 |
  | `table` | 價格表等表格 |
  | `practice` | 練習題（問題＋答案對照） |
  | `groupList` | 名稱＋數字的列表（例如群組人數統計） |
  | `gapnote` | 補充說明小提示框 |
  | `menu` | 單排小標籤 |
  | `iconmenu` | 帶icon的選單項目 |
  | `profilecard` | 人物/角色介紹卡片 |
  | `image` | 單張圖片＋圖說 |
  | `imagegrid` | 多張圖片並排（例如Day8的3張報名流程圖） |
  | `video` | 影片播放器（含控制條） |

### 4.2 審核台（`public/review/index.html` + `public/review/review.js`）

- **登入**：輸入 `ADMIN_KEY`，存在 `localStorage`，之後自動帶入
- **每天PIN碼一覽**：畫面最上方，方便上級決定何時口頭公佈給新人
- **夥伴列表**：每位新人目前進度（做到第幾天）、待審核項目數量，點開可看每天詳細回傳內容
- **核可／退回**：審核每筆送出的回饋，可附上文字回覆給新人
- **刪除**：可刪除單筆送出紀錄，或整個刪除某位夥伴（含其所有紀錄與上傳圖片）——**這兩個刪除動作都沒有二次確認彈窗**，是先前按需求刻意做成一鍵直接刪除
- **匯出備份／匯入備份**：把目前所有資料（trainees/submissions/pinUnlocks）匯出成一個JSON檔案下載，或上傳檔案還原資料。**這是因應 Render 免費方案沒有持久化硬碟而加的功能**，詳見第5節

---

## 5. ⚠️ 重要限制：資料持久化

這是接手這個專案最需要知道的一件事：

**這個系統的資料儲存方式是本機 JSON 檔案（`data/db.json`），不是真正的資料庫。** 如果部署在 Render 免費方案（目前的部署方式），伺服器閒置一段時間會自動「休眠」，下次有人訪問時會重新啟動一個全新的容器——**容器裡的檔案系統不會保留，`data/db.json` 會被重置成空白**，所有新人資料、審核紀錄都會消失。

目前因應方式：
1. 審核台加了**手動匯出/匯入備份**功能（上面4.2節），上級需要自己養成習慣定期點「匯出備份」存檔
2. 更長期的解法（還沒做，留給接手者參考）：
   - 升級 Render 到付費方案並加裝持久化硬碟（Persistent Disk），或
   - 把 `server/db.js` 改接真正的資料庫（例如免費方案的 Supabase / MongoDB Atlas / Render 自己的 PostgreSQL），這樣不管容器怎麼重啟資料都不會不見

如果要改用真正資料庫，**唯一需要修改的檔案是 `server/db.js`**（只要保持 `readDb()` / `writeDb()` / `genId()` 這三個函式的介面不變，`server/routes/api.js` 完全不需要改）。

---

## 6. 目前部署狀況

- **平台**：[Render](https://render.com)，Web Service，免費方案（Free plan）
- **設定檔**：根目錄 `render.yaml`（`buildCommand: npm install`，`startCommand: npm start`）
- **環境變數**：Render 後台「Environment」頁籤手動設定 `ADMIN_KEY`（目前沿用預設值 `changeme`，沒有改成自訂密碼）
- **自動部署**：Render 已連接 GitHub 這個 repo 的 `main` 分支，**只要 push 到 `main`，Render 會自動重新部署**（約1-2分鐘）
- **免費方案限制**：閒置會休眠（喚醒需等30-60秒）、沒有持久化硬碟（見第5節）、不支援SSH/Shell

這份交接檔案本身**沒有對現在線上運行的網站做任何變更**，純粹是整理既有程式碼。

---

## 7. 如果要修改某個功能，應該改哪裡

| 想做的事 | 該改的檔案 |
|---|---|
| 修改某一天的文字內容、新增/刪除區塊 | `server/tasks.js`（找到對應 `day:` 的物件，改 `blocks` 陣列） |
| 新增/刪除一整天 | `server/tasks.js` 的 `DAYS` 陣列（注意 `day` 數字要連續，`TASKS` 是自動從 `DAYS` 算出來的，不用手動改） |
| 改某天的PIN碼 | `server/tasks.js` 對應天數物件的 `pin` 欄位 |
| 新增一種新的內容排版方式（新的block type） | 同時改兩處：`server/tasks.js`（新增資料時用新type）+ `public/app.js` 的 `renderBlock()`（新增對應的case） |
| 調整新人端畫面外觀/版面 | `public/style.css`（全站共用，審核台也會受影響） |
| 調整新人端互動邏輯（例如送出表單的驗證規則） | `public/app.js` |
| 調整審核台互動邏輯 | `public/review/review.js` |
| 新增/修改 API 路由 | `server/routes/api.js` |
| 改資料儲存方式（換成真正資料庫） | `server/db.js`（保持三個函式介面不變） |
| 改審核台登入驗證規則 | `server/adminAuth.js` |
| 換封面圖/App圖示 | 取代 `public/assets/cover-team.jpg`（檔名不變），或改 `public/index.html` 裡 `<link rel="apple-touch-icon">` 的路徑 |
| 改部署平台設定 | `render.yaml`（如果換平台，這個檔案可能整個不需要，改用該平台的設定方式） |

---

## 8. 環境變數說明（`.env.example`）

```
ADMIN_KEY=changeme
```

只有這一個環境變數：
- **`ADMIN_KEY`**：審核台登入密碼，同時保護所有需要上級權限的 API（核可、退回、刪除、備份匯出/匯入、看PIN碼）。正式使用時務必改成不容易被猜到的字串。

**沒有使用任何第三方API、沒有串接任何外部資料庫或服務**，所以除了這一個密碼，沒有其他需要的金鑰或Token。

`.env` 本身已經被 `.gitignore` 排除，不會被提交進版本控制。

---

## 9. 已知的設計取捨（給接手者的額外背景）

- 刪除操作（刪夥伴、刪送出紀錄）故意不做二次確認彈窗，是先前需求方明確要求「一鍵刪除」換取操作速度，如果要加回確認彈窗是合理的安全性改進，但屬於刻意的產品決策，不是bug。
- 新人身份用 `localStorage` 存一個自訂id，沒有密碼機制，任何知道這個id的人理論上可以冒充該新人送出資料——目前規模下（小型內部團隊）屬於可接受風險，如果團隊擴大，建議之後加上真正的帳號系統。
- `ADMIN_KEY` 是單一共用密碼，沒有分權限、沒有個別帳號紀錄是誰核可了哪一筆——同上，是目前規模下的簡化設計。
