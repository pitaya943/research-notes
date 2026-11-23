# 視覺、多媒體、電腦圖學與延展實境:候選題目
查證日期 2026-11-23。等級:quote-verified / source-exists / model-only。

說明:本檔依指示不計判準 1(學習成本)。「夠不夠新」是 agent 的估計,最後要問真正的指導教授。

## 這個領域現在在做什麼

- 點雲配準(point cloud registration,把兩片 3D 掃描對齊)的主流是「先配對特徵、再用 RANSAC 或相容圖(compatibility graph)剔除離群對應」;2024-2026 的論文多半在比速度和高離群率下的成功率(OpenAlex 語意搜尋回傳的 12 筆皆屬此類,`source-exists`,只看了標題與摘要)。
- 即時繪圖的畫格生成(frame generation)分成內插(interpolation,會增加延遲)與外插(extrapolation,不增加延遲但要處理遮擋解除 disocclusion);2025-2026 每年都有 SIGGRAPH / SIGGRAPH Asia / TOG 的新方法(GFFE 的 20 篇引用文獻清單,`source-exists`,只看了標題)。
- XR 的全身姿態估計只靠頭盔與雙手控制器三個追蹤點,是一對多的病態問題;2024-2026 的作法是擴散模型、加環境點雲、加不確定度輸出(EnvPoser 全文,`quote-verified`;其餘方法只看了摘要,`source-exists`)。
- 3D Gaussian Splatting 的不確定度與 conformal 保證在 2026 下半年已有人做(見「沒有採用的方向」),這條線我放棄了。
- 整體判斷(`model-only`):這個領域大多數論文是「在基準上比數字」,符合學習者「有模型也有驗證」的題目不多;下面三題是刻意挑有機制或有保證的。

## 候選一覽

| 代號(英文 slug) | 題目(一句話問題) | 系上相鄰教授 | 缺口證據強度 | 驗證平台 | 延展到博論 | 主要風險 |
|---|---|---|---|---|---|---|
| ransac-stop-guided | 點雲配準的 RANSAC 用了過濾過、會自我更新的取樣集合之後,「信心水準 99.5% 就停」的停止條件還成立嗎?能不能給一個實際達到目標成功機率的停止規則? | 鍾國亮(直接相鄰) | 強:定義問題的論文自己寫「超出本文範圍」,4 篇引用文獻都沒做 | 3DMatch、KITTI、WHU-TLS(公開);鍾老師公開的 C++ 程式碼 | 統一框架(雜訊、取樣、局部最佳化一起算)→ 有保證的配準失敗偵測 | 理論上精確機率可能寫不出封閉式,只能做出保守上界 |
| framegen-error-budget | 畫格外插時,能不能在產生畫格之前,由深度、運動向量與著色梯度預測「哪些像素一定補不回來」,並據此決定外插、重新著色或整張重繪? | 蕭凱文(直接相鄰)、賴祐吉/戴文凱(即時繪圖) | 中:方法很擠,解析模型這個問法沒找到,但兩篇最接近的論文沒開到全文 | Falcor 繪圖框架 + Bistro、Sponza、San Miguel 場景(公開) | 延遲與瑕疵的感知模型(需要受試者實驗)→ VR 雙眼與雲端串流 | 2025-2026 的排程器論文可能已經用啟發式做到夠好,模型只多解釋力、不多效能 |
| hmd-pose-calibration | 只靠頭盔與雙手三點追蹤的全身姿態估計,輸出的關節不確定度是否校準(calibrated)?從日常動作換到運動動作時還校準嗎?能否對下半身誤差給覆蓋率保證? | 潘則佑(直接相鄰)、姚智原 | 中:最接近的 EnvPoser 有不確定度但全文沒有任何校準評估;arXiv 最新預印本這一步沒查成 | AMASS、EgoBody、GIMO(公開動作資料集) | 以不確定度驅動的感測器增補與互動設計(需要受試者實驗) | 容易變成「把 conformal 套到又一個任務」,太薄 |

我自己的排序:ransac-stop-guided > hmd-pose-calibration > framegen-error-budget。理由寫在各題與最後回報。

---

## ransac-stop-guided: 引導式取樣下 RANSAC 配準的停止條件

### 一句話
RANSAC 的停止條件假設「每次都從全部對應中均勻隨機抽樣」;當點雲配準先剔除離群、再讓取樣集合邊跑邊更新時,這個假設不成立,那麼宣稱的信心水準實際上達到多少,以及怎樣的停止規則才真的達到目標成功機率?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:抽到全內點(all-inlier)最小樣本的機率。經典公式用 p^k 近似,精確值是不放回抽樣的超幾何機率;兩者的比值在 Schönberger 等人的論文中寫成式 (7)(`quote-verified`,全文)。本題把它推到「取樣集合不是全集、而且每輪被更新」的情形:取樣集合的內點率與全集不同,集合更新又讓各次抽樣不獨立。
- 驗證:合成資料上可直接量「宣稱信心水準 vs. 實際成功率」(真值已知);真實資料用 3DMatch、KITTI、WHU-TLS,這三個資料集就是鍾老師 2026 年預印本用的(`quote-verified`:「the three public testing datasets, 3DMatch [56], KITTI [57], and WHU-TLS [58], are used to evaluate both the accuracy and runtime」)。
- 這是機率推導加實驗的題目,不需要 GPU 訓練,也不是比 SOTA。

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(引用 "Fixing the RANSAC Stopping Criterion",W4414566311,依日期排序):共 4 筆。"RANSAC Scoring Done Right"(arXiv 2606.27385,做的是評分函數而不是停止條件)、一篇深海採礦車視覺定位、一篇特徵匹配壓力測試、一篇 Galois 群數值計算。沒有一篇把停止條件推到引導式取樣或點雲配準(`source-exists`,只看標題;2606.27385 開了全文,其中 stopping 只出現在參考文獻與「varied termination criteria」一句,`quote-verified`)。
2. `search_works`(語意搜尋,2024 年起,"stopping criterion guarantee for RANSAC with guided non-uniform sampling in outlier-robust point cloud registration"):前 12 筆是 C-RANSAC(鍾)、"RANSAC Back to SOTA" 兩階段共識過濾、GOR-PCR、SPROSAC、PCR-99 等,全部是新的剔除或取樣方法;唯一談停止條件的就是上面那篇(`source-exists`,只看摘要)。
3. arXiv(`all:RANSAC AND ("stopping criterion" OR "termination criterion" OR "early termination")`,依投稿日排序):只回 5 筆,最新兩筆是鍾老師的 2604.26318 與 2503.07829,其餘是 BANSAC(2023)、MAGSAC(2018)、ANSAC(2017)(`source-exists`)。
- 結論:查到的範圍內缺口還在。保留一點:BANSAC 已提出一個用更新後機率的停止條件,但它是啟發式的(見下方引用),沒有說明實際達到的成功機率。

### 定義這個問題的論文
- Schönberger, Larsson, Pollefeys, "Fixing the RANSAC Stopping Criterion", arXiv 2503.07829(全文)。原文:「In our theoretical analysis, we ignore commonly known other factors, such as measurement noise, minimal solver stability, adaptive sampling, etc., which can lead to a discrepancy between the desired and actually achieved target success probability.」以及「Combining the different factors impacting the achieved target success probability into a unified framework is an important problem to be addressed.」(`quote-verified`)
- Chung, Lin, Chen, "Point Cloud Registration via Probabilistic Self-Update Local Correspondence and Line Vector Sets", arXiv 2604.26318(全文)。原文:「In the previous RANSAC interaction model, three termination conditions are employed ... However, it still cannot reach a good trade-off between accuracy and runtime across varying inlier rates.」文中的全域信心水準寫成 Cl^glo = 1-(1-|Ir^glo|)^(t^glo),門檻 99.5%,而新加的全域提早終止條件是一個由實驗挑出的最大互動輪數 R_max(`quote-verified`;公式是從 HTML 轉出的文字讀到的,上標可能有遺漏,要對照 PDF)。
- "BANSAC: A dynamic BAyesian Network for adaptive SAmple Consensus", arXiv 2309.08690(全文;作者姓名我沒有核對,故不列)。原文:「We exit the loop because there is a low chance of getting a better solution.」(`quote-verified`)——這是啟發式停止,沒有機率保證。

### 需要的資源(資料集、模擬器、硬體;拿不拿得到)
- 3DMatch、KITTI 的下載網址寫在 2604.26318 內文(`quote-verified`);WHU-TLS 的網址我沒看到完整字串(`source-exists`)。
- 程式碼:2604.26318 寫明 C++ 原始碼公開在 GitHub(ivpml84079 帳號,`quote-verified`);BANSAC 程式碼公開(摘要,`quote-verified`);2503.07829 列出 OpenCV 與 scikit-image 中受影響的程式行(`quote-verified`)。
- 硬體:一般 CPU 即可(`model-only`)。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:(1) 寫出「取樣集合為全集的子集、子集內點率未知」時的精確或保守停止機率;(2) 在合成資料上畫出宣稱 vs. 實際成功率的落差曲線;(3) 把修正後的規則換進鍾老師公開的演算法,報告三個資料集上成功率與時間的變化。
- 第二篇:把量測雜訊、局部最佳化、自我更新集合的相依性放進同一個框架(這正是 2503.07829 點名的未解問題)。
- 第三篇:由停止時的統計量給出「這次配準可能失敗」的有保證警示,接到 SLAM 的迴環偵測(鍾老師 2026 年另有一篇 loop closure detection 論文,`source-exists`,只看標題)。

### 最可能失敗的原因(一項,直說)
自我更新集合讓各輪抽樣彼此相依,精確機率很可能寫不出來,最後只剩一個偏保守、讓執行時間變長的上界;如果在三個資料集上成功率又沒有明顯改善,論文就只剩「指出問題」而沒有「解決問題」。

### 台科大資工系相鄰的教授(附依據)
- 鍾國亮(講座教授)。系上名冊:「深度學習理論與應用、影像處理與視訊壓縮、彩色影像處理與應用」(`quote-verified`,ntust-csie-faculty.md)。實驗室網站檔只抓到框架頁,內容等於空的(直說)。OpenAlex 作者檔 A5002873579 顯示 2024-2026 的論文是 C-RANSAC 點雲配準(JSTARS 2024)、"Feature Matching via Self-Adjusting Reliable Correspondence Set and Early Termination"(JSTARS 2025)、2604.26318(`source-exists`)。也就是說名冊寫的是壓縮與彩色影像,但他最近實際在做的正是這一題的對象。

---

## framegen-error-budget: 畫格外插的可預測誤差與重繪決策

### 一句話
外插畫格的誤差有多少是幾何上注定補不回來的(新露出的區域、變化中的陰影),能不能在產生畫格之前就由深度、運動向量與著色梯度估出來,並用它決定每個區塊該外插、重新著色還是整張重繪?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:遮擋解除面積是「深度不連續 × 相對運動 × 外插步數」的幾何量;著色變化可用時間梯度的一階外推估計。兩者合起來是每個區塊的誤差預算,決策問題是在固定運算預算下最小化預測誤差(`model-only`,這是我提出的建模方式,沒有查到現成論文這樣寫)。
- 驗證:在公開繪圖框架裡同時算外插畫格與真值畫格,比較預測誤差與實測誤差。Amulet 就是這樣做的:實作在 Falcor,場景是 Bistro Interior、Bistro Exterior、Intel Sponza、San Miguel,誤差用 FLIP(`quote-verified`,arXiv 2608.10423 全文)。GFFE 用 PSNR、SSIM、LPIPS、FovVideoVDP(`quote-verified`)。
- 誠實說明:只要論文想主張「玩家感覺比較好」或「延遲換瑕疵划算」,就需要受試者、高更新率螢幕或頭盔的使用者實驗;碩論最小版本可以只用影像誤差指標,不做受試者實驗。

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(引用 GFFE,W4404526111,依日期):20 筆。最新是 SIGGRAPH Asia 2026 的 "Tile-based Frame Extrapolation" 與 "FreeME: Target-Buffer-Free Mobile Extrapolation for Rendered Frames",以及 Amulet、"Consecutive Frame Extrapolation with Predictive Sparse Shading"(TOG 2025)、MoFlow(TOG 2025)、"Decoupled Motion Prediction for Real-time G-buffer Free Frame Extrapolation"(ACM MM 2025)。全部是新方法(`source-exists`,只看標題)。
2. `search_works`(2025 年起,"frame extrapolation" 或 "frame generation" 加 disocclusion / latency):33 筆,除上述外還有 X4Rate(4 倍外插)、一篇 UCSB 博士論文、一篇 Zenodo 上的低階硬體延遲實測資料。沒有看到解析誤差模型或誤差下界的標題(`source-exists`)。
3. arXiv(`all:"frame extrapolation"`,依投稿日):12 筆,繪圖相關的只有 Amulet(2026-08)、ReFrame(2025-06)、PatchEX(2024-07)、GFFE(2024-05)(`source-exists`)。加上「延遲 + 感知 / 使用者實驗」的第二次查詢被 arXiv 回覆 Rate exceeded,沒有結果。
- 結論:缺口無法完全判斷。方法競賽非常擠;「預測誤差再決策」這個想法,Amulet 已經用啟發式做了一部分(見下方引用),"Predictive Sparse Shading" 從標題看也是,但後者不開放取用,我沒讀到內文。

### 定義這個問題的論文
- Wu 等, "GFFE: G-buffer Free Frame Extrapolation for Low-latency Real-time Rendering", ACM TOG 2024, DOI 10.1145/3687923, arXiv 2406.18551(全文)。原文:「However, it fails when the disocclusion areas have never been shown before and are not the out-of-screen areas」;以及內插的代價:「the latency of interpolation methods is increased by at least one rendering time interval」(`quote-verified`)。
- Künzel 等, "Amulet: Frame Extrapolation Through Sparse Layered Scene Representation and Adaptive Shading", arXiv 2608.10423(全文)。摘要:「Using a predictive, gradient-based scheduler that assigns lifetimes for each tile, we enable adaptive shading updates under motion and dynamic lighting.」限制:「The primary limitation of Amulet is highly dynamic scenes involving non-rigid transformations ... In the worst case, every tile may require per-frame updates.」(`quote-verified`)
- Wu 等, "Consecutive Frame Extrapolation with Predictive Sparse Shading", ACM TOG 2025, DOI 10.1145/3763363(只有索引紀錄,`source-exists`)。

### 需要的資源(資料集、模擬器、硬體;拿不拿得到)
- Falcor 與 Bistro、Sponza、San Miguel 是公開的(我只在 Amulet 內文看到它們被使用,`quote-verified`;授權條款與下載方式沒查,`model-only`)。
- GFFE 的資料是作者自己在 Unreal Engine 擷取的 8 個場景(`quote-verified`),是否公開沒查到。
- 硬體:一張能跑延遲著色的桌機顯示卡(`model-only`)。感知實驗另需高更新率螢幕與受試者。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:只做相機運動加剛體運動的靜態光照場景;推導遮擋解除面積的預測式,在四個公開場景上報告預測與實測的相關性,再用它做「外插 / 重繪」二選一的決策,和固定比例外插比較。
- 第二篇:加入著色變化(陰影邊界、反射),處理 Amulet 自己承認做不好的非剛體動畫。
- 第三篇:延遲與瑕疵的感知交換模型,用受試者實驗擬合;或搬到 VR 雙眼畫格生成(StereoFG,SIGGRAPH Asia 2025,`source-exists`,只看標題)。

### 最可能失敗的原因(一項,直說)
現有的梯度式排程器在實務上已經夠好,解析模型只能解釋「為什麼有效」而拿不出更好的畫質或速度;這個領域的審稿人看的是畫質與毫秒數,純解釋性的結果很難發表。

### 台科大資工系相鄰的教授(附依據)
- 蕭凱文(助理教授)。實驗室網站檔的 Projects 頁:「Game Frame Interpolation is a technique used to increase the frame rate of rendered or recorded game footage by generating intermediate frames between existing ones.」(`quote-verified`)。另有與姚智原合著的 "Aliasing Detection in Rendered Images via a Multi-Task Learning"(PACMCGIT 2024,`quote-verified` 於網站檔,OpenAlex 也有紀錄)。
- 賴祐吉(教授)。名冊:「電腦遊戲技術、電腦圖學、電腦影像生成、電腦動畫、三D模型重建、電腦視覺等」(`quote-verified`)。實驗室網站檔是空的(0 bytes)。OpenAlex 顯示 2026 年有 "Real–time appearance–driven memory–efficient dense canopy synthesis"(與戴文凱合著)與 "Fidelity-preserved Visual Proxy Construction using Architectural Semantics"(CGF)(`source-exists`),屬即時繪圖,但不是畫格生成。

---

## hmd-pose-calibration: 三點追蹤全身姿態估計的不確定度校準

### 一句話
頭盔加雙手控制器的全身姿態估計模型輸出的關節不確定度,和實際誤差對得上嗎;當動作分佈從訓練用的日常動作換成運動動作時還對得上嗎;能不能對下半身關節誤差給出有覆蓋率保證的區間?

### 為什麼不會太薄(模型是什麼、驗證用什麼)
- 模型:異質變異數高斯迴歸(每個關節輸出平均與變異數)的校準定義,加上 conformal prediction(保形預測,用留出資料把區間修到指定覆蓋率)。難點有實質內容:動作序列的各畫格不獨立,而且測試動作與校準動作分佈不同,標準 conformal 的可交換性假設不成立(`model-only`)。
- 驗證:AMASS 訓練,EgoBody 與 GIMO 測試,這是 EnvPoser 的設定(`quote-verified`:「Training Stage I on the AMASS dataset」「two challenging public motion-scene interactions datasets: Egobody [51] and GIMO [56]」)。
- 誠實說明:數學與資料集驗證不需要受試者。但若要主張「顯示不確定度的虛擬化身讓使用者體驗更好」或「運動訓練回饋更可信」,就需要受試者與頭盔的使用者實驗,碩論最小版本不含這部分。
- 和現有候選 `eew-conformal` 用的是同一類數學工具(conformal),領域不同;兩題不要同時做。

### 缺口還在嗎(三項檢查各自的結果)
1. `list_citations`(引用 EnvPoser,W4413144921,依日期):16 筆。最新是 ICIG 2026 "A Lightweight Real-Time Correction System for Consumer VR Full-Body Tracking"、ACM MM 2026 "Aegis"、ECCV 2026 "ECHO: Ego-Centric Modeling of Human-Object Interactions"、WACV 2026 "Mem-MLP"、arXiv 2602.23618 "Egocentric Visibility-Aware Human Pose Estimation",其餘是毫米波與 LiDAR 動作擷取。標題中沒有校準或覆蓋率保證(`source-exists`,只看標題)。
2. `search_works`(語意搜尋,2024 年起,"uncertainty estimation of full-body pose from sparse head-mounted display and controller tracking signals"):前 12 筆是 EgoPoser、MobilePoser、Ultra Inertial Poser、EgoAllo、HMD-Poser、EnvPoser、S2Fusion、DivaTrack、DTP 等,摘要都在講準確度與一對多歧義,只有 EnvPoser 的摘要提到不確定度建模(`source-exists`,只看摘要)。
3. arXiv:查詢兩次,第一次回傳空白、第二次回覆 Rate exceeded。這一步沒有完成。
- 結論:缺口無法完全判斷。已查到的範圍內沒有人評估校準,但最新預印本沒查到;此外 EnvPoser 的 16 篇引用文獻我只看了標題。

### 定義這個問題的論文
- Xia 等, "EnvPoser: Environment-aware Realistic Human Motion Estimation from Sparse Observations with Uncertainty Modeling", CVPR 2025, arXiv 2412.10235(全文)。原文:「the sparsity of the input data—limited to tracking signals from the head and hands—introduces high uncertainty in estimating unobserved joints, particularly in the lower body.」它用一個不確定度迴歸頭輸出每個關節的 δ,再以 N(θ, δ) 重新取樣(`quote-verified`)。全文搜尋 "calibrat" 沒有任何結果,也就是它沒有檢查 δ 是否校準(`quote-verified`,以全文字串搜尋為據)。
- Jiang 等, "EgoPoser: Robust Real-Time Egocentric Pose Estimation from Sparse and Intermittent Observations Everywhere", ECCV 2024, arXiv 2308.06493(只有索引紀錄,`source-exists`)。
- Dai 等, "HMD-Poser: On-Device Real-time Human Motion Tracking from Scalable Sparse Observations", CVPR 2024, arXiv 2403.03561(只讀摘要,`source-exists`)。
- Yang 等, "DivaTrack: Diverse Bodies and Motions from Acceleration-Enhanced Three-Point Trackers", CGF 2024, DOI 10.1111/cgf.15057。摘要:「Because it is a highly under-constrained problem, inferring full-body pose from these inputs is challenging」(`source-exists`,只讀摘要)。

### 需要的資源(資料集、模擬器、硬體;拿不拿得到)
- AMASS、EgoBody、GIMO:在 EnvPoser 內文中被稱為公開資料集(`quote-verified`);實際申請流程與授權我沒查(`model-only`:這類資料集通常要填學術使用同意書)。
- 運動動作的公開動作擷取資料:沒查,這是「分佈偏移」那一半能不能做的關鍵,要先確認。
- 一張訓練用 GPU(`model-only`)。EnvPoser 等方法的程式碼是否公開沒查。

### 碩論的最小版本 / 延展到博論的路線
- 碩論:重現一個有不確定度輸出的三點追蹤模型;畫出各關節的可靠度圖(reliability diagram);加上適用於時間序列的 conformal 修正,報告同分佈與跨資料集兩種情況下的實際覆蓋率與區間寬度。
- 第二篇:分佈偏移下的保證(日常動作 → 運動動作),或用不確定度決定何時值得多戴一顆 IMU(HMD-Poser 支援頭盔加 2 或 3 顆 IMU 的輸入,摘要,`source-exists`)。
- 第三篇:把校準過的不確定度放進 VR 運動訓練系統,做受試者實驗(需要頭盔與參與者)。

### 最可能失敗的原因(一項,直說)
結果只是「不校準 → 套 conformal → 校準了」,沒有新的理論內容;審稿人會說這是把現成工具套到又一個任務。要避免就得在時間相依或分佈偏移上有真的推導,這部分我沒有把握做得出來。

### 台科大資工系相鄰的教授(附依據)
- 潘則佑(合聘教師,MIIS Lab)。名冊:「多媒體內容分析、電腦視覺、人機互動、延展實境」(`quote-verified`)。實驗室網站檔的著作清單有「Exploring VR Headset-Based Motion Capture for Supporting Self-Reflection with Motion Replay and LLM-Assisted A...」(標題在檔內被截斷)以及 VR 籃球、網球發球、標槍投擲訓練系統,和 IEEE VR 2026 full paper(`quote-verified`)。注意:名冊寫的是合聘,能不能當資工系碩士生的主指導教授要問系辦。
- 姚智原(副教授)。名冊:「電腦圖學、電腦視覺、延展實境、人機互動、醫療影像3D重建」(`quote-verified`)。網站檔的著作只列到 2019 年;OpenAlex 顯示 2026 年有 "Real-Time Sign Language Interpretation via Customized Sign Language Gloves and Motion Retrieval"(Sensors)以及與潘則佑合著的 VR 彈力回饋論文(AHs 2026)(`source-exists`)。

---

## 沒有採用的方向

- 3D Gaussian Splatting 的不確定度校準與 conformal 保證:arXiv 已有 "QuantileGS: Quantile-Based Conformal Bounds on RGB Rendering Errors for Novel View Synthesis with 3D Gaussian Splatting"(2611.21253,2026-11-16)與 "View-Structured Conformal Prediction for 3D Gaussian Splatting"(2609.10307,2026-09-09),這個問法已被做掉(`source-exists`,只看標題)。
- 3D Gaussian Splatting 的細節層次(level of detail)加螢幕空間誤差上界:OpenAlex 找到相鄰的 "On the Error Analysis of 3D Gaussian Splatting and an Optimal Projection Strategy"(ECCV 2024)與 "Learning Fast 3D Gaussian Splatting Rendering using Continuous Level of Detail"(CGF 2025),但 arXiv 查詢被限流,缺口沒辦法判斷,所以不列為候選(`source-exists`)。
- 影像與視訊壓縮的 rate-distortion 模型(名冊上鍾國亮的專長):OpenAlex 顯示他 2023 年以後的 8 篇論文是點雲配準、色彩校正、特徵匹配、迴環偵測,沒有一篇是視訊壓縮(`source-exists`),所以改選他現在實際在做的方向。
- 深偽偵測、拖曳式影像編輯(花凱龍實驗室 2024-2026 的主力,`quote-verified` 於網站檔):屬於比基準數字的題目,我沒找到可以寫成數學機制的切入點(`model-only`),也沒有另外做缺口檢查。
- 暈動症(cybersickness)與重定向行走(redirected walking)的感知門檻模型:驗證一定要受試者與頭盔,一年內含倫理審查風險高(`model-only`,沒有查文獻)。
- 程序化內容生成、遊戲 AI(戴文凱):網站檔的著作清單太長沒有細讀,這次沒有從中找候選;這是沒查,不是判定不可行。

## 來源

| id | 標題 | 開到哪個程度 |
|---|---|---|
| arXiv 2503.07829 | Fixing the RANSAC Stopping Criterion | 全文 |
| arXiv 2604.26318 | Point Cloud Registration via Probabilistic Self-Update Local Correspondence and Line Vector Sets | 全文(HTML 轉文字) |
| arXiv 2309.08690 | BANSAC: A dynamic BAyesian Network for adaptive SAmple Consensus | 全文(讀了摘要與 4.4 節) |
| arXiv 2606.27385 | RANSAC Scoring Done Right | 全文(只搜尋關鍵字) |
| DOI 10.1109/jstars.2024.3365516 | Centralized RANSAC-Based Point Cloud Registration With Fast Convergence and High Accuracy | 摘要 |
| DOI 10.1109/jstars.2025.3638412 | Feature Matching via Self-Adjusting Reliable Correspondence Set and Early Termination | 索引 |
| DOI 10.1109/jas.2026.125762 | Dual Codebook- and Scene Change-Based Loop Closure Detection Using SVM Outlier Removal in Verification | 索引 |
| DOI 10.1109/lra.2024.3502056 | RANSAC Back to SOTA: A Two-Stage Consensus Filtering for Real-Time 3D Registration | 摘要 |
| arXiv 2406.18551 / DOI 10.1145/3687923 | GFFE: G-buffer Free Frame Extrapolation for Low-latency Real-time Rendering | 全文 |
| arXiv 2608.10423 | Amulet: Frame Extrapolation Through Sparse Layered Scene Representation and Adaptive Shading | 全文 |
| DOI 10.1145/3763363 | Consecutive Frame Extrapolation with Predictive Sparse Shading | 索引 |
| DOI 10.1145/3829340.3842212 | Tile-based Frame Extrapolation | 索引 |
| DOI 10.1145/3829340.3842195 | FreeME: Target-Buffer-Free Mobile Extrapolation for Rendered Frames | 索引 |
| DOI 10.1145/3730400 | MoFlow: Motion-Guided Flows for Recurrent Rendered Frame Prediction | 索引 |
| DOI 10.1145/3757377.3763965 | StereoFG: Generating Stereo Frames from Centered Feature Stream | 索引 |
| arXiv 2412.10235 | EnvPoser: Environment-aware Realistic Human Motion Estimation from Sparse Observations with Uncertainty Modeling | 全文 |
| arXiv 2308.06493 | EgoPoser: Robust Real-Time Egocentric Pose Estimation from Sparse and Intermittent Observations Everywhere | 索引 |
| arXiv 2403.03561 | HMD-Poser: On-Device Real-time Human Motion Tracking from Scalable Sparse Observations | 摘要 |
| DOI 10.1111/cgf.15057 | DivaTrack: Diverse Bodies and Motions from Acceleration-Enhanced Three-Point Trackers | 摘要 |
| arXiv 2611.21253 | QuantileGS: Quantile-Based Conformal Bounds on RGB Rendering Errors for Novel View Synthesis with 3D Gaussian Splatting | 索引(標題) |
| arXiv 2609.10307 | View-Structured Conformal Prediction for 3D Gaussian Splatting | 索引(標題) |
| DOI 10.1007/978-3-031-72643-9_15 | On the Error Analysis of 3D Gaussian Splatting and an Optimal Projection Strategy | 索引 |
| DOI 10.1111/cgf.70069 | Learning Fast 3D Gaussian Splatting Rendering using Continuous Level of Detail | 索引 |
| DOI 10.1145/3675379 | Aliasing Detection in Rendered Images via a Multi-Task Learning | 索引 |
| DOI 10.1007/s11042-026-21807-4 | Real–time appearance–driven memory–efficient dense canopy synthesis | 索引 |
| DOI 10.1111/cgf.70608 | Fidelity-preserved Visual Proxy Construction using Architectural Semantics | 索引 |
| DOI 10.3390/s26154884 | Real-Time Sign Language Interpretation via Customized Sign Language Gloves and Motion Retrieval | 索引 |
| DOI 10.1145/3795011.3797350 | Exploring Elastic Force Feedback for Enhanced Physical Interaction in Virtual Reality Sports Experiences | 索引 |
| 本地檔 | topics/universe/ntust-csie-faculty.md;papers/_ntust/labs/ 下的 鍾國亮、花凱龍、戴文凱、賴祐吉、姚智原、蕭凱文、潘則佑 | 鍾國亮只有框架頁;賴祐吉 0 bytes;戴文凱只讀首頁部分;其餘讀了著作與專案清單 |

沒有使用 Consensus。arXiv API 在本次查證中多次回覆限流(429 / Rate exceeded),受影響的查詢已在各題中標明。
