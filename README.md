# VocaUp／單字複習前端變更說明

**文件更新時間戳記：** `2026-05-06T00:00:00+08:00`（2026 年 5 月 6 日）

以下彙整 **`index.html`** 中與「自選詞表／雲端章節複選」相關的功能擴充與行為修正（實際仍以程式碼為準）。

---

## 新增功能

### 1. 合併雲端章節進自選：卡片版型（方案三）

- **上排**：縮短書名（例：大全、補充等規則對應短稱，過長字串會截斷）。
- **次行**： **`N 章`** 徽章（與書名分行顯示）。
- **DAY 藥丸列**：列出所選 **`DAY*`**；當總寬超過可視區時，以 **CSS 動畫**做 **左右往返**（來回展示），並可 **`:hover` / `:active`** 暫停；遵守 **`prefers-reduced-motion: reduce`**。
- **資料欄位**：合併寫入歷史時會附 **`mergedFromFirebase: { categoryKey, dayKeys }`**，不必只靠 `displayName` 字串。

### 2. DAY 藥丸動畫的穩定量測

- 抽出 **`applyCustomDeckChipMarqueeSizing`**，搭配 **`ResizeObserver`** 監聽外層與軌道。
- **`renderCustomDeckPickGrid`** 結束後以 **`remeasureAllCustomDeckChipMarquees`** 多次排程（`rAF`、`setTimeout`、`document.fonts.ready`），避免雙欄排版或字型載入前先量到寬度 0，造成 **只有某一張卡在動**。

### 3. 自選列表操作介面

- 單張卡片上的 **重置／刪除** 按鈕改為 **上下排列**（`flex-col`），較適合窄卡寬。

### 4. Firebase 複選進自選的流程說明（既有能力之補充）

- 雲端 **DAY** 長按可進入 **複選合併→自選**；本次主要補強的是選取／點擊邏輯（見下文「修正問題」）。

---

## 修正問題

### 1. 長按複選後，點另一張 DAY 要按兩次才選得到

- **原因**：長按完成後用 **全域** `suppressNext*` 擋誤觸 `click`；放手時若未對「長按那一張」產生 `click`，標記殘留，**下一張卡的第一下 `click` 被錯吃**。
- **修正**：改為 **`suppressNextFirebaseDayClickForDayKey`**，只擋 **「長按起點」那一個 `dayKey`** 的下一個 `click`；點其他卡 **第一下即可** 加入／取消複選。

### 2. 合併詞表進度同步後，藥丸／合併資訊遺失

- **`syncCurrentDeckToHistory`** 在更新同一 `displayName` 的歷程項目時，會 **保留** 既有 **`mergedFromFirebase`**，避免複習進度寫回後 UI 後設資料被蓋掉。

### 3. 舊自選資料無 `mergedFromFirebase`

- 由 **`displayName`** 以 **`書名／DAY1＋DAY2…`**（或僅 **`DAY1＋DAY2…`**）做 **後援解析**；僅當各段符合 **`DAY`+數字** 模式才視為合併卡，減少一般檔名誤判。
