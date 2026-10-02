# 鐵片日誌 PWA

重量訓練紀錄 App：課表範本、組間休息計時、器械動作庫、進度圖表、資料備份。
這是 PWA（漸進式網頁 App），放上網站後，手機「加入主畫面」就能像 App 一樣使用，離線也能開。

## 檔案說明

| 檔案 | 用途 |
|---|---|
| `index.html` | App 本體（介面與程式都在這裡） |
| `manifest.webmanifest` | App 名稱、圖示、顏色，讓手機知道它可以安裝 |
| `sw.js` | Service worker，負責離線快取 |
| `icons/` | 各尺寸 App 圖示 |

> 注意：直接在電腦上雙擊 `index.html` 打開時，離線與安裝功能不會啟用。PWA 必須透過 **https 網址**開啟才有效，所以要先部署。

## 部署到 GitHub Pages（免費）

1. 註冊或登入 [GitHub](https://github.com)。
2. 右上角「＋」→ **New repository**。名稱例如 `ironlog`，選 **Public**，按 **Create repository**。
3. 在新的 repository 頁面點 **uploading an existing file**，把這個資料夾裡的**所有檔案和 `icons` 資料夾**拖進去（不要拖外層的資料夾本身，`index.html` 必須在最上層）。按 **Commit changes**。
4. 進入 **Settings → Pages**。Source 選 **Deploy from a branch**，Branch 選 `main`、資料夾選 `/ (root)`，按 **Save**。
5. 等一到兩分鐘，重新整理該頁，上方會出現網址，例如 `https://你的帳號.github.io/ironlog/`。

其他免費選擇：Netlify（[app.netlify.com/drop](https://app.netlify.com/drop) 直接拖整個資料夾）或 Cloudflare Pages，效果相同。

## 安裝到手機

**iPhone（必須用 Safari）**
1. 用 Safari 打開上面的網址。
2. 點下方的「分享」按鈕 → **加入主畫面** → 新增。

**Android（Chrome）**
1. 用 Chrome 打開網址。
2. 畫面下方出現「安裝應用程式」就直接點；沒有的話，點右上角 ⋮ → **安裝應用程式**（或「加到主畫面」）。

之後從主畫面圖示開啟，就是全螢幕的 App 畫面。

## 資料與備份（重要）

- 所有紀錄都**只存在那支手機的 App 裡**，不會上傳到任何地方。
- 在 iPhone 上，主畫面 App 和 Safari 分頁的資料是分開的，請固定從主畫面圖示開啟。
- 刪除 App、清除瀏覽器資料或換手機，紀錄就會消失。請定期到「進度」頁最下方按 **匯出備份**，把 `.json` 檔存到雲端硬碟或傳給自己。新手機安裝後按 **匯入備份** 即可還原。
- 首次開啟會看到範例資料，在右上角按「清除」即可從空白開始。

## 之後要更新 App

1. 修改 `index.html`（或請 Claude 幫你改）。
2. 打開 `sw.js`，把第二行 `VERSION` 的版本號加一，例如 `ironlog-1.0.0` → `ironlog-1.0.1`。**沒改版本號，手機可能一直顯示舊版。**
3. 把改過的檔案重新上傳到 GitHub（同檔名覆蓋）。
4. 手機上把 App 完全關掉再打開，必要時開兩次，就會換成新版。
