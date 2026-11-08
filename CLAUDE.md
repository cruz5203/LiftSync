# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 這是什麼

LiftSync 是一個重量訓練紀錄 PWA，介面是繁體中文（`zh-Hant-TW`）。沒有建置步驟、沒有套件管理、沒有任何相依套件。整個 App 就是 `index.html`（CSS 和一大段 `<script>` 都寫在裡面），加上 `sw.js`（離線快取）和 `manifest.webmanifest`，以靜態檔案部署到 GitHub Pages。README 是寫給不寫程式的擁有者看的；使用者看得到的文字、toast 訊息和 README 的新內容都要用中文。

## 執行與測試

```
python -m http.server 8000
```

- App：`http://localhost:8000/`
- 測試：`http://localhost:8000/test.html`。一定要透過伺服器開，`file://` 讀不到 iframe 的內容。頁面會顯示通過與失敗的數量，並列出每個失敗項目實際拿到的值。改完 `index.html` 重新整理這一頁就會重跑。
- `test.html?report` 跑完後會把結果 JSON POST 到 `__result`，給腳本自動跑用。單純的 `http.server` 不接受 POST，所以腳本要自己架一個小伺服器。
- 沒辦法只跑單一測試，每次都是整套跑完；可以用「只看失敗」只顯示失敗的項目。

`test.html` 的運作方式（新增測試前先讀）：
- 它在 iframe 裡載入**真正的** `index.html`，再把測試程式注入到 App 自己的 window 裡執行。這樣測試才能直接用 App 最上層的 `let`／`const`（`S`、`normalize`、`seed`、`renderAll`、`startRest`、`wEdit`……），這些名稱不會出現在 `window` 上。所以 App 裡改名字，測試可能會跟著壞。
- **第一階段**是 `<script type="text/plain" id="suite">` 那一段。它透過點擊和派發事件操作介面，並把 `beep`、`navigator.vibrate`、`Notification`、`navigator.serviceWorker` 換成假的，測試時不會真的響、震動或跳通知。`fresh()` 會重設回範例資料。
- **第二階段**是 `COLD` 陣列。每一項先用 `seed()` 把指定資料塞進 localStorage，開一個新的 iframe，再執行 `code` 字串，用 `ok(...)` 檢查結果，最後呼叫 `done()`。
- **第三階段**會真的註冊 `sw.js`，檢查預快取清單、舊快取有清掉、離線時有回應，以及 `sw.js` 的 `VERSION` 和 `index.html` 的 `<!-- LiftSync vX.Y.Z -->` 一致。環境不支援 service worker 時這一階段會跳過，總數變少是正常的。
- 測試頁會先備份 localStorage、跑完再還原。這個保證不能弄壞，因為測試頁和真正的訓練紀錄在同一個網域。
- 不要把測試數量寫死在任何地方，README 也一樣。

## 發布新版

版本號要改**兩個地方**，而且必須一樣（第三階段會檢查）：
- `sw.js` 第二行：`const VERSION = "liftsync-X.Y.Z";`
- `index.html` 最後的註解：`<!-- LiftSync vX.Y.Z -->`

沒改 `sw.js` 的版本號，手機不會拿到新的 `index.html`。新增離線需要的檔案時，要加進 `sw.js` 的 `SHELL`。

## `index.html` 的架構

程式是一個平鋪的大檔，用 `/* ---------- 名稱 ---------- */` 分段（tabs、exercise sheet、summary、draft workout、custom exercise library、rest timer、end-of-rest alert、routines、history、correct a saved workout、progress、backup、sample data）。搜尋這些分段標記就能找到位置。

**狀態。** 只有一個全域物件 `S`，透過 `save()` 以 JSON 存在 localStorage 的 `liftsync-v1`。結構是 `{ver:3, workouts, bodyweight, draft, sample, custom, routines, settings:{rest, exRest, alert}}`。`workouts` 一律**新的在前**，有好幾個函式依賴這個順序（例如 `lastSetsFor`、動作庫的「最近練過」）。`draft` 是進行中的訓練。第一次開啟時，`seed()` 會產生範例資料，並標記 `sample:true`。

**所有外來資料都要經過 `normalize(st)`。** 載入時會跑，匯入備份時也會跑。它會把不是陣列的欄位換成陣列、丟掉沒有日期或名稱的紀錄、確保每筆訓練都有不重複的 `id`（編輯畫面靠 id 找紀錄）、每天只留一筆體重。如果 normalize 本身出錯，就退回 `seed()`；壞掉的資料絕對不能讓 App 打不開。`S` 新增欄位時，要在這裡補預設值。

**舊的儲存鍵。** 改名前資料存在 `ironlog-v1` 和 `ironlog-v1-tab`，現在仍會讀它們當備援，冷啟動測試有涵蓋。

**畫面渲染。** 沒有框架。`el(tag, attrs, ...children)` 建立 DOM（`on` 開頭的屬性會變成事件監聽），`$` 就是 `getElementById`，`icon(name)` 引用 SVG 的 `<symbol id="i-name">`。每個區塊各有一個 `renderX()`，`renderAll()` 會全部呼叫一次。不要每點一下就重建整個大清單，之前有一個 commit 專門移除這種寫法。

**動作。** 內建動作庫是 `LIB` 這個字串（`部位|器材|名稱,名稱,…`），轉成 `BASE`。`allEx()` 等於 `BASE` 加上 `S.custom`。動作在各處都是用**名稱**對應：訓練紀錄、範本、`settings.exRest`。所以自訂動作改名時，這些地方都要一起改，歷史紀錄才不會斷掉。移除自訂動作只是讓它不再出現在選單裡。

**重訓與有氧。** 一組的欄位有兩種：重訓是 `{kg, reps}`，有氧（部位是「有氧」的動作）是 `{min, km}`。欄位定義集中在 `COLS`，畫面、驗證（`badSet`）、存檔（`cleanSet`）、帶入上次（`copySet`）都從 `cols(name)` 讀，不要再寫死 `kg`／`reps`。`isCardio(name)` 的判斷順序：自訂動作的部位 → 內建的重訓動作一律不是有氧 → 紀錄本身的欄位（`recordKind`）→ 內建有氧。紀錄排在內建有氧前面，是因為 v1.3.0 之前有人可能用「跳繩」「划船機」這類名字記過重訓，自訂動作移除後不能被新的內建有氧接手。這種情況下 `allEx()` 也會把那個內建有氧藏起來（`otherKind`），使用者才能用原本的部位把自訂動作加回來。`recordKind` 對歷史紀錄的查詢結果會快取，`save()` 時清掉；直接改 `S.workouts` 卻沒呼叫 `save()`，就會讀到舊結果。有氧不開始休息倒數、不算進 `vol()`、不進 `bests()`／`prWorkouts()`。自訂動作有紀錄後不能在有氧與重訓部位之間切換（`kindClash`）。

**共用的動作選單。** 只有一個 `#picker-panel` 底部面板，訓練中、範本編輯、紀錄修改三個畫面共用。`homeSheet(slot)` 把它搬到目前使用的畫面裡，`pickTarget()` 決定「加入」要寫到哪個清單（`wEdit` 是正在修改的歷史紀錄，`editing` 是正在編輯的範本，都不是就寫到 `S.draft`）。

**下一組與休息計時。**
- 下一組由 `activeSet()` 決定，依據是**打勾時間 `s.at`**：最近打勾的那個動作如果還有組沒做，下一組就在那裡；否則就是整份訓練裡第一個沒打勾的組。這個結果不會另外存成指標，所以取消打勾、移除動作、重開 App 時都不會錯亂。重點卡片和休息列都從它讀資料，兩邊必須保持一致。
- 休息計時存的是**結束時刻**（`restEnd`），存在 `liftsync-v1-rest`，不是累計秒數，所以手機休眠或 App 被關掉重開都不受影響。
  - `restoreRest()` 會丟掉超過 15 分鐘的休息、結束時刻離現在太遠的，以及沒有進行中訓練時的休息。
  - `tick()` 用 `restAlerted` 這個旗標確保震動、提示音、通知只觸發一次；休息已經結束超過 90 秒（`LATE_ALERT`）就不再提醒。
- 音效的 AudioContext 必須在使用者點擊時建立（`primeAudio`）。

**建置旗標。** `<html data-build="pwa">` 會啟用備份的匯出與匯入。只能在 PWA 裡運作的程式，要放在這個判斷後面。

## 慣例

- 程式寫得很緊湊，幾乎一行一個敘述，名稱很短。照著同樣的風格寫。
- 註解說明的是**為什麼**，通常是「不這樣做會出什麼錯」（例如「a non-array here throws during the first paint…」）。不明顯的防護都要保留這種註解。程式碼裡的註解目前是英文。
- 輸入框必須維持 `font-size:16px`，否則 iOS 點進去會放大。版面在窄螢幕不能橫向溢出，測試會用 360px 寬的 iframe 檢查。
- Commit 訊息是簡短的英文祈使句，描述行為上的改變（例如 "Keep the rest dock's next-set line and the next card in step"）。
