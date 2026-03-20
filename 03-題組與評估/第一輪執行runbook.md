# 第一輪執行 Runbook

這份 runbook 給第一輪探索測試直接用。它的目的是把研究包變成一套可操作流程，讓你每次跑都用同一個節奏，減少臨場亂改造成的雜訊。

## 方法說明

本版第一輪正式結果採 `subagent-run`。也就是說，writer backend 使用本環境子代理，而不是外部 `OpenAI/Claude API`。這個選擇來自實際條件：目前沒有 API 金鑰，但使用者已明確同意將 subagent 輸出作為正式第一輪結果。

因此這一輪至少要記兩個欄位：

- `model_name`：保留原本想對齊的測試目標欄位
- `writer_backend`：固定填 `subagent`

## 本輪目標

- 先看哪種介入法最有希望，不追求一次做完所有 360 份輸出。
- 優先比較 `baseline`、`rewrite-pipeline`、`classical/vernacular pivot pipeline`。
- 在 `OpenAI` 與 `Claude` 各跑一輪最小矩陣，先抓趨勢，再決定要不要補 `persona-only` 與 `few-shot-only`。

## 最小矩陣

- 題數：18 題
- 模型：2 個
- 條件：3 組
- 總輸出：108 份

公式：

`18 題 x 2 模型 x 3 條件 = 108 份`

## 建議順序

### 第一步：先建資料夾

建議在 `03-題組與評估` 下建立這些子資料夾：

- `outputs/openai/baseline`
- `outputs/openai/rewrite-pipeline`
- `outputs/openai/pivot-pipeline`
- `outputs/claude/baseline`
- `outputs/claude/rewrite-pipeline`
- `outputs/claude/pivot-pipeline`
- `blind-review`

### 第二步：固定模型版本

每一輪都記下：

- 模型名稱
- writer backend
- 模型版本或日期
- 溫度或其他採樣設定
- 當天日期

不要同一輪裡一邊跑一邊改模型設定。那樣後面根本沒法比。

### 第三步：逐題執行

每題都照這個順序：

1. 從 [第一輪探索題組.md](第一輪探索題組.md) 複製題目欄位。
2. 從 [第一輪Prompt模板包.md](../02-框架/第一輪Prompt模板包.md) 選條件。
3. 產生輸出後，原樣保存。
4. 把 `audit_notes` 拆掉，另存一份盲評版。
5. 在 [評分記錄模板.csv](評分記錄模板.csv) 補上基本欄位。

## 檔名規則

### 原始輸出

```text
{item_id}_{model_name}_{writer_backend}_{prompt_condition}_raw.md
```

例：

```text
A01_openai_subagent_rewrite-pipeline_raw.md
C03_claude_subagent_pivot-pipeline_raw.md
```

### 盲評版

```text
{item_id}_{random_code}_blind.md
```

盲評版要刪掉：

- 模型名稱
- prompt 條件
- audit_notes

### Judge 評分輸出

```text
{item_id}_{model_name}_{writer_backend}_{prompt_condition}_judge.json
```

## 人工盲評步驟

1. 每題至少找 2 位評審。
2. 每位評審只看盲評版。
3. 每位評審獨立填分，不要先討論。
4. 若兩位分數差距超過 2 分，再加第 3 位評審。

### 評審最少要填的欄位

- 自然度
- 模板味壓制度
- 翻譯腔控制
- 內容保真
- 語氣穩定
- 最刺眼的一個問題
- 最成功的一個地方
- 是否有過度修辭或 cosplay 味

## LLM-as-judge 步驟

Judge 請盡量避開和 Writer 同型同版。若 Writer 是 OpenAI，可先用 Claude judge；反過來也是一樣。

### Judge prompt

```text
請評估下列中文文本，依四項指標各打 1-5 分，並各寫一句理由。

指標：
1. 自然度：句法是否自然，有沒有機械感
2. 模板味：是否可見固定套路、姿態詞、整齊骨架
3. 內容保真：是否忠實保留題目或原文重點
4. 翻譯腔：是否殘留英文句法與顯化過度

額外判斷：
- 是否有聊天腔或協作殘影
- 是否有過度修辭或假作者感

請輸出 JSON，欄位如下：
{
  "naturalness": 1-5,
  "template_suppression": 1-5,
  "content_fidelity": 1-5,
  "translationese_control": 1-5,
  "chat_artifact": true/false,
  "overstyling": true/false,
  "notes": "..."
}
```

## 規則計數最小版

第一輪不用一次把所有欄位都做成自動化。先人工或半人工記這幾個就夠：

- 高頻姿態詞次數
- 對照句次數
- 三段式列舉次數
- `的` 鏈超過 2 層的次數
- 連接詞密度
- 代詞密度
- 被動句數量
- 題面詞重用率
- 聊天殘影次數

## 第一輪判讀重點

### 成功訊號

- `rewrite-pipeline` 明顯比 `baseline` 自然
- `pivot pipeline` 在英譯中重寫上壓低翻譯腔
- 模板味下降時，內容保真沒有一起掉下去

### 失敗訊號

- 文言中轉後還殘留假古文味
- persona 組只有 tone 變了，骨架沒變
- few-shot 組學到另一套固定模子
- AI 味下降，但整篇變得乾、薄、沒有判斷

## 建議記錄方式

每跑完一題，就補一行簡短觀察，別等全部跑完才回頭想。最有用的通常不是平均分，而是這兩句：

- 這一組到底改掉了哪種味道？
- 它又引進了哪種新問題？

## 第一輪結束後要做什麼

1. 先按題型分開看，不要把 18 題全灌成一個平均。
2. 對每個模型各自找出最穩的兩組。
3. 若 `pivot pipeline` 只在英譯中有效，就把它保留在那個場景，不要硬推全場景。
4. 把最有效的共通成分抽出來，才進第二輪收斂。
