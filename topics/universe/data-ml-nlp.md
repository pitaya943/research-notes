# 資料探勘、機器學習與自然語言/語音:候選題目
查證日期 2026-11-23。等級:quote-verified / source-exists / model-only。

說明:arXiv 搜尋 API 在本次查證中多次回 429(限流),三個題目的 arXiv 關鍵字搜尋各只成功一次;OpenAlex 的每日額度也一度用完。凡是只看到標題的論文,下面都寫「只看到索引」。

## 這個領域現在在做什麼

- 語音辨識後處理:主流是 generative error correction(GER,生成式錯誤修正),把 ASR 的 N-best 假設交給 LLM 改寫;2026 年的新工作開始問「LLM 改壞了多少」與「能不能給保證」,而不只是再降 WER。`source-exists`(OpenAlex 摘要:Damri & Laufer-Goldshtein ICLR 2026;arXiv 2610.30385 全文)
- 資料串流:concept drift(概念漂移)偵測的研究重心移到「標籤延遲或只有部分標籤」的情境,2026 年有實驗型 survey(Alencar et al., Digital Signal Processing)與評估框架(arXiv 2611.21914)。`source-exists`
- 向量檢索:filtered approximate nearest neighbor search(帶屬性過濾的近似最近鄰搜尋)是資料庫社群目前最擠的題目之一;ACORN(SIGMOD 2024)在 OpenAlex 上有 164 篇引用,最新的引用幾乎每週都有新的索引或 planner。`source-exists`
- 三個方向的共同趨勢:從「再做一個更準的方法」轉向「刻畫何時會壞、並給出可檢驗的保證」。這句是我對上面三點的歸納。`model-only`

## 候選一覽

| 代號 | 題目(一句話問題) | 系上相鄰教授 | 缺口證據強度 | 驗證平台 | 延展到博論 | 主要風險 |
|---|---|---|---|---|---|---|
| ger-damage-control | LLM 修正 ASR 輸出時,能否保證「把原本正確的句子改壞」的比率不超過 α,同時保住大部分修正收益? | 陳冠宇(直接相鄰) | 中強:傷害現象有全文證據;已有的保證針對的是另一種風險 | HyPoradise、HypR、CoVoGER | 分佈偏移下的保證 → 編輯層級的閘門 → 語音翻譯與帶聲音的修正 | 閘門訊號太弱,保證只能靠「幾乎都不修」達成 |
| drift-delay-anytime | 標籤延遲且只有部分標籤時,能否做出誤報率不隨串流長度膨脹的漂移偵測器,並刻畫延遲如何拉長偵測時間? | 戴碧如(領域相符,但近年著作不在串流)、鮑興國(異常偵測) | 中:限制由作者在全文自述;但 conformal martingale 一線的兩篇近作我只看到索引 | River、MOA;Electricity、Airlines、Credit Card Fraud、NSL-KDD、Covertype | 標籤相依延遲 → 帶保證的重訓時機 → 多串流監控 | 既有 conformal martingale 工作可能已涵蓋大半 |
| filtered-ann-plan-regret | 當過濾條件與向量空間相關時,pre/post/in-filter 三種策略的相變邊界移到哪裡?選錯策略的 regret 能否有界? | 吳怡樂(資料庫;近年著作偏影像)、陳冠宇(資訊檢索,較遠) | 弱到中:未解問題由作者自述,但領域極擠 | SIFT1M、YFCC-10M(Big-ANN filtered track)、pgvector/FAISS | 動態更新下的邊界 → 向量+關聯式查詢的成本模型校準 | 被大組搶先;教授相鄰度弱 |

我的排序:ger-damage-control > drift-delay-anytime > filtered-ann-plan-regret。理由見各節。

---

## ger-damage-control: 生成式 ASR 錯誤修正的「不造成傷害」保證

### 一句話
給定 ASR 的 1-best 與 N-best,加一個閘門決定「採用 LLM 的修正」或「保留 1-best」,使改壞率(修正後比 1-best 差的比例或期望 WER 惡化量)以高機率不超過使用者指定的 α,並在此限制下最大化修正收益。

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:這是 selective prediction(選擇性預測)加 risk control(風險控制)。損失函數是「相對於 1-best 的傷害」,閘門是一個門檻化的信心分數,門檻用 Learn then Test 或 conformal risk control 在校正集上選,得到有限樣本保證。`model-only`(這是我提出的建模方式)
- 現有的 repair/damage 分解可以直接當評估量:arXiv 2610.30385 把每一次 WER 變化精確拆成修好的 token、改壞的 token 與插入。`quote-verified`(摘要原文:"We decompose every WER change exactly into repaired tokens, damaged tokens and insertions")
- 驗證:Damri & Laufer-Goldshtein 在 HyPoradise 的三個資料集(TedLium-3、CommonVoice、CHiME-4)上做,取樣 50,000 句,用 LLaMA-2-7B 微調。`quote-verified`(ICLR 2026 全文:"We evaluate our approach on three datasets from the HyPoradise"、"Table 1: LTT WER (%) results with LLaMA-2-7B fine-tuning")

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(HyPoradise,W7133205981,共 104 篇引用,取最新 15 篇):最新的是 2610.30385(量化傷害)、「Ask, Don't Guess」(2610.24809,改成向使用者澄清)、「Read What You Hear」(Interspeech 2026,無參考答案的假設評估)、West Frisian 低資源研究。沒有一篇的標題是對傷害率給保證。`source-exists`(只看到索引)
2. OpenAlex 2025 年後搜尋(關鍵字一次、語意一次):最接近的是 **Damri & Laufer-Goldshtein, ICLR 2026**。它控制的是「相對於該模型與假設集合下最佳可達表現的期望相對 WER 惡化」,調的是 N-best 的大小,不是「要不要修」。`quote-verified`(摘要:"control the expected relative word error rate degradation compared to the best achievable performance for a given model and hypothesis set")。另有 RLLMCF(ASRU 2025)用三階段流程降低幻覺,摘要沒有提保證。`source-exists`(只讀摘要)
3. arXiv 搜尋 `"generative error correction" AND (ASR OR "speech recognition")`,依提交日排序 15 筆:2025–2026 的結果是合成資料、檢索增強、口音、構音障礙、agentic 記憶;沒有傷害率保證。`source-exists`(只看到標題)

判斷:缺口**還在,但不大**。Damri 已經把 risk control 帶進 GER,所以這個題目是「換一個更貼近部署需求的風險,加上偏移」,不是全新方向。要成為論文,必須做到 Damri 沒做的兩件事:(a) 以 1-best 為基準的傷害;(b) 校正集與測試集聲學條件不同時保證還成不成立。Damri 的保證依賴 exchangeability(可交換性)。`quote-verified`("relies only on the mild assumption of exchangeability between calibration and test data")

### 定義這個問題的論文
- Chen et al., "HyPoradise: An Open Baseline for Generative Speech Recognition with Large Language Models", NeurIPS 2023, DOI 10.52202/075280-1375。`source-exists`(只看到索引)
- Kudryavtsev et al., "Text-Only LLM Correctors Break Correct Transcripts: A Repair-Damage Analysis of Audio Grounding in ASR Error Correction", arXiv 2610.30385。原文:"With Gemini 2.5 Flash, text-only correction of its own draft breaks 13–32% of already-correct transcripts; with the audio, 1–3%." `quote-verified`
- Damri & Laufer-Goldshtein, "Confident and Adaptive Generative Speech Recognition via Risk Control", ICLR 2026(OpenAlex W7236519656)。原文:"They predominantly rely on a fixed hypothesis set size across all inputs" `quote-verified`
- Yamashita et al., "LLM-based Generative Error Correction for Rare Words with Synthetic Data and Phonetic Context", Interspeech 2025, DOI 10.21437/interspeech.2025-649。摘要:"neglecting phonetic cues, which leads to over-correction" `source-exists`(只讀摘要)

### 需要的資源
- HyPoradise:Damri 的全文直接使用它,標題自稱 open baseline。`source-exists`;實際下載我沒有試。
- HypR:陳冠宇實驗室自己的資料(Interspeech 2024),見下方教授依據。能否公開下載我沒查。`model-only`
- 運算:Damri 用 LLaMA-2-7B 微調,需要 GPU;實驗室有沒有,**沒查**。若只做校正(不重訓 LLM),Damri 說 "The framework requires only calibration without model retraining"。`quote-verified`

### 碩論的最小版本 / 延展到博論的路線
- 碩論:定義以 1-best 為基準的傷害損失;用 ASR 信心、N-best 一致度、LLM 輸出機率當閘門分數;在 HyPoradise 三個資料集上驗證保證成立,並畫出 α 對修正收益的曲線;再做一組「在乾淨語音校正、在噪音語音測試」看保證何時失效。
- 第二篇:偏移下仍成立的保證(加權校正或線上調整門檻),語言擴到 CoVoGER 的 15 種語言。`source-exists`(CoVoGER 摘要:"across 15 languages and 28 language pairs")
- 第三篇:從整句閘門細到編輯層級(每一處修改各自接受或拒絕),並延伸到語音翻譯與帶聲音輸入的修正器。

### 最可能失敗的原因
閘門分數分不開「會修好」與「會改壞」的句子。那樣保證仍然成立,但只能靠幾乎全部保留 1-best 來達成,結果是沒有收益的空保證。2610.30385 自己也說它的對照 "bound the effect of the audio and do not isolate it",表示這個現象的成因還不清楚。`quote-verified`

### 台科大資工系相鄰的教授(附依據)
- **陳冠宇**:教師名單「資訊檢索、文件摘要、語音辨識、自然語言處理、深度機器學習」。`quote-verified`(ntust-csie-faculty.md)。實驗室頁面列有 "Localized Optimization for LLM-based ASR Generative Error Correction"(ISCSLP 2026)、"LLM-Driven Hypothesis Set Refinement for Enhanced ASR Post-Processing"(APSIPA ASC 2025)、"HypR: A Comprehensive Study for ASR Hypothesis Revising with a Reference Corpus"(INTERSPEECH 2024)、"An Integrated Framework for ASR Post-Processing with Error Correction, Rescoring, and Text-Speech Matching"(2026)。`quote-verified`(papers/_ntust/labs/陳冠宇.txt)。這是三個候選裡相鄰度最高的:實驗室正在做同一個任務。

---

## drift-delay-anytime: 標籤延遲下、誤報率不隨串流長度膨脹的漂移偵測

### 一句話
標籤晚到、而且只有部分樣本有標籤時,設計一個漂移偵測器,使整段串流的誤報機率有一個與串流長度無關的上界,並推導偵測延遲如何隨標籤延遲與少數類別比例變化。

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:在不受類別先驗影響的座標(recall、specificity,多類別時用 confusion matrix 的列)上建一個 e-process 或 conformal test martingale(保形檢定鞅),用 Ville 不等式得到任意停止時間都成立的誤報界;標籤延遲進來後,統計量只能用「已到的標籤」更新,偵測延遲因此有一個下界。`model-only`(我提出的建模方式;座標的選擇來自 2611.21914)
- 「不受先驗影響的座標」已有現成的理論:arXiv 2611.21914 用它區分 prior shift 與 concept drift。`quote-verified`("We separate prior shift from concept drift by monitoring prior-free coordinates (recall and specificity)")
- 驗證:該文所有資料集都公開(Electricity、Airlines、Credit Card Fraud、NSL-KDD、Covertype),程式碼在 GitHub。`quote-verified`("Data availability. All data sets are public")。對照組用 River 與 MOA 的現成偵測器。`quote-verified`(2611.21914:"the standard river detectors";2611.17981:"All learners run with their MOA default configurations")

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(Grzenda, Gomes & Bifet 2019/2020 "Delayed labelling evaluation for data streams",W2989114215,61 篇引用,取最新 15 篇):最新的是 2611.21914(評估框架)、2611.17981(延遲下的 oversampling)、表格基礎模型上串流(2610.05352)、Alencar 等人的實驗型 survey、以及一篇 "Adaptive Conformal Risk Control for Payment Fraud Detection under Concept Drift and Delayed Feedback"(DOI 10.54691/kzrea147)。`source-exists`(只看到索引;最後一篇的期刊我不認識,沒有打開)
2. OpenAlex 2025 年後搜尋(關鍵字一次、語意一次):TPAMI 2026 的 "Latency-Aware Adaptive Learning for Nonstationary Data Streams"(處理延遲下的學習,不是偵測保證);"Optimal distribution-free sequential change detection with conformal martingales"(arXiv 2609.27179,摘要說對象是 independent observations,沒有提延遲標籤);**"Semi-supervised concept drift detection and adaptation based on conformal martingale framework"(Journal of Process Control 2025)與 "ConformalGuard ... Anytime-Valid Guarantees"(ADMA 2026)我只看到標題,沒有摘要可讀**。`source-exists`
3. arXiv 搜尋 `"concept drift" AND ("delayed labels" OR "label delay" OR "verification latency")`:13 筆,最新兩筆就是 2611.21914 與 2611.17981(2026-11-16、11-13)。`source-exists`

判斷:**缺口部分存在,無法完全判斷**。限制是作者自己寫的(見下方引用),而且是一週前的論文。但第 2 項裡那兩篇 conformal martingale 的工作我沒能讀內容;如果其中一篇已經處理延遲標籤,這個題目就只剩「不受先驗影響的座標」這一點新意。動手前必須先讀這兩篇。

### 定義這個問題的論文
- Grzenda, Gomes & Bifet, "Delayed labelling evaluation for data streams", Data Mining and Knowledge Discovery, DOI 10.1007/s10618-019-00654-y。`source-exists`(只看到索引)
- Prati & de Faria, "A Temporal Algebraic Framework for Data Stream Evaluation Under Delayed Labels and Distribution Shifts", arXiv 2611.21914。原文:"A calibrated threshold that depends on $\alpha$ and on the stream length is needed for a given run-level false alarm, and the power for a recall change is low at low prior. The variance model of the detector assumes independent arrivals" `quote-verified`
- 同上,關於標籤相依延遲:"The correction of selection bias is only partial when the delay depends on the label" `quote-verified`
- Awad & Sengar, "When Does Oversampling Help? ...", arXiv 2611.17981。原文:"hyperparameter optimisation under label delay is itself an open problem" `quote-verified`
- Alencar et al., "Concept drift detection in delayed and partially labeled data streams: An experimental survey", DOI 10.1016/j.dsp.2026.106325。`source-exists`(只讀摘要的前半)

### 需要的資源
全部是公開表格資料與開源套件,筆電可跑。`quote-verified`(資料公開一事,見上)。不需要 GPU。`model-only`

### 碩論的最小版本 / 延展到博論的路線
- 碩論:固定延遲、獨立到達的假設下,做出 e-process 偵測器並證明誤報界;在合成串流上驗證誤報率確實不隨長度上升(對照 DDM、ADWIN 等);在五個公開資料集上量偵測延遲。
- 第二篇:延遲與標籤相依、或由模型挑選要標哪些樣本(selective labels)時的修正;這正是 2611.21914 說只修正了一部分的地方。
- 第三篇:把偵測接到「何時重訓」的決策,給出累積損失的界;或做成多條串流同時監控、控制整體錯誤發現率的系統(這一步有後端與資料工程成分)。

### 最可能失敗的原因
相依到達。真實串流的錯誤是自相關的,2611.21914 已經指出它的變異數模型在原始順序下不成立;e-process 在相依資料下要嘛保證失效、要嘛保守到偵測不出東西。

### 台科大資工系相鄰的教授(附依據)
- **戴碧如**:教師名單「資料探勘、串流資料管理、生物資訊」;實驗室頁面研究領域列「資料串流管理」,課程有「資料串流探勘」,著作有 "A G-Means Update Ensemble Learning Approach for the Imbalanced Data Stream with Concept Drifts"(DaWaK 2016)。`quote-verified`(faculty 檔與 papers/_ntust/labs/戴碧如.txt)。**但要直說**:該頁面最新的著作是 2016 年;OpenAlex 上 2019 年後的 8 篇是社群推薦、假新聞、影響力最大化,以及一篇 PAKDD 2024 的多變量時間序列異常偵測。`source-exists`。也就是領域標籤相符,但近年沒有串流漂移的論文。
- **鮑興國**:教師名單「機器學習、資料科學、電腦視覺、資訊安全」;實驗室頁面列 "Anomaly Detection",著作有 "Self-supervised Federated Learning for Anomaly Detection"(IEEE BigData 2023)、"Reconsider Time Series Analysis for Insider Threat Detection"(IEEE BigData 2024)。`quote-verified`(papers/_ntust/labs/鮑興國.txt)。相鄰在「隨時間監控、控制誤報」這一面,不是漂移偵測本身。

---

## filtered-ann-plan-regret: 相關過濾條件下,filtered ANN 執行策略的相變邊界與 regret

### 一句話
過濾條件與查詢向量相關時(例如「類別=鞋」而查詢本身就是鞋的圖片),pre-filter、post-filter、in-filter 三種執行策略各自勝出的區間邊界會移到哪裡?一個同時考慮三種策略、帶遲滯的選擇器,在選擇率估計有誤差時的 recall 損失能否有界?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型已有雛形:post-filter 的邊界來自順序統計量,in-filter 的邊界來自圖上的 site percolation(位置滲流)。`quote-verified`(arXiv 2606.16341:"order statistics place the post-filter cliff at $s\approx k/K$, and site percolation places the in-filter cliff at $s_{c}\approx 0.83/M$ for graph degree $M$")
- 這個模型假設過濾條件與幾何無關;要加的是相關性。VADER 把相關性定義為 "the relationship between the filter and the vector space",並指出它會顯著改變搜尋難度。`source-exists`(只讀摘要)
- 驗證:SIFT1M(2606.16341 使用)與 NeurIPS'23 Big-ANN 競賽的 filtered track 資料 YFCC-10M。`quote-verified`(2611.16148:"YFCC-10M is the filtered-search dataset of the NeurIPS'23 Big-ANN competition")。2606.16341 的程式碼與預註冊公開在 GitHub。`quote-verified`
- 對學習者的背景:這題的實作面是 pgvector/PostgreSQL 的查詢規劃,與 SQL 和後端經驗直接相關。`model-only`

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(ACORN,W4399174383,164 篇引用,取最新 15 篇):2026 年 10–11 月就有 "Calibrated Early Termination with Bounded Optimality Gaps for Constrained Vector Search"(2611.16148)、"Per-Query Recall Guarantees ... in Dynamic Vector Databases"(2611.07579)、"Fidelity Synopses: Pre-Execution Plan Selection"(2610.41407)、兩篇新索引。`source-exists`
2. OpenAlex 2025 年後搜尋(關鍵字、語意各一次):VADER(Google 等,2609.38053,用學到的 recall 預測器處理選擇率與相關性)、"Efficient Filtered-ANN via Learning-based Query Planning"(2602.17914)、RACORN(2607.00768)、SIEVE(VLDB 2025)、Chronis 等人的 VLDB 2025 教學。`source-exists`(只讀摘要)
3. arXiv 搜尋 `filtered AND "nearest neighbor" AND selectivity`:15 筆中相關的有 "Recall-Bounded Staleness for Filtered Multi-Tenant ANN Graph Indexes under Concurrent Updates"(2610.15130)、"Query-aware Routing for Filtered Approximate Nearest Neighbors Search"(2606.19898)等。`source-exists`(只看到標題)

判斷:**缺口狹窄**。「相關過濾條件下的解析邊界」確實被 2606.16341 列為未做(見引用),VADER 這類工作是用學到的預測器繞過去,沒有給解析模型(依摘要判斷)。但這個領域每月都有大組的新論文,而且 2606.16341 的作者自己說 "the novelty is correspondingly modest"。`quote-verified`。一年內被搶先的機率在三個候選中最高。

### 定義這個問題的論文
- Patel et al., "ACORN: Performant and Predicate-Agnostic Search Over Vector Embeddings and Structured Data", Proc. ACM Manag. Data 2024, DOI 10.1145/3654923。`source-exists`(只看到索引)
- Mandarapu & Kunkunuru, "Filtered ANN as a Phase Transition: When Selectivity-Estimation Error Causes Plan Regret", arXiv 2606.16341。原文:"folding all three into one regret selector, the full adversarial correlated-predicate sweep, and a derived hysteresis rule are future work." `quote-verified`
- 同上:"a biased cost model opens a persistent miscalibration band that estimation-error robustness cannot fix." `quote-verified`
- Chatzakis et al., "VADER: Filtered Vector Search with Declarative Recall", arXiv 2609.38053。`source-exists`(只讀摘要)

### 需要的資源
公開資料集與開源索引(FAISS、pgvector、hnswlib),單機記憶體即可做百萬級;千萬級的 YFCC-10M 需要較大記憶體,數量我沒查。`model-only`

### 碩論的最小版本 / 延展到博論的路線
- 碩論:在 2606.16341 的模型上加一個相關性參數,推導兩條邊界的位移;在 SIFT1M 與 YFCC-10M 的真實標籤上檢驗;做一個三策略選擇器並量 regret。
- 第二篇:索引持續更新、多租戶時邊界如何漂移(2610.15130 的標題顯示已有人在做 staleness 這一面)。
- 第三篇:把校正過的成本模型放進向量加關聯式的查詢最佳化器(join、多個過濾條件)。

### 最可能失敗的原因
被搶先。原作者把這三件事明列為 future work,Google 與多個資料庫組都在同一題上。

### 台科大資工系相鄰的教授(附依據)
- **吳怡樂**:教師名單「資料庫系統、資料探勘、地理資訊系統、多媒體資料庫、影像分類」;實驗室頁面著作有 "Proposed Novel Method of Scalable Images Search Based on Binarized SIFT Features on Hbase"(NCS 2021)與 "Scalable Indexing Method for SIFT Features"。`quote-verified`(faculty 檔與 papers/_ntust/labs/吳怡樂.txt)。**但**頁面上 2022–2025 的著作是超解析度、NeRF、多物件追蹤;OpenAlex 2019 年後的 11 篇也以影像為主。`source-exists`。相鄰度:有歷史淵源,近年不活躍。
- **陳冠宇**:專長含「資訊檢索」,有 "RATE: A Retrieval-Augmented Transformer for Regional Earthquake Early Warning"。`quote-verified`(papers/_ntust/labs/陳冠宇.txt)。這是檢索的使用者,不是索引或查詢最佳化的研究者,相鄰度低。
- 結論:判準 6 勉強算符合,是三個候選裡最弱的。

---

## 沒有採用的方向

- **過濾式 HNSW 的滲流門檻本身**:已被 arXiv 2606.16341 做掉($s_c \approx 0.83/M$)。`quote-verified`
- **依輸入難度調整 N-best 大小並給保證**:已被 Damri & Laufer-Goldshtein(ICLR 2026)做掉。`quote-verified`
- **標籤延遲下的串流 oversampling**:arXiv 2611.17981 剛做完 15 個分類器、12 條串流的比較,結論是 oversampling 的邊際貢獻只在一個真實設定中可偵測。`quote-verified`(摘要)。純實驗比較沒有再做一次的空間。
- **每個查詢的 recall 保證(無過濾的 ANN)**:2026 年 10–11 月已有 2611.16148、2611.07579、VADER 三篇。`source-exists`
- **檢索增強生成(RAG)的 conformal 保證**:沒有查。技術上與既有候選 `eew-conformal` 重疊,所以沒有展開;這不是「已被做完」的判斷。
- **陳怡伶方向(社群網路分析、連結預測)**:實驗室頁面(papers/_ntust/labs/陳怡伶.txt)列出的最新著作是 2017 年(PerCom 2017),沒有近期方向可以貼上去,所以沒有以她為錨提候選。`quote-verified`(就該檔案內容而言;頁面可能沒更新)

## 來源

| id | 標題 | 開到的程度 |
|---|---|---|
| arXiv 2610.30385 | Text-Only LLM Correctors Break Correct Transcripts: A Repair-Damage Analysis of Audio Grounding in ASR Error Correction | 全文(讀了摘要、討論與限制、結論) |
| OpenAlex W7236519656(ICLR 2026) | Confident and Adaptive Generative Speech Recognition via Risk Control | 全文 PDF 轉文字(讀了摘要、引言、實驗設定、結論片段) |
| DOI 10.52202/075280-1375 | HyPoradise: An Open Baseline for Generative Speech Recognition with Large Language Models | 索引 |
| DOI 10.21437/interspeech.2025-649 | LLM-based Generative Error Correction for Rare Words with Synthetic Data and Phonetic Context | 摘要 |
| DOI 10.1109/asru65441.2025.11434775 | Fewer Hallucinations, More Verification: A Three-Stage LLM-Based Framework for ASR Error Correction | 摘要 |
| DOI 10.18653/v1/2025.emnlp-main.320 | CoVoGER: A Multilingual Multitask Benchmark for Speech-to-text Generative Error Correction with Large Language Models | 摘要 |
| DOI 10.1109/taslpro.2025.3551083 | ASR Error Correction Using Large Language Models | 摘要(未在正文引用其內容) |
| arXiv 2610.24809 | Ask, Don't Guess: Targeted Clarification for Speech Recognition Errors | 索引 |
| arXiv 2611.21914 | A Temporal Algebraic Framework for Data Stream Evaluation Under Delayed Labels and Distribution Shifts | 全文(讀了摘要、引言、結論與限制、資料聲明) |
| arXiv 2611.17981 | When Does Oversampling Help? A Decision Framework for Imbalanced Data Streams Under Label Delay and Concept Drift | 全文(讀了摘要、引言、相關工作片段) |
| DOI 10.1007/s10618-019-00654-y | Delayed labelling evaluation for data streams | 索引 |
| DOI 10.1016/j.dsp.2026.106325 | Concept drift detection in delayed and partially labeled data streams: An experimental survey | 摘要(截斷) |
| DOI 10.1109/tpami.2026.3742576 | Latency-Aware Adaptive Learning for Nonstationary Data Streams: Coordinating Concept Drift and Delayed Supervision | 摘要 |
| arXiv 2609.27179 | Optimal distribution-free sequential change detection with conformal martingales | 摘要(截斷) |
| DOI 10.1016/j.jprocont.2025.103374 | Semi-supervised concept drift detection and adaptation based on conformal martingale framework | 索引(未讀;動手前必讀) |
| DOI 10.1007/978-981-92-7738-4_3 | ConformalGuard: Unified Streaming Data Quality Monitoring via Hierarchical Conformal Martingales with Anytime-Valid Guarantees | 索引(未讀;動手前必讀) |
| DOI 10.54691/kzrea147 | Adaptive Conformal Risk Control for Payment Fraud Detection under Concept Drift and Delayed Feedback | 索引 |
| arXiv 2606.16341 | Filtered ANN as a Phase Transition: When Selectivity-Estimation Error Causes Plan Regret | 全文(讀了摘要、限制、結論) |
| DOI 10.1145/3654923 | ACORN: Performant and Predicate-Agnostic Search Over Vector Embeddings and Structured Data | 索引 |
| arXiv 2609.38053 | VADER: Filtered Vector Search with Declarative Recall | 摘要 |
| arXiv 2611.16148 | Calibrated Early Termination with Bounded Optimality Gaps for Constrained Vector Search | 全文已下載(讀了摘要與提到過濾、資料集的段落) |
| arXiv 2610.41407 | Fidelity Synopses: Pre-Execution Plan Selection for Approximate Vector Search | 全文已下載(讀了摘要、限制) |
| arXiv 2602.17914 | Efficient Filtered-ANN via Learning-based Query Planning | 摘要 |
| arXiv 2607.00768 | RACORN: Robust Predicate-Agnostic Filtered Vector Search under Extreme Selectivity | 摘要 |
| DOI 10.14778/3749646.3749725 | SIEVE: Effective Filtered Vector Search with Collection of Indexes | 摘要 |
| DOI 10.14778/3750601.3750700 | Filtered Vector Search: State-of-the-Art and Research Opportunities | 摘要 |
| arXiv 2611.07579 | Per-Query Recall Guarantees for Approximate Nearest Neighbor Search in Dynamic Vector Databases | 索引 |
| arXiv 2610.15130 | Recall-Bounded Staleness for Filtered Multi-Tenant ANN Graph Indexes under Concurrent Updates | 索引 |
| arXiv 2606.19898 | Query-aware Routing for Filtered Approximate Nearest Neighbors Search | 索引 |
| 本地檔 | topics/universe/ntust-csie-faculty.md;papers/_ntust/labs/{鮑興國,戴碧如,吳怡樂,陳冠宇,陳怡伶}.txt | 已讀(五個檔都有內容,沒有空檔) |
| OpenAlex 作者查詢 | Bi-Ru Dai(A5077426816)、Yi-Leh Wu(A5002503247)2019 年後著作列表 | 索引 |

沒有用 Consensus(0 次)。
