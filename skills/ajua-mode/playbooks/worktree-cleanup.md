# 清理 worktree

清掉用完的 worktree 與 branch。分類只是建議，不是許可。刪掉尚未併入的工作無法復原，必須先讓使用者確認。

1. **列出清單。** 記下磁碟可用空間。用 `git worktree list` 取得每個 worktree 的路徑與 branch，不手打路徑，手打會漏掉放在別處的 worktree。Claude Code 的子代理 worktree 在專案的 `.claude/worktrees/` 底下，branch 以 `worktree-agent-` 開頭。
2. **逐一檢查並分類。** 寫一支腳本對每個 worktree 檢查：`git -C <路徑> status --porcelain` 看未 commit 的改動，已追蹤的修改與未追蹤的檔案分開計數；`git branch --merged` 看 branch 是否已併入目前的 branch；`git log -1 --format=%cr <branch>` 看最後一次 commit 的時間；資料夾大小。依結果分成：乾淨且已併入、有未 commit 的修改、只有未追蹤的檔案、尚未併入，並提出建議的刪除清單。
3. **以使用者的清單為準。** 向使用者取得仍在使用或要保留的對話與 worktree，逐一核對每個候選。清單與分類衝突時，照使用者的清單。
4. **確認沒有進行中的工作在用它。** 最近有 commit 或改動的，以及任何有疑慮的項目，讀對應的對話紀錄，確認該對話是否仍在進行、碰了哪些 worktree。進行中的對話會透過子代理在別的 worktree 產生東西，名稱看不出關聯，這些也算使用中。對話紀錄量大，交給子代理，一份紀錄一個，放到背景執行。模型依入口檔的「模型」一節，角色 `routine`。
5. **遇到不可復原的損失就停。** 有未 commit 的修改：秀出 diff，取得使用者決定。乾淨的 worktree 刪掉後可從 branch 復原，未 commit 的工作會直接消失。未追蹤的檔案一律列出檔名，取得使用者確認後才刪。尚未併入的 branch，刪除前取得使用者確認。乾淨、已併入、未使用中的直接繼續。
6. **移除已確認的項目。** 逐一執行 `git worktree remove --force <路徑>`。資料夾因被忽略的建置產物殘留時，刪掉資料夾，再執行 `git worktree prune`。已併入的 branch 用 `git branch -d` 刪除；尚未併入的，使用者確認後才用 `git branch -D`。重新列出清單，記下磁碟可用空間。

這份流程會刪除使用者的狀態，沒有其他審查能攔下失誤，上面的關卡就是審查。

**回報：** 前後的磁碟可用空間與回收的空間、已移除的 worktree 與 branch、每個保留項目的一行理由（哪個工作在用，或有未 commit 的工作）。
