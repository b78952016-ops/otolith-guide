# 魚耳石圖鑑專案指引

## 專案說明
將 FB 相簿「魚耳石和貝殼塚」的耳石紀錄整理成靜態 HTML 圖鑑網站。
純 HTML + CSS + JS，資料內嵌在 `index.html` 的 `<script>` 區塊，無外部框架。

## 檔案位置
- 主檔：`D:\耳石和貝殼塚\耳石圖鑑\index.html`
- 圖片：`D:\耳石和貝殼塚\耳石圖鑑\images\`
  - 命名規則：`{id}_otolith.jpg`、`{id}_fish.jpg`、`{id}_cooking.jpg`
  - 個位數 ID（1–9）補零：`01_`、`02_`…`09_`；兩位數以上不補零：`10_`、`11_`…
  - 棲地示意圖：`{id}_habitat.svg`（同樣補零規則）

## ID 編號規則

fishData 的 ID 對應 FB 相簿「魚耳石和貝殼塚」中**耳石貼文的時間順序**：
- 只計算耳石貼文，貝殼或其他非耳石貼文一律略過
- 第 n 篇耳石貼文 = ID n
- 照片所屬 ID 由使用者對照 FB 貼文順序後直接告知，不需推斷

## 新增條目照片流程

原始照片來自 `D:\耳石和貝殼塚\挑耳石影片\魚和耳石照片\`，命名格式為 `{序號}.{魚名}{類型}.{日期}.jpg/HEIC`。

### 步驟
1. **HEIC 轉 JPG**（若有）：用 ffmpeg 轉換：`ffmpeg -i input.heic -q:v 2 -update 1 output.jpg -y`
2. **對照 FB 留言圖片判斷照片類型**：
   - 開 FB 貼文的 fbUrl，找留言裡的圖片（只看不下載）
   - 對照本機照片內容，判斷哪張是生魚、哪張是料理
   - 料理照通常在留言第 2–3 張
3. **重新命名並移到 `images/`**：
   - 耳石照 → `{id}_otolith.jpg`
   - 生魚照 → `{id}_fish.jpg`（真空包裝或砧板生魚）
   - 料理照 → `{id}_cooking.jpg`（主要料理）、`{id}_cooking2.jpg`（次要）
   - 多張生魚照 → `{id}_fish2.jpg`、`{id}_fish3.jpg`...
4. **更新 index.html** 對應條目的 `otolithPhoto`、`fishPhoto`、`cookingPhoto`、`date`、`location`、`cooking`、`lat`、`lng`

### 注意
- FB 留言圖片只用來對照辨識，**不下載**，網站用本機照片
- HEIC 原始檔留在來源資料夾，不移入 `images/`

---

## 照片分類定義

三種照片的視覺判斷標準：

| 類型 | 檔名後綴 | 視覺特徵 |
|------|----------|----------|
| **耳石照** | `_otolith.jpg` | 黑底背景 ＋ 台幣一元硬幣當比例尺，主體是白色半透明的耳石 |
| **生魚照** | `_fish.jpg` | 簡單單色背景（砧板、鐵盤、白盤、真空包裝），只有魚，未烹調，沒有其他食材、醬汁、配菜 |
| **料理照** | `_cooking.jpg` | 已烹調，通常有醬汁、配菜、多種食材，或呈現複雜盤飾；魚頭湯、燉煮、清蒸整魚（有醬汁蔥薑）皆屬此類 |

**邊界案例說明：**
- 吃完後的魚骨殘骸 → 料理照（屬 cooking）
- 多條魚或多種生物並排但未烹調 → 生魚照（fish 可含多種）
- 貝殼照片 → 不屬於任何一類，**直接刪除**

---

## 照片審核與修正規範

每次新增或批次匯入照片後，應視覺審核所有圖片，確認分類正確。以下為 2026-05 審核中歸納的常見錯誤與處理方式：

### 錯誤類型與處理

**A. `_fish` 和 `_cooking` 是同一張料理照（_fish 誤標）**
- 現象：同一張已烹調的照片被複製成兩個檔，分別命名 fish 和 cooking
- 處理：刪除 `_fish.jpg`，保留 `_cooking.jpg`；fishData 的 `fishPhoto` 改為 `null`

**B. `_cooking` 槽放了耳石照（重複耳石）**
- 現象：`_cooking.jpg` 和 `_otolith.jpg` 是同一張耳石照
- 處理：刪除 `_cooking.jpg`；cookingPhoto 改為 `null`

**C. `_fish` 是料理照且與 `_cooking` 不同（兩張不同的料理）**
- 處理：將 `_fish.jpg` 改名為 `_cooking2.jpg`；fishPhoto 改為 `null`

**D. `_cooking` 槽是料理照，但 `_cooking` 被刪後空缺（B 類刪除後）**
- 處理：將 `_fish.jpg`（其實是料理）改名為 `_cooking.jpg`；同步更新 fishData 的 cookingPhoto

**E. 照片 ID 放錯（某筆的照片跑到別筆的槽）**
- 現象：`N_fish.jpg` 和 `(N-1)_cooking.jpg` 內容完全相同
- 處理：刪除錯誤 ID 的那張；受影響欄位改為 `null`

**F. `_cooking` 實際上是未烹調的生魚照**
- 處理：改名為 `_fish2.jpg`（若 fish 已有）；cookingPhoto 改為 `null`

**G. 完全無關的照片（餐廳外觀、貝殼標本等）**
- 處理：直接刪除；對應欄位改為 `null`

### 審核工具

用 Claude 視覺逐批讀圖時，每批不超過 40 張，以 `Read` 工具逐一讀取，不可猜測。

---

## 條目搬移規則（重要）

當一筆條目的圖片檔名需要改 ID（例如 `06_*` → `47_*`），**必須同步處理以下三件事**，缺一不可：

1. **images/ 照片檔案**：`{old_id}_otolith.jpg`、`_fish.jpg`、`_cooking.jpg` 全部改名
2. **habitatDiagram SVG**：`{old_id}_habitat.svg` 也要改名為 `{new_id}_habitat.svg`
3. **fishData 條目內容**：舊 ID 的魚名、日期、地點、描述、habitat、fbUrl 等所有欄位，一起移到新 ID 的條目

照片、SVG、fishData 是同一筆條目的三個部分，任何 ID 搬移都必須三個一起動。

---

## 網頁排序規則

- 卡片顯示順序：**依 ID 降冪**（最新 ID 在最上面）
- 實作：`filteredFish()` 函式最後的 `.sort((a, b) => b.id - a.id)`
- 不用日期排序，因為 date 可能是 null，也可能與 ID 順序不一致

---

## 本機預覽
```
python -m http.server 8765 --directory "D:/耳石和貝殼塚/耳石圖鑑"
```
瀏覽器開 `http://localhost:8765`

## commonName（俗名）填寫規則

- **外國魚種**：用該國語言的慣用名稱當俗名（日本魚用日文名，例如：姬鱒 → ヒメマス、香魚 → アユ、日本鰤 → ブリ）
- **中文名即俗名**（正式中文名與民間叫法完全相同）：commonName 填入與 chineseName 相同的名稱（例如：鬼頭刀、秋刀魚、白鯧）
- **俗名是縮寫或簡稱**：填縮寫（例如：白帶魚 → 白帶、尖鰭細身飛魚 → 飛魚）
- **確實沒有通用俗名**：填 null，不勉強

---

## 資料結構（fishData 陣列每筆欄位）
```javascript
{
  id, commonName, folkName, scientific, family,
  date, location, cooking,
  lat, lng,                        // 地圖定位
  otolithPhoto, fishPhoto, fishPhoto2, fishPhoto3,  // 生魚照最多三張，無照片填 null（省略即可）
  cookingPhoto, cookingPhoto2,             // 料理照最多兩張，無照片填 null（省略即可）
  fbUrl,
  habitat,                         // 習性介紹文字（\n 換行）
  habitatTFRI,                     // 台灣魚類資料庫記載文字
  habitatRefs: [{label, url}],     // 參考連結（FishBase / TFRI）
  habitatDiagram,                  // 棲地示意圖路徑，如 "images/01_habitat.svg"
  notes                            // 耳石筆記全文（FB 原文）
}
```

---

## 棲地示意圖繪製規範

新增 `{id}_habitat.svg` 時，依照以下規範維持全套一致風格。

### 版面（固定不變）
- `viewBox="0 0 500 195"`，外框 `<rect>` `stroke="#2a4060"` `rx="6"`
- 字型一律 `font-family="Microsoft JhengHei,sans-serif"`

### 右側深度尺
- 垂直線：`x1="478"` `x2="478"`，`stroke="#8a9bb0"` `opacity="0.5"`
- 刻度線：x 從 474 到 478
- 數字：`x="480"`，字型 8.5px，顏色 `#8a9bb0`
- 主要棲息帶刻度改用金色 `#c9a84c`，`stroke-width="1.2"`

### 標籤位置
- 左上棲地名稱：`fill="#8ab8d0"` 9.5px
- 左側深度範圍：`fill="#c9a84c"` 9px
- 底部中央地理分布：`fill="#b09040"` 9px，`text-anchor="middle"`
- 主要棲息帶文字：`fill="#c9a84c"` 8.5px（兩行）

### 水體漸層
| 類型 | 色碼 |
|------|------|
| 深海 / 大陸棚 | `#1a3a5c` → `#0a1828`（35%）→ `#020610` |
| 熱帶珊瑚礁 | `#1a6080` → `#0e3855`（55%）→ `#082030` |
| 熱帶加光線 | 額外加 `stroke="#3abcd8"` `opacity="0.04"` 斜線 3 條 |

### 水面波紋
- `stroke="#3abcd8"` 或 `#4a9abc"`，`stroke-width="1–1.5"`，`opacity 0.3–0.75`
- 用 Q 命令畫波浪（x 每 26px 一個峰谷）

### 底質畫法
**沙泥底**
- `<path>` 曲線 Q 命令，`linearGradient` 棕黃色調
- 沙紋：細 `<path>` Q 命令，`stroke="#a09070"` `opacity="0.55"`

**礁岩底（鋸齒型）**
- 主體：多點 `<polygon>`，`fill="url(#rock)"`
- 受光亮面：同形狀偏移的 `<polygon>`，`opacity="0.7"`
- 裂縫：`<line>` `stroke="#141008"` `stroke-width="1.4–1.8"` `opacity="0.5–0.6"`

**珊瑚礁（圓頂 Spur-and-groove）**
- 每個 Spur：cubic bezier `<path>`，`fill="url(#rock04a)"` 深棕
  - 受光亮面：`<path>` 覆蓋上緣，`fill="url(#rock04b)"` 稍亮，`opacity="0.58"`
  - 頂部突出：小 `<polygon>`
  - 裂縫：`<line>`
- Sandy groove：`<path>` Q 曲線，深沙色 `linearGradient`
- 沙紋：`<path>` Q 命令，`stroke="#3a3020"` `opacity="0.5"`

**深海礁坡（剖面圖）**
- 主坡：斜向大 `<polygon>`
- 上緣鋸齒：次 `<polygon>` `opacity="0.4"`
- 岩層紋理：`<path>` Q 命令，平行坡向，`stroke="#3a3828"` `opacity="0.38–0.5"`
- 棚坡轉折點：`<circle>` `fill="#c9a84c"` `opacity="0.45"`

### 魚的畫法
- 身體：`<ellipse>`；背鰭 `<path>`；尾鰭兩葉 `<line>` `stroke-linecap="round"`
- 眼睛：大眼魚（燧鯛型）`r="8.5"` + 兩層反光點；一般魚 `r="3–4.5"`
- 游泳角度：`rotate(N)` 對齊坡度（大陸棚 ~18° 坡 → `rotate(18)`）
- 主角魚 `opacity="0.90–0.92"`，背景魚 `opacity="0.78–0.82"`
- 特殊特徵：
  - 尾鰭白邊（白緣星鱠）→ `stroke="#ffffff"`
  - 絲狀尾延伸（長尾濱鯛）→ 額外細 `<line>`
  - 埋沙（沙鮻）→ 沙色 `<ellipse>` 蓋住下半身

### 珊瑚畫法
**枝狀珊瑚**
```svg
<g stroke="#c06020" fill="none">
  <!-- 主幹 + 分枝 line，端點 circle fill="#e08040" -->
</g>
```
**塊狀珊瑚**
```svg
<ellipse fill="#7a5828"/>
<!-- 弧線紋理 path stroke="#a07040" -->
<!-- 表面點 circle fill="#8a6030" -->
```

### 標注箭頭
- `<marker>` 定義箭頭，`fill="#c9a84c"`
- 連接線：`stroke="#c9a84c"` `stroke-dasharray="3,2"`
- 文字：`fill="#c9a84c"` 9px

### 海雪粒子（深海場景）
- 散布 5–9 個 `<circle>`，`fill="#fff"` `r="0.7–1.2"`，`opacity="0.15–0.32"`
- 集中在水體上半部

## 捕獲方式（captureMethod）規範

### 支援的 15 種方式（對應 CAPTURE_ICONS key）
`一支釣`、`延繩釣`、`拖網`、`底拖網`、`中層拖網`、`圍網`、`定置網`、`刺網`、`流刺網`、`籠具`、`養殖`、`梁漁`、`棒受網`、`手鏢`、`曳繩釣`

> 手釣已合併入「一支釣」；梁漁為日本傳統竹木柵欄河川漁法（やな漁）；棒受網為夜間集魚燈誘魚＋大型撈網，主要用於秋刀魚；手鏢涵蓋手鏢旗魚法（從船頭投擲）與潛水鏢魚（水下射魚）；曳繩釣為船行拖釣（trolling），船尾拖線掛擬餌誘魚，鬼頭刀、旗魚常用

### 拖網三類的填法（重要）
| 填入值 | 條件 |
|--------|------|
| `拖網` | 來源只寫「拖網」，未指明類型 |
| `底拖網` | 來源**明確寫出**底拖網，且棲地為砂泥底（無礁岩） |
| `中層拖網` | 來源**明確寫出**中層拖網，或明確描述在水中層作業 |

**礁岩排除規則**：棲地有礁岩的魚種絕對不會有底拖網（網子會壞掉），可直接排除。

### 資料查核順序（每筆魚）
1. `description`（耳石筆記）—— 有明確漁法記載**必須納入**，第一優先
2. `habitatTFRI` 文字 —— 通常有「可用 XXX 漁獲」
3. 台灣魚類資料庫（fishdb.sinica.edu.tw）
4. 湧升海洋部落格（https://oceaninc.pixnet.net/blog）
5. FishBase —— 封鎖自動抓取（403），需瀏覽器手查
6. 整合後**只有一種方式**時，須再查 FAO、Wikipedia 等補充
7. 來源只寫「拖網」→ 填 `拖網`，不可自行推斷為底拖或中層

---

## 常用地點座標

| 地點 | lat | lng | 備註 |
|------|-----|-----|------|
| 草屯三合院 | 23.9969545 | 120.6644118 | 南投縣草屯鎮墩煌路三段138巷5號 |

---

### 已完成的示意圖
| 檔名 | 魚種 | 特色 |
|------|------|------|
| 01_habitat.svg | 達氏橘燧鯛 | 大陸棚緩坡 ~18°，深度 0–600m |
| 02_habitat.svg | 白鯧 | 淺海沙泥底 |
| 03_habitat.svg | 長尾濱鯛 | 深海礁岩陡坡剖面，100–400m，絲狀尾 |
| 04_habitat.svg | 白緣星鱠 | 熱帶珊瑚礁，三大圓頂礁岩塊 |
| 05_habitat.svg | 山女魚 | 淡水溪流 |
| 06_habitat.svg | 多鱗沙鮻 | 紅樹林河口＋近海沙泥底，埋沙標注 |
