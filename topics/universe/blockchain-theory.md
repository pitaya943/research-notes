# 區塊鏈網路的效能/誘因建模,以及演算法與計算幾何:候選題目
查證日期 2026-11-23。等級:quote-verified / source-exists / model-only。

「source-exists」在本檔一律表示:只看過 OpenAlex 或 arXiv 的索引紀錄或摘要,沒有讀全文。
判準 1(學習成本)依指示不計。

## 這個領域現在在做什麼

- 手續費機制(transaction fee mechanism, TFM)的純機制設計理論仍在出論文,但系上沒有人做這一塊;`ntust-csie-faculty.md` 自己就寫「交易手續費機制本身不在頁面專長內」[quote-verified:faculty 檔第 70 行]。
- 2025–2026 年有一批論文把 mempool(待確認交易池)當成排隊系統來分析:Bitcoin 上有 M/G^K/1 priority queue(2606.01274)、M/M^b/1/n(ICBC 2026)、自架節點資料的結構估計(2604.17183);Ethereum 的 EIP-1559 上有 priority queue 模型(Inami & Phung-Duc 2025)與 MDP 模型(2605.12794)[source-exists,各篇摘要]。
- Ethereum 共識層的「延遲」正被當成經濟變數:timing games(出塊者故意晚出塊以多拿 MEV)自 2305.09032 之後有 46 篇引用,2026 年已有 Markov chain 模型(TIFS)、實證與緩解(TDSC)、ePBS 下的拍賣理論(2607.11240)、地理位置賽局(2610.18812)[source-exists:OpenAlex 引用清單與摘要]。
- 網路層本身:GossipSub 的 mesh 大小已有理論模型加主網量測(INFOCOM 2026 Eth2.0-NA),data availability sampling(DAS,資料可用性抽樣)的傳播以模擬與叢集實驗為主(PANDAS, Middleware 2025)[source-exists,摘要]。
- 計算幾何的 terrain guarding(地形看守)理論面相當成熟(1.5D 有 PTAS 與精確解法),2023 年後的新作集中在 imprecise(不精確輸入)與 watchtower 變形,數量很少:OpenAlex 關鍵字搜尋 2023 年以後只有 4 篇 [source-exists]。

## 候選一覽

| 代號 | 題目(一句話問題) | 系上相鄰教授 | 缺口證據強度 | 驗證平台 | 延展到博論 | 主要風險 |
|---|---|---|---|---|---|---|
| `eip1559-wait-queue` | EIP-1559 下交易的等待時間分佈,能否由「容量可伸縮、價格回授、依 tip 排序」的排隊模型預測,並通過公開 mempool 資料的檢驗? | 周詩梵、馮輝文 | 中偏強:現有模型只對模擬或合成到達驗證;實證論文沒有模型 | Mempool Dumpster、Xatu(都已確認可下載) | 需求彈性閉迴路 → 多資源(blob)→ 次秒級出塊的 L2 | 主網大多數時間不壅塞,等待時間幾乎都是一個區塊,模型沒東西可預測 |
| `timing-delay-propagation` | 給定實測的區塊傳播延遲分佈,出塊者的最適延遲與 missed-slot 機率是多少?slot 結構改變時如何移動? | 周詩梵(最貼近)、馮輝文 | 中偏弱:很擁擠,兩篇最接近的 2026 論文讀不到全文 | Xatu(beacon 事件與 libp2p 表)、relay data API(都已確認) | 區塊大小/blob 數對延遲的影響 → ePBS → DAS 傳播的截止機率 | 已被 TIFS 2026 / TDSC 2026 做掉大半,碩論只剩「再量一次」 |
| `terrain-guard-uncertain` | 地形高度只知道在 ±ε 內時,保證看得到整條 1.5D terrain 的最少 guard 數怎麼算?在真實高程剖面上要多付幾成? | 項天瑞 | 缺口無法判斷:兩篇關鍵論文只有索引紀錄 | SRTM 高程檔(已確認可下載)、TGPIL 測資(只見於論文) | 2.5D 地形 → 有限視距 → 充電器/基地台佈建 | 理論部分可能已被 2023/2025 兩篇涵蓋;且項老師 2019 年後沒有幾何論文 |

我的排序:`eip1559-wait-queue` > `timing-delay-propagation` > `terrain-guard-uncertain`。

---

## eip1559-wait-queue: EIP-1559 等待時間的排隊模型與實測檢驗

### 一句話
EIP-1559 把每個區塊的容量做成可伸縮(有 target 與上限),並用 base fee 逐區塊回授調價;一筆出價為 (max fee, tip) 的交易要等幾個區塊才被納入,這個分佈能不能用一個排隊模型算出來,而且跟公開 mempool 資料對得上?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:bulk-service queue(一次服務一批的排隊)加上三個 EIP-1559 特有的機制:服務批量隨壅塞伸縮、准入門檻(base fee)由前一批的使用率決定、批內依 tip 排優先序。輸出是「等待區塊數」的分佈與穩定條件。這是排隊理論的標準工具,但價格回授讓到達與服務互相耦合,不是套公式就有答案 [model-only:這是我對題目的建模描述]。
- 驗證:每筆公開交易的「第一次被看到的時間」與「被納入的區塊」都有公開資料(見下方資源),可以直接算實測等待時間再跟模型比。
- 已知的現象可當檢驗目標:Liu 等人量到 London 升級後等待時間顯著下降,但只給了可能原因,沒有模型 [quote-verified:2201.05574]:"We find that the waiting time significantly declines after the London hard fork, possibly as a result of easier gas price bidding and variable-sized blocks."

### 缺口還在嗎(三項檢查各自的結果)
1. **引用清單**。2201.05574 有 156 篇引用;最新 20 篇(看標題)裡沒有「用排隊模型擬合 Ethereum 實測等待時間」的論文,最接近的是 GDAQ(一個操作層面的交易分級框架)與 2610.23305(多鏈下 adaptive fee 的不穩定性)[source-exists]。Inami & Phung-Duc 2025 有 10 篇引用,包含同一組的 "Performance Analysis of Blockchain with Rollups"(APCC 2025)、一篇 vacation queue 模型、以及 2605.12794;沒有看到加上實測資料的後續 [source-exists,標題]。
2. **OpenAlex 2025 年後語意搜尋**(查詢:queueing model / waiting time / EIP-1559 / mempool)。回傳的相關論文:
   - Inami & Phung-Duc, 10.3390/math13061010:priority queue 模型,摘要說理論值「match well with those obtained from Monte Carlo simulations」,也就是只對模擬驗證 [source-exists,摘要;MDPI 全文回 403,沒讀到]。
   - 2605.12794(Fardno & Etesami):MDP 模型。全文明說不處理 tip [quote-verified]:"In this paper, we do not consider tips in our analysis and instead assume that, whenever the capacity constraint $c\cdot B$ is violated, an adversary selects which transactions are included in the block"。隨機到達的實驗用的是合成的 Poisson 到達 [quote-verified:"Let $A[i,j]\sim\mathrm{Poisson}(\mu_{i})$ denote the number of arriving transactions"],沒有主網資料。
   - 2606.01274、ICBC 2026(Tan 等)、2604.17183:都是 Bitcoin(first-price、固定容量),不含價格回授 [source-exists,摘要]。
   - 2502.20031(Babaioff & Nisan):patient bidders 下 EIP-1559 的福利分析,是機制理論不是延遲模型 [source-exists,摘要]。
   - 10.3390/math14071232:把 EIP-1559 搬到 600 ms 出塊的鏈,agent-based 模擬 [source-exists,摘要]。
3. **arXiv 最新預印本**(`abs:"EIP-1559" AND (queue|waiting|delay|latency)`,依投稿日排序)。只回 3 篇:2610.23305、2506.07988、2201.05574。arXiv API 今天被限流,另外兩個查詢回 "Rate exceeded",所以這一步不完整 [source-exists]。

結論:沒有找到「模型含 tip 優先序與價格回授,且用主網等待時間驗證」的論文。缺口目前成立,但這一帶很活躍(筑波大學 Phung-Duc 組連續在出),一年內被補上的機率不低。

### 定義這個問題的論文
- Liu, Lu, Nayak, Zhang, Zhang, Zhao, "Empirical Analysis of EIP-1559: Transaction Fees, Waiting Time, and Consensus Security", arXiv:2201.05574(全文)。量測方法與限制 [quote-verified]:"While the waiting time (delay) is widely used in theoretical models of users' utility function, few have found an effective way to directly measure and analyze it in the blockchain setting."
- Inami & Phung-Duc, "Analysis of Dynamic Transaction Fee Blockchain Using Queueing Theory", Mathematics 13(6):1010, DOI 10.3390/math13061010(只讀摘要)。
- Fardno & Etesami, "Dynamic Transaction Scheduling and Pricing in the Ethereum Mempool", arXiv:2605.12794(全文)。
- Roughgarden, arXiv:2106.01340(你已有的閱讀指南;提供 base fee 與 tip 的正式定義)。

### 需要的資源
- **Flashbots Mempool Dumpster**:首頁回 HTTP 200,列出 2023-08 到 2026-11 每個月的目錄,並寫明 "The data is dedicated to the public domain under the CC-0 license";2026-10 的月目錄也回 200 [quote-verified:本 session 以 curl 開啟 mempool-dumpster.flashbots.net]。我沒有下載任何一天的檔案,所以欄位內容是 [model-only]。
- **ethPandaOps Xatu**:網站自述 "Open data on the Ethereum network: beacon chain events, mempool activity, canonical chain data, MEV, and p2p networking" [quote-verified:curl ethpandaops.io/data/xatu/]。`mempool_transaction/2026/11/1.parquet` 以 range 請求回 206,檔案存在 [quote-verified:HTTP 狀態];沒有解析內容。
- 硬體:筆電加一顆外接硬碟即可 [model-only]。不需要自架節點。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:(a) 固定到達率下的模型與等待分佈;(b) 用一段壅塞期間與一段平常期間的資料各驗證一次;(c) 指出模型在需求突發時哪裡失準。
- 第 2 篇:到達率隨 base fee 改變(需求彈性;地圖裡的 2606.13555 提供估計值),分析閉迴路的穩定與振盪。
- 第 3 篇:兩種資源各有自己的價格(gas 與 blob gas)的雙資源排隊,或次秒級出塊的 L2。

### 最可能失敗的原因
主網大多數時間不壅塞,公開交易幾乎都在下一個區塊被納入,等待時間分佈退化成一個點,模型沒有東西可以預測;而且走私有通道的交易不在 mempool 資料裡 [model-only:我沒有在本 session 量過這兩件事。動手前先花一天用 Mempool Dumpster 算等待時間的分佈,若 95% 以上都是一個區塊就該丟棄]。

### 台科大資工系相鄰的教授
- **周詩梵**:lab 檔的 Research Interests 列 "Blockchain Networks" 與 "Performance Modeling and Analysis";每年春季開 "Introduction to Blockchain and Its Applications" [quote-verified:papers/_ntust/labs/周詩梵.txt]。但注意:他已發表的區塊鏈論文是應用框架(AIoT 資料交易、UAV 配送溯源、IPFS 儲存成本),OpenAlex 列出他 22 篇,我看了最新的 12 篇標題,其中沒有排隊模型 [source-exists]。
- **馮輝文**:faculty 檔專長含「排隊理論、效能分析」[quote-verified];lab 檔著作有 "Departure processes of BMAP/G/1 queues," Queueing Systems, 2001 [quote-verified]。但該頁面寫「最近更新日期:2006-02-19」,近年是否還做排隊分析,本檔查不到;沒有任何區塊鏈著作的跡象。
- 判準 6:符合(兩位各蓋一半:一位懂區塊鏈網路,一位懂排隊)。沒有一位同時做兩者。

---

## timing-delay-propagation: 傳播延遲分佈決定的出塊延遲均衡

### 一句話
Ethereum 的出塊者晚一點出塊可以多拿 MEV,但太晚會來不及在 4 秒的 attestation 截止前傳到投票者;給定實測的傳播延遲分佈,最適延遲是多少、missed slot 機率是多少,而且這個預測跟各大 staking 業者實際的出塊時間對得上嗎?

### 為什麼不會太薄
- 模型:出塊者選延遲 δ,最大化「區塊價值(隨 δ 增加)×被多數投票者及時收到的機率(由傳播延遲的 CDF 決定)」。兩個輸入都量得到。原始論文已觀察到兩者的形狀 [quote-verified:2305.09032]:"We observe that the share value stays close to one as long as the block is signed within the first two seconds of the slot. Once the two second threshold is crossed, there is a substantial drop-off"。
- 驗證:Xatu 有 beacon 節點看到區塊的時間與 libp2p gossip 的紀錄;relay 的 data API 有每個 slot 的得標紀錄。可以逐業者比較「模型預測的 δ*」與「實際出塊時間」。
- 這題把手續費/MEV 的誘因直接接到網路延遲模型上,正是「區塊鏈網路 + 效能建模」的交集。

### 缺口還在嗎
1. **引用清單**(2305.09032,共 46 篇,看最新 20 篇標題)。直接相關:2602.11147(2-Prop,AAMAS 2026)、2607.11240(ePBS 的區塊建造拍賣)、2610.18812(出塊者地理位置賽局)、2609.20322(延遲下限與去中心化)、10.1007/s11403-026-00505-2(reorg 的 agent-based 模型)、10.1109/tnsm.2025.3573246(Mišić 組的 fork 機率分析)[source-exists]。
2. **OpenAlex 2025 年後語意搜尋**。回傳三篇很接近的:
   - Qi 等,"Modeling the Performance-Security Trade-Off of Gasper's Block Proposal Mechanism Under Latency-Driven Attacks",IEEE TIFS 2026,DOI 10.1109/tifs.2026.3651978:Markov chain 模型算 fork 機率與吞吐量,摘要說以 "extensive simulations" 驗證 [source-exists,摘要;閉鎖,沒讀全文]。
   - Ji 等,"Strategic Timing Games in Ethereum: An Empirical Study and Mitigation",IEEE TDSC 2026,DOI 10.1109/tdsc.2026.3737958 [source-exists,摘要被截斷;閉鎖]。
   - Issa,"Timing Games on Ethereum in 2026: The Attester Externality of Block-Proposal Delay…",Zenodo 預印本 10.5281/zenodo.23473829:用 Xatu 28 天資料量延遲的外部性 [source-exists,摘要;未經同儕審查]。
   - 另有 INFOCOM 2026 的 Eth2.0-NA(10.1109/infocom59046.2026.11571391):傳播 hop 數與 mesh 大小的理論模型,並用多個探測節點驗證 [source-exists,摘要]。這表示「傳播模型」那一半已有人做。
3. **arXiv**:`abs:"timing games"` 回 12 篇,全是不相干的 timed games;較早的 `all:"timing games" AND (ethereum OR proof-of-stake)` 只回 2305.09032。arXiv 被限流,查詢不完整 [source-exists]。

結論:**缺口部分成立,但我無法確認**。看得到的空隙是:2602.11147 的延遲分佈是假設的,不是量的 [quote-verified]:"As empirically observed in practice, typically, packet delays follow a Gamma distribution [41]. Hence, we assume $f_{P}$s follow the same.";TIFS 那篇依摘要只對模擬驗證。但 TDSC 2026 的實證做了什麼、有沒有擬合均衡,我讀不到。動手前必須先透過學校圖書館讀完 TIFS 與 TDSC 兩篇。

### 定義這個問題的論文
- Schwarz-Schilling 等,"Time is Money: Strategic Timing Games in Proof-of-Stake Protocols",arXiv:2305.09032(全文,已在 papers/)。未解之處 [quote-verified]:"Tackling the root cause of timing games remains an open challenge."
- Rasheed M 等,"Let Leaders Play Games: Improving Timing in Leader-based Consensus",arXiv:2602.11147(全文)。
- Qi 等,TIFS 2026,DOI 10.1109/tifs.2026.3651978(摘要)。
- Médard 等,"Pricing Innovation Under Latency Constraints: A Mean-Field Analysis of Coded Payload Delivery",arXiv:2603.20426(全文):把「解碼完成時間的分佈」當成定價的核心,但用的是簡化到達模型 [quote-verified]:"analyzes the arrival-time statistics of different sharding and coding schemes under a simplified Poisson shard-arrival model"。

### 需要的資源
- **Xatu**:`beacon_api_eth_v1_events_block/2026/11/1.parquet` 回 HTTP 200、約 9.0 MB,開頭是 `PAR1`(parquet 檔);`libp2p_gossipsub_beacon_block/2026/11/1.parquet` 回 200、約 3.7 MB [quote-verified:本 session 的 curl 結果]。沒有解析欄位。
- **Relay data API**:`boost-relay.flashbots.net/relay/v1/data/bidtraces/proposer_payload_delivered?limit=1` 回一筆 JSON,含 slot 15714254、builder_pubkey、proposer_pubkey 等欄位 [quote-verified]。
- 2305.09032 用過的 mevboost.pics 公開資料與 blockprint [quote-verified:論文第 274 行提到];我沒有確認它們今天還在。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:從 Xatu 擬合傳播延遲 CDF(依區塊大小分層)→ 解單一出塊者的最適延遲 → 對照各業者實際出塊時間 → 一個反事實(slot 縮短時 δ* 怎麼變)。
- 第 2 篇:ePBS 上線後 slot 內多了 builder 揭露 payload 的階段,重做模型(2607.11240 只做了拍賣面)。
- 第 3 篇:DAS 的欄位傳播(PANDAS 的 4 秒截止)與 blob 數上限的關係,把「blob 多 → 傳播慢 → 截止機率降」寫成解析式。

### 最可能失敗的原因
太擁擠。TIFS 2026 已有 Markov 模型、TDSC 2026 已有實證、INFOCOM 2026 已有傳播模型;碩論很可能變成「把三篇拼起來再量一次」,新意不足。

### 台科大資工系相鄰的教授
- **周詩梵**:lab 檔列有 "Cai-Sheng Feng, Shih-Fan Chou, Kuang-Hui Huang, Edmund M. Yeh and Ai-Chun Pang, 'Latency Matters: How Named Data Networking Impacts Data Dissemination for Blockchain,' submitted to IEEE Transactions on Networking, 2025" [quote-verified:lab 檔]。這是全系最貼近本題的一行。該論文是投稿中,OpenAlex 找不到,內容我沒讀過。
- **馮輝文**:faculty 檔「排隊理論、效能分析、模擬技術」[quote-verified];可支援 Markov/延遲模型,但無區塊鏈著作。
- 判準 6:符合,而且比 `eip1559-wait-queue` 更貼近周老師實際做的東西(網路延遲對區塊鏈的影響)。

---

## terrain-guard-uncertain: 高度不確定的 1.5D terrain guarding

### 一句話
地形剖面(x-monotone 折線)每個頂點的高度只知道落在 ±ε 內(DEM 的垂直誤差),要選最少的 guard 使得不論真實地形是哪一個都看得到整條地形;這個問題的複雜度與近似比是什麼,在真實高程剖面上比不考慮誤差的最佳解多用多少 guard?

### 為什麼不會太薄
- 模型:1.5D terrain guarding 加上 robust(對所有可能輸入都成立)的要求。精確版本已有完整的理論與可跑的精確解法,所以有東西可以比 [quote-verified:1509.08285]:"It solves instances of the TGP with up to $10^{6}$ vertices within roughly 1–2 minutes on a standard desktop computer"。
- 驗證:理論結果(困難度或近似比)加上真實 DEM 剖面的實驗(robust 解比名目解多幾成)。Friedrichs 等人的實驗只用隨機地形 [quote-verified]:"We test four classes of random terrains from the 2015-08-06 version of the TGPIL",所以「真實地形」本身就是沒做過的實驗條件 [model-only:我沒有搜尋到用真實 DEM 的 terrain guarding 實驗,但搜尋不完整]。
- 坦白說:這題比兩個區塊鏈題薄。成熟論文的去處是 CCCG、CGTA 這一級,不是頂會 [model-only]。

### 缺口還在嗎
1. **引用清單**。1509.08285(Friedrichs 等)27 篇引用,最新 12 篇裡相關的有 "Robustly Guarding Polygons"(SoCG 2024,多邊形而非地形)、"One-sided terrain guarding and chordal graphs"(DAM 2024)、"Parameter Analysis for Guarding Terrains"(Algorithmica 2022)[source-exists,標題]。項老師自己的 1805.02447 只有 3 篇引用,最新是 2020 年 [source-exists]。
2. **OpenAlex 2023 年後**。關鍵字搜尋只回 4 篇,其中兩篇正面撞題:
   - Keikha, Löffler, Saumell, Valtr,"Guarding a 1.5D Terrain with Imprecise Viewpoints",IWOCA 2025,DOI 10.1007/978-3-031-98740-3_1。OpenAlex 沒有摘要,Utrecht 典藏頁回 HTTP 500,**內容沒讀到** [source-exists,僅索引]。標題說不精確的是觀測點,不是地形。
   - McCoy, Zhu, Dutt,"Guarding Precise and Imprecise Polyhedral Terrains with Segments",COCOA 2023,DOI 10.1007/978-3-031-49614-1_24。閉鎖、無摘要 [source-exists,僅索引]。標題說處理了不精確的多面體地形(2.5D),用線段當 guard。
3. **arXiv**:`terrain guarding` 相關查詢回 10 篇,2020 年以後只有 2609.15510(弱可見多邊形,不是地形);`terrain AND imprecise AND guarding` 回 0 篇 [source-exists]。

結論:**缺口無法判斷**。1.5D、點/頂點 guard、高度不確定這個組合,我沒有找到論文;但兩篇最接近的論文都只有標題,COCOA 2023 那篇可能已經涵蓋理論核心。先讀這兩篇再決定。

### 定義這個問題的論文
- Friedrichs, Hemmer, King, Schmidt,"The Continuous 1.5D Terrain Guarding Problem: Discretization, Optimal Solutions, and PTAS",arXiv:1509.08285(全文)。留下的問題 [quote-verified]:"We would like to find an optimal discretization. But what is an optimal discretization?"
- Lai & Hsiang,"Continuous Terrain Guarding with Two-Sided Guards",arXiv:1805.02447(全文)[quote-verified]:"To the best of our knowledge, the 1.5D CTG problem with two-sided guards has never been examined."
- Lai & Hsiang,"A Linear-Time Approximation Algorithm for the Orthogonal Terrain Guarding Problem",arXiv:1804.11088(只看索引)。
- Keikha 等 IWOCA 2025;McCoy 等 COCOA 2023(都只有索引)。

### 需要的資源
- **SRTM 高程**:`s3.amazonaws.com/elevation-tiles-prod/skadi/N25/N25E121.hgt.gz`(涵蓋台北一帶的一度方格)HEAD 請求回 HTTP 200 [quote-verified:HTTP 狀態;沒有下載]。
- **TGPIL**(Terrain Guarding Problem Instance Library):1509.08285 引用為 [21];我沒有確認它還能下載 [source-exists]。
- ILP 求解器(開源的即可)與 CGAL 之類的精確幾何函式庫 [model-only]。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:定義 robust 版本 → 證明可離散化(仿 Friedrichs 的 witness/guard 集合)→ ILP 精確解 → 在 SRTM 剖面上量「robust 的代價」隨 ε 的曲線。
- 第 2 篇:近似演算法或困難度證明。
- 第 3 篇:有限視距或 2.5D,並接到感測器/無線充電器佈建(項老師 2019 年 Sensors 的題目)。
- 延展性比兩個區塊鏈題弱:這個子領域一年只有個位數論文,博論要靠換到相鄰問題。

### 最可能失敗的原因
讀完 COCOA 2023 與 IWOCA 2025 後發現理論核心已被做掉,剩下的只是一個實驗報告,撐不起「理論與實證都要有」。

### 台科大資工系相鄰的教授
- **項天瑞**:faculty 檔專長「機器人行動規劃、無線感測網路、幾何計算」,空間計算實驗室 [quote-verified:faculty 檔第 59 行]。**lab 檔是空的**(網站只回 72 bytes),所以沒有任何來自他個人網站的依據。
- OpenAlex(作者 A5084567996,38 篇)確認他與 Wei-Yu Lai 合著 1805.02447、1804.11088,以及 "Wireless Charging Deployment in Sensor Networks"(Sensors 2019)、"Deploying base stations for simple user traces in mobile networks"(ISPACS 2015)[source-exists]。
- 必須知道的事:OpenAlex 上他 2020 年以後的論文是眼底影像分割、美學評分、Graph RAG、手繪地圖導航、群組服務配置;最後一篇幾何論文是 2018–2019 年 [source-exists]。他現在是否還收幾何題目,要直接問。
- 判準 6:名義上符合;實際上目前無法判斷。

---

## 沒有採用的方向

- **純 TFM 機制理論**(放寬假設、collusion-resilience、leaderless 協定):系上無相鄰教師;faculty 檔自己註明手續費機制不在周老師頁面專長內。
- **GossipSub / DAS 傳播延遲的解析模型**(獨立成題):INFOCOM 2026 的 Eth2.0-NA 已有「理論模型 + 主網量測」,DEBS 2026 的 Lean Gossip 做了完整參數掃描 [source-exists,摘要]。剩下的 DAS 部分收進 `timing-delay-propagation` 的博論路線。
- **Bitcoin 手續費排隊模型**:2606.01274、ICBC 2026、2604.17183 三篇在 2026 年已各做一種,後者還有自架節點的 mempool 資料 [source-exists]。
- **Blob 費用市場的實證**:2026 年已有 2610.16588(彈性與福利)、2610.15256(多代理 RL)、SSRN 10.2139/ssrn.7190458(blob 納入的個體經濟)[source-exists,僅標題或摘要];偏經濟計量,離系上專長遠。
- **無線充電器佈建的幾何近似演算法**(項老師 2019 年的題目):2025–2026 的新作幾乎全是啟發式演算法(Grundy coloring 加各種群體最佳化)[source-exists,摘要],可寫成數學但難以成為成熟論文。

## 來源

| id | 標題 | 開到哪裡 |
|---|---|---|
| arXiv:2201.05574 | Empirical Analysis of EIP-1559: Transaction Fees, Waiting Time, and Consensus Security | 全文(grep 相關段落) |
| arXiv:2605.12794 | Dynamic Transaction Scheduling and Pricing in the Ethereum Mempool | 全文(grep 相關段落) |
| DOI 10.3390/math13061010 | Analysis of Dynamic Transaction Fee Blockchain Using Queueing Theory | 摘要(全文 403) |
| arXiv:2606.01274 | Strategic Users in a Priority Queue with Bulk Service on Blockchains | 摘要 |
| DOI 10.1109/icbc67748.2026.11575459 | Finite-Queue Bitcoin Transaction-Confirmation Time Modelling | 摘要 |
| arXiv:2604.17183 | A Model and Estimation of the Bitcoin Transaction Fee | 摘要 |
| arXiv:2502.20031 | On the Welfare of EIP-1559 with Patient Bidders | 摘要 |
| DOI 10.3390/math14071232 | Dynamic Fee Markets at Sub-Second Timescales | 摘要 |
| arXiv:2610.23305 | On the (In)stability of Adaptive Fee Mechanisms in Multi-Chain Systems | 索引 |
| arXiv:2305.09032 | Time is Money: Strategic Timing Games in Proof-of-Stake Protocols | 全文(本地檔) |
| arXiv:2602.11147 | Let Leaders Play Games: Improving Timing in Leader-based Consensus | 全文(grep 相關段落) |
| arXiv:2603.20426 | Pricing Innovation Under Latency Constraints | 全文(grep 相關段落) |
| DOI 10.1109/tifs.2026.3651978 | Modeling the Performance-Security Trade-Off of Gasper's Block Proposal Mechanism Under Latency-Driven Attacks | 摘要 |
| DOI 10.1109/tdsc.2026.3737958 | Strategic Timing Games in Ethereum: An Empirical Study and Mitigation | 截斷的摘要 |
| DOI 10.5281/zenodo.23473829 | Timing Games on Ethereum in 2026: The Attester Externality… | 截斷的摘要 |
| DOI 10.1109/infocom59046.2026.11571391 | Eth2.0-NA: Modeling Message Propagation to Optimize Mesh Size in Ethereum 2.0 Network | 摘要 |
| DOI 10.1145/3809481.3812610 | Lean Gossip: Big Gains From Small Talk | 摘要 |
| DOI 10.1145/3721462.3770769 | PANDAS: Peer-to-peer, Adaptive Networking Allowing Data Availability Sampling… | 摘要 |
| arXiv:2607.11240 | From PBS to ePBS: the Microstructure of Block Building | 摘要 |
| arXiv:2610.18812 | The Geography of (Concurrent) Block Proposing | 摘要 |
| arXiv:2609.20322 | The Price of Decentralization, Paid Twice | 摘要 |
| arXiv:1509.08285 | The Continuous 1.5D Terrain Guarding Problem: Discretization, Optimal Solutions, and PTAS | 全文(grep 相關段落) |
| arXiv:1805.02447 | Continuous Terrain Guarding with Two-Sided Guards | 全文(grep 相關段落) |
| arXiv:1804.11088 | A Linear-Time Approximation Algorithm for the Orthogonal Terrain Guarding Problem | 索引 |
| DOI 10.1007/978-3-031-98740-3_1 | Guarding a 1.5D Terrain with Imprecise Viewpoints | 索引(無摘要) |
| DOI 10.1007/978-3-031-49614-1_24 | Guarding Precise and Imprecise Polyhedral Terrains with Segments | 索引(無摘要) |
| DOI 10.3390/s19010201 | Wireless Charging Deployment in Sensor Networks | 索引 |
| 資料 | mempool-dumpster.flashbots.net;ethpandaops.io/data/xatu 與 data.ethpandaops.io 的三個 parquet;boost-relay.flashbots.net data API;AWS elevation-tiles-prod SRTM | 以 curl 確認 HTTP 回應;沒有解析資料內容 |
| 本地 | topics/universe/ntust-csie-faculty.md;papers/_ntust/labs/周詩梵.txt、馮輝文.txt、項天瑞.txt(空) | 全文 |

沒查的:Consensus(一次都沒用)、Google Scholar、DBLP、Semantic Scholar。arXiv API 在本 session 多次回 "Rate exceeded" 或 503,所以每個候選的第 3 項檢查都不完整。
