# ajua-stack

Claude Code 外掛。讓 Claude 照固定流程工作，並以實際證據驗證成果。

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

小改動直接說即可，不需要這個指令。完整用法見[導覽](docs/guide/README.md)。

## 指令

| 指令 | 用途 |
|---|---|
| `/ajua-stack:ajua-mode` | 入口。依任務選流程表並執行 |
| `/ajua-stack:how` | 說明某個部分怎麼運作 |
| `/ajua-stack:why` | 從版本紀錄查為什麼這樣做 |
| `/ajua-stack:teach` | 一步步講解，直到理解 |
| `/ajua-stack:bro` | 把上一則回覆改成白話 |
| `/ajua-stack:catch-up` | 說明目前進度，列出等你決定的事 |
| `/ajua-stack:recall` | 從過去的對話重建某個主題的脈絡 |
| `/ajua-stack:sharpen` | 把你的提示詞改得更到位 |
| `/ajua-stack:blast-radius` | 查一個改動會連帶影響哪些地方 |
| `/ajua-stack:verify-setup` | 替專案建立驗證方法，之後 Claude 能自行驗證 |
| `/ajua-stack:interrogate` | 派多個子代理找出改動的毛病 |
| `/ajua-stack:arena` | 多個子代理各做一版，取各版的長處 |
| `/ajua-stack:swarm` | 多個子代理分頭處理，彙整成一份報告 |
| `/ajua-stack:worklog` | 把每個決定記成可事後檢查的紀錄 |
| `/ajua-stack:comments` | 檢查程式碼註解，只留有必要的 |
| `/ajua-stack:writing` | 中文技術寫作規範 |
| `/ajua-stack:typescript` | TypeScript 寫法規範 |
| `/ajua-stack:reflect` | 把這次的教訓整理成技能的修改提案 |
| `/ajua-stack:correct` | 把重複出現的錯誤變成自動檢查 |
| `/ajua-stack:automate-me` | 從過去的對話歸納你的工作風格，做成專屬技能 |
| `/ajua-stack:setup` | 設定各角色使用的模型 |

## 流程表

`ajua-mode` 依任務自動選用。

| 任務 | 流程表 |
|---|---|
| 搞懂運作方式或原因 | 搞懂 |
| 東西壞了 | 修問題 |
| 新功能，或改變現有行為 | 加功能 |
| 整理程式，行為不變 | 整理 |
| 做法未定 | 試做比較 |
| 太慢 | 效能調校 |
| 反覆改進一個指標 | 指標爬升 |
| 執行中的異常症狀 | 執行期診斷 |
| 分析效能紀錄檔 | 紀錄檔診斷 |
| 畫面改版或重寫，要與原本一致 | 畫面比對 |
| 清掉用完的 worktree | 清理 worktree |
| 工作做到一半要停 | 安全暫停 |
| 接續中斷的工作 | 接手工作 |
| 大工程 | 多階段計畫 |
| 做到完成為止，中途不停 | 長時間自主執行 |
| 新增或修改技能 | 撰寫技能 |

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
