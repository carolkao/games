# 小遊戲樂園 — 專案備忘（給之後的 Claude session）

## 基本資訊
- 本機路徑：`/Users/ckao/Carol/Game`；遠端：`https://github.com/carolkao/games`（public）。
- 線上網址：https://carolkao.github.io/games/ ——GitHub Pages，legacy build，來源是 `main` 的根目錄；push 到 `main` 後約 1–2 分鐘自動部署。
- 純靜態、沒有 build step。`index.html` 是選單，`games/<name>/index.html` 是各自獨立的單檔遊戲。
- 選單 `GAMES` 陣列裡的遊戲**一律全部顯示**（早期的 `hidden` / `?query` 隱藏機制已移除，不要加回來）。新增遊戲的步驟見 `README.md`。

## 本機預覽
- `.claude/launch.json` 已設定 `game`（`npx serve -l 8080 .`），用 `preview_start {name:"game"}` 開。
- `serve` 會把 `xxx/index.html?query` 轉址成 `/index` 並丟掉 query，所以直接開 `/games/<name>/`，要改狀態就用 JS 設變數。
- 驗證遊戲的手法（直接驅動 `update()`、截圖的時機、踩過的坑）寫在 [`games/isometric-town/NOTES.md`](games/isometric-town/NOTES.md)。

## Push / 認證
- gh token 放在使用者本機的 `~/.zsh/shared_env.zsh`（不在 repo 裡，也不要寫進 repo）。
- 要 push 時，`source` 和 `git push` 必須在**同一個 Bash 指令**：`source ~/.zsh/shared_env.zsh && git push`。`gh auth setup-git` 已經跑過。不要去探測憑證，也不要用 `git -c credential.helper=...`。
- 2026-11-26 實測：`shared_env.zsh` 設進環境的 `GH_TOKEN` 已失效（`gh auth status` 說 invalid，`git push --dry-run` 回 `Authentication failed`，`gh pr list` 回 401）。`shared_env.zsh` 最後修改時間是 2026-11-10，晚於最後一次成功 push（約 11-07），無法確定是舊 token 到期還是之後換成了無效的新值。
- `gh auth status` 同時顯示 keyring 裡有 `carolkao` 的有效登入（scopes：repo、workflow…），只是被環境變數 `GH_TOKEN` 蓋掉。可能的解法是 push 時 `env -u GH_TOKEN -u GITHUB_TOKEN git push`，**尚未實測**，改用前先問使用者；否則請使用者更新 `shared_env.zsh` 裡的 key。

## 慣例
- `games/forest-cabin/` 是立體小鎮的玩法參考原版，使用者要求**不要改動**；立體版是獨立的 `games/isometric-town/`。
- `games/correct-typos/`（改錯字打怪）是另一個 session 加的，題庫 docx 放在它底下的 `questions/`。
- commit 結尾加：`Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>`；PR 描述結尾加：`🤖 Generated with [Claude Code](https://claude.com/claude-code)`。

## 分支現況（2026-11-26）
- `main` = `origin/main`。
- `feature/isometric-town`（本機＋遠端）內容已 merge（PR #2），只多兩個與 `main` 重複的 correct-typos commit（`691516f`、`342039d`）。可以安全刪除，尚未刪。
- `origin/claude/99-multiplication-game-4e429d` 是別的 session 建的，沒動過。
