# anti-AI prompt 模型規格

## 目的

這份文件定義研究案要測的五種 prompt 條件：`baseline`、`persona-only`、`few-shot-only`、`rewrite-pipeline`、`classical/vernacular pivot pipeline`。它的用途很單純：把每組介入法的變數先釘住，讓後面的測試有可比性。

本文預設主場景是 `OpenAI/Claude` 的中文論說與評論文輸出。任務類型延伸到研究摘要改寫與英譯中重寫。文獻基礎主要來自三束材料：

- LLM 風格收斂與同質化研究，如 [Zanotto & Aroyehun, 2025](https://aclanthology.org/2025.emnlp-main.1163/) 與 [Padmakumar & He, 2024](https://proceedings.iclr.cc/paper_files/paper/2024/hash/02dec8877fb7c6aa9a79f81661baca7c-Abstract-Conference.html)
- translationese／MTese 研究，如 [Hu, Li, Kübler, 2018](https://aclanthology.org/W18-1603/) 與 [Kong & Macken, 2025](https://aclanthology.org/2025.mtsummit-1.8/)
- style transfer 與 prompt 介入研究，如 [Krishna et al., 2022](https://aclanthology.org/2022.acl-short.94/)、[Liu et al., 2024](https://aclanthology.org/2024.lrec-main.1328/)、[Madaan et al., 2023](https://arxiv.org/abs/2303.17651)

另有兩條實務脈絡會放進實驗：

- `humanizer` 與維基 AI 寫作特徵頁提供的 heuristics，可當最後一輪稽核依據，但不能獨力充當學術證明。
- 既有材料中的「文言中轉 -> 新文化白話」路徑，先視為強假說，等對照測試來驗。

## 共通設定

### 1. 固定輸入欄位

每一組 prompt 都要保留這些欄位，避免測到一半變成不同任務。

- `task_type`：說理評論、研究摘要改寫、英譯中重寫
- `target_audience`：一般讀者、研究生、專業讀者
- `length_target`：字數範圍
- `source_material`：題目、摘要、原文
- `content_constraints`：不得遺漏的資訊
- `tone_constraints`：語氣限制

### 2. 共通語言約束

每組 prompt 都加入以下共通要求，避免模型一路滑回助理預設。

- 使用台灣中文。
- 優先用動詞與名詞，少堆抽象名詞。
- 能省略主語時就省略。
- 少用過密的邏輯路標，如「然而」「因此」「換言之」。
- 避免聊天腔、討好腔、佔位句、知識截止免責。
- 若原文是英文，先拆 clause，再決定中文重組，不能沿英文語序直推。

### 3. 共通輸出格式

除非任務另有要求，每組都統一輸出：

1. 正文
2. `audit_notes`，用短句列出自己修掉的模板痕跡

這樣做有兩個好處。第一，方便盲評前把第二段刪掉。第二，後處理時能看出模型到底修了什麼。

## Baseline

### 用途

提供最接近一般助理預設的對照組，作為所有介入法的比較底盤。

### 適用情境

- 要看 prompt 介入到底幫了多少
- 要抓模型原生模板味與翻譯腔

### 主要 prompt 結構

```text
你是一位中文寫作者。請根據以下材料完成任務。

任務類型：{task_type}
讀者：{target_audience}
字數：{length_target}
限制：{content_constraints}

材料：
{source_material}

請直接輸出成品。
```

### 風險

- 容易落回助理腔
- 對題面詞重用偏高
- 說理常走最短路徑，句法平均化明顯

### 預期效果

可穩定產出可讀文本，但 AI 味通常最明顯，適合作為基準。

## Persona-only

### 用途

測試角色與風格標定，能壓低多少模板味。

### 適用情境

- 任務偏評論、隨筆、短論
- 需要快速改掉助理預設聲調

### 主要 prompt 結構

```text
你現在是一位中文論說文作者，文風要求如下：

- 語氣：誠懇、明白、節制，有判斷
- 句法：長短句交錯，避免排比感
- 說理：先交代表面合理性，再補條件、範圍、因果
- 禁用：聊天腔、口號式總結、過度工整的三段式

任務類型：{task_type}
讀者：{target_audience}
字數：{length_target}
限制：{content_constraints}

材料：
{source_material}

請直接輸出成品。
```

### 風險

- 模型可能只學到表面 tone，沒改掉句法骨架
- 若角色描述太花，容易變成模仿秀或「cosplay 味」
- 不同模型對 persona 的敏感度落差很大，參考 [persona prompting 研究](https://arxiv.org/abs/2407.02099)

### 預期效果

比 baseline 更像有作者，但效果常不穩，尤其在研究摘要與翻譯任務上。

## Few-shot-only

### 用途

用少量高品質例文示範想要的節奏、視角與語氣。

### 適用情境

- 有清楚目標文風
- 任務重在「怎麼寫」而不是「寫什麼」

### 主要 prompt 結構

```text
下面有三個示例，請觀察它們的句法節奏、論證方式與語氣。
不要抄內容，只學寫法。

[Example 1]
{example_1}

[Example 2]
{example_2}

[Example 3]
{example_3}

現在完成下列任務：
任務類型：{task_type}
讀者：{target_audience}
字數：{length_target}
限制：{content_constraints}

材料：
{source_material}
```

### 風險

- 示例太像，模型會學到另一套新模板
- 示例太短，能學到 tone，學不到篇章節奏
- 示例品質若有翻譯腔，會整批帶歪

### 預期效果

通常比 persona-only 更穩。對句長分布、轉場節奏、收尾方式的控制較好。參考 [Prompt-and-Rerank](https://aclanthology.org/2022.emnlp-main.141/) 與官方 few-shot 建議的經驗，示例多樣性是關鍵。

## Rewrite-pipeline

### 用途

把生成拆成兩步或三步，先出草稿，再做定向改寫，專門處理模板味與 translationese。

### 適用情境

- 研究摘要改寫
- 英譯中重寫
- 需要兼顧內容保真與文風調整

### 主要 prompt 結構

```text
請依三步完成任務。

Step 1. 先抓材料的核心意思與不能漏掉的資訊。
Step 2. 依中文習慣寫出初稿。先求清楚，再求有風格。
Step 3. 掃描初稿，逐條檢查：
- 題面詞是否重用過多
- 連接詞是否過密
- 主語代詞是否補得太滿
- 是否有過長的「的」字鏈
- 是否殘留英文被動、抽象名詞與從句痕跡
- 是否有聊天腔、討好腔、模板收尾

最後輸出修正版，並附 audit_notes。

任務類型：{task_type}
讀者：{target_audience}
字數：{length_target}
限制：{content_constraints}

材料：
{source_material}
```

### 風險

- 同一模型自我修正時，可能把原本語氣磨得更平，參考 [Pride and Prejudice](https://aclanthology.org/2024.acl-long.826/)
- 若檢查清單太長，模型會偏向保守與過度編輯

### 預期效果

這組通常最平衡。它對 translationese 的壓制特別有利，也較能保住內容。可參照 [Prompt-Based Editing](https://aclanthology.org/2023.findings-emnlp.381/) 與 [Step-by-Step](https://aclanthology.org/2024.lrec-main.1328/) 的思路。

## Classical/Vernacular Pivot Pipeline

### 用途

把英文句法殘影先打散，再用另一套中文規律重建。這是本研究最重要的實驗組。

### 適用情境

- 英譯中重寫
- 說理文本的模板味壓制
- 原稿抽象名詞過多、句法硬、翻譯腔重

### 主要 prompt 結構

```text
請依四步完成任務。

Phase 1. 先讀懂材料，只抓意思，不照原句序。
Phase 2. 用半文言或古典筆意重述事情，重點是打散英文句法與抽象名詞鏈。
Phase 3. 只根據 Phase 2 的結果，重寫成新文化白話式中文：明白、短句、有人味、少模板句。
Phase 4. 做最後稽核，刪掉殘留的翻譯腔、聊天腔、過整齊的段落節奏。

輸出：
1. 正文
2. audit_notes

任務類型：{task_type}
讀者：{target_audience}
字數：{length_target}
限制：{content_constraints}

材料：
{source_material}
```

### 風險

- 若 Phase 2 修辭過濃，Phase 3 可能留下過度文飾
- 若模型對古典語感掌握不穩，可能出現假古文或莫名典故
- 目前學術證據多屬機制層支持，缺直接 benchmark，需靠本研究補實測

### 預期效果

理論上最能切斷英語句法與模型預設模板。若做得好，中文會更鬆、更活，也比較不像助理直接吐出來的答案。

## Final Audit Pass

這一步是五組都共享的後處理。目的不是追求華麗，而是把明顯 AI 殘影收乾淨。

### 檢查項目

- 有沒有姿態詞與空泛總結詞連發，如「值得注意的是」「具有重要意義」「凸顯了」
- 有沒有句法模板連續出現，如對照句、三段式、列表化段落
- 有沒有 translationese 痕跡，如代詞過滿、被動殘影、`的` 鏈、連接詞過密
- 有沒有聊天與協作殘影，如「希望這對你有幫助」「如果你需要我可以再補」
- 有沒有樣式殘影，如粗體濫用、破折號、過多標題感

### 統一提示詞

```text
請只做最後稽核，不要重寫立場與內容重心。
逐條檢查：
1. 刪去空泛姿態詞與模板收尾
2. 打散過整齊的句法與段落
3. 去掉翻譯腔：代詞、被動、連接詞、「的」字鏈、抽象名詞
4. 去掉聊天腔與協作式尾巴
5. 若某處已自然，保持原樣，不要為了變化而硬改
輸出修正版與 audit_notes。
```

### 使用原則

- `final audit pass` 要晚於主要寫作步驟，否則模型一開始就會變得太拘謹
- 這一步要限制改動範圍，避免把原有風格磨平

## 建議的實驗順序

若資源有限，先跑這個順序：

1. `baseline`
2. `rewrite-pipeline`
3. `classical/vernacular pivot pipeline`
4. `persona-only`
5. `few-shot-only`

理由很簡單。`rewrite-pipeline` 最容易收穫穩定收益，`pivot pipeline` 最有研究價值，另外兩組則有助於辨認「風格控制」和「內容重組」各自貢獻了多少。

## 實作注意事項

- 所有組別都要固定同一題組與同一字數範圍。
- `few-shot` 示例至少要有句長差異、收尾差異、論證差異，不能只換題材。
- `pivot pipeline` 的第二階段重點在句法去源語化，不在仿古。
- `audit_notes` 在正式盲評前要移除，避免評審被提示。
