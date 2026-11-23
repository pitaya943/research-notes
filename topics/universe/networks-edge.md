# 網路、邊緣計算與量子網路:候選題目
查證日期 2026-11-23。等級:quote-verified / source-exists / model-only。

查證工具說明:arXiv API 今天多數時候回 `Rate exceeded` 或 429,所以 arXiv 的「最新 preprint」檢查改用 arXiv 網頁搜尋(arxiv.org/search,依公告日期排序)。Consensus 沒有使用。標成 source-exists 的論文我只看了索引紀錄或摘要;只看到標題的會另外寫「只看標題」。

## 這個領域現在在做什麼
- 排隊理論這兩年的新題目是「帶預測的排隊」(queueing with predictions:工作大小不是已知,而是 ML 預測出來、會有誤差),主要由 LLM 推論排程帶動;2025 年的綜述明說多伺服器系統的理論幾乎還沒做。(quote-verified,arXiv 2503.07545 全文)
- 量子網路的效能分析正在把古典網路的工具搬過來:Age of Information(AoI,資訊新鮮度)在 2025–2026 年被改寫成 Age of Entanglement(AoE,糾纏新鮮度)等指標,目前只有三節點鏈與單一 switch 的結果。(source-exists,摘要:arXiv 2602.23985、2601.11698、2603.23476;OpenAlex W7204175654)
- 糾纏緩衝(entanglement buffering:用新產生的糾纏對去純化記憶體裡正在劣化的那一對)已有解析解,但「最佳純化策略」被原作者列為未解。(quote-verified,npj Quantum Information 2026 全文)
- 5G 省電機制(DRX、BWP 切換)的 Markov/semi-Markov 模型在 2023–2026 年持續有人發表,空間已經很擠。(source-exists,OpenAlex 引用清單,只看標題)
- Serverless 冷啟動有公開的大型 trace(Huawei,85 億筆請求),但題目多為系統實作,解析模型零散。(source-exists,arXiv 2410.06145 摘要)

## 候選一覽
| 代號 | 題目(一句話問題) | 系上相鄰教授 | 缺口證據強度 | 驗證平台 | 延展到博論 | 主要風險 |
|---|---|---|---|---|---|---|
| predicted-dispatch-fog | 工作大小只有 ML 預測值時,多節點霧/邊緣叢集用「power-of-d 分派 + 最短預測工作優先」的平均回應時間是多少?預測要多準、多便宜才划算? | 沈上翔、馮輝文、賓拿雅 | 強(綜述全文明列為 open question;2025 起只找到 LLM 取向的後續) | 自寫離散事件模擬 + Huawei 公開 trace;k3s 小叢集(model-only) | 異質 CPU/GPU 節點 → 預測成本 → 邊緣 LLM 推論 | 多伺服器排隊理論本身就難;若只做模擬會被說成「又一個排程器」 |
| age-of-entanglement-chain | n 跳中繼鏈(含 cutoff 與純化)上,讓端到端 AoE 最小的生成/交換策略長什麼樣?swap-ASAP 差多少? | 賓拿雅、Pankaj Kumar | 中強(定義論文 2026-02 才出現,只有 3 篇引用;原文把長鏈留給未來) | 自寫 MDP 求解 + 量子網路模擬器(系上有 QComNetsim) | 多使用者排程(Whittle index)→ 與路由合併 → AoE 與金鑰新鮮度 | 題目太新、移動快,一年內可能被 Ulukus/Yener 等組做掉;AoE 是否只是「延遲+保真度」換名字會被質疑 |
| purify-buffer-policy | 純化式糾纏緩衝的最佳純化策略是什麼(可用率與消耗時保真度的取捨)?緩衝後面接 swap 時結論還成立嗎? | Pankaj Kumar、賓拿雅 | 中(原作者明寫 open question;但 2025–2026 有幾篇相鄰論文我只看了標題) | 原論文公開分析程式碼(GitHub);Markov chain 數值解 | 緩衝接入中繼鏈 → 多條長壽記憶體 → 與保真度感知路由結合 | 原作者(QuTech)自己最可能先解掉;物理雜訊模型的學習量大 |

我的排序:predicted-dispatch-fog > age-of-entanglement-chain > purify-buffer-policy。理由見各節與檔尾的回報。

## predicted-dispatch-fog: 帶預測誤差的多節點分派與排程
### 一句話
在 N 台(可異質)霧/邊緣節點上,每個工作到達時只有 ML 預測的服務時間,分派用 power-of-d(隨機抽 d 台挑最好的),節點內用 SPJF(shortest predicted job first,最短預測工作優先);問平均回應時間的解析式,以及預測誤差與預測成本的「划算門檻」。

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:mean-field / fluid limit(節點數趨近無限時,佇列狀態分佈滿足一組 ODE)。綜述指出現有的 fluid limit 只做到 1-bit 預測(長/短兩類),而且需要 Poisson 到達與指數服務時間。(quote-verified,2503.07545 §2.5:"a variant of the power of two choices is considered in the setting of 1-bit predictions by deriving the fluid limit differential equations")把它推廣到 K 類預測或連續預測,是一個邊界清楚的數學問題。(model-only:這是我對可行切法的判斷)
- 驗證:有限 N 的離散事件模擬對 ODE 預測;再用真實到達過程重放。Huawei 公開了一個月、85 億筆請求的 serverless trace。(source-exists,2410.06145 摘要:"releases and analyzes a month-long trace of 85 billion user requests")
- 相鄰的既有結果:「測試工作大小再分派」不一定划算,取決於負載、系統大小與工作大小變異係數。(source-exists,2304.00899 摘要)這正好是本題「預測成本」那一半的起點。

### 缺口還在嗎(三項檢查各自的結果)
1. list_citations(2503.07545 的期刊版 W4412572711,45 篇引用,看最新 15 篇,只看標題):幾乎都是 LLM 服務(記憶體受限、KV-cache、autoscaling 的排隊模型,例如 arXiv 2609.20957、2606.15555)與單佇列排程(Mathematics 2026 "Prediction-Driven Non-Preemptive Scheduling with Bounded Overtaking");沒有看到標題是「多位元預測下的 power-of-d」的。(source-exists)
2. search_works(2025 起,關鍵字:predicted service times / job size estimates × load balancing / dispatching / power-of-d / multiserver):只有 1 筆(UNAS,LLM 服務排程,JCC 2026),不是這個問題。(source-exists)
3. arXiv 網頁搜尋("predicted service times" OR "job size estimates" OR "predicted job sizes"):10 筆,最新是 2610.32503 "Pareto-optimal Non-Clairvoyant Scheduling with Predictions"(只看標題)、2504.10743 "Robust Gittins for Stochastic Scheduling"(只看標題);分派方向的是 2019 年的 1905.12155 "The Supermarket Model with Known and Predicted Service Times"(只看標題)與 2304.00899。(source-exists)
結論:缺口看起來還在,但第 1 項只看了 45 篇中的 15 篇標題。

### 定義這個問題的論文
- Mitzenmacher, Shahout, "Queueing, Predictions, and LLMs: Challenges and Open Problems", arXiv 2503.07545;期刊版 Stochastic Systems 15(3), DOI 10.1287/stsy.2025.0106。原文:"Using predictions in larger systems or networks of queues remains relatively open for further study." 以及 open question:"can we develop fluid limit models to analyze the power of two choices when using (more than 1-bit) predictions?"(quote-verified)
- "Load Balancing with Job-Size Testing: Performance Improvement or Degradation?", arXiv 2304.00899(source-exists,摘要)
- "SkipPredict: When to Invest in Predictions for Scheduling", arXiv 2402.03564 / DOI 10.52202/079017-3352(source-exists,只看索引)
- Pramono, Shen, "Adaptive fog resource management through ML-based prediction SJF scheduling", Cluster Computing 2026, DOI 10.1007/s10586-026-06372-7(source-exists,摘要;這是系上沈上翔老師的論文,做的是系統框架 ARAMS,沒有從摘要看到解析模型)

### 需要的資源
- Huawei trace:論文說已釋出(source-exists);下載位置我沒有實際開過。
- 模擬器自己寫(Python/C++)即可;小型 k3s 叢集做實機驗證是加分項。(model-only)
- 不需要特殊硬體。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:同質節點、K 類預測、給定混淆矩陣(預測類別 vs 真實類別)下的 power-of-d + SPJF fluid limit,加上預測延遲;給出「預測划不划算」的相圖,並用模擬與 trace 驗證。
- 第 2 篇:異質節點(CPU/GPU,對接 ARAMS 的情境)與不可搶占服務。
- 第 3 篇:預測器線上學習(預測誤差隨負載改變)或邊緣 LLM 推論(工作有記憶體佔用);這一步會進入目前很擠的 LLM 排程文獻。

### 最可能失敗的原因
多類 SPJF 的佇列狀態描述可能大到 ODE 沒有封閉形式,只剩數值解;那樣理論貢獻會變薄。綜述自己也說:"multiple server systems resist analysis even without predictions"。(quote-verified)

### 台科大資工系相鄰的教授(附依據)
- 沈上翔:實驗室頁列有 2026 年論文 "Adaptive fog resource management through ML-based prediction SJF scheduling"(Cluster Comput 29, 586 (2026))。(quote-verified,labs/沈上翔.txt;OpenAlex 也確認作者單位為 NTUST)
- 馮輝文:系網專長列「模擬技術、排隊理論、效能分析」(quote-verified,ntust-csie-faculty.md);個人網頁著作有 Queueing Systems 期刊的 BMAP/G/1 論文,但該網頁「最近更新日期:2006-02-19」,近年在做什麼從網頁看不出來。(quote-verified,labs/馮輝文.txt)
- 賓拿雅:"His primary research interests include network optimization and queuing analysis",並開授 "Queuing Theory (CS5148701)"。(quote-verified,labs/賓拿雅.txt)

## age-of-entanglement-chain: 長中繼鏈上的 Age of Entanglement
### 一句話
把 AoE(端到端糾纏對「多久沒更新」再加上退相干造成的品質下降)從三節點鏈推到 n 跳鏈,問最佳的生成/儲存/交換策略有沒有門檻結構,以及固定策略(例如 swap-ASAP 加 cutoff)的平均 AoE 能不能寫成封閉式。

### 為什麼不會太薄
- 模型:平均成本 MDP(Markov decision process)。定義論文就是這樣建模,並用 relative value iteration 求解。(source-exists,2602.23985 摘要;全文 §III 確認是 "three-node linear quantum repeater chain",quote-verified)
- 解析的一半有先例:quantum switch 上的同類指標 AoEE 已經對兩族低複雜度策略得到封閉式。(source-exists,2601.11698 摘要)鏈狀拓撲還沒有對應結果。(model-only:我沒有逐篇排除)
- 驗證:MDP 數值解對模擬。系上已有自製模擬器 QComNetsim(見下);NetSquid、SeQUeNCe 等公開模擬器我這次沒有查證。(model-only)

### 缺口還在嗎
1. list_citations(2602.23985,INFOCOM 2026,W7166566586):共 3 篇。arXiv 2611.17865 "Starlight: Whittle-Driven Fidelity Rescue for Satellite-Terrestrial Quantum Networks"(只看標題)、ISIT 2026 "Age of Quantum Information: Fidelity-Aware Teleportation Scheduling"(摘要:時槽模型、傳送排程)、arXiv 2603.23476(switch、Whittle index)。沒有一篇的標題或摘要是 n 跳鏈。(source-exists)
2. search_works(2022 起,"age of information" × entanglement/quantum network/QKD):10 筆中相關的 4 筆,另兩篇是 MeditCom 2025 "Age of Information for Quantum Communication Channels with Monogamy of Entanglement" 與 APNOMS 2025 "AoI Minimization with Quantum Key OTP for Secure IoT Data Collection"。(source-exists,摘要)
3. arXiv("age of entanglement" OR "age of quantum information";另一個查詢加 repeater):2602.23985、2601.11698、2603.23476,沒有更新的鏈狀結果。(source-exists)
結論:缺口還在,但 2611.17865 是本月的 preprint,我沒有開內文,它可能已經處理部分網路層問題。

### 定義這個問題的論文
- Ceran, "Age of Entanglement in Satellite Repeater Chains with Intermittent Availability", arXiv 2602.23985,DOI 10.1109/infocom59046.2026.11571216。原文(結論):"This work also provides a baseline for future studies on AoE optimization in longer repeater chains and possibly the development of learning-based control algorithms to enable more scalable solutions in large-scale quantum networks."(quote-verified)
- Mitrolaris, Banerjee, Ulukus, "Age-Based Scheduling for a Memory-Constrained Quantum Switch", arXiv 2601.11698(source-exists,摘要)
- Banerjee, Mitrolaris, Ulukus, "Index-Based Scheduling for a Resource-Constrained Quantum Switch", arXiv 2603.23476(source-exists,摘要)
- Welling, Yener, "Age of Quantum Information: Fidelity-Aware Teleportation Scheduling", ISIT 2026, DOI 10.1109/isit62367.2026.11653667(source-exists,摘要)

### 需要的資源
只需要運算與模擬器;不需要量子硬體。定義論文沒有提到公開程式碼(全文搜尋 github 無結果,quote-verified),要自己重做基線。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:同質 n 跳鏈、固定策略族(swap-ASAP + cutoff)的平均 AoE 封閉式或遞迴式;小 n 用 MDP 求最佳策略,量出固定策略的差距;模擬驗證。
- 第 2 篇:多對使用者共用鏈路時的 AoE 排程(restless bandit / Whittle index),接到 Pankaj Kumar 的分區路由。
- 第 3 篇:AoE 與應用層需求(QKD 金鑰新鮮度)的聯合最佳化。注意這一步會靠近清單上已有的 `qkd-keypool-backpressure`,要刻意切開。

### 最可能失敗的原因
被搶先。這條線 2026 年 1–11 月已有至少 5 篇(上列),兩個成熟的 AoI 團隊都在做;碩士一年的節奏可能跟不上。

### 台科大資工系相鄰的教授(附依據)
- 賓拿雅:系上唯一同時做 AoI 與量子網路的人。實驗室頁:"Age-of-Information Aware Mobility-Based Vehicular-Fog Formation Using Deep Reinforcement Learning ... IEEE Transactions on Intelligent Transportation Systems, February 2026";"Fidelity-Aware Inter-Zone Based Routing for Throughput Maximization in Quantum Networks ... IEEE Journal on Selected Areas in Communications, October 2026"。(quote-verified,labs/賓拿雅.txt)
- Pankaj Kumar:Research Expertise 列 "Routing, Entanglement Distribution, Quantum Repeaters, Resource Allocation and Optimization";博論 "Zone-Based Entanglement Routing in Quantum Networks"。(quote-verified,labs/PankajKumar.txt)

## purify-buffer-policy: 純化式糾纏緩衝的最佳策略
### 一句話
兩節點各有一顆長壽記憶體存糾纏、n 顆短壽記憶體產生新糾纏(1GnB 系統);每個時槽要決定拿幾對新糾纏、用哪種協定去純化存著的那一對。問:讓「可用率」與「被消耗時的平均保真度」最佳的策略是什麼,緩衝後面接 entanglement swapping 時是否還成立。

### 為什麼不會太薄
- 模型:離散時間 Markov chain,原論文已導出可用率與平均消耗保真度的解析式。(source-exists,2502.20240 摘要;期刊版全文確認,quote-verified:"We derive analytical expressions for the entanglement availability and the average fidelity upon consumption.")
- 原論文的結論「純化要盡量頻繁」只在它考慮的策略族內成立,而且可用率下界是否緊也沒證。(quote-verified:"We leave this proof as future work.")兩者都是可以寫成定理的目標。
- 驗證:論文說 "all results in this manuscript are analytical",分析程式碼放在 GitHub。(quote-verified;我的輸出把網址截斷了,沒有實際開 repo)

### 缺口還在嗎
1. list_citations(前作 "Entanglement buffering with two quantum memories",Quantum 2024,W4402192716,23 篇,全部看過標題):最新的是 arXiv 2610.26900(多請求的純化排程,MILP 與 RL,網路層;摘要)、2609.21152 "ReFINE: Scheduling of Distillation and Coding for Rate-Fidelity Tradeoff"(只看標題)、QCNC 2026 "Dimensioning of Quantum Memories for Distilled Quantum EPR Packets"(arXiv 2604.13964,Markov chain 配置記憶體;摘要)、2505.02286 "Entanglement Purification in Quantum Networks: Guaranteed Improvement and Optimal Time"(只看標題)。(source-exists)
2. search_works(2023 起,entanglement buffering purification quantum memories analytical model):只回 1 筆,就是 2024 年的前作;這個查詢太嚴,證據力弱。(source-exists)
3. arXiv("entanglement buffer" OR "entanglement buffering"):3 筆(2502.20240、2312.05099、2311.10052),沒有 2025-02 之後的新作;另一個查詢(purification × buffer × policy/scheduling)只回 2502.20240。(source-exists)
結論:「1GnB 最佳策略」本身沒有看到後續;但 2505.02286 與 2609.21152 我只看了標題,它們可能已回答部分問題。這一項記為「缺口部分無法判斷」。

### 定義這個問題的論文
- Iñesta, Davies, Kar, Wehner, "Entanglement buffering with multiple quantum memories", npj Quantum Information 12, 64 (2026),DOI 10.1038/s41534-025-01161-3,arXiv 2502.20240。原文:"Notably, determining the optimal ordering in which simple protocols should be applied to newly generated links (e.g., concatenated, nested, or banded) is left as future work. Additionally, finding policies that optimize availability or average consumed fidelity remains an important open question."(quote-verified;原文在 nested、banded 後各有一個參考文獻編號,此處省略)
- Davies, Iñesta, Wehner, "Entanglement buffering with two quantum memories", Quantum 2024,DOI 10.22331/q-2024-09-03-1458,arXiv 2311.10052(source-exists,摘要)

### 需要的資源
只要運算。需要先學 Werner state、DEJMPS 等純化協定的輸出保真度公式(學習成本這次不計)。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:把 1GnB 寫成 MDP,數值求最佳策略,與原論文的啟發式比較;嘗試證明下界緊或給反例;用模擬驗證。
- 第 2 篇:緩衝接在兩段鏈中間(buffer + swap),看「盡量頻繁純化」是否仍最佳。
- 第 3 篇:多顆長壽記憶體(mGnB)與保真度感知路由結合,接到 Pankaj Kumar 的路由工作。

### 最可能失敗的原因
原作者明確把它列為下一步,而他們已連續發表兩篇;一年內被原團隊做完的機率不低。

### 台科大資工系相鄰的教授(附依據)
- Pankaj Kumar:專長列 "Quantum Repeaters, Resource Allocation and Optimization";著作有 "Trace-distance based end-to-end entanglement fidelity with information preservation in quantum networks"(JNCA 2025)與 "Fidelity-aware inter-zone-based routing for throughput maximization in quantum networks"(JSAC 2026)。(quote-verified,labs/PankajKumar.txt)他做的是路由層,不是鏈路層的純化策略,所以是「相鄰」而不是「同題」。
- 賓拿雅:Quantum Research Lab 主持人;與 Pankaj Kumar 等人有 "QComNetsim: A validated quantum network simulator with cross-platform benchmarking"(IEEE GLOBECOM)。(quote-verified,labs/PankajKumar.txt、labs/賓拿雅.txt)

## 沒有採用的方向
- **5G NR 的 BWP 切換 + DRX 省電的排隊模型**(緊鄰周詩梵 "Joint BWP Switching and Data-Control Resource Allocation in New Radio Networks",labs/周詩梵.txt,quote-verified):已有隨機模型(Telecommunication Systems 2023,DOI 10.1007/s11235-023-01007-3)與考慮切換延遲的聯合設計(IEEE TMC 2025,DOI 10.1109/tmc.2025.3547978),2025–2026 還有一串 semi-Markov DRX 論文(source-exists,只看標題)。剩下的空間太窄;XR 流量版本我沒查到東西,也沒能確認是空的。
- **Serverless/邊緣冷啟動的 keep-alive 排隊模型**(鄰近金台齡的邊緣計算、沈上翔的 "Enabling Service Cache in Edge Clouds"):基礎模型 2020 年就有(DOI 10.1109/tcc.2020.3033373,120 篇引用),2026-10 有統一排隊框架(arXiv 2610.35564,摘要),keep-alive 當作快取的觀點也有人做(arXiv 2603.03091 摘要、2408.00957 只看標題)。在預算內找不出一個乾淨的缺口。
- **容器映像層(image layer)共享的邊緣快取解析模型**(緊鄰沈上翔 "Named Image-Layer Networks for Containers",TNSM 2026):OpenAlex 關鍵字查詢回 0 筆、語意查詢被限流、arXiv 查詢只回一篇不相關的。缺口無法判斷,沒有繼續。
- **LLM 推論的排隊與預測排程**:2025–2026 已非常擁擠(OpenAlex 語意搜尋前 12 筆全是這個題目),而且和學習者清單上的 KV cache 題目重疊。
- **地震預警感測網路的 AoI**(鄰近金台齡):arXiv 查詢時被限流,沒有查成,不能下結論。

## 來源
| id | 標題 | 開到哪個程度 |
|---|---|---|
| arXiv 2503.07545 / DOI 10.1287/stsy.2025.0106 | Queueing, Predictions, and LLMs: Challenges and Open Problems | 全文(arXiv 版);期刊頁被擋 |
| arXiv 2304.00899 | Load Balancing with Job-Size Testing: Performance Improvement or Degradation? | 摘要 |
| arXiv 2402.03564 / DOI 10.52202/079017-3352 | SkipPredict: When to Invest in Predictions for Scheduling | 索引 |
| arXiv 1905.12155 | The Supermarket Model with Known and Predicted Service Times | 只看標題 |
| arXiv 2610.32503 | Pareto-optimal Non-Clairvoyant Scheduling with Predictions | 只看標題 |
| DOI 10.1007/s10586-026-06372-7 | Adaptive fog resource management through ML-based prediction SJF scheduling | 摘要(截斷) |
| arXiv 2410.06145 / DOI 10.1145/3689031.3696073 | Serverless Cold Starts and Where to Find Them | 摘要 |
| arXiv 2602.23985 / DOI 10.1109/infocom59046.2026.11571216 | Age of Entanglement in Satellite Repeater Chains with Intermittent Availability | 全文 |
| arXiv 2601.11698 | Age-Based Scheduling for a Memory-Constrained Quantum Switch | 摘要 |
| arXiv 2603.23476 | Index-Based Scheduling for a Resource-Constrained Quantum Switch | 摘要 |
| DOI 10.1109/isit62367.2026.11653667 | Age of Quantum Information: Fidelity-Aware Teleportation Scheduling | 摘要(截斷) |
| arXiv 2611.17865 | Starlight: Whittle-Driven Fidelity Rescue for Satellite-Terrestrial Quantum Networks | 只看標題 |
| DOI 10.1109/meditcom64437.2025.11104412 | Age of Information for Quantum Communication Channels with Monogamy of Entanglement | 摘要(截斷) |
| DOI 10.23919/apnoms67058.2025.11181281 | AoI Minimization with Quantum Key OTP for Secure IoT Data Collection | 摘要(截斷) |
| DOI 10.1038/s41534-025-01161-3 / arXiv 2502.20240 | Entanglement buffering with multiple quantum memories | 全文(nature.com 開放取用頁) |
| DOI 10.22331/q-2024-09-03-1458 / arXiv 2311.10052 | Entanglement buffering with two quantum memories | 摘要 |
| arXiv 2610.26900 | Concurrent Entanglement Purification Scheduling in Dynamic Quantum Networks | 摘要 |
| arXiv 2604.13964 | Dimensioning of Quantum Memories for Distilled Quantum EPR Packets | 摘要 |
| arXiv 2609.21152 | ReFINE: Scheduling of Distillation and Coding for Rate-Fidelity Tradeoff in Quantum Networks | 只看標題 |
| arXiv 2505.02286 | Entanglement Purification in Quantum Networks: Guaranteed Improvement and Optimal Time | 只看標題 |
| DOI 10.1007/s11235-023-01007-3 | Stochastic modeling for bandwidth part switching based DRX mechanism in 5G NR networks | 索引 |
| DOI 10.1109/tmc.2025.3547978 | Device Power Saving With Time-Frequency Adaptation: Joint BWP-DRX Design With BWP Switching Delay Considered | 索引 |
| DOI 10.1109/tcc.2020.3033373 | Performance Modeling of Serverless Computing Platforms | 索引 |
| arXiv 2610.35564 | A Unified Queueing-Based Framework for Comparative Evaluation of Serverless Autoscaling Strategies under Bursty Workloads | 摘要 |
| arXiv 2603.03091 | Custom Keep-Alive Cache Policies | 摘要 |
| arXiv 2408.00957 | Caching Aided Multi-Tenant Serverless Computing | 只看標題 |
| 本地檔 | topics/universe/ntust-csie-faculty.md;papers/_ntust/labs/ 的 馮輝文、金台齡、沈上翔、周詩梵、賓拿雅、PankajKumar | 全部讀過(長檔讀了前段與關鍵字比對到的行) |
