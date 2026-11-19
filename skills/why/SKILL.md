---
name: why
description: 從版本紀錄查某個東西為什麼這樣做，引用紀錄原文，說明決策與取捨。
disable-model-invocation: true
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
argument-hint: 要問的檔案、函式或行為
---

# Why

用於「X 為什麼是這樣運作」、「為什麼選 Y」、設計理由、regression、有資料支持的門檻值。要問執行時的行為，用 `/ajua-stack:how`。

從紀錄回答，不從程式碼回答。用 `git log`、`git blame` 找出引入目標的 commit，以及之後每一次變更的 commit，讀完整的 commit 訊息。線索不足時再讀其他來源，例如專案內的設計文件與註解。

- 引用作者寫下的內容。
- 把自己的推論標示為推論。
- 說明紀錄沒有回答的部分。
