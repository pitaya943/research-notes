# 資安與可信任 AI(台科大資工系的做法):候選題目
查證日期 2026-11-23。等級:quote-verified / source-exists / model-only。

說明:`quote-verified` 表示我在本次開啟的全文(或系上教師檔、實驗室網頁擷取檔)裡找到那句話;`source-exists` 表示只看到索引紀錄或摘要;`model-only` 是我自己的判斷或提案,沒有查證。每一題的「模型」與「最小版本」都是我的提案,一律是 `model-only`,要由指導教授判斷可不可行。

## 這個領域現在在做什麼

- 聯邦式(federated,多方不交換資料共同訓練)LLM 微調的安全性在 2025–2026 很熱:Ye 等人(ICLR 2025)指出惡意客戶可破壞安全對齊(safety alignment),之後出現 Safe-FedLLM(ACL 2026)、FedDetox(IJCNN 2026)、Gradient Assembly Poisoning(arXiv 2601.00566)、Ghost in the Cloud(ICLR 2026)等攻防論文。`source-exists`(OpenAlex 引用清單與摘要)
- 惡意程式偵測器的可證明強健性(certified robustness,對某一類擾動給出數學保證)集中在 Windows PE 的 byte 層級:RS-Del(arXiv 2302.01757)、Gibert 等人的 (de)randomized smoothing 系列(AISec 2023、ACM TOPS 2026)。`source-exists`;RS-Del 為 `quote-verified`
- 隱私稽核(privacy auditing,用實際攻擊量出差分隱私 ε 的經驗值)的主力是 Google 團隊:聯邦版的 one-shot 估計(arXiv 2302.03098,ICLR 2024)與 2026 年針對 LoRA 微調 LLM 的集中式稽核(arXiv 2606.10481)。`quote-verified`(兩篇全文都開過)
- 系上的重心:鄭欣明做 IoT 惡意程式偵測器的對抗攻擊與強健性、O-RAN 安全;王紹睿有 2026–2028 國科會計畫「聯邦式大型語言模型的資安弱點與防禦方法」;許之凡做 FHEW/TFHE 加密推論與聯邦學習隱私。`quote-verified`(`papers/_ntust/labs/` 各檔)

## 候選一覽

| 代號(英文 slug) | 題目(一句話問題) | 系上相鄰教授 | 缺口證據強度 | 驗證平台 | 延展到博論 | 主要風險 |
|---|---|---|---|---|---|---|
| `fedlora-privacy-audit` | 客戶只上傳 LoRA 因子 (A, B) 且伺服器分開平均時,one-shot canary 估出的 ε 還準嗎?偏差隨 rank 怎麼變? | 王紹睿、許之凡(林佳志:只有專長標籤) | 中強:arXiv 兩個查詢 0 筆,OpenAlex 9 筆無一命中 | 2302.03098 用的 Stackoverflow / EMNIST;2606.10481 用的 Enron | 稽核 → 用稽核結果收緊 DP 會計 → 稽核合成資料與異質隱私等級 | 答案可能是「直接套用就行」,變薄 |
| `iot-malware-transform-cert` | 對結構特徵(CFG / 呼叫圖)的 IoT ELF 惡意程式偵測器,能否給出以「語意保持轉換次數」為半徑的強健性證書,而不是 byte 編輯距離? | 鄭欣明、鄧惟中(借調中)、鮑興國 | 中:OpenAlex 精確查詢只回 2 筆無關書章;但有 2 篇鄰近預印本只讀到摘要 | 鄭欣明 2024 論文的攻擊(轉換集合已定義);資料集公開與否未確認 | 指令層證書 → 圖層證書 → 跨架構(P-code)與動態特徵 | 證書半徑遠小於實際轉換造成的改動,結果是空的 |
| `fedit-safety-detectability` | 在資料異質的聯邦 LoRA 指令微調中,破壞安全對齊的惡意更新何時「統計上不可偵測」?臨界惡意比例如何隨異質度與 rank 變? | 王紹睿(許之凡:FedTrust) | 弱到中:題目很擠,ICLR 2026 已有「單一攻擊者」的經驗結果;沒看到理論門檻,但 arXiv 查詢多次失敗 | OpenFedLLM、Beavertails、AdvBench(Ye 等人全文所用) | 門檻模型 → 帶保證的聚合規則 → 與 DP、後門合併分析 | 被大組搶先;需要 GPU |

我沒有提出第四題;看過的 FHE 方向見「沒有採用的方向」。

---

## fedlora-privacy-audit: 聯邦 LoRA 微調下 one-shot 隱私稽核是否仍然成立

### 一句話
Andrew 等人的 one-shot 方法假設「模型是所有客戶更新的線性組合加高斯雜訊」;聯邦 LoRA 把 A、B 分開平均後,全域更新是 (平均 A)(平均 B),不再是線性組合。問:這時 canary 餘弦統計量的虛無分佈與 ε 估計還對嗎,偏差是 rank 與客戶數的什麼函數?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型(我的提案,`model-only`):把 canary 放在因子空間 (A, B) 或乘積空間 ΔW 兩種做法分別寫成假設檢定;推導分開平均造成的交叉項 (Ā·B̄ − mean(A_i B_i)) 對檢定統計量平均值與變異數的影響;在單輪高斯機制下求閉式,與解析 ε 比對。
- 原方法的理論支點是高維與線性:「We show that in the tractable case of a single application of the Gaussian mechanism, our method provably recovers the true, analytical ε in the limit of high dimensionality.」`quote-verified`(2302.03098 全文)
- 線性假設原文:「each model iterate is a linear combination of all updates received thus far, plus Gaussian noise.」`quote-verified`(2302.03098)
- 分開平均的結構性後果已被攻擊論文點出:「the effective model update emerges from the multiplicative interaction between aggregated components. This interaction fundamentally differs from additive gradient aggregation」`quote-verified`(2601.00566 全文)
- LoRA 在 DP 聯邦下有「compounded noise amplification」`source-exists`(2602.19926,只讀摘要):這是可以用稽核量化的對象。

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(2302.03098,最新 15 筆,共 45 筆):最新引用是醫學影像生成的 canary 估計(SIU 2026)、合成資料黑箱稽核、KV-Auditor、UniAud、The Last Iterate Advantage 等;標題中沒有 LoRA 或 LLM 聯邦微調。`source-exists`(只看標題;第 16–45 筆沒看)
2. `search_works`(2025 起,federated AND LoRA AND 稽核/成員推論):9 筆,最接近的是 FedRand(arXiv 2503.07216,隨機 LoRA 子參數更新以提升隱私)與 DP refactorization(arXiv 2605.08443),都是防禦設計,不是稽核。`source-exists`(只看標題)
3. arXiv:`"privacy auditing" AND LoRA` 回 0 筆;`federated AND "language model" AND "privacy auditing"` 回 0 筆;`"privacy auditing" AND federated` 回 5 筆,無 LoRA。`source-exists`
- 2606.10481(Google,2026-06)做的是集中式、樣本層級 canary,並研究 LoRA rank 對記憶的影響;我在全文搜尋 federated / user-level,只在參考文獻出現一次。`quote-verified`(全文搜尋結果)
- 判斷:缺口目前看起來還在,但同一組人(Andrew、Kairouz、McMahan 同時是兩篇的作者)最有能力補上。後半句是 `model-only`。

### 定義這個問題的論文
- arXiv 2302.03098,One-shot Empirical Privacy Estimation for Federated Learning(ICLR 2024)。引文:「We stress that our empirical ε estimate should not be construed as a formal bound on the worst-case privacy leakage.」`quote-verified`
- arXiv 2606.10481,Advancing the State-of-the-Art in Empirical Privacy Auditing。引文:「We find that the canary entropy that maximizes memorization increases with LoRA rank」`quote-verified`
- arXiv 2601.00566,Low Rank Comes with Low Security(說明分開聚合的乘法結構)。`quote-verified`
- arXiv 2602.19926,Rethinking LoRA for Privacy-Preserving Federated Learning in Large Models(ICLR 2026)。`source-exists`(摘要)

### 需要的資源
- 資料:2302.03098 的實驗用 Stackoverflow 與 EMNIST(全文表 4、表 5 標題),2606.10481 用 Enron 與 Gemma3。`quote-verified`。這些資料集能否直接下載我沒有在本次確認。
- 計算:小模型(例如 2601.00566 用過的 GPT-2)應可在單張 GPU 完成;這是 `model-only`。
- 不需要特定人提供的資料。

### 碩論的最小版本 / 延展到博論的路線
- 最小版本(`model-only`):(a) 單輪高斯機制下因子空間 canary 的閉式分析;(b) 在一個小模型上重做 one-shot 估計,比較「乘積聚合」與「A、B 分開平均」兩種規則下 ε_est 與解析 ε 的差;(c) rank ∈ {4, 16, 64} 掃描。
- 第二篇:用稽核出的鬆弛量設計較緊的 LoRA 專用 DP 會計或雜訊配置(接 2602.19926 的雜訊放大問題)。
- 第三篇:異質隱私等級客戶(接許之凡的 HPFL)或合成資料釋出(接 2606.10481 的 data audit)的聯邦稽核。

### 最可能失敗的原因
LoRA 因子的維度仍然很高,交叉項在實務客戶數下可能小到可以忽略,結論變成「原方法直接可用」,只剩一篇實驗報告的份量。`model-only`

### 台科大資工系相鄰的教授(附依據)
- 王紹睿:計畫「2026-2028 國科會計畫: 聯邦式大型語言模型的資安弱點與防禦方法」;研究興趣「隱私強化技術: 解決人工智慧系統(聯邦式學習、大語言模型等)的用戶資料隱私問題,包括使用差分隱私、安全多方運算等技術」。`quote-verified`(`王紹睿.txt`)
- 許之凡:論文「Federated Learning Using Multi-Modal Sensors with Heterogeneous Privacy Sensitivity Levels」(ACM TOMM 2024)。`quote-verified`(`許之凡.txt`)
- 林佳志:教師表列「可信任人工智慧、隱私強化技術、機密運算、硬體信任根」。`quote-verified`(`ntust-csie-faculty.md`);沒有實驗室網頁檔,無法確認實際做不做 DP 稽核。

---

## iot-malware-transform-cert: 以語意保持轉換為半徑的 IoT 惡意程式偵測強健性證書

### 一句話
現有證書以 byte 編輯距離或 patch 大小為半徑;鄭欣明實驗室的攻擊用的是函式內聯、控制流平坦化等程式層轉換。問:對 CFG / 呼叫圖特徵的偵測器,能否證明「任意 k 次指定轉換都改不了判定」,k 可以做到多大?

### 為什麼不會太薄
- 模型(`model-only` 提案):把每種轉換對圖特徵(或指令 chunk 序列)的影響上界化為一個「轉換成本」;在指令 chunk 上套 RS-Del 的隨機刪除平滑,或在圖上做邊/節點刪除平滑,推出以轉換次數計的證書半徑;再量 certified accuracy 對半徑的曲線。
- RS-Del 自己說明威脅模型其實是語意物件:「attackers are likely to edit semantic objects in an executable (e.g., whole instructions), rather than at the byte-level.」並且方法「compatible with chunking」。`quote-verified`(2302.01757 全文)
- 攻擊端已由系上定義好:「semantic-preserving binary diversification techniques, including function inlining, branch function insertion, control flow graph flattening, and basic block merging and reordering」,對結構式偵測器「achieving evasion rates of up to 100% on certain detectors, with an average binary size increase of just 8.35%」。`source-exists`(OpenAlex 摘要,DOI 10.1109/access.2024.3513713)

### 缺口還在嗎
1. `list_citations`(RS-Del,最新 12 筆,共 33 筆):Gibert 等人 ACM TOPS 2026(end-to-end 偵測器的 (de)randomized smoothing)、EXE-Bench(TIFS 2026)、ByteShield、Certifiably Robust Malware Detectors by Design(IFIP SEC 2025)、Certified Robustness Under Bounded Levenshtein Distance(ICLR 2025)、以及一篇立場文「Position: Certified Robustness Does Not (Yet) Imply Model Security」。全是 Windows / byte 或一般序列。`source-exists`(只看標題)
   `list_citations`(Gibert AISec 2023,共 19 筆):同一群 Windows PE 工作,加上 arXiv 2604.20495。`source-exists`
2. `search_works`(2024 起,malware AND (IoT OR ELF OR CFG OR opcode) AND certified/provable robustness):2 筆,都是無關的書章。`source-exists`
3. arXiv:`malware AND "randomized smoothing"` 8 筆,`malware AND ("certified robustness" OR "problem-space")` 12 筆,無 IoT / ELF / 圖特徵的證書;`malware AND IoT AND adversarial AND certified` 0 筆(該查詢前兩次連線失敗,第三次成功)。`source-exists`
- 兩篇必須先讀全文的鄰近工作,我只讀了摘要:arXiv 2604.20495 用特徵消融的平滑,對 PyMetaEngine 的變形給「feature-space perturbations」的證書;arXiv 2508.10038 提出「certifiably robust malware detection by design」的架構 ERDALT。`source-exists`。如果其中一篇已涵蓋圖特徵與程式層轉換,這題的缺口就小很多:**缺口部分無法判斷**。

### 定義這個問題的論文
- arXiv 2302.01757,RS-Del: Edit Distance Robustness Certificates for Sequence Classifiers via Randomized Deletion。引文:「RS-Del can achieve a certified accuracy of 91% at an edit distance radius of 128 bytes」`quote-verified`
- DOI 10.1109/access.2024.3513713,An Adversarial Attack on ML-Based IoT Malware Detection Using Binary Diversification Techniques(Mwangi、Cheng)。`source-exists`(摘要)
- DOI 10.1145/3689427,Improving Robustness in IoT Malware Detection through Execution Order Analysis(Lin、Wang、Cheng、Lee)。`source-exists`(摘要前段)
- arXiv 2508.10038、arXiv 2604.20495。`source-exists`(摘要)

### 需要的資源
- 資料集:鄭欣明論文摘要只說「a large-scale dataset of IoT ELF binaries」,沒說是否公開。`source-exists`。實驗室網頁提到「IoT honeynets at scale」(`quote-verified`,`鄭欣明.txt`),資料很可能是實驗室自有的。**判準 4(資源拿得到)目前無法判斷**,要直接問老師。
- 轉換工具:要能對 ELF 做上述四種轉換;是否有公開工具我沒有查。
- 計算:平滑分類器要對每個樣本取樣多次,靜態特徵下應可負擔。`model-only`

### 碩論的最小版本 / 延展到博論的路線
- 最小版本(`model-only`):只取一種特徵(例如基本區塊序列)與兩種轉換(區塊重排、分支函式插入),推出每次轉換的 chunk 編輯距離上界,套 RS-Del 得到以轉換次數計的證書,並用鄭欣明 2024 的攻擊做經驗對照。
- 第二篇:圖層級證書(對 CFG / 呼叫圖的結構擾動),處理控制流平坦化這種全域轉換。
- 第三篇:跨 CPU 架構(接鄧惟中的 P-code 中間表示)或動態/執行順序特徵的證書。

### 最可能失敗的原因
控制流平坦化與函式內聯會改掉幾乎整個圖,任何以編輯距離為基礎的半徑都遠小於一次轉換的改動,證書是空的。那時論文只剩「這條路走不通」的負面結果,需要改走不變特徵路線(可能正是 2508.10038 做的事)。`model-only`

### 台科大資工系相鄰的教授(附依據)
- 鄭欣明:「AI Security: Adversarial & backdoor attacks on ML models, robustness evaluation of malware detectors」;期刊 J41「An adversarial attack on ML-based IoT malware detection using binary diversification techniques」、J42「Improving robustness in IoT malware detection through execution order analysis」。`quote-verified`(`鄭欣明.txt`)
- 鄧惟中:「Cross-Architecture IoT Malware Analysis Using P-Code Intermediate Representation」(ARTMAN '25)。`quote-verified`(`鄧惟中.txt`)。但同檔寫「Seconded to Telecom Technology Center from August 2025」,借調中,能否收學生要確認。
- 鮑興國:資安項目列「Anomaly Detection, Botnet, Malware Detection, Gamebot」。`quote-verified`(`鮑興國.txt`);只有領域標籤層級的相鄰。

---

## fedit-safety-detectability: 聯邦 LoRA 指令微調中安全攻擊的可偵測性門檻

### 一句話
已知少數(甚至一個)惡意客戶就能破壞全域 LLM 的安全對齊,而且穩健聚合擋不住;沒看到的是「為什麼」的量化模型。問:在客戶資料異質度 ζ、LoRA rank r、惡意比例 ρ 下,惡意更新在安全子空間上的投影何時落在良性更新的變異之內(不可偵測),臨界 ρ*(ζ, r) 是多少?

### 為什麼不會太薄
- 模型(`model-only` 提案):把每輪聚合更新分解成安全子空間分量與其餘分量;良性分量視為均值 0、變異由異質度決定的雜訊,惡意分量是固定漂移;把伺服器的偵測寫成假設檢定,得到不可偵測區域與累積漂移越過拒答邊界所需輪數。
- 問題的經驗面已確立:「classical defense methods in FL (six representatives are considered) fail to defend against our attack method, which at most brings 4% safety improvement」`quote-verified`(2406.10630 全文)
- 作者自己把模型層過濾稱為未解:「relieving the need for filtering out malicious clients via model-level operation which is currently unsolvable.」`quote-verified`(2406.10630)
- 低秩結構造成跨輪累積:「even a minority of malicious clients can gradually bias the global update direction while remaining within per-round validation bounds.」`quote-verified`(2601.00566 全文)

### 缺口還在嗎
1. `list_citations`(2406.10630,最新 20 筆,共 35 筆):Safe-FedLLM(ACL 2026)、FedDetox(IJCNN 2026)、ShadeEdit(AAAI 2026)、Ghost in the Cloud(ICLR 2026)、2601.00566、圖表示學習輔助的模型操縱(arXiv 2605.07961)等。攻防都很活躍。`source-exists`
2. `search_works`(2025 起,federated + LoRA/LLM + safety alignment + Byzantine/robust aggregation/subspace):3 筆。Ghost in the Cloud 的摘要說「a single attacker can significantly compromise the safety alignment of LLMs through malicious training, and existing defenses like robust aggregation or trust-based frameworks fail under this setting due to data heterogeneity」(摘要被截斷,後半提出什麼我沒看到)。`source-exists`
   Safe-FedLLM 摘要:LoRA 更新「exhibit distinct behavioral patterns that can be effectively distinguished by lightweight classifiers」,並稱在高惡意比例下仍有效。`source-exists`(摘要)。這與「不可偵測」的假設直接衝突,兩者的威脅模型差異必須先讀全文釐清。
3. arXiv:`federated AND LoRA AND safety AND malicious` 2 筆(Safe-FedLLM 2601.07177、PEFT-as-an-Attack 2411.19335);另外三個更貼近的查詢連線失敗,沒有結果。**arXiv 檢查不完整。**
- 判斷:經驗性的攻防已被做了很多;「可偵測性門檻的理論模型」我沒看到,但檢查不完整,而且 Ghost in the Cloud 全文可能已有理論分析。**缺口無法判斷**,偏向「還有一小塊」。

### 定義這個問題的論文
- arXiv 2406.10630,Emerging Safety Attack and Defense in Federated Instruction Tuning of Large Language Models(ICLR 2025)。`quote-verified`
- arXiv 2601.00566,Low Rank Comes with Low Security: Gradient Assembly Poisoning Attacks against Distributed LoRA-based LLM Systems。`quote-verified`
- OpenAlex W7167559498,Ghost in the Cloud: Your Geo-Distributed Large Language Models Training is Easily Manipulated(ICLR 2026)。`source-exists`(截斷摘要)
- DOI 10.18653/v1/2026.acl-long.1120,Safe-FedLLM。`source-exists`(摘要)

### 需要的資源
- Ye 等人的設定:OpenFedLLM 框架、Llama2-7B(8-bit 量化)、LoRA rank 32、100 輪;良性資料 LMSYS-Chat、WildChat;惡意資料 Beavertails;安全評測 AdvBench。`quote-verified`(2406.10630 全文)。各資料集的授權與下載我沒有確認。
- 計算:7B 模型的聯邦 LoRA 需要 GPU;一張 24 GB 顯示卡夠不夠是 `model-only`。

### 碩論的最小版本 / 延展到博論的路線
- 最小版本(`model-only`):在 Ye 等人的設定上量出良性與惡意更新在安全子空間投影的分佈,擬合門檻模型,預測「ρ 與異質度」平面上的攻擊成功邊界,再用實驗檢驗預測。
- 第二篇:由門檻模型導出帶保證的伺服器端投影聚合(接王紹睿的 SelGrad:selective gradient projection)。
- 第三篇:與 DP 雜訊、後門攻擊合併的統一分析(接 `fedlora-privacy-audit` 與王紹睿的 FedMVC)。

### 最可能失敗的原因
這是 ICLR / ACL 等級研究組正在密集發表的題目,一年內很可能有人發表同樣的門檻分析;碩士生單張 GPU 的實驗規模也難以說服審稿人。`model-only`

### 台科大資工系相鄰的教授(附依據)
- 王紹睿:計畫「聯邦式大型語言模型的資安弱點與防禦方法」(NSTC 115-2221-E-011-126-MY2);投稿中論文「SelGrad: Selective Gradient Projection for Efficient Safety Alignment Against Harmful Fine-Tuning」(IEEE TDSC,In Major Revision);「FedMVC: Defense against backdoor attacks in Federated Learning…」(Journal of Supercomputing 2026)。`quote-verified`(`王紹睿.txt`)。這是三題中與單一教授現行計畫貼得最近的一題。
- 許之凡:「FedTrust: Towards Building Secure Robust and Trustworthy Moderators for Federated Learning」(IEEE MIPR 2022)。`quote-verified`(`許之凡.txt`);年份較早,相鄰程度中等。

---

## 沒有採用的方向

- **FHEW/TFHE 加密推論的「失敗機率-量化位元-準確率」聯合模型**(相鄰:許之凡,PETs 2025 論文 DOI 10.56553/popets-2025-0172,全文已開)。論文靠「DNN's inherent tolerance for approximation errors」換速度,結尾只說「We leave concrete optimization of parameters and performances (for CKKS and others) as an interesting research direction」(`quote-verified`)。我找不到一句明確陳述的未解問題;引用它的 4 篇都是工程改進(`source-exists`)。缺口無法判斷,而且偏密碼工程,不像資安題目。
- **零知識決策樹推論的電路成本**(相鄰:鄧惟中,ARTMAN '25 論文)。arXiv 查詢連線失敗,沒有任何結果;加上教授借調中。沒查成,不列。
- **O-RAN 的 low-rate DoS / xApp 異常偵測**(相鄰:鄭欣明,J43、J45)。與清單上已有的 `xapp-conflict-game`、`5g-core-metastability` 太近。
- **byte 層級的 Windows 惡意程式證書**:Gibert 等人已從 AISec 2023 做到 ACM TOPS 2026,另有 RS-Del、ByteShield、EXE-Bench(`source-exists`),已被做得很滿。

## 來源

| id | 標題 | 開到哪個程度 |
|---|---|---|
| arXiv 2302.03098 | One-shot Empirical Privacy Estimation for Federated Learning | 全文 |
| arXiv 2606.10481 | Advancing the State-of-the-Art in Empirical Privacy Auditing | 全文 |
| arXiv 2406.10630 | Emerging Safety Attack and Defense in Federated Instruction Tuning of Large Language Models | 全文 |
| arXiv 2601.00566 | Low Rank Comes with Low Security: Gradient Assembly Poisoning Attacks against Distributed LoRA-based LLM Systems | 全文 |
| arXiv 2302.01757 | RS-Del: Edit Distance Robustness Certificates for Sequence Classifiers via Randomized Deletion | 全文 |
| DOI 10.56553/popets-2025-0172 | Optimizing Encrypted Neural Networks: Model Design, Quantization and Fine-Tuning Using FHEW/TFHE | 全文(PDF) |
| arXiv 2602.19926 | Rethinking LoRA for Privacy-Preserving Federated Learning in Large Models | 摘要 |
| arXiv 2604.20495 | Towards Certified Malware Detection: Provable Guarantees Against Evasion Attacks | 摘要 |
| arXiv 2508.10038 | Certifiably robust malware detectors by design | 摘要 |
| arXiv 2604.06833 | FedDetox: Robust Federated SLM Alignment via On-Device Data Sanitization | 摘要 |
| DOI 10.18653/v1/2026.acl-long.1120(arXiv 2601.07177) | Safe-FedLLM: Delving into the Safety of Federated Large Language Models | 摘要 |
| OpenAlex W7167559498 | Ghost in the Cloud: Your Geo-Distributed Large Language Models Training is Easily Manipulated | 截斷的摘要 |
| DOI 10.1109/access.2024.3513713 | An Adversarial Attack on ML-Based IoT Malware Detection Using Binary Diversification Techniques | 摘要 |
| DOI 10.1145/3689427 | Improving Robustness in IoT Malware Detection through Execution Order Analysis | 截斷的摘要 |
| DOI 10.1145/3817114 | Harmful Fine-tuning Attacks and Defenses for Large Language Models: A Survey | 截斷的摘要 |
| DOI 10.1145/3605764.3623914 | Certified Robustness of Static Deep Learning-based Malware Detectors against Patch and Append Attacks | 截斷的摘要 |
| DOI 10.1145/3827613 | Certified Adversarial Robustness of End-to-End Malware Detectors via (De)Randomized Smoothing | 索引 |
| arXiv 2501.13676 | Certified Robustness Under Bounded Levenshtein Distance | 索引 |
| arXiv 2506.13024 | Position: Certified Robustness Does Not (Yet) Imply Model Security | 索引 |
| arXiv 2512.09883 | ByteShield: Adversarially Robust End-to-End Malware Detection through Byte Masking | 索引 |
| DOI 10.1109/tifs.2026.3734783 | EXE-Bench: Ranking the Tradeoffs of AI-Based Windows Malware Detectors for Real-World Usability | 索引 |
| arXiv 2503.07216 | FedRand: Enhancing Privacy in Federated Learning with Randomized LoRA Subparameter Updates | 索引 |
| arXiv 2605.08443 | Improving Parameter-Efficient Federated Learning with Differentially Private Refactorization | 索引 |
| arXiv 2411.19335 | PEFT-as-an-Attack! Jailbreaking Language Models during Federated Parameter-Efficient Fine-Tuning | 索引 |
| arXiv 2605.07961 | Graph Representation Learning Augmented Model Manipulation on Federated Fine-Tuning of LLMs | 索引 |
| DOI 10.1609/aaai.v40i41.40787 | ShadeEdit: A Utility-Preserving and Defense-Evasive Knowledge Manipulation Attack in Federated LLMs | 索引 |

系上資料:`topics/universe/ntust-csie-faculty.md`;`papers/_ntust/labs/` 的 鄭欣明、王紹睿、許之凡、鄧惟中、鮑興國 五個檔(林佳志沒有檔)。

沒有查成的事:Consensus 沒有使用;arXiv API 多次逾時或回 429,候選三與零知識方向的幾個查詢沒有結果;Google Scholar、DBLP、Semantic Scholar 依慣例不可用,沒有查。
