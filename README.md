# 曼陀羅筆記

把八格填滿的九宮格（曼陀羅 / Mandal-Art）筆記工具。單一 HTML 檔案，沒有建置流程、沒有框架依賴。

**線上網址**：https://abbestben.github.io/mandala-notes/

---

## 功能總覽

- **單張**：一次一張九宮格，中心寫主題，周圍八格自由填，任一格可以「往下想」變成新的中心
- **81 格**：目前這張紙加上四周八張子紙的縮圖總覽
- **攤開**：所有寫過的紙同時攤在畫布上，依原本的方位展開，可拖曳調整位置、防止互相遮擋、一鍵「整理排版」
- **全貌**：整個主題的階層樹狀清單
- **歸納**：把攤開收集到的內容一鍵謄成一份獨立的大綱，可拖曳排序、縮排改層級、直接編輯文字，跟原始九宮格內容分開存放
- **自動編號**：每張紙、每一格都會依照「NW,N,NE,W,E,SW,S,SE → 5,6,7,4,8,3,2,9」的規則自動算出階層位置（例如 4-5-6），不用手動編號
- **關鍵字自動配圖**：打字時即時比對內建的中文關鍵字對照表（約 200 條），猜到合適的字就在格子上方顯示對應的 emoji 插圖，純本機比對、不用 AI、不花錢
- **跨裝置同步**：用 Email／密碼登入後，資料會同步存到 Firebase，Mac 和手機用同一組帳密就能共用同一份資料，即時同步（不用手動存檔/傳檔）
- **PWA**：手機瀏覽器「加入主畫面」後會有自己的圖示、開啟時沒有網址列，且有基本的離線殼層
- **備份**：首頁「備份全部（JSON）」／「匯入備份」，可以匯出/還原所有資料

---

## 開發紀錄

依開發順序：

1. 從 claude.ai 上的 Artifact 搬過來，建立本機 git 專案
2. 新增「攤開」畫布檢視，可拖曳、有連接線
3. 攤開的卡片加上自動階層編號
4. 編號從紅色圓標改成右下角灰字小標（避免搶視覺）
5. 預設關閉「填滿八格才能往下」的鎖，順序不重要
6. 新增「歸納」大綱編輯器（拖曳排序、縮排、編輯、從收集一鍵匯入）
7. 每格加上關鍵字自動配圖，圖庫從 35 條擴充到約 200 條，圖示尺寸放大到有記憶點
8. 攤開視圖加入碰撞閃避與「整理排版」按鈕，避免卡片互相重疊
9. 接上 Firebase（Email/密碼登入 + Firestore），支援跨裝置即時同步
10. 上架 GitHub Pages，加上 PWA（manifest.json、九宮格造型 App 圖示、service worker）

詳細每一步的程式碼變更，看 `git log` 就是完整紀錄，每筆 commit 訊息都寫了改了什麼、為什麼改。

---

## 技術架構

- **`index.html`**：整個 App。`<style>` 是所有 CSS，`<script>` 是所有邏輯（一個大 IIFE，沒有外部框架）
- **`manifest.json`** / **`sw.js`**：PWA 設定與離線快取
- **`icon-192.png` / `icon-512.png` / `icon-180.png`**：App 圖示（九宮格造型，深藍底白格線）
- **資料儲存**：
  - 沒登入：只存瀏覽器 `localStorage`
  - 登入後：`localStorage` 照存，同時 debounce 900ms 後推到 Firestore `users/{uid}/sheets/{專案id}`，並用 `onSnapshot` 監聽即時更新，讓其他裝置的變更自動同步過來
  - 每個專案（九宮格主題）是一包 JSON，裡面包含所有節點內容、攤開畫布的卡片座標、歸納大綱——都在同一包裡，所以只要專案有同步，這三者都會一起同步，不用分開處理

## Firebase 專案

- 專案 ID：`mandala-notes`
- 控制台：https://console.firebase.google.com/project/mandala-notes
- 用 Email/Password 登入方式（不是匿名登入），這樣同一組帳密才能在不同裝置間共用資料
- Firestore 安全規則（已設定）：只有登入者本人能讀寫自己 `uid` 底下的資料

---

## 日後要修改，怎麼進行？

### 方式一：找 Claude Code 幫忙改（推薦）

1. 開一個新的 Claude Code 對話，指定資料夾為這裡（`/Users/huangben/Documents/曼陀羅筆記`）
2. 直接說想改什麼，例如「圖庫再加 OO」「攤開的卡片想要更大」「歸納想要支援子項目縮排更多層」
3. 改完會幫你 `git commit` + `git push`，GitHub Pages 大約 1-2 分鐘內自動重新部署
4. 手機、Mac 重新整理網頁就會看到新版本（不用重新安裝 PWA）

### 方式二：自己手動改

1. 用文字編輯器打開 `index.html` 直接改
2. 本機測試：
   ```bash
   cd "/Users/huangben/Documents/曼陀羅筆記"
   python3 -m http.server 8765
   ```
   瀏覽器開 http://localhost:8765
3. 確認沒問題後推上去：
   ```bash
   git add -A
   git commit -m "說明這次改了什麼"
   git push
   ```
4. push 後 GitHub Pages 會自動重新部署

### 常見修改位置速查（用編輯器搜尋這些關鍵字）

| 想改什麼 | 搜尋關鍵字 |
|---|---|
| 圖庫的關鍵字對照表 | `ICON_MAP` |
| 編號規則（方位對應數字） | `const NUM` |
| 九宮格填滿才能往下的鎖 | `prefs.strict` |
| 攤開視圖的卡片間距、防重疊 | `spacingAt`、`CARD_COLLIDE` |
| Firebase 設定值 | `FIREBASE_CONFIG` |
| 歸納大綱的邏輯 | `ensureOutline`、`renderOutline` |
| App 名稱、圖示、PWA 設定 | `manifest.json` |

---

## 備份與救援

- 平常：首頁「備份全部（JSON）」定期存一份到雲端硬碟最保險
- 真的資料異常：去 Firebase 主控台的 Firestore Database，`users/{你的uid}/sheets` 底下可以直接看到每個專案的原始 JSON
