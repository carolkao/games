# 立體小鎮（isometric-town）開發筆記

單檔 `index.html`（約 1,200 行，canvas，無 build）。畫面參考手遊 Township 的等距風格，玩法是把 `games/forest-cabin/` v1~v3 搬過來：商店／工人 → 小屋升級 → 種子花園／花工人 → 花攤／客人／寵物。

## 技術重點
- **2:1 等距投影**：`isoWorld(c,r) = {x: ORIGIN_X+(c-r)*A, y: ORIGIN_Y+(c+r)*B}`，`A=32, B=16`；`isoPos` 再減掉鏡頭 `cam`。
- **深度排序**：地磚先畫，其餘物件放進 `drawables`，依 `c+r` 排序（painter's algorithm）；精靈都是 billboard。
- **鏡頭**：跟著玩家捲動。`VW/VH` 依裝置決定（手機全螢幕，桌面 760x560 縮放），世界大小 `WORLD_W/WORLD_H`。
- **手機判斷**：`matchMedia("(max-width: 700px), (pointer: coarse)")`（只看 touch 事件不可靠）。方向輸入經 `screenDirToGrid` 轉成格子方向。
- **圍欄碰撞**：`nearFence()` 用連續距離，半徑 `FENCE_BLOCK_R=0.58`。必須 > 0.5，否則相鄰柱子的圓之間有縫，斜著走會穿過去；用 `Math.round` 的格子判斷則會把玩家卡死在原地。
- **玩家**：向量畫的 chibi。`drawLimb(x1,y1,ang,len,width,color)`（`ang=0` 為垂直向下）、`drawTool`；採集揮動幅度 `sin(progress*PI)`；`P.harvestType` 決定工具與動作，`P.facing` 決定朝向。`drawCarriedItems` 把採到的東西堆疊在頭上。

## 進程（`gamePhase`）
| 階段 | 進入條件 | 內容 |
|---|---|---|
| 0 | 開局 | 採木頭／石頭、賣給小屋；三個商店：⚡速度（上限 3）、🎒背包（上限 3，容量 `10+5*等級`）、👷工人（上限 5，每個最多帶 6 個） |
| 1 | `UPG.speed>=3 && UPG.bag>=3 && workers.length>=3` | 小屋升級：投入 🪵25 🪨20 🪙50（可分次投入，工人也會幫忙投） |
| 2 | 小屋升級完成 → `unlockGarden()` | 種子商店、鬱金香花園、花工人（`FW_COST=80`，最多 3 個）；第二次小屋升級 🪵40 🪨30 🪙100 🌷15 |
| 3 | 第二次小屋升級完成 | 花攤（建造 🪵10 🪨5，花賣 25/朵）、客人自動來買、三隻寵物（`petShops`：旺財採集範圍+50%、咪咪工人速度+30%、小鳥生長速度+20%） |

數值都集中在檔案中段的常數：`UPG_COST`、`MAX_UPG`、`WOOD_PRICE/STONE_PRICE/FLOWER_PRICE`、`SEED_PRICE`、`HOUSE_UPGRADE_COST`、`HOUSE_UPGRADE2_COST`、`FSTAND_BUILD`。要調平衡就改這些。

購買／投入都是「站在附近停留」自動觸發（`BUY_TIME=1.2s`），沒有按鈕。

## 地圖座標（格子 c,r）
小屋 `(2,2)`；商店 `(2,6)` `(2,9)` `(2,12)`；種子商店 `(13,14)`；花農 `(18,14)`；花攤 `(12,17)`；寵物 `(15,17)` `(17,17)` `(19,17)`。

## 踩過的坑（已修，改動時別重犯）
- `drawShadow` 要在 `ctx.translate` **之前**呼叫，否則偏移被套兩次，會出現飄在旁邊的陰影斑塊。
- 小屋「賣出」區塊要放在「升級投入」區塊**之後**，否則賣掉了就湊不齊升級材料。
- 工人到小屋時要先拿身上的資源投入進行中的升級，再賣剩下的（不然工人只賺金幣）。
- 文字用 `fillTextClamped` 避免被畫布邊緣切掉（小屋標籤出過問題）。
- 沒種子時要有提示（toast），否則玩家不知道為什麼不能種；未解鎖的花園用脈動的 🔒 提示。
- 手機上沒有懸浮說明，所以用常駐的 `#objective` 橫幅顯示 `currentObjective()`。
- `catOwned()` 在 `WorkerAI` 裡以 `typeof catOwned==='function'` 保護（它定義在 class 之後）。

## 怎麼驗證（不靠真人鍵盤）
內建瀏覽器的分頁常在背景，`requestAnimationFrame` 會停，所以用 `javascript_tool` 直接驅動遊戲迴圈：

```js
// 用 IIFE 包起來，不然多次執行會因 const 重複宣告而報錯
(() => {
  UPG.speed = 3; UPG.bag = 3;
  for (let i = 0; i < 3; i++) workers.push(new WorkerAI(2, 7));
  for (let i = 0; i < 60; i++) update(1/60);   // 模擬 1 秒
  render();                                     // 強制重畫，截圖前一定要做
  return { gamePhase, gold: P.gold, obj: currentObjective() };
})()
// 2026-11-26 實測：gamePhase 0 → 1，console 無錯誤
```
- 移動玩家：直接設 `P.col / P.row`（座標超出範圍會被夾回去），再跑幾秒 `update()` 看是否自動採集／購買／賣出。
- 看有沒有錯：`read_console_messages`。看畫面：先 `render()` 再截圖；手機版面用 `resize_window` preset `mobile`。
- 放大看角色動作：把 `render` 暫時換成空函式，再用 `ctx.scale(3)`～`4` 搭配替換過的 `isoPos` 單獨畫 `drawPlayer`。

## 驗證紀錄與限制
- 已用腳本從階段 0 跑到 3，並請 4 個 subagent（第一輪、第二輪、完整經濟 A／B）當第一次玩的玩家回饋，問題都已修。
- **沒做過**：真人用鍵盤／滑鼠長時間遊玩的手感、在真實手機上的觸控（虛擬搖桿）測試。
