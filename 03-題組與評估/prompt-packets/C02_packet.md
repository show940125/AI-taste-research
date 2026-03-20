# C02 Prompt Packet

## 使用說明

- 這份 packet 是給第一輪最小矩陣直接貼用的。
- `model_name` 保留正式測試目標欄位，用來標示原本打算比較的對象。
- 第一輪 writer 來源經使用者確認，統一採 `subagent-run`，不直接呼叫 OpenAI 或 Claude API。
- 因此正式記錄時，請在 manifest 的 `writer_backend` 欄位填 `subagent`。

## 固定欄位

- `item_id`: C02
- `task_group`: C
- `task_type`: 英譯中重寫
- `target_audience`: 一般讀者
- `length_target`: 350-550 字
- `content_constraints`: 保留因果、條件、限制三層。
- `tone_constraints`: 不要保留英文抽象名詞鏈。
- `writer_backend`: subagent

## Source Material

```text
Translate and rewrite the following into natural Chinese.

Source:
Public trust in institutions is often discussed as if it were a moral resource that citizens either possess or lack. In practice, it behaves more like a running estimate. People revise it after each visible success, each bureaucratic delay, and each moment when official language diverges from lived reality.
```

## Baseline

```text
你是一位中文寫作者。請根據以下材料完成任務。

任務類型：英譯中重寫
讀者：一般讀者
字數：350-550 字
內容限制：保留因果、條件、限制三層。
語氣限制：不要保留英文抽象名詞鏈。

材料：
Translate and rewrite the following into natural Chinese.

Source:
Public trust in institutions is often discussed as if it were a moral resource that citizens either possess or lack. In practice, it behaves more like a running estimate. People revise it after each visible success, each bureaucratic delay, and each moment when official language diverges from lived reality.

要求：
- 使用台灣中文
- 輸出一篇完整成品
- 最後另列 `audit_notes`，用短句寫出你自己看到的風格風險
```

## Rewrite-pipeline

```text
請依三步完成任務。

Step 1. 先整理材料的核心意思與不能漏掉的資訊，不要照原句序重寫。
Step 2. 依中文習慣寫出初稿。先求清楚，再求節奏與作者感。
Step 3. 檢查初稿，逐條修正：
- 題面詞是否重用過多
- 連接詞是否過密
- 主語代詞是否補得太滿
- 是否有過長的「的」字鏈
- 是否殘留英文被動、抽象名詞與從句痕跡
- 是否有聊天腔、討好腔、模板收尾
- 是否整篇段落太齊、太像同一模子

任務類型：英譯中重寫
讀者：一般讀者
字數：350-550 字
內容限制：保留因果、條件、限制三層。
語氣限制：不要保留英文抽象名詞鏈。

材料：
Translate and rewrite the following into natural Chinese.

Source:
Public trust in institutions is often discussed as if it were a moral resource that citizens either possess or lack. In practice, it behaves more like a running estimate. People revise it after each visible success, each bureaucratic delay, and each moment when official language diverges from lived reality.

輸出：
1. 正文
2. audit_notes
```

## Pivot-pipeline

```text
請依四步完成任務。

Phase 1. 先讀懂材料，只抓意思與關係，不照原句序。
Phase 2. 用半文言或古典筆意把事情重述一遍。重點是打散英文句法、抽象名詞鏈與現成模板骨架，不求華麗，不求堆典故。
Phase 3. 只根據 Phase 2 的內容，重寫成新文化白話式中文。要求明白、短句、節制、有人的判斷，不要聊天腔，不要公文腔。
Phase 4. 做最後稽核，修掉殘留的翻譯腔、聊天腔、過整齊的段落節奏與空泛姿態詞。

任務類型：英譯中重寫
讀者：一般讀者
字數：350-550 字
內容限制：保留因果、條件、限制三層。
語氣限制：不要保留英文抽象名詞鏈。

材料：
Translate and rewrite the following into natural Chinese.

Source:
Public trust in institutions is often discussed as if it were a moral resource that citizens either possess or lack. In practice, it behaves more like a running estimate. People revise it after each visible success, each bureaucratic delay, and each moment when official language diverges from lived reality.

輸出：
1. 正文
2. audit_notes
```
