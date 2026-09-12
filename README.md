# 透天厝家務勾稽表

單一 HTML 檔案的互動式家務勾稽表，依樓層（一樓～四樓＋全棟）分類，可勾選完成、記錄完成日期，並可自行新增／修改／刪除區域與項目。資料透過瀏覽器 localStorage 自動儲存在使用者裝置上。

## 放到 GitHub 並啟用網頁（GitHub Pages）

1. 到 GitHub 建立一個新的 repository（公開或私人皆可），例如叫 `household-chores`。
2. 把 `household-chores.html` 上傳到這個 repository（可直接在網頁上用「Add file → Upload files」拖曳上傳，不需要指令列）。
3. 進入 repository 的 **Settings → Pages**。
4. 在「Build and deployment」的 Source 選擇 `Deploy from a branch`，Branch 選 `main`（或你的預設分支）、資料夾選 `/root`，按 Save。
5. 等 1～2 分鐘，GitHub 會給你一個網址，格式通常是：
   `https://<你的帳號>.github.io/household-chores/household-chores.html`
6. 之後打開這個網址，就是可以直接使用、資料會留在瀏覽器裡的家務勾稽表。把檔名改成 `index.html` 上傳，網址就可以省略檔名，直接用 `https://<你的帳號>.github.io/household-chores/`。

## 關於資料儲存

- 資料儲存在**瀏覽器的 localStorage**，只會留在同一台裝置、同一個瀏覽器上，清除瀏覽器資料會遺失，換裝置或換瀏覽器也不會同步。
- 如果之後想要「多人共用、跨裝置同步」的記錄（例如家人各自在手機上打勾，彼此都看得到），需要接一個簡單的後端或資料庫（例如 Firebase、Supabase 等），屬於進一步的功能，可以再另外評估。
