# 儲存系統、作業/嵌入式系統、編譯器與 EDA:候選題目
查證日期 2026-11-23。等級:quote-verified / source-exists / model-only。

查證方式的限制:arXiv 的 API(export.arxiv.org)整段時間都回 429(Rate exceeded),所以「最新 preprint」改用 arxiv.org 的搜尋頁(依公告日期排序)與直接下載 arxiv.org/html 全文。Consensus 沒有使用。ACM Digital Library 回 403,所以 FSIMR、IFFS、OTDB 等論文只看到索引紀錄。

## 這個領域現在在做什麼
- [quote-verified: 2610.02676] SSD 這一側的焦點是「主機告訴 SSD 資料該放哪」的介面。FDP(Flexible Data Placement,NVMe 的彈性資料放置)讓主機用 RUH(reclaim unit handle,回收單元代碼)把壽命相近的資料寫在一起,以降低 WAF(write amplification factor,寫入放大倍率);企業級 FDP SSD 已經出貨,"the device WAF remains close to ~1" 是資料中心的目標。
- [source-exists: OpenAlex 清單,只看標題] 引用 FDP 定義論文(EuroSys 2025)的 28 篇裡,2026 年的多半是「怎麼分流」的系統(FDP-Rocks、io.stream、eFDP、FDPEmu),另有把 KV cache 放上去的 LifeKV。
- [quote-verified: atc19-hajkazemi] 硬碟這一側,IMR(interlaced magnetic recording,交錯式磁記錄)仍然沒有實機:"we cannot be sure of its precise characteristics until real prototypes are available"。[source-exists: OpenAlex] 但 2024 到 2026 年仍有約 10 篇 IMR 資料管理論文,台灣與中國的團隊為主。
- [quote-verified: 2605.10860] 編譯器這一側,RISC-V 向量擴充 RVV 1.0 的實機已經買得到,GCC 15 與 Clang 21 都能自動向量化,但 "current compiler cost models do not yet fully capture" 實機上的 predication(遮罩)與 strided load(跨步載入)成本。
- [quote-verified: 謝仁偉.txt] 系上謝仁偉老師 2024 到 2026 年的論文正好橫跨這幾條線:3D NAND 可靠度、ZNS 上的 LSM-tree、IMR 與 SMR。

## 候選一覽
| 代號(英文 slug) | 題目(一句話問題) | 系上相鄰教授 | 缺口證據強度 | 驗證平台 | 延展到博論 | 主要風險 |
|---|---|---|---|---|---|---|
| fdp-collector-estimation | FDP SSD 的垃圾回收策略藏在韌體裡;主機能不能只靠看得到的遙測估出它的「挑選力」,並在部署前預測分流值不值得? | 謝仁偉(主)、連翊涵 | 強:2026-11 的論文把它寫成待做事項,三項檢查都沒找到有人做 | FEMU / FDPVirt 模擬器(可設定回收策略,當作標準答案);實機 FDP SSD 可有可無 | 估計器 → 閉環自適應分流 → 多租戶 handle 分配 | 提出這個問題的團隊自己先做完 |
| imr-track-map-bound | IMR 的 track 對映演算法離最優還有多遠?給定更新偏斜與空間使用率,寫入放大的下界是多少? | 連翊涵(主)、謝仁偉 | 中偏弱:沒看到談下界的標題,但近三年的論文都讀不到全文 | IMRSim(開源)、CloudPhysics/MSR trace;沒有實機 | 推廣到 SMR、HAMR-IMR 與 FDP 的「受限覆寫媒體」統一模型 | IMR 一直沒有產品,題目可能被視為空中樓閣 |
| rvv-vectorizer-cost-calibration | LLVM 對 RVV 的向量化成本模型在實機上判錯多少?用實機量測校準、且知道向量長度的成本模型能修回多少? | 黃元欣(主)、劉一宇 | 中:實機論文指出成本模型沒抓到,唯一的建模論文只做 GCC、gem5、離線驗證 | TSVC、PolyBench、KTH 微基準;BananaPi BPI-F3 實機或 gem5 | 成本模型 → LMUL/VF 聯合選擇 → 跨 SVE/RVV 的可攜成本模型 | 上游 LLVM 每一版都在改,結果半年就過期 |

我的排序:fdp-collector-estimation > rvv-vectorizer-cost-calibration > imr-track-map-bound。理由見各節與文末。

---

## fdp-collector-estimation: 從主機端估計 FDP SSD 隱藏的垃圾回收「挑選力」
### 一句話
FDP 讓主機決定資料放哪,但「回收時挑哪一塊」仍由韌體決定且不公開;能不能設計一組探測寫入,從主機看得到的數字反推回收策略的 selectivity(挑選力),再預測某個分流方案在這顆 SSD 上能省多少 WAF?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- [quote-verified: 2611.04234] 已有理論說明「分流標籤值多少」取決於回收策略:"label value peaks when the drive searches moderately for blocks holding little valid data, falls when it always takes the block with the least, and turns negative when it ignores how much valid data blocks hold"。所以不知道回收策略,就無法預測分流的效益。
- [quote-verified: 2611.04234 的參考文獻] 數學基礎是現成的:WAF 的 mean-field(平均場)與 Markov 模型(Desnoyers 2014 "Analytic models of SSD write performance"、Van Houdt 2013/2014、Bux & Iliadis 2010)。我只在參考文獻清單看到這些,沒有開啟原文。
- [model-only] 模型的形狀:把回收策略參數化成一個連續的挑選力參數(例如 d-choices:隨機抽 d 塊取最空的),用已知的平均場模型寫出「WAF 對 (挑選力, 空間使用率, 工作負載偏斜)」的函數,再做反問題:給一組探測工作負載下量到的 WAF 曲線,估計挑選力。要回答的理論問題是可辨識性(identifiability):哪些探測負載能把挑選力與 over-provisioning(預留空間)分開。
- 驗證:在模擬器裡把回收策略當成已知的標準答案,檢查估計值;再用估計值預測另一組負載的 WAF,和模擬結果比。

### 缺口還在嗎(三項檢查各自的結果)
1. **引用定義論文的最新論文**(OpenAlex,W4408870153 "Towards Efficient Flash Caches with Emerging NVMe Flexible Data Placement SSDs",共 28 篇,全部看過標題):沒有一篇的標題是估計或推斷回收策略。最接近的是 2611.04234,它把這件事列為待做。[source-exists]
2. **2025 年起已發表的論文**(OpenAlex 關鍵字 "flexible data placement" 加 SSD write amplification,14 筆;另以語意搜尋「從主機黑箱推斷 SSD 垃圾回收策略」,25 筆看前 8 筆):沒有命中。[source-exists,只看標題]
3. **arXiv**(arxiv.org 搜尋頁 "flexible data placement",6 筆):2611.06461、2611.04234、2610.02676、2603.09605、2507.10573、2503.11665。沒有估計器。[source-exists]
- **要當心的一筆**:[quote-verified: 2611.04234 參考文獻 16] 有一篇匿名投稿中的論文 "When does data placement pay? a closed-form, pre-deployment ceiling for write amplification and endurance in log-structured systems"(Under review)。它做的是「部署前的上限」,可能很接近。我無法開啟它。
- **我原本想做的題目已經被做掉**:「標籤有誤差、handle 數有限時 WAF 會怎樣」。[quote-verified: 2611.04234] "a two-class label is worth 65% more spare when each scheme runs at its best setting, but becomes a net loss at 20% label error";"Labels stating which data die together keep most of their value at 16 FDP handles"。所以題目往它留下的缺口移。

### 定義這個問題的論文
- arXiv:2611.04234,Zhao et al.,"The Unreported Variable: Collector Selectivity in Data Placement Evaluation"(2026-11-03)。開到全文。原文:"Estimate the hidden variable. Under FDP the host sees only telemetry. An estimator of effective selectivity from observables — the distribution of valid fractions at reclaim, relocation latency, the derivative of write amplification with respect to host writes — would let an operator predict what a labelling scheme is worth on a drive before deploying it." [quote-verified]
- 同一篇的限制:"We use simulation because it is the only way to hold the workload, the labels, the device and the spare capacity fixed while varying victim selection alone";"The shape is measured here; the coefficients for a real drive need its collector." [quote-verified]
- arXiv:2611.06461,Moon et al.,"eBPF-Driven Adaptive Lifetime-Aware Data Placement for Shared FDP SSDs"。開到全文。它把「類別比 handle 多」寫成分配問題:"Classes and must be learned, and may exceed. Choose which classes share a stream ... to minimize the live bytes GC copies"(HTML 轉文字時數學符號掉了)。[quote-verified]
- arXiv:2610.02676,Lan et al.,"FDP: The Data Placement Promise of Modern NVMe SSDs"。開到全文。實機量測,結論之一是 "the limited number of RUHs in current FDP SSDs may not be a practical barrier to effective data separation"。[quote-verified]
- doi:10.1145/3689031.3696091(arXiv:2503.11665),Allison et al.,EuroSys 2025。只看摘要。[source-exists]

### 需要的資源(資料集、模擬器、硬體;拿不拿得到)
- FEMU:[quote-verified: 2611.06461] eFDP 的實驗用 "an FEMU-emulated [13] and an enterprise FDP SSD"。FEMU 是否內建可切換的回收策略,我沒有查。
- FDPVirt / FDPVirt+(NVMeVirt 為底的 FDP 模擬器)、FDPEmu(ISPASS 2026):[source-exists,只看摘要]。是否開源未查。
- WARP(FAST 2026,"Characterizing and emulating FDP SSDs with WARP"):只在兩篇論文的參考文獻裡看到,OpenAlex 查不到紀錄。[source-exists,弱]
- 公開 block trace:[quote-verified: 2611.04234] 該文用了 "public block traces" 與一條 mail-server trace。拿得到。
- 實機 FDP SSD:[model-only] 不在公開市場上零售,拿不拿得到要問教授。最小版本不需要它。

### 碩論的最小版本 / 延展到博論的路線
- **碩論**:在一個可控的模擬器上實作 3 到 4 種回收策略;推導挑選力參數的估計式與可辨識條件;用 trace 驗證「估計 → 預測 WAF」的誤差。不碰實機。
- **第二篇**:把估計器接進 eFDP 那類閉環系統,線上估計並在回收策略或負載改變時重新分配 handle。
- **第三篇**:多租戶共享 FDP SSD 時,在回收策略未知的情況下怎麼分 handle 才有效益保證;或者在實機上驗證(需要廠商或教授的硬體)。

### 最可能失敗的原因
提出問題的人最有條件先做完。2611.04234 的作者已經有模擬器、理論與那篇匿名投稿;他們把估計器寫成「建議」,很可能就是下一篇。[model-only]

### 台科大資工系相鄰的教授(附依據)
- **謝仁偉**。[quote-verified: 謝仁偉.txt] "其研究方向主要在於高效能儲存系統與即時系統";論文 "Mitigating Write Amplification of Dual-mode Flash Memory"(ISOCC 2023)、"SplitLSM: Unlocking Write Efficiency with Zone-Aware Compaction for LSM-Trees on Zoned Namespace SSDs"(NVMSA 2025)、"FreshGC: Avoid Stale Free Blocks in 3D NAND Flash Memory"(RACS 2026)。寫入放大與垃圾回收是他實際在發表的題目。
- **連翊涵**。[quote-verified: ntust-csie-faculty.md] 專長「記憶體與儲存系統、檔案系統、作業系統、嵌入式系統」。她的論文集中在 SMR/IMR 與檔案系統,flash 這一側是相鄰而非重疊。

---

## imr-track-map-bound: IMR track 對映的寫入放大下界
### 一句話
IMR 的底層 track 一改寫就要連帶重寫相鄰的上層 track;在給定資料更新偏斜與空間使用率時,任何 track 對映策略的寫入放大至少是多少,現有演算法離這個下界多遠?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- [quote-verified: atc19-hajkazemi] 機制很清楚:"“bottom” (e.g. even-numbered) tracks cannot be written without data loss on the adjoining “top” (e.g. odd-numbered) ones",所以改底層 track 要 RMW(read-modify-write,讀出、修改、寫回)。
- [quote-verified: atc19-hajkazemi] 已有效能模型與三個啟發式演算法(track flipping、selective track caching、dynamic track mapping),做法是 "moving hot data to top tracks and cold data to bottom ones"。
- [model-only] 還缺的是最優性:把「哪些資料放上層、哪些放下層、何時搬」寫成線上問題,推導一個只依賴更新頻率分布與使用率的下界(類似 flash 上 hot/cold 分離的 WAF 下界),再量現有演算法的差距。這樣論文同時有定理和 trace 模擬。
- 驗證:IMRSim 加公開 trace。

### 缺口還在嗎(三項檢查各自的結果)
1. **引用 TrackLace(TC 2020,W3018260413)的最新論文**(共 23 篇,看最新 15 篇的標題):IFFS(TCAD 2025,檔案系統)、DTB 與 DTB+(用強化學習減少 RMW)、"Enabling Data-Deduplication-Assisted Data Relocation for Interlaced Magnetic Recording"(DAC 2025)、SecureIMR、OTDB(TACO 2026,key-value store)。全是新的設計,沒有一個標題談下界或最優性。[source-exists,只看標題]
2. **2024 年起已發表**(OpenAlex "interlaced magnetic recording",10 筆;2019 到 2023 年另有 18 筆,全部看過標題):同上。[source-exists]
3. **arXiv**(搜尋頁 "interlaced magnetic recording"):只有 1 筆,就是 IMRSim。這個社群幾乎不放 arXiv,所以這一項的檢查力很弱。
- **判定:缺口無法完全判斷。** 近三年的論文都在付費牆後,我只讀了 2019 年的 ATC 論文與 2022 年的 IMRSim 全文。IFFS 或 DTB+ 的內文可能已經有某種分析模型。動手前必須先讀這兩篇。

### 定義這個問題的論文
- Hajkazemi, Kulkarni, Desnoyers, Feldman,"Track-based Translation Layers for Interlaced Magnetic Recording",USENIX ATC 2019(usenix.org 公開 PDF,開到全文)。原文:"Further research is needed to determine whether extensions of this work (e.g. track flipping+caching) will yield conventional drive-level performance for IMR with acceptable memory cost." 以及 "At present we neither know the actual ratio of top to bottom track size in a specific real IMR drive, nor the practical range of this parameter for feasible drives; therefore this 90% figure is highly speculative." [quote-verified]
- arXiv:2206.14368,Zeng et al.,"IMRSim: A Disk Simulator for Interlaced Magnetic Recording Technology"(開到全文)。原文:"there has been no related disk simulator and product available to the public, which motivates us to develop an open-source IMR disk simulator"。[quote-verified]
- doi:10.1109/tc.2020.2988257,Wu et al.,"TrackLace: Data Management for Interlaced Magnetic Recording"。只看索引。[source-exists]
- doi:10.1145/3607922,Lien et al.,"FSIMR: File-system-aware Data Management for Interlaced Magnetic Recording"(TECS 2023)。只看索引;下載被 ACM 擋下(403)。[source-exists]

### 需要的資源
- IMRSim:[quote-verified: 2206.14368] 開源,以 Linux kernel block device driver 實作。現在的 kernel 還能不能編,未查。
- Trace:[quote-verified: atc19-hajkazemi] 該文用 CloudPhysics 的 trace(致謝裡寫明是對方提供,是否公開未查)。MSR trace 是否適用為 [model-only]。
- 實機:沒有。這一點任何人都一樣。

### 碩論的最小版本 / 延展到博論的路線
- **碩論**:單一磁區層級的模型,推導下界,並用 IMRSim 重現 3 到 4 個既有演算法,量它們與下界的差距;若差距大,提出一個逼近下界的簡單策略。
- **第二篇**:把檔案系統的資訊納入(FSIMR 的路線),問「知道檔案語意能把下界再壓多少」。
- **第三篇**:把同一套分析推廣到 SMR 與 FDP,寫成「覆寫受限媒體」的統一模型。這一步可以和 fdp-collector-estimation 接上。[model-only]

### 最可能失敗的原因
IMR 從 2016 年提出到現在仍沒有產品。[quote-verified: atc19-hajkazemi] 連定義論文都稱它 "still a new—or even speculative—technology"。審稿人可以直接問「誰需要這個下界」,而你無法用實機回答。這也限制了博論的出路。

### 台科大資工系相鄰的教授(附依據)
- **連翊涵**。[quote-verified: 連翊涵.txt] 期刊論文第 1 篇就是 "FSIMR: File-system-aware Data Management for Interlaced Magnetic Recording"(ACM TECS,與 CODES+ISSS'23 整合);另有兩篇 SMR 論文。[source-exists: OpenAlex W4386580331] 作者欄確認有 Yi-Han Lien。這是全系與這個題目最貼近的一位。
- **謝仁偉**。[quote-verified: 謝仁偉.txt] "IMRAID5: Design for RAID 5 with Interlaced Magnetic Recording"(ACM RACS 2024)、"CBW: Reducing Data Write Back with Circular Bubble Write for DM-SMR Disks"(ICS 2024)。

---

## rvv-vectorizer-cost-calibration: 以實機校準的 RVV 向量化成本模型
### 一句話
LLVM 決定「這個迴圈要不要向量化、用多寬」時靠的是靜態成本模型;在 RVV 1.0 實機上,這個模型判錯的比例是多少,能否用微基準量測校準出一個知道向量長度的成本模型,並放回編譯器裡量到實際加速?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- [quote-verified: 2611.03917] 機制:"LLVM’s Loop Vectorizer analyzes each loop and, using a target-specific cost model, decides whether vectorizing is likely to be profitable, and at what width."
- [quote-verified: 2605.10860] 實機上已量到模型沒抓到的兩項成本:"predication masks introduce 35% overhead w.r.t unmasked vsetvl for tailing elements, and strided vector ld/st instructions incur up to 4 the cost of unit-stride equivalents"(原文應為 4 倍,HTML 轉文字時乘號掉了)。
- [model-only] 模型的形狀:迴圈的預測時間等於各類指令成本的線性組合,係數隨 VL(向量長度)、LMUL(暫存器分組倍數)、是否遮罩、記憶體存取樣式而變;用微基準做迴歸求係數;評估指標是「判斷錯誤率」(該向量化卻沒做,或做了反而變慢)與修正後的實測加速。
- 驗證有公開基準:[source-exists: 兩篇摘要] TSVC 與 PolyBenchC-4.2.1。

### 缺口還在嗎(三項檢查各自的結果)
1. **引用 "RISC-V Vectorization Coverage for HPC: A TSVC-Based Analysis"(SC'25 Workshops,W4416003937)的論文**(共 5 篇,全部看過):2605.10860(實機評估,Euro-Par 2026)、2611.03917(bf16/fp16 退化)、一篇 XNNPACK 能耗、一篇 ARM SVE 量子模擬、一篇 VLA 的 ML 程式碼生成。沒有一篇提出校準過的成本模型。[source-exists]
2. **2025 年起已發表**(OpenAlex,8 筆):最接近的是 "Evaluation and Benefit Modeling of Auto-Vectorization Cost Models for RISC-V RVV"(EICCT 2026,doi:10.1109/eicct69950.2026.11564611)。[source-exists,只看摘要] 它評估的是 GCC、平台是 gem5,提出 "a vector length-aware static benefit model",但只做到 "offline verification",自己的結論是 "can be used as a supplementary analysis basis"。**這篇已經佔了「VL-aware 成本模型」這個說法**;剩下的缺口是 LLVM、實機、以及真的接回編譯器。
3. **arXiv**(搜尋頁 "RISC-V vector auto-vectorization" 8 筆;"vectorization cost model compiler" 11 筆):沒有 RVV 成本模型校準的論文;1909.13639 NeuroVectorizer 是用強化學習選向量化參數的舊作(只看標題)。
- 沒查到的地方:LLVM 的 mailing list、Phabricator/GitHub PR、RISC-V Summit 的演講不在任何學術索引裡。上游工程師很可能已經在改這些成本表,我無法從這裡確認。

### 定義這個問題的論文
- arXiv:2605.10860,Shi et al.,"Closer in the Gap: Towards Portable Performance on RISC-V Vector Processors"(Euro-Par 2026;開到全文)。原文:"we find that predication overhead and stride load pose performance bottlenecks that current compiler cost models do not yet fully capture." [quote-verified]
- doi:10.1109/eicct69950.2026.11564611,Yao et al.(EICCT 2026)。只看摘要。摘要原文:"in the scenario of unknown vector length, the fixed overhead and the difference of loop structure are not considered enough, and there is a problem of overestimation and underestimation of revenue." [source-exists]
- doi:10.1145/3731599.3767535,Lai et al.,"RISC-V Vectorization Coverage for HPC: A TSVC-Based Analysis"。只看摘要。[source-exists] OpenAlex 的作者欄有 National Tsing Hua University 的 Jenq-Kuen Lee,台灣有人在做這條線。
- arXiv:2611.03917,Kouru & Chun,"Silent Degradation during Auto-Vectorization of bf16/fp16 on RISC-V Under LLVM"(開到全文)。原文:"widened bf16/fp16 operations need extra care in the cost model because of their effect on LMUL and register pressure"。[quote-verified]

### 需要的資源
- 實機:[quote-verified: 2605.10860] "Milk-V Jupiter and BananaPi BPI-F3, both are based on the Spacemit(R) X60 CPU with 8 cores";價格與在台灣能否買到為 [model-only],我認為是一般開發板價位。
- 微基準:[quote-verified: 2605.10860] 作者公開在 github.com/KTH-ScaLab/rvv-evaluation。
- 模擬器:[source-exists: EICCT 摘要] gem5 可跑不同向量長度。實機只有一種向量長度,要談「VL-aware」就需要 gem5 或 QEMU。
- 編譯器:LLVM 開源。[quote-verified: 2605.10860] "LLVM 21 advances by extending the RISC-V backend to support a rich set of CPU models, including the SpacemiT scheduling model."

### 碩論的最小版本 / 延展到博論的路線
- **碩論**:在一塊 X60 板子上,用 TSVC 加 PolyBench 量 LLVM 成本模型的判錯率;以微基準校準係數;把校準後的成本表接回 LLVM 的 RISC-V TTI(target transform info),報告實測加速與退化案例。
- **第二篇**:LMUL 與 VF(vectorization factor)的聯合選擇,把暫存器壓力放進模型。[quote-verified: 2605.10860] "Default LMUL selection proves near-optimal in most cases, though GCC 15 shows greater potential for performance gains through larger LMUL tuning",所以這一步的空間有限,要先量。
- **第三篇**:從微基準自動生成成本模型,跨 RVV 與 ARM SVE 兩種可變長度向量架構;或用 gem5 研究不同 VLEN 下的可攜性。[model-only]

### 最可能失敗的原因
這比較像工程調校而不是研究問題。校準結果綁定一顆 CPU(X60)和一個 LLVM 版本;上游可能在你寫完前就把同樣的修正合進去。要撐成論文,貢獻必須是「自動校準的方法與判錯率的量化」,而不是那張成本表。

### 台科大資工系相鄰的教授(附依據)
- **黃元欣**。[quote-verified: 黃元欣.txt] Research Interests:"Parallel Processing and Parallelizing Compilers / Compilation Techniques for Embedded Systems / Computer Architecture";本學期開「高等編譯器設計」。注意:他的網頁只抓到 1,133 bytes,沒有論文清單,所以我無法確認他近年實際發表什麼,只能確認專長與授課。
- **劉一宇**。[quote-verified: 劉一宇.txt] 計畫 "RISC-V架構技術與創新應用組教材發展計畫 ─ 指令層級平行化技術, 2024/07/01-2025/06/30" 與 "RISC-V處理器架構、模擬軟體與運用"。他的論文主軸是封裝載板繞線,RISC-V 是教學計畫,所以是相鄰而非主力。

---

## 沒有採用的方向
- **「標籤預測有誤差、RUH 數有限時的 WAF 解析模型」**:已被 arXiv:2611.04234(2026-11-03)做掉,見上。[quote-verified]
- **Skyrmion racetrack memory 的資料放置**(謝老師 RACS 2026 的 "Characterizing Frequency-Aware Placement for Skyrmion Racetrack Memory in Block Storage"):OpenAlex 2023 年起只找到 3 筆相關論文,沒看到公開硬體;有沒有公開模擬器我沒查。社群太小,延展性不明。[source-exists]
- **封裝載板繞線**(劉老師的主力,TCAD 2026、TODAES 2026):以 "package substrate routing benchmark open-source" 搜尋 OpenAlex,0 筆相關。我無法確認有公開 benchmark;沒有公開資料就不符合判準 4。這是「沒找到」,不是「確定沒有」。
- **3D NAND 晶片層級的可靠度**(FreshGC、read retry table、CellRejuvo):需要晶片測試平台,而且與已保留的 kv-offload-aging-flash 重疊。
- **ZNS 上的 LSM-tree compaction**(謝老師的 SplitLSM):與清單上的 kv-zns-write-amp 太近,沒有另外查。

## 順帶查到、與既有候選有關的兩件事
- **kv-zns-write-amp 有新的競爭者**:"LifeKV: Reducing SSD Write Amplification with Lifetime-Aware KV Cache Placement in Multi-Turn LLM Serving"(IISWC 2026,doi:10.1109/iiswc71020.2026.00028)。只看到索引紀錄的標題,沒讀摘要。評估那個候選時要先讀它。[source-exists]
- **kv-offload-aging-flash 的判準 6 現在有依據了**:[quote-verified: 謝仁偉.txt] 謝老師有 "Read Retry Mechanism for 3D NAND Flash Memory: Observations, Analyses, and Solutions"(NVMSA 2024, pp.49-54)與 "Adaptive Read Retry Table Extension for Enhanced Data Reliability in 3D NAND Flash Memory"(DAC 2026)。那一頁寫「依報告;本 session 未查證」的地方可以更新;他的實驗室有沒有晶片測試平台仍要當面問。

## 來源
| id | 標題 | 開到哪個程度 |
|---|---|---|
| arXiv:2611.04234 | The Unreported Variable: Collector Selectivity in Data Placement Evaluation | 全文(HTML) |
| arXiv:2611.06461 | eBPF-Driven Adaptive Lifetime-Aware Data Placement for Shared FDP SSDs | 全文(HTML),只讀了問題定義、摘要與結論 |
| arXiv:2610.02676 | FDP: The Data Placement Promise of Modern NVMe SSDs | 全文(HTML),關鍵字檢查 |
| doi:10.1145/3689031.3696091(arXiv:2503.11665) | Towards Efficient Flash Caches with Emerging NVMe Flexible Data Placement SSDs | 摘要 |
| doi:10.1145/3828523.3857126 | FDP-Rocks: An Effective Use Case of FDP in RocksDB | 摘要 |
| doi:10.1109/ispass69572.2026.00041 | FDPEmu: How to Separate Workloads for Better WAF on FDP SSDs | 摘要 |
| doi:10.1109/clusterworkshops61563.2024.00057 | FDPVirt: Flexible Data Placement SSD Emulator | 摘要 |
| doi:10.1145/3837053.3837343 | io.stream: cgroup-Granular FDP for Unmodified Workloads | 索引 |
| doi:10.1109/iiswc71020.2026.00028 | LifeKV: Reducing SSD Write Amplification with Lifetime-Aware KV Cache Placement in Multi-Turn LLM Serving | 索引 |
| (無 id)USENIX FAST 2026 | Characterizing and emulating FDP SSDs with WARP | 只在他人的參考文獻中看到 |
| USENIX ATC 2019(usenix.org/system/files/atc19-hajkazemi.pdf) | Track-based Translation Layers for Interlaced Magnetic Recording | 全文(PDF) |
| arXiv:2206.14368 | IMRSim: A Disk Simulator for Interlaced Magnetic Recording Technology | 全文(HTML) |
| doi:10.1109/tc.2020.2988257 | TrackLace: Data Management for Interlaced Magnetic Recording | 索引 |
| doi:10.1145/3607922 | FSIMR: File-system-aware Data Management for Interlaced Magnetic Recording | 索引 |
| doi:10.1109/tcad.2025.3642739 | IFFS: An Interlaced Magnetic Recording Friendly File System | 索引 |
| doi:10.1016/j.sysarc.2025.103479 | DTB+: An enhanced data management strategy for efficient RMW reduction in IMR drives | 索引 |
| doi:10.1109/dac63849.2025.11133405 | Enabling Data-Deduplication-Assisted Data Relocation for Interlaced Magnetic Recording | 索引 |
| doi:10.1145/3844512 | OTDB: A Key-Value Store Tailored for Overlapping Track-Based Disks | 索引 |
| arXiv:2605.10860 | Closer in the Gap: Towards Portable Performance on RISC-V Vector Processors | 全文(HTML) |
| arXiv:2611.03917 | Silent Degradation during Auto-Vectorization of bf16/fp16 on RISC-V Under LLVM | 全文(HTML),只讀了摘要與成本模型段落 |
| doi:10.1109/eicct69950.2026.11564611 | Evaluation and Benefit Modeling of Auto-Vectorization Cost Models for RISC-V RVV | 摘要 |
| doi:10.1145/3731599.3767535 | RISC-V Vectorization Coverage for HPC: A TSVC-Based Analysis | 摘要 |
| 本機檔案 | papers/_ntust/labs/謝仁偉.txt、連翊涵.txt、黃元欣.txt、劉一宇.txt;topics/universe/ntust-csie-faculty.md | 全文(黃元欣.txt 只有 1,133 bytes,沒有論文清單) |
