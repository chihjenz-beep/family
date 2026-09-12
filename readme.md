# 透天厝家務勾稽表

單一 HTML 檔案的互動式家務勾稽表，依樓層（一樓～四樓＋全棟）分類，可勾選完成、記錄完成日期，並可自行新增／修改／刪除區域與項目。

支援兩種儲存模式：
- **本機模式（預設）**：資料存在瀏覽器 localStorage，只有目前這台裝置看得到。
- **雲端同步模式（選用）**：接上免費的 Firebase Realtime Database 後，全家/志工共用同一個網址，不用登入，任何人打開都看到、修改的是同一份資料。

---

## 第一步：放到 GitHub 並啟用網頁（GitHub Pages）

1. 到 GitHub 建立一個新的 repository，例如叫 `household-chores`。
2. 把 `household-chores.html` 和 `README.md` 上傳到這個 repository 的**根目錄**（同一層），可直接在網頁上用「Add file → Upload files」拖曳上傳。
3. 進入 repository 的 **Settings → Pages**。
4. Source 選 `Deploy from a branch`，Branch 選 `main`、資料夾選 `/root`，按 Save。
5. 等 1～2 分鐘，會拿到一個網址，例如：
   `https://<你的帳號>.github.io/household-chores/household-chores.html`
   （想省略檔名的話，把檔案改叫 `index.html` 再上傳即可。）

這一步做完，家務表就能用，但資料只存在每個人自己的瀏覽器裡（本機模式）。想要大家共用同一份，才需要下面第二步。

---

## 第二步（選用）：開啟雲端同步

這一步會用到 **Firebase**（Google 提供的免費雲端服務），不需要寫程式，只要複製貼上幾個設定值。

1. 前往 https://console.firebase.google.com ，用 Google 帳號登入。
2. 點「新增專案 / Add project」，輸入專案名稱（例如 `household-chores`），一路下一步建立完成（可以關閉 Google Analytics，不需要）。
3. 進入專案後，左側選單找 **建構 / Build → Realtime Database**，點「建立資料庫 / Create Database」。
   - 地區選離台灣近的（例如新加坡 asia-southeast1）。
   - 安全性規則先選 **測試模式 / Test mode**（之後可再調整，見下方「安全性提醒」）。
4. 建好後，回到左上角齒輪圖示 → **專案設定 / Project settings**，往下捲到「你的應用程式 / Your apps」，點 `</>`（網頁圖示）新增一個網頁應用程式，命名隨意，不用勾 Firebase Hosting。
5. 系統會顯示一段 `firebaseConfig` 設定，長得像：
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "household-chores-xxxx.firebaseapp.com",
     databaseURL: "https://household-chores-xxxx-default-rtdb.asia-southeast1.firebasedatabase.app",
     projectId: "household-chores-xxxx",
     storageBucket: "household-chores-xxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
6. 打開 `household-chores.html`，找到檔案裡的 `FIREBASE_CONFIG`（在 `<script>` 區塊靠前面的位置），把上面對應的值一一貼進去、存檔：
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "household-chores-xxxx.firebaseapp.com",
     databaseURL: "https://household-chores-xxxx-default-rtdb.asia-southeast1.firebasedatabase.app",
     projectId: "household-chores-xxxx",
     storageBucket: "household-chores-xxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
7. 把改好的 `household-chores.html` 重新上傳覆蓋 GitHub 上的舊檔（GitHub 網頁上直接編輯該檔案、貼上新內容存檔即可，或刪除舊檔重新上傳）。
8. 等 GitHub Pages 重新部署（通常 1 分鐘內），打開網址，畫面右上角會顯示「雲端同步中」，這時所有人打開同一個網址看到的就是同一份、即時同步的資料。

### 安全性提醒

測試模式的 Realtime Database 規則是完全公開讀寫、且沒有到期限制，任何知道資料庫網址的人理論上都能讀寫。對內部小型使用（家人、志工）通常足夠，但如果想更保險一點，可以進 Firebase 主控台的 **Realtime Database → 規則 / Rules**，把規則改成只開放這個工具用到的路徑，例如：

```json
{
  "rules": {
    "household-chores": {
      "shared": {
        ".read": true,
        ".write": true
      }
    },
    "$other": {
      ".read": false,
      ".write": false
    }
  }
}
```

這樣即使有人拿到資料庫網址，也只能存取家務表這一份資料，不會動到專案裡的其他東西。

### 已知限制

- 這是「最後寫入者為準」的同步方式：如果兩個人**同時**在編輯同一個欄位（例如同時修改同一個項目名稱），比較晚存檔的那邊會蓋掉另一邊，不會自動合併。避免同時改同一格即可。
- 若沒有設定 `FIREBASE_CONFIG`（保持空白），程式會自動退回「本機模式」，不影響單機使用。
