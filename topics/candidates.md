# 候選題目

狀態:`待評估`、`未定`(評估過但缺一項事實)、`保留`、`丟棄`、`選定`。

前十個題目來自 2026-11-18 的報告。該報告引用的論文在本 session 都還沒有開啟查證,所以它對每個題目的描述目前都是 `model-only`。

| 代號 | 題目 | 狀態 | 理由(一行) | 評估頁 |
|---|---|---|---|---|
| kv-offload-aging-flash | 可靠度感知的 KV cache 卸載:老化 flash 上「載入或重算」的最優決策 | 保留 | 我認為還是很有機會;最大問題是市售 SSD 不回報重試次數。有四條可能的驗證途徑(自行磨耗量測、OpenSSD、FEMU、問教授),都無法馬上得到答覆,故先保留 | [頁面](kv-offload-aging-flash.md) |
| kv-zns-write-amp | KV cache 生命週期與 ZNS zone 對齊:寫入放大的解析模型 | 待評估 | | |
| ifc-bounded-declassification | 有界洩漏的降級:讓資訊流控制的 LLM agent 不再過度封鎖 | 未定 | 缺:在多 agent、有狀態的系統上 IFC 的效用損失有多大;另外學習成本超過上限 | [頁面](ifc-bounded-declassification.md) |
| agent-injection-epidemic | 多 agent 系統中 prompt injection 的傳播:流行病模型與閾值 | 待評估 | | |
| qkd-keypool-backpressure | QKD 網路的金鑰池排隊模型與吞吐量最優路由 | 待評估 | | |
| entanglement-cutoff-multipath | 帶 cutoff 的多路徑糾纏分配:速率與保真度的前緣 | 待評估 | | |
| xapp-conflict-game | O-RAN xApp 衝突的賽局模型與無政府代價 | 待評估 | | |
| 5g-core-metastability | 5G 核心網控制面在信令風暴下的排隊模型與准入控制 | 待評估 | | |
| eew-conformal | 具保證涵蓋率的地震預警:時空分佈偏移下的 conformal prediction | 待評估 | | |
| tee-llm-perf-model | 機密運算下 LLM 推論的效能模型 | 待評估 | | |
| kv-hbf-read-retry | 同一個問題搬到 HBF(High-Bandwidth Flash):read retry 對「維持 HBM 級讀取頻寬」的影響 | 待評估 | 由 kv-offload-aging-flash 的評估衍生;2026 下半年的論文多在 HBF 上 | |
| tfm | 區塊鏈交易手續費機制(transaction fee mechanism);見 `maps/transaction-fee-mechanisms.md` | 待評估 | 這是一個領域,還不是題目;需要先收斂到其中一個方向 | |
| fdp-collector-estimation | 從主機端遙測估計 FDP SSD 隱藏的垃圾回收選擇性 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/storage-systems.md) |
| rvv-vectorizer-cost-calibration | LLVM RVV 向量化成本模型在真實硬體上的誤判與校正 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/storage-systems.md) |
| imr-track-map-bound | 交錯式磁記錄軌道對應的寫入放大下界 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/storage-systems.md) |
| predicted-dispatch-fog | 工作大小只有預測值時 fog 叢集分派的回應時間模型 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/networks-edge.md) |
| age-of-entanglement-chain | n 跳量子中繼鏈上最小化 Age of Entanglement 的策略 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/networks-edge.md) |
| purify-buffer-policy | 糾纏緩衝區的最優純化策略 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/networks-edge.md) |
| fedlora-privacy-audit | 聯邦 LoRA 微調下 one-shot 隱私稽核是否仍成立 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/security.md) |
| iot-malware-transform-cert | 以語意保持轉換為半徑的 IoT 惡意程式偵測強健性證書 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/security.md) |
| fedit-safety-detectability | 聯邦 LoRA 指令微調中安全攻擊的可偵測性門檻 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/security.md) |
| ransac-stop-guided | 引導式取樣下 RANSAC 停止規則的機率保證 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/vision-graphics-xr.md) |
| hmd-pose-calibration | 頭顯全身姿態估計的不確定度校準與涵蓋率保證 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/vision-graphics-xr.md) |
| framegen-error-budget | 外插畫面生成的誤差預算與排程 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/vision-graphics-xr.md) |
| ger-damage-control | LLM 修正語音辨識時「改壞率」的保證 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/data-ml-nlp.md) |
| drift-delay-anytime | 延遲標籤下概念漂移偵測的誤報上界 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/data-ml-nlp.md) |
| filtered-ann-plan-regret | 過濾式向量搜尋的執行計畫選擇與 regret | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/data-ml-nlp.md) |
| eip1559-wait-queue | EIP-1559 交易等待時間的排隊模型與 mempool 資料驗證 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/blockchain-theory.md) |
| timing-delay-propagation | 區塊傳播延遲下提案者的最優延遲與錯過 slot 機率 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/blockchain-theory.md) |
| terrain-guard-uncertain | 高度不確定的 1.5D 地形守衛問題 | 待評估 | 2026-11-23 題目宇宙報告提出;尚未單獨評估 | [報告](universe/blockchain-theory.md) |

2026-11-23 新增的 18 個候選來自 `universe/README.md`,依台科大資工系的師資分群提出。
