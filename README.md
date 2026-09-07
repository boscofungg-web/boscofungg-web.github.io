# 每週固定時間表 — 網頁版

`weekly.pdf` 嘅網頁版本。同一套配色、同一套「可排課空檔」計法，
不過學生可以直接喺網頁㩒個空檔就預約，唔使等你出新 PDF。

- **一個 `index.html` 檔搞掂**，冇 build、冇框架、冇 npm。
- 週表 + 月曆兩個檢視。
- 新增課堂係一個彈出視窗：課堂名稱、星期、開始／結束，**預設每週重複**。
- 可排課空檔自動計，演算法照搬 `gen_weekly.py` 嘅 `free_slots()`。

---

## 檔案

| 檔案 | 做咩 |
|---|---|
| `index.html` | 成個網站。所有設定喺頂部嘅 `CALENDAR_CONFIG` |
| `firestore.rules` | Firebase 安全規則，貼落 Firebase 主控台 |
| `README.md` | 你而家睇緊嘅嘢 |

---

## 一、放上 GitHub Pages

1. GitHub 右上角 **+** → **New repository**
2. 名要**一模一樣**係 `boscofungg-web.github.io`（你嘅 username + `.github.io`），選 **Public**
3. **Add file → Upload files**，拉 `index.html` 入去 → **Commit changes**
4. **Settings → Pages**：Source 揀 `Deploy from a branch`，分支 `main`，資料夾 `/ (root)` → Save
5. 等一兩分鐘，開 `https://boscofungg-web.github.io`

之後改嘢：喺 repo 入面開 `index.html` → 㩒鉛筆 → 改 → **Commit changes**，一分鐘內生效。

> 補充：`.github.io` 呢個網址永久免費，連 HTTPS 都免費。
> 自己買 `boscofung.com` 之類就要每年約 US$10，GitHub 唔賣網域。
> 免費替代：`is-a.dev`、`eu.org`、`js.org`（要係 JS 相關項目）。

---

## 二、接駁 Firebase — 令所有人睇到最新版、又改得到

**呢步唔做嘅話，每個訪客嘅課堂只存喺佢自己部瀏覽器，互相睇唔到。**
GitHub Pages 淨係派檔案，佢冇資料庫。要共用就要一個資料庫，
Firebase Firestore 免費額度（每日 5 萬次讀、2 萬次寫）遠遠夠一個時間表用。

### 點解接駁之後就自動同步

程式用嘅係 Firestore 嘅 `onSnapshot()`。佢唔係「入頁面時載入一次」，
而係**開住一條長連線**：任何人一改資料，Firebase 就即刻推去所有開緊個網頁嘅人，
畫面自己重繪，唔使 refresh。所以：

- 你喺電話改咗個時間 → 學生嗰邊幾秒內自己更新
- 學生預約咗一個空檔 → 你嗰邊即刻見到，個空檔亦即刻縮細
- 佢哋改嘅嘢係寫入資料庫，唔係寫入瀏覽器，所以人人都保存得到

### 設定步驟

1. 去 **console.firebase.google.com**，用 Google 帳戶登入。
2. **建立專案**（Create a project）。名隨便。Google Analytics **關咗佢**，唔需要。
3. 左邊 **Build → Firestore Database → Create database**
   - 揀 **Start in production mode**
   - 地區揀 **asia-east2（香港）**
4. 齒輪 **⚙ → Project settings**，碌到 **Your apps**，㩒 **`</>`**（網頁）圖示，
   改個暱稱 → **Register app**。
5. Firebase 會出一段 `firebaseConfig`。**抄低啲值。**
6. 開 `index.html`，喺頂部 `CALENDAR_CONFIG` 度，把 `firebase: null` 換成：

```js
firebase: {
  apiKey: "AIzaSy...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.firebasestorage.app",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
}
```

7. 返 Firebase → **Firestore Database → 規則（Rules）**，
   把 `firestore.rules` 全部內容貼上去 → **發布（Publish）**。
   **呢步唔可以慳。** production mode 預設係全部拒絕，唔貼規則咩都寫唔到。
8. 把 `index.html` commit 返上 GitHub。一分鐘後就係共用時間表。

### 第一次開會發生咩事

資料庫係空嘅時候，網頁會**自動把 `index.html` 入面嗰份預設時間表寫上 Firestore**
（就係由你 `gen_weekly.py` 搬過嚟嗰 22 堂）。之後所有改動都以資料庫為準，
`index.html` 裡面嘅預設值就唔再理。所以：

- 想改課堂 → **喺網頁改**，唔好再改 `index.html`
- 想推倒重來 → 去 Firestore 主控台刪晒 `lessons` collection，再 refresh 個網頁

### `apiKey` 唔係密碼

每個 Firebase 網頁 app 都會喺公開 JavaScript 入面帶住個 apiKey，
佢只係「識別你個 project」，唔會授權任何嘢。真正管存取嘅係第 7 步嗰份規則。

---

## 三、管理模式

一般訪客可以新增課堂、以及改／刪**自己加嘅**課堂，掂唔到人哋嘅。

你自己喺網址尾加密語就有全部權限：

```
https://boscofungg-web.github.io/#owner
```

進入之後標題旁會出現「管理模式」標籤，而且：

- 改得、刪得任何一堂
- 彈出視窗多咗「備註」（課室／科目代碼）
- 新增嘅課堂直接算「已確認」，唔係「待確認」

**分享出去之前記得改密語**，喺 `index.html`：

```js
ownerKey: "owner",   →   ownerKey: "只有你知嘅字",
```

> 老實講一句：呢個係門鎖，唔係夾萬。密語寫喺網頁嘅 JavaScript 入面，
> 識開 developer tools 嘅人搵得到。對住一班你識嘅學生完全夠用 —— 佢防意外，唔防黑客。
> 真係要嚴謹就要上 Firebase Authentication，由伺服器驗身份。
>
> 同一道理：唔好喺備註入面寫任何畀陌生人睇到會有問題嘅嘢。個時間表本身就係公開嘅。

---

## 四、可以改嘅設定

全部喺 `index.html` 頂部嘅 `CALENDAR_CONFIG`：

| 設定 | 做咩 |
|---|---|
| `title` / `subtitle` | 大標題、細標題 |
| `footNote` / `copyright` | 頁尾左右兩邊嘅字 |
| `ownerKey` | 管理模式密語 |
| `categories` | 顏色分類。見下面第五節 |
| `defaultCategory` | 學生預約時預設用邊個分類 |
| `dayStart` / `dayEnd` | 週表格線顯示嘅範圍。而家係 `08:00`–`23:00` |
| `bookableFrom` / `bookableTo` | 自動計「可排課空檔」嘅範圍。而家係 `09:00`–`21:00`，即係朝早 8–9 點同夜晚 9–11 點唔會出空檔（同你 `gen_weekly.py` 嘅 `T0, T1 = 9, 21` 一樣）|
| `lockedWindows` | 封鎖時段，見下面第七節 |
| `categoryPassword` | 受保護分類嘅密碼，見下面第八節 |
| `travelBuffer` | 緊接港大課堂嘅空檔要扣幾多分鐘（預設 30，等於原本 `BUF = 0.5`） |
| `minSlot` | 短過幾多分鐘唔當可排課（預設 60，等於原本 `MINSLOT = 1.0`） |
| `openDays` | 邊幾日收新課，`[一,二,三,四,五,六,日]`。而家係 `[true,true,false,true,true,false,false]` |
| `extraWindows` | 某日只開放部分時段。而家係 `{ 2: [["09:30","11:30"]] }`，即星期三只開 09:30–11:30 |
| `holidays` | 假期。而家有 9 月 25、26 號中秋。當日唔會計可排課。**假期名淨係喺月曆格仔右上角出，週表標題唔會顯示** |
| `firebase` | Firebase 設定，或 `null` |

顏色喺 `<style>` 開頭嘅 `:root`：`--uni` 藍（港大）、`--work` 綠（新東方）、
`--open` 橙（可排課），同 `calendar.tex` 嘅 `brandink` / `brandmain` / `brandaccent` 一樣。
深色模式已經整好，唔使另外處理。

---

## 五、顏色分類

彈出視窗嘅「顏色分類」用嘅係 `calendar.tex` 原本嗰四隻色，冇加新色：

| 顏色 | `colour` 值 | 預設名 | LaTeX 對應 |
|---|---|---|---|
| 藍 | `uni` | 港大課堂 | `brandink` |
| 綠 | `work` | 新東方教學 | `brandmain` |
| 橙（實色） | `amber` | 其他 / 私人 | `brandaccent` |
| 橙（半透明虛線） | `open` | 可排課 | 同自動計嗰啲空檔一模一樣 |
| 灰 | `slate` | （而家冇用） | `restcol` |

`slate` 灰色仍然用得，只係預設冇分類揀佢。想要就把某個分類嘅 `colour` 改成 `"slate"`。

改名、加減分類都喺 `index.html` 嘅 `categories`：

```js
categories: [
  { key: "uni",   label: "港大課堂",    colour: "uni",   travel: true  },
  { key: "work",  label: "新東方教學",  colour: "work",  travel: false },
  { key: "other", label: "其他 / 私人", colour: "amber", travel: false },
  { key: "open",  label: "可排課",      colour: "open",  travel: false, fixedTitle: "可排課" }
]
```

- **`key`** 係存落資料庫嘅代號。**改咗就對唔返舊資料**（舊課堂會跌返做預設分類），
  所以想改名就淨係改 `label`，唔好郁 `key`。
- **`colour`** 只可以係 `uni` / `work` / `amber` / `open` / `slate` 五個之一。
- **`travel: true`** 代表呢類課堂前後要預留交通時間（本來 `gen_weekly.py` 只有港大課堂要）。
  揀掣上會有個「＋交通」小標記。想新開嘅分類都預留，就設 `travel: true`。
- **`fixedTitle`** 會鎖死個名。揀咗「可排課」之後，課堂名稱會自動填成「可排課」
  兼且變咗唯讀（灰橙色底），任何人都改唔到，咁手動同自動嘅空檔就完全一樣。
  換返第二個顏色，之前打過嗰個名會自動還返。
  唔想鎖就把 `fixedTitle` 整行刪走。
- **`locked: true`** 代表要密碼先改得，見第八節。

圖例、頁尾時數統計、`.ics` 匯出嘅分類名都係跟住呢個清單自動更新，唔使另外改。

### 「可排課」呢個分類同自動計嗰啲有咩分別

樣係一模一樣，但行為唔同，值得搞清楚：

- **自動嗰啲**由 `free_slots()` 計出嚟，係「執完所有課之後剩返嘅空位」。
  你加多一堂，佢即刻自動縮細。開放日先會出現。
- **手動嗰個分類**係你自己擺落去嘅一格，同一堂課冇分別 —— 
  即係話**佢會佔住嗰段時間**，自動計嗰陣會當佢係「有嘢做」，
  所以嗰個位唔會再另外出一格自動空檔。

用途：**喺唔開放嘅日子手動開一格**。例如星期六本來 `openDays` 係 `false`，
自動計唔會出任何空檔，但你想開個補堂時段 —— 手動擺一格「可排課空檔」就得。

頁尾嘅「可排課 X 小時」會把自動同手動兩邊加埋一齊數，唔會數兩次。

> 學生㩒手動嗰格會開到「課堂詳情」（唯讀），唔係預約表格 ——
> 因為喺程式眼中佢係一堂課，唔係一個空位。
> 想學生㩒得落去直接預約就同我講，改得。

**所有人都揀得色**，唔使管理模式。預設會揀住 `defaultCategory`（而家係新東方教學），
學生想改就改。

> 值得諗一諗：呢個係方便，唔係管制。冇任何嘢阻止學生把自己嘅堂標成「港大課堂」藍色。
> 對住你識嘅一班學生冇所謂，亂咗你自己㩒返轉頭就得。
> 如果將來覺得煩，想改返「只有管理模式先揀得色」，喺 `index.html` 搵：
>
> ```html
> <div class="field" id="kindField">
> ```
>
> 加返個 `hidden`：
>
> ```html
> <div class="field" id="kindField" hidden>
> ```
>
> 再喺下面 `if (IS_OWNER) {` 嗰段加返一行
> `document.getElementById("kindField").hidden = false;` 就得。

> ⚠️ 加咗新分類之後，`firestore.rules` 都要一齊更新，否則 Firebase 會拒絕寫入。
> 搵 `d.kind in [...]` 嗰行，把新 `key` 加埋落去，再喺 Firebase 重新發布規則。
> 而家嗰行係：`d.kind in ['uni', 'work', 'other', 'open', 'admin']`
>（`admin` 留住係為咗舊資料，冇用過就當佢唔存在。）

---

## 六、重複課堂點運作

一堂「每週重複」嘅課堂會記低一個**開始日期**（`from`），只會由嗰日開始出現。

- 你今個星期加一堂逢星期四嘅課 → 今個星期同以後每個星期四都有，
  **上個星期四唔會突然多咗一堂**。
- 開表格嗰陣個說明會話你知：「由你而家睇緊嗰一星期開始，之後每星期出現。以前嘅星期唔會補返。」
  即係話你想由邊個星期開始，就先揀去嗰個星期再撳新增。
- 開返一堂已經存在嘅重複課堂，說明會變成「由 2026-09-10 開始，每星期重複」。

### 刪除重複課堂

撳「刪除」會問你揀邊樣：

| 揀 | 結果 |
|---|---|
| **只刪這一次** | 淨係嗰一日消失，之前之後照舊（記喺 `skip` 入面） |
| **刪除呢次同之後** | 嗰日同之後全部消失，**之前嘅照舊保留**（記低 `until`） |

由第一次出現嗰日撳「刪除呢次同之後」，等於成個系列刪走，程式會直接刪張紀錄。

> 一個限制講清楚：**改時間唔會分開新舊。**
> 你把一堂逢星期四 19:00 嘅課改成 20:00，佢由開始日期起所有星期四都會變 20:00，
> 唔會淨係改將來嗰啲。真係要分開就「刪除呢次同之後」，再喺新嗰星期開一堂新嘅。

### 已經喺資料庫嘅課堂唔受影響

呢個改動係**向後兼容**嘅。你原本嗰 22 堂、同埋學生已經預約咗嘅，全部冇 `from` 呢個欄位，
程式當佢哋「所有星期都出現」—— 即係同以前一模一樣，過去嘅星期照樣睇得到，
唔使你手動補資料，亦唔會有嘢消失。

只有**之後新開**嘅重複課堂先會有開始日期。

---

## 七、封鎖時段 `lockedWindows`

```js
lockedWindows: {
  2: [["17:00", "23:00"]],   // 星期三 夜晚唔開放
  5: [["18:00", "23:00"]],   // 星期六 夜晚唔開放
  6: [["08:00", "23:00"]]    // 星期日 全日唔開放
},
```

索引同 `openDays` 一樣：`0`=一 … `6`=日。一日可以封幾段。

封鎖之後會發生三件事：

1. 格線上打斜紋，中間有個「唔開放預約」標籤 —— **睇得見嘅鎖先係有用嘅鎖**
   （星期標題而家淨係得星期同日子，開放狀態一律睇格線）
2. 自動空檔唔會踩入去
3. 有人想喺嗰度預約會俾彈返轉頭，出提示叫佢揀第二個時間

拖曳都拖唔到落封鎖區。**管理模式唔受限制** —— 自己張表自己話事，
不過表格會出黃色提示話你知呢個時間一般訪客預約唔到。

> `lockedWindows` 同 `openDays` 唔同：
> `openDays` 管「自動空檔出唔出」，`lockedWindows` 管「畀唔畀人預約」。
> 星期三本來就唔係開放日，但冇 `lockedWindows` 之前，
> 學生一樣可以喺格線度拖個時間出嚟預約 —— 而家先真係鎖死。

---

## 八、受保護分類（改港大課堂要密碼）

`categories` 入面加 `locked: true`，嗰個分類就要密碼先改／刪得到：

```js
{ key: "uni", label: "港大課堂", colour: "uni", travel: true, locked: true },
```

```js
categoryPassword: "CHANGE-ME",
```

規則：

- 學生想改一堂港大課堂、或者想把自己嗰堂標成港大課堂 → 要打密碼
- 打啱一次，同一個分頁之後都唔使再打（`sessionStorage`，閂咗個 tab 就重設）
- **管理模式免問** —— 你已經有 `ownerKey`，唔使打兩個密碼

### ⚠️ 揀密碼之前要知

`categoryPassword` 會**原封不動咁出現喺公開 GitHub repo 嘅 `index.html` 入面**。
任何人 View Source 都睇得到，搜尋器都索引得到。

所以：

1. **一定要揀一個你冇喺其他地方用過嘅字。**
2. **千祈唔好用你 GitHub、電郵、銀行、學校嗰啲密碼。**
3. 佢嘅作用係「防手誤」—— 阻止學生手快撳錯改咗你堂大學課，
   唔係「防人」。同 `ownerKey` 一樣，係門鎖唔係夾萬。

同一道理都適用於 `ownerKey`。兩個都揀啲隨手嘅字就得，例如 `wed-lock-2026`。

---

## 九、點用

| | |
|---|---|
| 預約／新增 | 㩒橙色「可排課」格，或者喺格線度拖，或者㩒「+ 新增課堂」 |
| 改一堂／換色 | 㩒嗰個課堂格 |
| 刪重複課堂 | 㩒刪除 → 會問你「只刪這一次」定「刪除呢次同之後」，見第六節 |
| 轉檢視 | 月曆／週表掣，或者㩒 `M`、`W` |
| 睇邊日開放 | 睇格線 —— 橙色虛線格 = 可排課，斜紋 = 唔開放預約 |
| 睇假期 | 開月曆檢視，假期名喺格仔右上角 |
| 前後移動 | 方向鍵左右，`T` 返今日，`N` 新增 |
| 匯出 | 下載 `.ics`，把未來 16 週展開，可匯入 Google／Apple 日曆 |

拖曳以 15 分鐘為單位，表格入面亦可以直接打任何時間 —— 11:30 至 12:15 冇問題。
撞時間會出黃色提示，但唔會阻止你儲存（有時你真係想 double book）。

---

© By Bosco Fung, All rights reserved ・ 內部教學用途
