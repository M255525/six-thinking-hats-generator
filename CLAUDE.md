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
- **完成進度條**（`#aiProgressWrap`/`setProgress(done, total)`）：呼叫 AI 時顯示視覺化進度，單一帽模式是 0/1→1/1，依序模式是即時 0/6→…→6/6——續跑時（見下）已沿用的成功結果會讓初始進度直接從對應的完成數開始，而不是每次都從 0 歸零；成功與失敗都算「已處理」推進進度（進度條反映批次處理進度，不是成功率，成功/失敗細節由下方每頂帽子的卡片與 `aiReport` 呈現）。已用 Playwright + 延遲 mock `fetch` 驗證中途進度（3/6=50%）與最終進度（6/6=100%）皆正確即時更新。
- **依序模式重試會續跑、不會全部重打**：每筆結果（成功或失敗）都多存一個 `prompt` 欄位，記錄它是用哪一段組成後的提示詞文字生成的。再次按「送給 AI 生成分析」時，會先比對 `state.aiOutput.sequence[i].prompt` 是否等於這次的 `items[i].text`——相等且沒有 `error` 就直接沿用、不重打；否則（含尚未生成過、上次失敗、或主題已改導致 prompt 對不上）才呼叫 API。這是為了避免使用者因某一頂失敗（例如金鑰打錯）重試時，把前面幾頂已成功、花過 API 費用的結果一起丟棄重打。已用 Playwright + mock `fetch` 驗證：中途失敗後重試只會呼叫未完成的那幾頂；改了問題主題後重試則會視全部結果過期、六頂都重新呼叫。
- `parseMarkdownTable()`：把 AI 回覆中的 GitHub 風格 markdown 表格轉成真正的 `<table>`（`.hat-table`）；抓不到表格格式（例如 AI 沒有照格式回覆）就回傳 `null`，外層退回原始文字的 `pre-wrap` 顯示，不會拋錯或空白。
- **BYOK AI 串接**：與 `ai-prompt-generator`／`Prompt/`／`sbir-generator/sbir-gen-s` 同一套模式——瀏覽器直連 `fetch()`，Claude 需 `anthropic-dangerous-direct-browser-access` header，Gemini 金鑰放 `x-goog-api-key`，OpenAI/OpenRouter 用 Bearer；429/500/503/529 自動重試最多 2 次。設定存 `localStorage`（key: `sixHatsApiConfig`）。
- 狀態存 `localStorage`：`sixHatsState`（topic/mode/activeHat/assembled/aiOutput）、`sixHatsSavedItems`（已儲存清單）、`sixHatsMarquee`（跑馬燈快取）。

## PWA 加入主畫面 ＋ 手機適用

`manifest.json`／`service-worker.js`／`icons/`／`#installBtn` 安裝 IIFE 逐字比照 `ai-prompt-generator` 已修過多輪 bug 的版本（見該專案 CLAUDE.md 的「PWA 加入主畫面」「iOS／iPadOS／macOS 相容性補強」「回饋機制與快取踩坑修正」三段）：network-first + 同源快取備援（不追求離線可用，`{cache:'reload'}` 繞過 HTTP 快取）；安裝腳本是獨立 IIFE，`notify()` 自己操作 `#toast` DOM，不依賴主程式 `showToast`（跨 IIFE 看不到會是 `undefined`）；iOS／macOS Safari 的 5 個判斷式（`isIOSDevice`/`isMacDesktop`/`isSafariEngine`/`isStandalone`/`fallbackMessage()`）原樣複用。圖示（`icons/`）是這次現畫的：藍底（`#2563eb`，對應 `--accent` 藍帽色）＋白色帽子剪影（跟 `index.html` 的 `hatSvg()` 同一個造型：橢圓帽簷＋圓頂帽冠），一次性產生腳本（`gen_icons.py`）用完即刪、未進版控。已用 Playwright 實測：192px 寬視窗下水平零溢出、320px 更窄視窗也零溢出，按鈕與帽架皆正常換行；`beforeinstallprompt` 有被 Chromium 偵測到（安裝按鈕走原生流程而非 fallback 分支，跟 `ai-prompt-generator` 當初的實測結果一致）。

沒有另外加 `@media (max-width:480px)` 之類更細的斷點——原本 `.hero`/`.type` 用 `clamp()`、`flex-wrap` 已經涵蓋大多數手機寬度，只在既有的 `@media (max-width:600px)` 基礎上微調就足夠通過測試，沒有為此新增大量手機專屬 CSS。

## 匯出 PDF（含浮水印）＋ 內容去 Markdown 符號

- **浮水印**：`#printWatermark`／`#wmImg` 這段（含內嵌 base64 圖檔）是用 Python 腳本直接從 `mandala-thinking/index.html` 逐行搬過來的（找到該行寫入目標檔案，全程沒有經過對話視窗顯示內容，避免大體積 base64 佔用 context），跟 `IPA_Kano`／`restaurant-feasibility-calculator`／`mandala-thinking` 用同一張已去背 PNG（480×297，「馬克老師 AI・工具・學習・成長」）。平常 `display:none`，只在 `@media print` 用 `position:fixed`+`opacity:.11` 顯示，每一頁列印都會重複出現。
- **列印範圍**：只印「③ 送給 AI 生成分析」的結果——`#topicPanel`／`#promptPanel`／`#savedPanel`／跑馬燈／頂欄／按鈕群一律在 `@media print` 隱藏；改用 `#printHeader`（列印時才 `display:block`）在最上方補上「問題主題／模式／產出時間」這行 context，因為問題主題欄位本身被隱藏了。六頂依序模式的 `.hat-card` 是 `<details>`，收合狀態列印會是空的，`printAiBtn` 點擊時會先把全部 `.hat-card` 的 `open` 設成 `true` 再呼叫 `window.print()`。
- **Markdown 符號清理**：`template()` 新增第 5 條規則，明確要求 AI 說明文字不要用 `#`／`*`，只有表格本身保留標準 Markdown 表格語法（因為 `parseMarkdownTable()` 靠它解析）。這只是「要求」，AI 不一定完全遵守，所以另外加了防呆：`stripBold()`（去掉 `**粗體**` 星號只留文字）用在表格的表頭／儲存格與 fallback 純文字；`stripMarkdownNoise()`（額外去標題井字號、條列符號改成「・」）只用在表格以外的自由文字（`parseMarkdownTable()` 的 before-text 與整段解析失敗時的 fallback），不會誤傷表格分隔列（`|---|---|`）本身，因為那一段在解析階段就已經被抽走。已用 mock `fetch`（內容故意夾帶 `#`／`**`／`*`）端對端驗證清理邏輯正確、且不影響表格解析。

## 部署

已推公開 GitHub repo `M255525/six-thinking-hats-generator`，用 `.github/workflows/deploy-pages.yml`（逐字複製 `mandala-thinking` 的版本）以 Actions workflow 部署 GitHub Pages（非 legacy branch-source；`gh api -X POST repos/.../pages -f build_type=workflow` 建立 Pages 站台後，`gh workflow run` 觸發第一次部署），已上線：<https://m255525.github.io/six-thinking-hats-generator/>（手冊：<https://m255525.github.io/six-thinking-hats-generator/manual.html>）。

## 本次刻意不做（範圍依使用者實際請求，未主動擴增）

- **無序號授權**：使用者僅要求跑馬燈／使用警語／創作者資訊／使用手冊，未提及序號授權或鎖定，比照 `coffee-ig-planner`／`mandala-thinking`／`social-post-grader` 無授權的先例，不主動加鎖。
- **無桌面版 exe**：未被要求，會引入打包/維護額外面（PWA 已於後續回合依使用者要求補上，見下）。
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

- 桌面版 exe 打包
- 序號授權（使用者本次未要求；若之後要鎖工具，比照 `ai-prompt-generator` 的「鎖整個工具 12 個月」模式加回）
