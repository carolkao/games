# 小遊戲樂園

一個放小遊戲的靜態網站。`index.html` 是入口頁，列出所有遊戲；每個遊戲各自獨立放在 `games/<game-name>/` 資料夾底下。

## 目前的遊戲

- [`games/multiplication-tetris/`](games/multiplication-tetris/) — 九九乘法消消樂
- [`games/forest-cabin/`](games/forest-cabin/) — 森林小屋

## 新增一個遊戲

1. 在 `games/` 底下建一個新資料夾，裡面放該遊戲的 `index.html`（純靜態頁面，不需要 build step）。
2. 在根目錄的 `index.html` 裡的 `GAMES` 陣列多加一筆 `{ icon, title, description, path }`，`path` 指到剛剛的資料夾。

## 本機預覽

```
npx serve -l 8080 .
```

或用 Claude Code 的 `.claude/launch.json`（已設定好 `game` 這個 preview server）。
