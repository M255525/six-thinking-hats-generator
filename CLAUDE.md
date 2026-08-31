# CLAUDE.md — six-thinking-hats-generator

「六頂思考帽分析產生器」——單檔前端工具，把愛德華．狄波諾（Edward de Bono）的六頂思考帽方法做成填問題主題即可用的提示詞產生器：可選單一帽視角，或依「白→紅→黑→黃→綠→藍」固定順序一次產出六頂分析，也可選填 BYOK API 金鑰直接取得 AI 生成的完整分析（含自動解析成表格）。

## 緣起

工具的核心提示詞範本是使用者直接貼給我的原始格式，`template()` 逐字保留其措辭與結構，只替換思考帽色與問題主題兩個變數，未自行改寫。

## 架構

單一 `index.html`：內嵌 CSS/JS、無外部資源、無建置步驟。視覺主題刻意採**淺色「白板會議室」風格**（`--bg #eef1f5`），是工作區裡少數的淺色調工具，與其餘多數深色系姊妹工具做出區隔——呼應「六頂思考帽」本身就是白板／會議室工作坊教學情境。

- **簽名元素**：六個內嵌 SVG 帽子圖示（`hatSvg()`）以各帽子的真實代表色繪製，並排成「帽架」（`.hat-rack`）。點選任一頂帽子會把 `<html data-hat="...">` 換掉，CSS 依 `[data-hat=...]` 選擇器整組換色（`--accent`／`--accent-soft`／`--accent-ink`），讓整頁配色跟著「戴上那頂帽子」——這是全頁唯一的「換色互動」，其餘按鈕／面板一律沿用中性配色，避免喧賓奪主。
- **兩種模式**（`state.mode`）：`single`（選一頂帽子產生一份分析）／`sequence`（固定依「白,紅,黑,黃,綠,藍」六帽依序各產生一份，這個順序直接取自使用者原始範本裡 `{{思考帽||白,紅,黑,黃,綠,藍}}` 的列舉順序，不可任意調換）。`HAT_ORDER` 陣列是唯一真實來源，新增/調整帽子時只改這裡即可。
- `template(hatChar, topic)`：組成提示詞的唯一函式，`hatChar` 是單一色字（如「白」），輸出字串裡是 `六頂思考帽中${hatChar}帽的觀點`——修改文字時注意「帽」字是模板自己補的，不要把它塞進 `HATS[key].name`。
- **核心互動模型**（與 `ai-prompt-generator` 同一套設計哲學）：「🖊 組成提示詞」純前端字串組裝，不需金鑰；「🚀 送給 AI 生成分析」選用功能，串接使用者自己的 BYOK LLM。單一模式呼叫一次；依序模式**逐一 await**六次呼叫並即時更新每頂帽子的卡片狀態（不會平行發送六個請求，也不會因某一頂失敗就中斷其餘五頂——失敗的那頂卡片顯示錯誤訊息，其餘照常）。
- `parseMarkdownTable()`：把 AI 回覆中的 GitHub 風格 markdown 表格轉成真正的 `<table>`（`.hat-table`）；抓不到表格格式（例如 AI 沒有照格式回覆）就回傳 `null`，外層退回原始文字的 `pre-wrap` 顯示，不會拋錯或空白。
- **BYOK AI 串接**：與 `ai-prompt-generator`／`Prompt/`／`sbir-generator/sbir-gen-s` 同一套模式——瀏覽器直連 `fetch()`，Claude 需 `anthropic-dangerous-direct-browser-access` header，Gemini 金鑰放 `x-goog-api-key`，OpenAI/OpenRouter 用 Bearer；429/500/503/529 自動重試最多 2 次。設定存 `localStorage`（key: `sixHatsApiConfig`）。
- 狀態存 `localStorage`：`sixHatsState`（topic/mode/activeHat/assembled/aiOutput）、`sixHatsSavedItems`（已儲存清單）、`sixHatsMarquee`（跑馬燈快取）。

## 本次刻意不做（範圍依使用者實際請求，未主動擴增）

- **無序號授權**：使用者僅要求跑馬燈／使用警語／創作者資訊／使用手冊，未提及序號授權或鎖定，比照 `coffee-ig-planner`／`mandala-thinking`／`social-post-grader` 無授權的先例，不主動加鎖。
- **無 PWA／無桌面版 exe／未部署 GitHub Pages**：未被要求，且會引入 manifest/service-worker/打包等額外維護面。若日後要加，PWA 做法可直接比照 `ai-prompt-generator` 的 5 個判斷式（iOS／macOS Safari 相容性）整段複用。
- **無 Google Sheet 後端**：跑馬燈是唯一對外呼叫，沿用工作區既有共用授權伺服器（與 `Prompt/`／`ai-video-studio` 系列同一個 Google Sheet），除此之外零後端。

## 頂部共用跑馬燈

做法逐字比照 `ai-prompt-generator/index.html`（同一個共用 Google Apps Script 端點與 Google Sheet），`localStorage` key 改為 `sixHatsMarquee`。改跑馬燈內容直接編輯共用 Sheet 即可。

## 隱私與警語

無伺服器端經手使用者資料；問題主題、組成結果、AI 產生結果、已儲存清單皆只存在使用者瀏覽器的 localStorage。首頁與手冊皆明列使用警語：AI 生成內容需自行查核、請勿輸入真實個資或機密資料、僅供教學與個人使用禁止商業化。

## 指令

無建置/測試指令。修改 `index.html` 後直接用瀏覽器開啟驗證，或暫起 `python -m http.server 8804` 測完關閉（8804 是本專案佔用的埠號，記入工作區埠號清單）。修改內嵌 `<script>` 後可用以下方式快速檢查語法：

```bash
python -c "
import re
html = open('index.html', encoding='utf-8').read()
open('_check.js','w',encoding='utf-8').write(re.findall(r'<script>(.*?)</script>', html, re.S)[0])
"
node --check _check.js
```

已用 Playwright 端對端驗證：帽子切換即時換色（`data-hat` + CSS 變數）、單一帽／六頂依序兩種模式的提示詞組裝、markdown 表格解析、已儲存清單的儲存／載入流程，皆正常運作，主控台無錯誤（favicon 404 除外，無影響）。

## 本次未做（後續視需要再處理）

- git 初始化（比照工作區慣例每個子資料夾各自是獨立 git 儲存庫）尚未執行，需使用者確認後再做
- 根目錄 `專案目錄.docx` 尚待補上本專案的列
