---
name: setup
description: 設定 ajua-stack 四個角色各用哪個模型與推理等級，寫入每個新對話都會載入的規則檔。
disable-model-invocation: true
allowed-tools: Read(${CLAUDE_PLUGIN_ROOT}/**)
---

# Setup

把各角色的模型寫進 `~/.claude/rules/ajua-stack-models.md`。Claude Code 每個新對話都會載入這個檔案。

四個角色：

- `build`：實作與除錯。
- `judgment`：判斷與文字。
- `routine`：機械式修改與驗證。
- `panel`：審查與比稿的小組。值是清單，一個項目派一個子代理，清單長度就是人數。

## 1. 偵測

列出 `Agent` 工具 schema 裡 `model` 的可用值與 `effort` 的可用值。這是唯一可靠的來源。一個模型都偵測不到時，要使用者貼上可用的模型名稱。沒確認可用的模型不寫。推理等級最高用到 `xhigh`。

## 2. 問預算

用 `AskUserQuestion` 問，三個選項：

- `large`：推理等級 `xhigh`
- `medium`：推理等級 `high`
- `small`：推理等級 `medium`

規則檔已存在時，讀出它的 `# budget` 行，在問題裡點出目前的預算。第 3 步以它現有的各角色值為起點。

## 3. 套用預算

預設值是第 5 步格式裡的值，對應 `large`。

1. **降級。** `large` 不變，`medium` 把每個推理等級降一級，`small` 降兩級。階梯是 `xhigh` > `high` > `medium` > `low`，降到 `low` 就停住。
2. **推理等級不可用。** 目標等級不在偵測到的 `effort` 值裡，改用偵測到的、低於目標的最高等級。
3. **模型不可用。** `panel` 拿掉偵測不到的項目。其他角色的模型偵測不到時，把這個角色標成「需要選擇」。

## 4. 確認

顯示四個角色與各自的值，標出這次會改動的角色與「需要選擇」的角色。`panel` 剩不到兩個項目時，標成「需要選擇」。用 `AskUserQuestion` 問要照原樣接受，還是修改某個角色。可選的值只限偵測到的模型，推理等級不超過 `xhigh`。`panel` 不足兩個時告訴使用者：比稿與審查至少要兩個子代理。所有角色都有值後才寫入。

## 5. 寫入

整個檔案覆寫，讓重跑結果一致。不加 frontmatter。不寫進 `CLAUDE.md`。格式：

```
# ajua-stack 模型設定。每行一個角色，刪掉某行就回到預設。
# budget: large
build: opus xhigh
judgment: opus xhigh
routine: opus high
panel: fable high, opus high, sonnet high
```

## 6. 告知

告訴使用者規則已寫入，下一個對話開始生效。要修改時重跑 `/ajua-stack:setup`。

專案還沒有可重複執行的驗證方法（沒有 `verify-*` 技能），提議一次用 `/ajua-stack:verify-setup` 建立。對方拒絕就結束，不再提。
