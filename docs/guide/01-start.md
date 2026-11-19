# 安裝與第一個任務

這章讓你裝好外掛、跑完第一個任務，並讓 Claude 能自己驗證成果。

## 安裝與更新

安裝：

```
/plugin install ajua-stack --marketplace ajuadotme/ajua-stack
```

更新（在一般終端機執行）：

```
claude plugin update ajua-stack@ajua-stack
```

所有指令都要寫完整前綴：`/ajua-stack:ajua-mode`、`/ajua-stack:how`、`/ajua-stack:teach`。少了前綴，Claude Code 找不到指令。

## 第一個任務

挑一件真實但很小的事。格式是目標、完成條件、要看的證據：

```
/ajua-stack:ajua-mode 再玩一次的按鈕沒反應。修好後實際按一次，把結果截圖給我。
```

`ajua-mode` 會依序做這些事：

1. 用一兩句話複述你的目標，然後開始。
2. 依任務挑一份流程表，把步驟列成任務清單。這個例子會挑「修問題」。
3. 實際執行並驗證，不只確認能編譯。
4. 驗證通過後在本機 commit，再用白話回報，附上證據。

你不用指定流程表。描述任務就好。

## 版本紀錄

`ajua-mode` 動手前會先檢查 git 狀態。

- 專案沒有 git：它會停下來詢問。沒有版本紀錄，改壞了無法復原。
- 有你尚未存檔的改動：它不會動那些改動，也不會混進自己的 commit，並會問你要不要先存檔。
- 它只在本機 commit。不 push，不開 PR。你說要推送才推。

平常直接在專案資料夾工作。`arena` 的執行者、`interrogate` 的審查者，以及會改檔案的 `swarm` 工作者都會用 worktree，結束後自動收掉。

## 會先問你的事

下列情況，Claude 會先問，不會直接做：

- 刪除沒進版本紀錄的東西。
- push、reset --hard、改寫歷史。
- 部署、發布、對外發訊息。
- 產品方向，例如功能取捨、畫面、文案。

## 成本

`ajua-mode` 會派子代理做實作與驗證。遇到要比稿、分頭處理或風險高的改動，還會用到 `arena`、`swarm`、`interrogate`，花費更高。`recall`、`automate-me`、`worklog` 也會派子代理。小改動直接說即可：

```
把設定頁的標題改成「偏好設定」。
```

## 讓 Claude 能自行驗證

Claude 只有能自己檢查成果，才能一直做到通過為止。否則每個結果都要你手動確認。用 `verify-setup` 替專案建立驗證方法：

```
/ajua-stack:verify-setup
```

它會找出專案怎麼啟動、怎麼操作、能取得哪些證據，並在 `.claude/skills/verify-<app>/` 產生專案專用的驗證技能，並在 `features/` 底下為 3 到 5 個主要功能寫操作說明。它只問你觀察不到的事。交付前它會實際跑一遍，沒跑通就不算完成。

之後 `ajua-mode` 驗證時會自動使用這個技能。專案換了啟動方式或介面，再執行一次。

下一章：[怎麼下指令](./02-prompting.md)
