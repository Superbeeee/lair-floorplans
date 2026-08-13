# lair-floorplans

L'AiR Space 展館平面圖（floorplan）的資料來源。透過 GitHub Pages 以靜態檔提供，
前台在載入展館頁時直接 fetch 這裡的 JSON。

## ⚠️ 改這裡會直接影響線上機台

這個 repo **沒有佈版流程做為緩衝**。push 到 `main` 之後，現場機台只要重新整理就會吃到新版本。

- 改之前先確認你知道這份圖正在被哪個展場使用
- 改完自己開一次前台確認，不要 push 完就走
- 不確定的話先開 PR，不要直接推 `main`

## 目錄結構

```
<客戶或案件代號>/
  <樓層>.json     平面圖資料（幾何 + 攤位 + node 綁定）
  <樓層>.jpg      底圖
```

目前：

| 路徑 | 說明 |
|---|---|
| `taipei-trade/1f.json` `2f.json` `3f.json` | 台北世貿 1–3 樓 |

## 檔名不可以改

JSON 的網址是寫死在前端 `src/data/specialProjects.ts` 裡的。**檔名一改，前端就要跟著改程式並重新佈版**，
等於失去「改圖不用佈版」的意義。

所以：

- ❌ 不要用 `2f-0812.json` 這種帶日期的檔名
- ✅ 檔名固定為 `2f.json`，版本紀錄交給 git history

要回溯某個特定版本，用帶 commit SHA 的不可變網址：

```
https://raw.githubusercontent.com/Superbeeee/lair-floorplans/<commit-sha>/taipei-trade/2f.json
```

## `baseImage.url` 必須是完整網址

底圖要跟 JSON 放在同一個 origin，並在 JSON 裡寫完整網址：

```json
"baseImage": {
  "url": "https://superbeeee.github.io/lair-floorplans/taipei-trade/2f.jpg",
  "width": 1700,
  "height": 1700
}
```

寫成 `/custom/...` 這種站內絕對路徑的話，會對「L'AiR 網站自己的 origin」解析，
底圖就還是綁在 dist 裡，換圖仍需佈版。

`width` / `height` 必須等於圖檔實際像素 —— grid 座標換算以此為基準，填錯整張路線會偏。

## 前端怎麼取用

前端 `src/utils/floorPlanSource.ts` 依序退讓：

1. **遠端**（本 repo，`cache: 'no-store'`）
2. **localStorage** 上次成功的版本
3. **打包在 dist 內的備援**

每一段都過形狀驗證，所以推壞的 JSON 不會讓平面圖半殘，會自動退到上一段可用的版本
（但也代表**你可能以為推上去了、現場卻還在用舊版**——改完務必自己確認一次）。

## 產生 JSON 的工具

L'AiR 後台的平面圖編輯器：`/overview/exhibition-editor/{專案數字 id}`
畫完後「匯出 JSON」，改好 `baseImage.url` 再放進這裡。
要續編就用編輯器的「匯入 JSON」載入現有檔案。
