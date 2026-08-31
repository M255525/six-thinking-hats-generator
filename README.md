# 六頂思考帽分析產生器

把愛德華．狄波諾（Edward de Bono）的六頂思考帽方法做成填問題主題即可用的提示詞產生器：選一頂帽子看單一角度，或依「白→紅→黑→黃→綠→藍」固定順序一次戴上六頂，逐步產出結構化分析。

🔗 **線上使用**：<https://m255525.github.io/six-thinking-hats-generator/>

不需要授權序號，開啟網址即可使用。

## 這是什麼

六頂思考帽把「立場」拆成六種可以輪流切換的角度，避免討論時事實、情緒、風險、創意混在一起互相打架：

| 帽子 | 核心特性 |
|---|---|
| ⚪ 白帽 | 客觀事實與數據 |
| 🔴 紅帽 | 直覺與情緒感受 |
| ⚫ 黑帽 | 風險與批判思考 |
| 🟡 黃帽 | 正面價值與可行性 |
| 🟢 綠帽 | 創意與新可能 |
| 🔵 藍帽 | 流程掌控與統整 |

「🖊 組成提示詞」是純前端字串組裝（不需金鑰、可複製）；「🚀 送給 AI 生成分析」是選用功能，需自備 API 金鑰，直接取得 AI 產生的分析結果，回覆的 markdown 表格會自動轉成真正的表格顯示。

## 功能

- **兩種模式**：單一帽（點選任一頂帽子，整頁配色會跟著換成該帽子的顏色）／六頂依序（固定順序一次產出六份分析，逐一呼叫、單頂失敗不影響其他頂）
- **重新生成不必全部重打**：六頂依序模式再次生成時只補未完成或失敗的帽子；也可勾選「🔁 強制重新生成」指定某幾頂已成功的帽子重打，不動其餘結果
- **內建範例**：5 組虛構情境（調漲售價、導入 AI 客服、全面遠距辦公等），一鍵套用快速上手
- **BYOK**：支援 Claude／OpenAI／Gemini／OpenRouter 四選一，API 金鑰只存在瀏覽器 localStorage，不經過任何後端伺服器
- **已儲存的分析**：可將主題與產出（連同 AI 產生結果）存成有名字的紀錄，之後載入、下載 .txt 或刪除
- **匯出 PDF**：AI 分析結果可直接列印／存成 PDF，內容適合直接閱讀（不會出現 `#`／`*` 等 Markdown 符號），並附浮水印
- **加入主畫面**：支援 PWA 安裝，手機與電腦瀏覽器皆可加入主畫面當作 App 開啟，介面已針對手機畫面優化

## 怎麼用

1. 開啟 <https://m255525.github.io/six-thinking-hats-generator/>
2. 在「① 問題主題」填入一句話描述的問題，或套用範例
3. 選擇模式（單一帽／六頂依序），單一帽模式可點選帽子切換角度
4. 按「🖊 組成提示詞」取得可複製的提示詞文字；或展開「API 連線設定」貼上你自己的金鑰，按「🚀 送給 AI 生成分析」直接取得分析結果
5. 滿意的結果可在「④ 已儲存的分析」取名儲存

詳細操作說明見 [manual.html](https://m255525.github.io/six-thinking-hats-generator/manual.html)。

### API 金鑰申請網址

| 服務商 | 申請網址 |
|---|---|
| Claude（Anthropic） | <https://console.anthropic.com/> |
| OpenAI | <https://platform.openai.com/api-keys> |
| Gemini（Google AI Studio） | <https://aistudio.google.com/apikey> |
| OpenRouter | <https://openrouter.ai/keys> |

## 技術架構

純前端單檔工具，**沒有任何建置流程、框架、npm 依賴**：

| 項目 | 做法 |
|---|---|
| 提示詞組裝 | 純前端字串模板，不連網 |
| AI 生成分析 | 瀏覽器直接 `fetch` 你選擇的 LLM 服務商官方 API（無後端代理） |
| 金鑰儲存 | `localStorage`，只在使用者自己的瀏覽器裡 |
| markdown 表格解析 | 純前端正則解析，失敗自動退回原始文字顯示 |
| 頂部跑馬燈 | 與工作區其他工具共用同一個公告來源（可選、失敗不影響主功能） |

## 本機開發

不需要任何建置工具或安裝依賴，純靜態檔案：

```bash
git clone https://github.com/M255525/six-thinking-hats-generator.git
cd six-thinking-hats-generator
python -m http.server 8804
```

開啟 `http://localhost:8804`。

## 檔案結構

```
index.html         主程式（跑馬燈 + 帽架 + 問題主題 + 提示詞組裝 + AI 生成 + 儲存清單 + 匯出 PDF + PWA）
manual.html         操作手冊
manifest.json        PWA 安裝設定
service-worker.js    PWA 離線快取（network-first）
icons/               PWA 圖示
CLAUDE.md            開發筆記／架構決策紀錄
```

## 隱私與資料

本 repo 公開的只有程式碼。你填寫的問題主題、組成的提示詞、AI 產生結果只存在自己瀏覽器的 localStorage；按下「送給 AI 生成分析」時，這些內容會直接連線送到你選擇的 AI 服務商，不經過本工具作者或任何第三方伺服器。

## 授權/用途

僅供教學與個人使用，禁止未經授權公開發布、販售或商業化使用。
