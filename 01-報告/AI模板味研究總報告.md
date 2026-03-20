# AI模板味研究總報告

## 摘要

本文專注研大型語言模型寫出來的文字，到底為什麼會有一股「模板味」。我們想弄清楚，這股味道在中文裡究竟是什麼模樣？哪些部分是機器寫作的通病？哪些又是英文思維硬翻成中文留下的痕跡？如果想用提示詞把這股怪味去掉，我們究竟該禁用某些詞彙、讓模型扮演特定角色，還是乾脆改造寫作流程？研究分成兩段。前半段回顧同質化、translationese、style transfer 與實務 heuristics 等文獻，後半段用本研究自行建立的第一輪正式結果做初步實測：8 題、24 份正式輸出，條件為 `baseline`、`rewrite-pipeline`、`pivot-pipeline`，writer backend 統一為 `subagent`。結果顯示，高頻姿態詞並非主要問題，真正穩定的訊號落在句法骨架、被動殘影、顯性連接與段落節奏。若以壓低英譯中殘影為主要目標，`pivot-pipeline` 的效果最好；若以保住資訊量為優先，`rewrite-pipeline` 更穩定。本文最後提出一套系列 prompt 模型，重心放在先診斷，再改寫；必要時做跨層中轉，最後用有邊界的 heuristics 做終稿稽核。

**關鍵詞：** AI味、模板味、translationese、英譯中、prompt design、style transfer、中文寫作

## 一、研究緣起

我們平常看機器寫的文章，總覺得刺眼。大家常挑毛病：有些詞太常見了，有些句子太整齊了，整篇文章就像一個模子印出來的。這些看法沒錯，但沒說到點子上。真正讓人一眼看破手腳的，不在單獨一個詞，也不在某句口頭禪。關鍵在於，整篇文章太平穩、太工整。就像人拿砂紙死死打磨過，縫隙沒了，起伏也沒了。

這毛病在中文裡特別明顯。因為機器的「AI味」，往往和「翻譯腔」混雜在一起。句子表面上讀得通順，你仔細一嚼，就會發現它其實是按照英文的邏輯想好了，再硬搬進中文裡。代名詞滿天飛，連接詞像路標一樣插得到處都是，抽象名詞一個接一個。到了段落最後，還總愛下一個漂亮卻空洞的結論。於是，文章裡就有了兩種怪味：一種是機器求安全的平庸味，一種是英文邏輯的殘影。我們做這個研究，就是要弄清楚這兩種味道，然後找出辦法來對付它們。

因此，本研究的目的，就是把這兩股味道拆開看，並把拆解結果做成可用的對抗方案。問題有三個：

1. 中文模板味的穩定特徵是什麼。
2. 哪些特徵和 translationese 關係最深。
3. prompt 應該怎麼設計，才有機會真正壓低這些痕跡，而不是把一種模板換成另一種模板。

## 二、概念與研究問題

### 2.1 模板味

本文說的「模板味」，是指文章結構太容易預測，重複性太高。起承轉合太過平順，段落長短太過平均，詞彙用得太安全、太籠統。這問題比單純的「詞彙重複」嚴重得多，也比「文筆差」隱晦。它呈現出的是分布過度集中，變異不足的問題。

### 2.2 AI味

至於「AI味」，那是讀者的整體感受。除了模板味，還包括刻意討好人的語氣、囉唆的解釋，以及視覺排版上的痕跡。如果模板味是骨架，AI味就是整體的氣息。

### 2.3 英譯中殘影

本文用「英譯中殘影」，指的是模型先用英文結構打腹稿，再把內容硬塞進中文裡。最常見的毛病包括：一連串的抽象名詞、太過明顯的連接詞、濫用被動語句、代名詞太多，以及長長的「的」字結構、題面詞重用率偏高。這和翻譯研究中的 translationese、MTese、third code 密切相連。

### 2.4 研究問題

本文聚焦四個問題：

1. 模板味最大的破綻真的是否為高頻詞?
2. 哪些題型最容易把英譯中殘影放大。
3. `rewrite-pipeline` 與 `pivot-pipeline` 兩種改寫方法各能修掉什麼毛病？
4. 實務上是否能從這些結果反推一套更有效的 anti-AI prompt 模型。

## 三、文獻回顧

### 3.1 同質化與均值回歸

近年的研究對這件事已有相當一致的描述。Zanotto 與 Aroyehun（2025）比較多領域人類文本與多個模型文本，指出模型生成文的變異顯著較低，而且較新的模型彼此更像。這個結果很重要，因為它把「一眼像 AI」從主觀抱怨，往「統計上過度集中」推進了一步。Zamaraeva、Flickinger、Bond 與 Gómez-Rodríguez（2025）則從形式句法的角度補證，發現模型新聞文本與人類新聞文本在句法型別分布上有穩定差異。兩項研究放在一起看，說明模板味源自於輸出分布偏窄，而非只是使用者過於挑剔。

Durandard、Dhawan 與 Poibeau（2025）更細地指出，模型對題面貼得太近，風格適配卻偏弱。模型常迅速重用問題裡的概念與語詞，回答雖然不至於離題，聲調卻過於 generic。這很接近中文使用者熟悉的感覺：有答案，卻不似人寫的。

Padmakumar 與 He（2024）從共寫研究補上另一個角度。當模型參與寫作，跨作者文本的相似度會上升，詞彙與內容多樣性會下降。換句話說，模型一面生成內容，一面也把寫作往幾條熟路上推。

### 3.2 translationese 與中文句法

若只談同質化，還不夠。中文世界對 AI 味的敏感，很多時候其實來自翻譯腔。Baker（1993）早就把譯文看成一種「第三碼」，常伴隨顯化、簡化、常規化與去歧義。她談的是翻譯，今天拿來看模型中文，仍然十分有用。

Hu、Li 與 Kübler（2018）用句法特徵區分譯文中文與原生中文，發現只看句法就能分得很開。辨識力最高的特徵，包括限定詞偏多、主語位置代詞偏多、`NP + 的` 修飾偏多與並列結構偏多。這些特徵剛好也是中文讀者常覺得「怪怪的」地方。

Kong 與 Macken（2025）進一步比較英譯中的 LLM 與 NMT，指出原生中文和這些譯文幾乎可被明確區分；其中顯著特徵包括句長偏短、轉折與因果連接詞偏密。這項研究把一件常被說成語感問題的事，明確壓成可數的指標。

### 3.3 prompt 介入與 style transfer

Reif 等人（2022）證明，任意風格轉換可透過改寫指令完成，但若提示只停在「請寫得自然」，控制力仍然有限。這提醒我們，對抗 AI 味不能只靠抽象要求。

Liu 等人（2024）提出 step-by-step 的做法，先切區段，再改寫，再補齊，比一次整篇重寫更穩。對中文來說，這尤其關鍵，因為很多問題卡在局部句法與篇章節奏，不必每次都推倒重來。

Self-Refine 相關研究則提醒另一個風險：模型自己改自己，往往能讓字面更順，卻未必能讓文字更像人。Madaan 等人（2023）提出自我回饋迭代流程，Xu 等人（2024）則指出自我精修可能放大模型對自身輸出的偏愛。若沒有外部 rubric，自我修正很容易修成更光滑的模型文。

### 3.4 heuristics 的價值與邊界

`blader/humanizer` 與維基的「AI生成文的特徵」頁面提供了另一種材料（blader, n.d.; Wikipedia contributors, 2026）。它們不是嚴格的學術證明，卻有極高的編修價值。它們把大量一線編輯的直覺整理成可操作清單，像姿態膨脹、模糊歸因、否定排比、三段式、聊天善後句、破折號濫用等，適合拿來做標註表、後處理、終稿 audit，非常好用。

## 四、材料與方法

### 4.1 研究材料

本研究的理論部分，依據下列四束材料：

1. LLM 同質化與內容多樣性研究。
2. translationese 與英漢句法研究。
3. style transfer 與 prompt 改寫研究。
4. `humanizer` 與維基特徵頁。

實務部分則加入兩份本地材料：[賈寶適翻譯.txt](../賈寶適翻譯.txt) 與 [翻译理论要点及提示词优化指南.md](../翻译理论要点及提示词优化指南.md)。這兩份文字雖不是學術論文，卻很重要，因為它們把「文言中轉，再轉新文化白話」這條路講得很清楚，能當研究假說的起點。

### 4.2 第一輪正式結果

第一輪正式結果共 24 份輸出，詳見 [第一輪正式結果_manifest.csv](../03-題組與評估/第一輪正式結果_manifest.csv)。條件如下：

- 題目數：8 題
- 題型：說理評論 2 題、研究摘要改寫 3 題、英譯中重寫 3 題
- 條件：`baseline`、`rewrite-pipeline`、`pivot-pipeline`
- writer backend：`subagent`

這批結果可以比較 prompt 條件差異，但不外推到特定 API 產品。

### 4.3 評估指標

本輪先做三件事：

1. 規則計數：連接詞、代詞、被動句、對照骨架、長 `的` 鏈、段落數。
2. 細讀樣本：觀察同一題不同條件在節奏、語氣與翻譯腔上的差異。
3. 交叉比對：把數量結果和樣本閱讀放在一起看，避免只看統計，也避免只靠印象。

人工盲評與 LLM-as-judge 的完整框架已建於 [題組與評估框架.md](../03-題組與評估/題組與評估框架.md)，但這一版報告先不假裝那些分數已經存在。

## 五、第一輪結果

### 5.1 整體比較

規則計數結果見 [第一輪結果初步分析.md](../03-題組與評估/第一輪結果初步分析.md)。若只看整體平均，可得下表：

| 條件 | 平均字數 | 連接詞 | 代詞 | 被動句 | 對照骨架 | `的` 鏈 > 2 | 段落數 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `baseline` | 317.62 | 0.62 | 3.38 | 1.75 | 1.00 | 0.38 | 2.50 |
| `rewrite-pipeline` | 294.88 | 0.88 | 4.25 | 0.88 | 0.75 | 0.12 | 3.00 |
| `pivot-pipeline` | 264.25 | 0.75 | 2.75 | 0.75 | 0.50 | 0.25 | 3.00 |

這張表說明一件事：模板味最穩的訊號，還是在句法與篇章，不在姿態詞。24 份文本裡，高頻姿態詞其實不多；可被動、代詞、對照骨架、段落收束方式，差異很清楚。

`pivot-pipeline` 在三組裡最像「換骨」。字數縮得最多，代詞最少，被動句最低，對照骨架也最低。`rewrite-pipeline` 比較像保守修整：長 `的` 鏈會減少，被動也能壓下來，但代詞數反而上升，表示它在保留資訊時仍沿用原本的指稱方式。

### 5.2 依題型比較

#### 說理評論

說理評論題裡，`baseline` 最容易露出對照骨架。平均每篇 2 次，幾乎就是肉眼能看見的套路。`rewrite-pipeline` 與 `pivot-pipeline` 都能把這個數字減半，但路徑不同：`rewrite` 靠修枝，`pivot` 靠重組。若任務重在壓模板骨架，兩者都有用；若還想讓句子更像中文作者自己說出來的話，`pivot` 走得更遠。

#### 研究摘要改寫

這組的差異不若英譯中劇烈。因為輸入本來就密，模型寫作時自然會偏保守。`rewrite-pipeline` 在資訊保留上較穩，可是也容易留下整齊版摘要文的痕跡。`pivot-pipeline` 讀起來更像評論，可篇幅也縮得更快。對研究摘要來說，下一輪若要擴充，人工內容保真評分非常必要。

#### 英譯中重寫

這組最值得看。`baseline` 的被動句平均達 4，遠高於其他組。這表示英語式句法骨架被直接搬進中文。`rewrite-pipeline` 雖能把被動降到 1.67，可代詞升到 5.33，顯示顯性指稱仍重。`pivot-pipeline` 在這組效果最乾淨：被動句降到 1.67，對照骨架歸零，代詞數回落。若目標是清理英譯中殘影，它目前是最強的方案。

### 5.3 樣本細讀

#### A01：遠端管理題

`baseline` 把話講得很順，順到骨架都露出來了。先讓步，再轉折，再收結論，句法對稱，讀者很容易猜到下一步。`rewrite-pipeline` 有把顯性骨架削掉一些，也減少了那種「給你一個完整答案」的助理姿態。`pivot-pipeline` 則明顯換了呼吸方式，像「把帳算得太省」「把漏洞照亮」這樣的句子，已經離開教科書式的說明，轉到中文評論的說法。

#### C01：便利與隱性勞動題

這題最能看出英譯中殘影。`baseline` 幾乎沿著原文的抽象論證往前走，看得到英語句法在後面拉扯。`rewrite-pipeline` 能把表層磨順，卻還帶著解說者的手勢。`pivot-pipeline` 直接換了句法出發點，讓抽象關係落回較短的中文句子；讀者會感到，這件事終於被用中文講出來了。

## 六、討論

### 6.1 模板味的主戰場仍在骨架

這一輪結果支持一個很清楚的判斷：模板味不能化約成高頻詞。若只看姿態詞，這批正式結果幾乎不嚴重；若看句法與篇章，差異立刻浮上來。真正穩的訊號，是對照骨架反覆出現、被動句在英文來源題裡居高不下、段落收得太齊、語意轉折被講得太明。

### 6.2 `rewrite` 與 `pivot` 在做不同工作

兩者都能降味，卻不是同一類工具。`rewrite-pipeline` 適合在內容保真壓力高的題目裡用。它像修枝剪葉，讓句子順一點，讓某些露餡的位置退下去。`pivot-pipeline` 更像換骨。它先把原有骨架打散，再用另一套中文節奏重組。代價也很明顯：篇幅會縮，判斷會變緊，有時情緒色澤也會加重。

### 6.3 文言中轉的價值

從這輪結果看，「文言中轉，再轉新文化白話」至少已經脫離玄學猜測。它起作用的地方，是切斷源語句法。當模型先被迫把內容拉離英語骨架，再回到較短、較緊、較意合的中文句子時，翻譯腔明顯下降。這條路還需要更大規模的實測，但它已經過了「純猜想」那一關。

### 6.4 heuristics 應放在終稿層

`humanizer` 與維基規則在這輪沒有直接做作者判定，卻很好用。它們最適合放在最後那一道門：幫你找出聊天殘影、空泛收尾、破折號濫用、列表化段落。若把它們放到最前面，容易把文字修得過於拘謹；若放在最後，反而像一張有效的清單。

## 七、對抗 AI 味的 prompt 模型

綜合文獻與第一輪結果，較可行的 prompt 更像一個按順序運作的流程，不像一串禁令。它先把任務說清楚，再決定風險落在哪一層，接著選擇適當的改寫主幹，最後才做終稿稽核。這種設計比「模仿某位作者」更穩，因為它處理的是產生模板味的路徑；表面口氣只是末端現象。

### 7.1 任務定義與文類定位

第一步要把文類、讀者、篇幅和不可遺漏的資訊說死。這看來只是基本功，實際上非常關鍵。模型一旦拿不準文類，就會往助理預設滑：先概括，再平衡，再收在一個無害的總結。任務定義寫得清楚，可以先擋掉這條熟路。

### 7.2 先診斷，再下藥

真正動筆之前，應先要求模型辨認目前最可能出問題的位置。若主要風險在模板骨架，處理重點就該放在句法與段落功能；若主要風險在英譯中殘影，就要回頭處理被動句、顯性連接、代詞密度與抽象名詞鏈；若問題集中在互動殘影與樣式痕跡，則應留到終稿層處理。這個診斷步驟可以不外顯，但不能省略，否則後面很容易用錯方法。

### 7.3 兩條主幹：`rewrite` 與 `pivot`

若文本本來就是中文，只是句型太整齊、收束太光滑，優先用 `rewrite-pipeline`。它適合在保留資訊的前提下，局部改寫句法、拆解對照骨架、鬆開段落節奏。若文本明顯受英語骨架牽引，或論證一路沿抽象概念直推，則應改用 `pivot-pipeline`。它不求把原句修順，做法是先把原有骨架打散，再用較意合、較短促的中文句子重建論證。第一輪結果已經顯示，凡是被動句、代詞、題面詞重用與長 `的` 鏈堆在一起的文本，多半更適合走 pivot。

### 7.4 終稿稽核與人工保留權

規則層檢查應放在最後，不該擺在最前面。終稿前再掃一次聊天前綴、善後句、空泛姿態詞、過整齊的對照骨架、被動殘影、代詞過滿，以及破折號、粗體、列表化格式，能有效清掉殘留痕跡，又不至於把全文修成另一種僵硬模板。只是這一步也有邊界。模型不能自己宣布「我已經沒味道了」，最後仍要有人回頭讀，確認節奏、判斷和篇章起伏有沒有真的站住。

## 八、限制

本研究仍有幾個限制，必須講清楚。第一輪正式結果的 writer backend 是 `subagent`，不是外部產品 API；這一輪可以拿來比較條件差異，還不能直接宣稱對所有模型產品都一樣有效。

這版報告也還沒有人工盲評分數。現有結論主要靠規則計數與細讀樣本，因此性質仍屬初步。

`persona-only` 與 `few-shot-only` 還沒進正式比較，所以本文對它們的判斷主要來自文獻與設計推論，不來自本輪實測。

`pivot-pipeline` 雖然成效亮眼，但它有壓縮篇幅與加重語勢的風險。這件事不能靠直覺判斷，第二輪最好明確設內容保真與過度修辭兩道檢查。

## 九、結論

這份研究做到目前，已足以支持幾個較硬的判斷。中文 AI 味的主問題不在幾個壞詞，主戰場在骨架；只禁詞，效果有限。translationese 在中文場景裡也不是旁支，它常和模板味纏在一起，尤其在英譯中與抽象說理題裡最明顯。若不正面處理源語骨架，很多「去味」都只停在表層美容。

第一輪正式結果也讓兩條路的差別變得清楚。`rewrite-pipeline` 較穩，適合內容保真壓力高的題目；`pivot-pipeline` 對切斷英語骨架更有力，代價則是篇幅容易縮、語勢容易變緊。兩者沒有誰能全面通吃，因為它們處理的是不同類型的病灶。

因此，對抗 AI 味較可靠的路，和禁詞清單不同，也比空泛地要求「像人一點」更能落地。真正可操作的方法，是先分層診斷，再選擇改寫主幹，最後用有邊界的 heuristics 清理殘影。文言中轉再轉新文化白話，正是這套方法裡目前最值得保留的一支。它未必在所有題型都最強，但在這輪結果裡，已經證明自己確實能打斷英語骨架。

## 參考文獻

Baker, Mona. 1993. "Corpus Linguistics and Translation Studies: Implications and Applications." In *Text and Technology: In Honour of John Sinclair*, edited by Mona Baker, Gill Francis, and Elena Tognini-Bonelli, 233-250. Amsterdam: John Benjamins. https://doi.org/10.1075/z.64.15bak

blader. n.d. *humanizer*. GitHub repository. Accessed 2026-03-20. https://github.com/blader/humanizer/tree/main

Durandard, Noe, Saurabh Dhawan, and Thierry Poibeau. 2025. "LLMs stick to the point, humans to style: Semantic and Stylistic Alignment in Human and LLM Communication." In *Proceedings of the 26th Annual Meeting of the Special Interest Group on Discourse and Dialogue*, 206-213. Avignon, France: Association for Computational Linguistics. https://aclanthology.org/2025.sigdial-1.16/

Hu, Hai, Wen Li, and Sandra Kubler. 2018. "Detecting Syntactic Features of Translated Chinese." In *Proceedings of the Second Workshop on Stylistic Variation*, 20-28. New Orleans: Association for Computational Linguistics. https://doi.org/10.18653/v1/W18-1603

Kong, Delu, and Lieve Macken. 2025. "Decoding Machine Translationese in English-Chinese News: LLMs vs. NMTs." In *Proceedings of Machine Translation Summit XX: Volume 1*, 99-112. Geneva, Switzerland: European Association for Machine Translation. https://aclanthology.org/2025.mtsummit-1.8/

Liu, Pusheng, Lianwei Wu, Linyong Wang, Sensen Guo, and Yang Liu. 2024. "Step-by-Step: Controlling Arbitrary Style in Text with Large Language Models." In *Proceedings of the 2024 Joint International Conference on Computational Linguistics, Language Resources and Evaluation (LREC-COLING 2024)*, 15285-15295. Torino, Italia: ELRA and ICCL. https://aclanthology.org/2024.lrec-main.1328/

Madaan, Aman, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang, Shashank Gupta, Bodhisattwa Prasad Majumder, Katherine Hermann, Sean Welleck, Amir Yazdanbakhsh, and Peter Clark. 2023. "Self-Refine: Iterative Refinement with Self-Feedback." *arXiv* 2303.17651. https://arxiv.org/abs/2303.17651

Padmakumar, Vishakh, and He He. 2024. "Does Writing with Language Models Reduce Content Diversity?" In *International Conference on Learning Representations 2024*. https://proceedings.iclr.cc/paper_files/paper/2024/hash/02dec8877fb7c6aa9a79f81661baca7c-Abstract-Conference.html

Reif, Emily, Daphne Ippolito, Ann Yuan, Andy Coenen, Chris Callison-Burch, and Jason Wei. 2022. "A Recipe for Arbitrary Text Style Transfer with Large Language Models." In *Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 2: Short Papers)*, 837-848. Dublin, Ireland: Association for Computational Linguistics. https://doi.org/10.18653/v1/2022.acl-short.94

Wikipedia contributors. 2026. "Wikipedia:AI生成文的特徵." *Chinese Wikipedia*. Accessed 2026-03-20. https://zh.wikipedia.org/wiki/Wikipedia:AI%E7%94%9F%E6%88%90%E6%96%87%E7%9A%84%E7%89%B9%E5%BE%B5

Xu, Wenda, Guanglei Zhu, Xuandong Zhao, Liangming Pan, Lei Li, and William Wang. 2024. "Pride and Prejudice: LLM Amplifies Self-Bias in Self-Refinement." In *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 15474-15492. Bangkok, Thailand: Association for Computational Linguistics. https://doi.org/10.18653/v1/2024.acl-long.826

Zamaraeva, Olga, Dan Flickinger, Francis Bond, and Carlos Gomez-Rodriguez. 2025. "Comparing LLM-generated and human-authored news text using formal syntactic theory." In *Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, 9041-9060. Vienna, Austria: Association for Computational Linguistics. https://doi.org/10.18653/v1/2025.acl-long.443

Zanotto, Sergio E., and Segun Aroyehun. 2025. "Linguistic and Embedding-Based Profiling of Texts Generated by Humans and Large Language Models." In *Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing*, 22841-22858. Suzhou, China: Association for Computational Linguistics. https://doi.org/10.18653/v1/2025.emnlp-main.1163

《賈寶適翻譯》. 未刊手稿. [../賈寶適翻譯.txt](../賈寶適翻譯.txt)

《翻译理论要点及提示词优化指南》. 未刊工作文件. [../翻译理论要点及提示词优化指南.md](../翻译理论要点及提示词优化指南.md)
