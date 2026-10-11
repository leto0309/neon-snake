# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概要

霓虹風格貪食蛇。整個遊戲只有 `index.html` 一個檔案，HTML、CSS、JavaScript 全部寫在裡面。沒有套件、建置工具、linter 或測試框架，請維持零依賴、單一檔案。

## 執行

直接用瀏覽器開啟 `index.html`，或在專案資料夾啟動本機伺服器：

```bash
python -m http.server 8000
```

然後前往 http://localhost:8000。沒有自動化測試，修改後要在瀏覽器實際玩一次來驗證；觸控滑動需要用手機或 DevTools 的裝置模擬來測。

## 架構

全部 JS 包在一個 IIFE 裡，狀態放在閉包內的 `let` 變數（`snake`、`dir`、`dirQueue`、`food`、`score`、`tickMs`、`particles`……）。

- **狀態機**：`state` 只有 `'ready' | 'running' | 'paused' | 'over'` 四種值，由 `start()` / `pause()` / `resume()` / `gameOver()` / `win()` 切換。鍵盤、按鈕和觸控處理都依 `state` 決定行為，新增輸入時要照顧到每一種狀態。
- **邏輯與繪圖分離**：`requestAnimationFrame(loop)` 每一幀都會呼叫 `draw()`，但遊戲邏輯 `step()` 以固定時間步 `tickMs` 推進。落後時最多補 3 步，避免切回分頁時蛇瞬移。粒子和蛇身色相（`hue`）則是每幀更新，不受 `tickMs` 影響。
- **座標系統**：蛇和食物用格子座標（0…`GRID-1`，`GRID = 20`）；粒子用畫布像素座標。`cell = canvas.width / GRID` 已包含 `devicePixelRatio`，所以繪圖的長度都要用 `cell` 的倍數來表示，不要寫死像素值。
- **轉向佇列**：`turn()` 把方向放進 `dirQueue`（最多 3 個），並和*佇列最後一個*方向比較來擋掉反向，這樣快速連按兩次也不會原地掉頭。`step()` 每步只取出一個方向。
- **碰撞**：沒吃到食物時尾巴會移走，所以檢查撞到自己時要排除最後一節（`snake.slice(0, -1)`）。修改移動邏輯時要保留這個細節。
- **速度與等級**：每吃一個食物得 10 分，`tickMs` 減 4，最低到 `MIN_TICK`。HUD 的「速度」是由 `tickMs` 換算出來的（`1 + round((BASE_TICK - tickMs) / 8)`），改了減速幅度要一起確認這條公式。
- **勝利條件**：`snake.length === GRID * GRID`。
- **最高分**：存在 `localStorage` 的 `neon-snake-best`，讀寫都包在 try/catch 裡（瀏覽器可能封鎖儲存）。

## 慣例

- 介面文字、程式註解和 README 都使用繁體中文（`lang="zh-Hant"`）。
- 配色在 `:root` 有 CSS 變數，但 canvas 繪圖直接寫死同樣的色碼（如 `#ff3fa4`、`#39ffb6`、`#07060f`）。換配色時兩邊都要改。
- `showOverlay()` 用 `innerHTML` 設定內文，只能放靜態字串或數字。
- 操作方式記在三個地方：`keydown` 處理程式、頁面底部的 `.hint` 文字，以及 README 的操作表格。改按鍵時三處要一起更新。
