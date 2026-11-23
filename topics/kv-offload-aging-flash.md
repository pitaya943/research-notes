---
candidate: kv-offload-aging-flash
date: 2026-11-22
verdict: keep
sources:
  swarm: {ref: "arXiv:2603.17803", title: "Swarm: Co-Activation Aware KVCache Offloading Across Multiple SSDs"}
  kvdrive: {ref: "arXiv:2605.18071", title: "KVDrive: A Holistic Multi-Tier KV Cache Management System for Long-Context LLM Inference"}
  saria: {ref: "arXiv:2610.31169", title: "SARIA: Persistence-Aware KV Caching with Coordinated DRAM-SSD Reload for LLM Agents"}
  laflash: {ref: "arXiv:2611.22746", title: "LaFlash: Capacity-Driven Acceleration for Coding Agent Workflows via QLC-HBF"}
  hbfsim: {ref: "arXiv:2610.23503", title: "HBFSim: A Simulator for Exploring HBF in Memory Hierarchies for LLMs"}
  storage-view: {ref: "arXiv:2611.10178", title: "The Storage View Of KV Cache: Characterizing Cache Behavior Across LLM Workloads"}
  flatcake: {ref: "arXiv:2610.38355", title: "A Flat KV Cache Storage Architecture for LLM Serving"}
  raro: {ref: "arXiv:2508.19530", title: "RARO: Reliability-aware Conversion with Enhanced Read Performance for QLC SSDs"}
  park-rr: {ref: "arXiv:2104.09611", title: "Reducing Solid-State Drive Read Latency by Optimizing Read-Retry"}
  ye-nzrr: {ref: "doi:10.1145/3620665.3640372", title: "Achieving Near-Zero Read Retry for 3D NAND Flash Memory"}
  gandalf: {ref: "arXiv:2610.30791", title: "Gandalf: Flash Shall Not Die! Increasing Flash Storage Lifetime for KV Cache Offloading in LLM Serving"}
  lachesis: {ref: "arXiv:2610.08378", title: "Lachesis: Lifetime-Aware KV Cache Placement for Agent Serving across HBM and High-Bandwidth Flash"}
  hbf-postech: {ref: "arXiv:2608.13868", title: "Exploring High-Bandwidth Flash for Modern LLM Inference: Opportunities and Challenges"}
  read-reclaim: {ref: "doi:10.1109/lca.2026.3742410", title: "Read-Reclaim-Induced Endurance Degradation in High-Bandwidth Flash for LLM Inference"}
---
# 可靠度感知的 KV cache 卸載:老化 flash 上「載入或重算」的最優決策

2026-11-22 第二次評估:讀了 park-rr 與 raro 的內文,並對 laflash、gandalf、lachesis、hbf-postech 等五篇做了全文關鍵字檢查。其餘論文仍只看摘要。

## 一句話
SSD 老化後讀取變慢,那麼「從 SSD 載入 KV cache」和「在 GPU 上重算」的划算分界點會怎麼移動?

## 這個領域在做什麼
- [source-exists: swarm] LLM 推論時 KV cache(每個 token 算過的中間結果)是記憶體消耗的主因;把它從 GPU 搬到 DRAM 再搬到 SSD 比較便宜,但 SSD 的讀取受頻寬限制。
- [source-exists: saria] 多輪對話的 agent 會重複使用很長的前綴,所以把 KV cache 存起來重用可以省下重算的時間;但全部寫進 SSD 會快速消耗 SSD 的寫入壽命。
- [source-exists: raro] 高密度 flash(QLC)可靠度有限,會頻繁觸發 read retry(讀取重試),嚴重拖慢讀取。
- [model-only] 這個題目要把上面兩條線接起來:read retry 讓 KV 載入變慢,進而拉長第一個 token 的等待時間。

## 缺的那項事實:查到了什麼
問題:高寫入次數、短存放時間下,read retry 讓循序讀取吞吐量下降多少?

- **每多一次重試的代價:已確定,而且很大。** [quote-verified: park-rr] 讀一頁的時間隨重試次數 N_RR 線性增加:"tRETRY = N_RR × (tR + tDMA + tECC)"。所以讀取時間約為無重試時的 (1 + N_RR) 倍。
- [quote-verified: raro] 在 FEMU 模擬的 QLC 上,"one retry and ten retries reduce the sequential read bandwidth by 50.0% and 92.0%"。這與 1/(1+N_RR) 一致(1/2 與 1/11)。注意這是模擬器的結果,不是實機量測。
- **零存放時間下會重試幾次:有數字,但只到圖的精度。** [source-exists: park-rr, Figure 5, 由 agent 從圖上讀出,約略值] 160 顆真實 3D TLC 晶片,存放 0 個月時:0 P/E 幾乎不重試;1K P/E 平均不到 1 次;2K P/E 平均約 2 次、最多約 3 到 4 次。換算成讀取吞吐量:1K P/E 時下降不到一半,2K P/E 時約降到原本的 1/3。
- **反證:韌體可以把這個效應幾乎消掉。** [source-exists: ye-nzrr, 只看摘要] 針對晶片調整 read retry table 後,8K P/E、長存放時間下平均重試次數低於 0.003;同一摘要說當時的既有方法在 3K P/E 就超過 3 次。
- 還不知道的:圖上存放時間的刻度是 0 與 3 個月,KV cache 實際是幾小時到幾天,落在兩者之間;以及 QLC 與 2026 年的晶片是否相同。

結論:原本擔心的「只慢一成」不太可能。這個效應要嘛很大(只要有一次重試,吞吐量就減半),要嘛被韌體消掉而接近零。它**不是小,而是取決於你看不到的韌體**。

## 需要的知識
| 知識 | 我有沒有 | 補起來要多久 |
|---|---|---|
| LLM 推論內部:KV cache、prefill、TTFT、vLLM 這類系統 | 不確定,請你確認 | 約 2 週 |
| NAND flash 可靠度:P/E cycle、retention、RBER、read retry、LDPC | 沒有 | 約 3 週 |
| SSD 模擬器 FEMU 的使用與修改 | 沒有 | 約 2 到 3 週 |
| 成本模型與門檻策略的最優性證明 | 大學程度的最佳化應該夠 | 約 1 週 |

合計約 8 到 9 週,剛好在上限邊緣。[model-only]

## 缺口還在嗎
2026-11-22 在 arXiv 以三組關鍵字搜尋(KV cache 加 SSD/flash;KV cache 加 read retry/wear/endurance/reliability/aging;KV cache 加 recompute 加 storage),各看最新 12 筆的標題,並開啟其中 7 篇的摘要。

- **寫入端的壽命已經有人做。** [source-exists: saria] 選擇性寫入以減少磨耗;[source-exists: laflash] 寫入壽命限制了資料放置;[source-exists: hbfsim] 模擬出 flash 在 KV 寫入下可用 2.0 到 6.9 年。
- **讀取端的老化(read retry 讓載入變慢)在這些摘要裡沒有出現。** 所以這個特定缺口目前看起來還在。
- **但這個領域非常擁擠,而且速度很快。** 光是 2026 年 10 到 11 月,arXiv 上就有至少 7 篇 KV cache 加 SSD 的系統論文。
- 限制(第一次評估):只查了 arXiv、只看摘要。

2026-11-22 第二次,三項檢查各自的結果:
1. **引用 park-rr 的最新論文**(OpenAlex,共 83 篇,看最新 25 篇):其中 3 篇是 LLM 加 flash(laflash、SlipStream、一篇 HBM 加 flash 的 CAL 短文)。[quote-verified: laflash] 它只把 read retry 列為未來的挑戰:"device-level reliability issues during operation, such as read-retry and retention degradation"。沒有人量化它對 KV 載入的影響。
2. **近十二個月已發表的論文**(OpenAlex,2025 年起,27 筆):寫入壽命這一側又多了很多。[source-exists: gandalf] 自稱第一個有 SSD 壽命意識的 KV 卸載系統;[source-exists: lachesis] 依資料壽命決定放置;[source-exists: read-reclaim] 重複讀取造成 read disturb,引發 read reclaim 而消耗壽命。最後這篇是讀取端的可靠度問題,但談的是壽命,不是載入變慢。
3. **arXiv**:"read retry" 加上 "KV cache" 或 LLM,0 筆。對 gandalf、lachesis、hbf-postech 與另外兩篇 HBF 特性分析做全文檢查,"retry" 出現 0 次。

所以「老化讓載入變慢,進而移動載入或重算的分界點」目前沒有人做。兩個要注意的變化:
- **場景正在從 SSD 移到 HBF(High-Bandwidth Flash,封裝在 GPU 旁的高頻寬 flash)。** 2026 年下半年的論文大多在談 HBF。[quote-verified: hbf-postech] HBF 的好處取決於 "sustaining HBM-comparable read bandwidth"。read retry 正好破壞這一點,所以這個題目放到 HBF 上可能更有份量。[model-only]
- **最可能搶先的人已經在場。** gandalf 的作者群包含 FEMU 的作者;park-rr 的作者也寫了 hbf-postech。這兩組人同時懂 flash 可靠度和 KV cache。

## 一年內做得完嗎
最小可交付的版本 [model-only]:成本模型,加上用公開 trace 驅動的模擬(把 read retry 造成的延遲當成參數注入),再加一組小規模的真實量測。**不做** FEMU 與真實 GPU 推論的完整整合;那是報告標出的主要工程風險。

粗估:學習 2 個月、建模 2 個月、模擬與量測 4 個月、寫作 2 個月,留 2 個月緩衝。

## 需要的資源
- 公開 trace:[source-exists: storage-view] 該論文分析了 11 組 production 與公開 trace,所以公開 trace 存在。拿得到。
- FEMU:開源。它能否設定 read retry 與老化參數是報告的說法,未查證。
- GPU:做真實量測時需要。拿不拿得到未知。
- 老化 flash 的重試次數分佈:[source-exists: park-rr] Figure 5 有 TLC 在 0/1K/2K P/E、0 到 12 個月的分佈,可以當模擬的輸入。QLC 與短於 3 個月的資料沒有。
- 在真實 SSD 上觀察重試次數:市售 SSD 不會把它報給主機。要嘛用 FEMU 注入,要嘛需要有晶片測試平台的實驗室。未確認。

## 最可能失敗的原因
**效應取決於韌體,而你控制不了也看不到韌體。** 2026-11-22 之前寫的是「效應可能太小」;查過之後,這個理由改了。重試一旦發生,影響很大;但 [source-exists: ye-nzrr] 顯示好的 read retry table 可以讓它接近零。審稿人會問:「現在的企業級 SSD 還會重試嗎?」如果只能用模擬器注入參數來回答,實證的部分就站不穩。[model-only]

## 對照判準
| 判準 | 結果 | 依據 |
|---|---|---|
| 1. 跟得上 | 邊緣 | 估計 8 到 9 週,flash 可靠度是全新領域 |
| 2. 缺口還在 | 符合,被搶先的風險高 | 三項檢查都沒找到讀取端老化對 KV 載入的研究;但兩組最有能力做的人已在這個領域發表 |
| 3. 有模型也有驗證 | 符合 | 載入時間正比於 (1 + N_RR) 有文獻依據;加 trace 驅動模擬。但「門檻策略最優」的定理可能很淺 |
| 4. 資源拿得到 | 目前無法判斷 | 重試次數分佈有公開的圖可用;但實機上觀察重試需要晶片測試平台,GPU 也未確認 |
| 5. 一年做得完 | 符合(限最小版本) | 前提是不做完整系統整合 |
| 6. 有人能指導 | 符合 | 依報告;本 session 未查證 |

## 結論
**保留**(2026-11-22,我的決定)。

理由:我認為還是很有機會。當前最大的問題是市售 SSD 不會回報重試次數;如果拿不到可以計算重試次數的 SSD,可驗證的部分會很弱。但下面四條途徑有機會解決這個問題,而它們都無法現在馬上得到答覆,所以先保留,等有答覆再決定。

要去確認的途徑(由 agent 提出,我採用;皆為 [model-only],尚未查證):
1. 只量主機端看得到的東西:把一顆市售 SSD 寫到高 P/E,量循序讀取吞吐量隨磨耗與存放時間的變化。模型要預測的是載入時間,不一定需要重試次數本身。粗估約一週。
2. 開放韌體的平台(例如 Cosmos+ OpenSSD),可以在韌體裡自己計數。要確認拿不拿得到硬體。
3. FEMU 注入重試參數;只能當輔助,不能當唯一的驗證。
4. 問做 flash 的教授:實驗室有沒有晶片測試平台,或廠商的 log page 有沒有重試計數。2026-11-23 補充:[quote-verified: 謝仁偉實驗室網站著作清單] 系上謝仁偉教授有兩篇 read retry 的論文(NVMSA 2024、DAC 2026),前一篇在真實平台上量測,應該先問他。

什麼情況下重新考慮:四條途徑都有答覆之後。如果沒有任何一條能觀察或推得重試的影響,丟棄。
