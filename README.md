# ajua-stack

Claude Code 外掛。讓 Claude 照固定流程工作，並以實際證據驗證成果。為獨立開發、不讀程式碼的使用者設計。

## 安裝

```
/plugin install ajua-stack --marketplace ajuadotme/ajua-stack
```

## 使用

```
/ajua-stack:ajua-mode <目標>。<完成條件>。<要看的證據>。
```

```
/ajua-stack:ajua-mode 再玩一次的按鈕沒反應。修好後按一次，把結果截圖給我。
```

小改動直接說即可，不需要這個指令。

## 指令

| 指令 | 用途 |
|---|---|
| `/ajua-stack:ajua-mode` | 入口。依任務選流程表並執行 |
| `/ajua-stack:how` | 說明某個部分怎麼運作 |
| `/ajua-stack:teach` | 一步步講解，直到理解 |
| `/ajua-stack:bro` | 把上一則回覆改成白話 |
| `/ajua-stack:catch-up` | 說明目前進度，列出等你決定的事 |
| `/ajua-stack:verify-setup` | 替專案建立驗證方法，之後 Claude 能自行驗證 |

## 流程表

`ajua-mode` 依任務自動選用。

| 任務 | 流程表 |
|---|---|
| 搞懂運作方式或原因 | 搞懂 |
| 東西壞了 | 修問題 |
| 新功能，或改變現有行為 | 加功能 |
| 整理程式，行為不變 | 整理 |
| 做法未定 | 試做比較 |

## 行為

- 先複述目標，再開始。
- 完成前實際執行，並附上證據。
- 驗證通過後在本機 commit。不 push，不開 PR。
- 專案沒有 git 時，先詢問。
- 刪除、push、部署、產品方向的決定，先詢問。

## 更新

```powershell
claude plugin update ajua-stack@ajua-stack
```

## 授權

[MIT](LICENSE)
