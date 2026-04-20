# 論文閱讀筆記

## 1. 論文資訊
* **題目**：Enabling Programmable Inference and ISAC at the 6GR Edge with dApps  
* **作者**：Michele Polese, Rajeev Gangula, Tommaso Melodia
* **單位**：Institute for Intelligent Networked Systems, Northeastern University, Boston, MA, U.S.A 
* **來源（IEEE 格式）**：M. Polese, R. Gangula, and T. Melodia,
“Enabling Programmable Inference and ISAC at the 6GR Edge with dApps,”
[arXiv preprint arXiv:2603.29146](https://arxiv.org/abs/2603.29146), 2026.  

---

## 2. 問題定義（Introduction）

### 論文要解決什麼問題？

**答：**  
解決「如何在現有 O-RAN 架構下，實現可程式化（programmable）的推論與 ISAC 系統」的問題。

1. **動態生命週期管理問題**  
   現有系統無法支援推論與 ISAC 演算法的動態部署與調整，  
   而這些需求會隨時間與基地台位置改變。

2. **AI 驅動流水線不足**  
   缺乏支援 AI 驅動 ISAC 所需的完整資料流程，  
   包含資料收集、訓練、測試與部署。

3. **資源配置缺乏彈性**  
   現有系統無法在「通訊、感測、推論」三者之間  
   進行即時且動態的資源分配。

---

### 為什麼這個問題重要？

1. **開拓新營收來源**  
    6G 網路的設計背景與 5G 不同，營運商希望透過提供超越單純連網的服務  
   （如環境感測與 AI 推理）來開啟新的收入流。

2. **經濟效益**  
   ISAC 技術允許通訊與感測共享相同的空口（air interface）、頻譜與基礎  
   設施，具有顯著的經濟優勢。

3. **網路優化**  
    透過 AI 推論識別 RAN 數據中的結構與脈絡，可用於網路優化及增值服務。

---

### 為什麼這個問題困難？

**答：**  
傳統架構缺乏以下關鍵能力，導致作者認為在舊架構上實現即時感測是「在架構上不可行的（architecturally infeasible）」：

1. **即時實體層數據存取（Real-time PHY/MAC data access）:**  
   感測演算法需要直接存取 I/Q 樣本、通道狀態資訊（CSI）和探測參考信號  
   （SRS）。然而現有的 O-RAN 介面（如 E2）在頻寬與延遲上無法滿足感測  
   所需的時間尺度。

2. **缺乏 AI 研發流水線（AI pipelines））:**  
   AI 驅動的 ISAC 需要端到端的 MLOps（數據收集、訓練、驗證與  
   部署），但目前的 RAN 生態系統尚未在開放且標準化的框架中發展出這些能力。

3. **感測增強型無線電（ISAC-enhanced radios）:**  
   不同的感測拓撲（如單站、雙站）對無線電單元（RU）有不同要求  
   （如全雙工、同步、波束成形），架構必須能支援異質且可更換的 RU。

4. **演算法生命週期管理（Algorithm life cycle）：**  
  感測應用隨站點（如鄉村 vs. 室內）與時間而異，架構必須能支援像  
xApps/rApps 一樣動態部署與更換演算法

5. **動態堆疊與資源配置（Dynamic stack configuration）：**  
   營運商需要在通訊與感測任務之間動態分配資源。由於感測與通訊共  
   享硬體加速器（如 GPU），必須防止感測推論影響到即時通訊的運作。

---

## 3. 現有方法（Existing Methods）

目前針對通訊感測一體化（ISAC）與推論的解決方法主要集中在 3GPP 標準化、現有的 O-RAN 架構以及 5G NR 定位機制，但它們各自存在顯著的缺點：

---

### 3GPP 標準化方法

**現狀：**  
3GPP 已在 Release 19 定義 ISAC 使用場景與需求，  
並於 Release 20 開始研究相關架構與資料暴露 API。

**缺點：**
 目前的研究重心仍過度集中在波形設計（Waveform design）、通道  
 模型以及核心網整合，對於 RAN 層級的即時數據存取、多節點感測  
 協調以及演算法的生命週期管理仍缺乏深入探討。
 
---

### O-RAN 的控制迴路與介面 (xApps/rApps)

**現狀：**  
透過 Near-RT RIC（xApps）進行近即時控制，  
以及 Non-RT RIC（rApps）進行策略管理與優化。

**缺點：**

1. **E2 介面限制**  
   現有的 E2 介面是為「控制面遙測」設計的，  
   不支援傳輸感測所需的用戶面 I/Q 數據流。

2. **效能不足（延遲與頻寬）**  
   針對感測所需的時間尺度（次毫秒級），E2 介面  
   在頻寬與延遲上均無法滿足需求，尤其在需要匯  
   聚多站點數據時更顯吃力。

3. **缺乏動態部署能力**  
   缺乏針對不同站點需求進行動態演算法配置與更  
   新的能力。

---

### 現有 5G NR 定位機制

**現狀：**  
基地台（gNB）從 UE 訊號中提取定時測量值（如 UL-RTOA），  
並回報給核心網中的定位功能（LMF）。

**缺點：**

1. **數據嚴重遺失**  
   在實體層（PHY）處理邊界，會丟棄包含振幅、相位與多路徑  
   結構的完整通道衝擊響應（CIR）數據，僅保留標量結果。

2. **感測精度受限**  
   這種「標量報告」方式無法進行跨觀察點的信號子空間處理，  
   導致在多路徑環境或觀察次數較少時，感測精度大幅下降甚至失效。

---

### 現有學術研究方法

**現狀：**  
已有研究探討在蜂巢式網路中整合感測功能。

**缺點：**
 大多侷限於單節點拓撲或特定的波形設計，對於如何建立  
 端到端的 **AI 流水線（MLOps）**以及如何在共享硬體  
 基礎設施上動態平衡通訊與感測資源，仍缺乏完整的系統級解決方案。

---

## 4. 系統模型（System Model）

#### 1. 問題的系統化定義（Problem Formulation）

在傳統 5G 與 O-RAN 架構中，RAN 的資料處理流程為：

UE → gNB → PHY Processing → Feature Extraction → Core Network


然而，此架構存在以下關鍵限制：

- 原始實體層數據（I/Q samples、CSI）在 PHY 層即被丟棄  
- 僅保留低維度標量特徵（如 RTOA）  
- 無法支援高精度感測（ISAC）與 AI 推論  

> **核心問題：**  
> 現有 RAN 架構缺乏對「原始實體層數據」的即時存取能力，  
> 導致無法實現可程式化的感測與推論功能  

---

#### 2. 系統輸入與輸出（Inputs / Outputs）

#### (1) Inputs

- **PHY/MAC 數據**
  - I/Q samples  
  - CSI  
  - SRS  

- **Operator Intent**
  - 無人機偵測、環境感測等  

- **Network Telemetry**
  - RU/DU 設定  
  - GPU/NPU 使用狀況  

- **Training Data**
  - AI 模型訓練資料  

---

#### (2) Outputs

- **Sensing Results**
  - 目標偵測  
  - 距離／速度估測  

- **Inference Results**
  - 物體分類  
  - 追蹤  

- **Network Optimization**
  - Beam selection  
  - 資源配置  

- **Model Outputs**
  - 訓練完成模型  
  - 部署策略  

---

#### 3. 提出之系統架構（Proposed Architecture）

為解決上述問題，論文提出一個分層式架構：   

              ┌──────────────┐
              │   SMO Layer  │
              │ (Orchestration) │
              └──────────────┘
                     ↑
              ┌──────────────┐
              │   RIC Layer  │
              │ (xApps/rApps)│
              └──────────────┘
                      ↑  
    UE → gNB → DU → dApps (Edge Layer)  
    ↑
    E3 Interface


---

#### 4. 核心設計（Key Components）

#### (1) E3 Interface

- 提供高頻寬（Gbps）資料傳輸  
- 支援原始 I/Q、CSI 存取  
- 解決 E2 無法傳輸感測數據的問題  

---

#### (2) dApps（Edge Layer）

- 部署於 DU 附近  
- 執行即時（sub-ms）推論與感測  
- 支援可程式化（programmable）部署  

---

#### (3) Hierarchical Control

- **Edge（dApps）**：即時推論  
- **RIC**：多節點融合  
- **SMO**：模型與服務管理  

---

#### 5. 系統流程（Workflow）

UE Signal
→ RAN (I/Q, CSI)
→ E3 Interface
→ dApps (Real-time Inference)
→ RIC (Fusion)
→ SMO (Model Management)
→ Outputs


---

#### 6. 系統目標（Objectives）

- **Low Latency**：sub-ms inference  
- **High Accuracy**：完整訊號資訊  
- **Programmability**：動態部署  
- **Resource Efficiency**：共享資源  

---

#### 7. 小結（Key Insight）

> 本論文透過引入 E3 介面與 edge dApps，  
> 使系統能直接存取原始實體層數據，  
> 並在 RAN 中實現可程式化的感測與 AI 推論。

---

## 5. 提出方法（Proposed Method）與效能驗證（Evaluation）

### 作者提出什麼概念解決此問題？

為了解決現有 O-RAN 架構在「即時數據存取」與「動態管理」上的不足，  
作者提出一套以 **dApps 與 E3 介面為核心的層次化 ISAC 架構**：

1. **dApps（分散式應用）**  
   部署於 DU（Distributed Unit）旁的即時處理模組，可在 **亞毫秒  
   （sub-ms）時間尺度**下直接存取實體層數據，用於即時推論與感測。

2. **E3 介面（新介面設計）**  
   專門用於傳輸高頻寬原始數據（I/Q、CSI、SRS），使 dApps 能直接  
   取得完整訊號資訊，解決現有 E2 介面無法支援感測的問題。

3. **層次化架構（Hierarchical Architecture）**

   - **邊緣層（dApps）**  
     執行即時感測與 AI 推論（低延遲）

   - **控制層（xApps / rApps）**  
     負責多節點資料融合與網路協調

   - **編排層（ISAC Orchestrator / SMO）**  
     於 SMO 中，負責模型生命週期管理（訓練、部署、更新），
     包含集中式訓練與意圖驅動的模型部署。

4. **動態堆疊配置（Dynamic Stack Configuration）**  
   在 O-Cloud 上透過容器化或硬體分區，動態分配資源給  
   通訊與感測任務，避免互相干擾。

---

### 用哪些效能指標證明問題已被解決？

作者透過 **理論分析（CRLB） + 實驗驗證（5G Testbed）**，  
使用以下指標證明系統有效性：

---

#### 1. 範圍與速度誤差（Range & Velocity RMSE）

* 利用 **Cramér-Rao Lower Bound（CRLB）** 分析感測精度，  
  感測精度與數據傳輸量直接相關。  
* 結果顯示：  
  - 當 E3 提供約 3 Gbps 數據時  
  - 範圍誤差 < 0.2 m  
  - 速度誤差 ≈ 5 m/s  

👉 證明「高頻寬即時數據」對感測精度至關重要

---

#### 2. 範圍估計誤差分佈（Empirical CDF）

* 比較：
  - 傳統 5G 標量回報  
  - 基於 dApps 子空間估計方法  

* 結果：  
  - dApps 方法在 **20 次觀測下，超過 90% 的估計能達到亞米級 (sub-meter) 精度**  
  - 傳統方法幾乎失效  

👉 證明新架構在低觀測條件下仍具高準確度

---

#### 3. 感測數據開銷（Sensing Data-rate Overhead）

* 衡量 E3 介面所需承載流量（Mbps / Gbps）  
* 用於評估感測任務對通訊資源的影響  

👉 證明透過本地處理（edge processing）可有效管理高流量數據

---

#### 4. 偵測概率與虛警率（Detection Probability & False Alarm Rate）

* 用於評估感測系統的可靠性  
* 同時作為 **模型重新訓練與更新的觸發條件**

👉 確保系統長期穩定運作

---

### 總結

作者透過：

- **新架構（dApps + E3 + 分層設計）**
- **理論分析（CRLB）**
- **實驗驗證（5G 平台）**

證明其方法能：

✔ 提升感測精度  
✔ 降低延遲  
✔ 支援即時 AI 推論  
✔ 解決現有架構的系統限制  
---

## 6. 作者如何證明方法有效？

作者透過 **理論分析（CRLB）** 與 **實際 5G 測試平台實驗**，  
證明所提出的 dApps + E3 架構能有效解決：

- 即時數據存取問題  
- 感測精度不足問題  

---

### 1. 理論分析：單站感測（CRLB Analysis）

**測試場景：**  
- 目標：偵測無人機  
- 距離：500 m  
- 速度：25 m/s  

**分析重點：**  
針對「單站全雙工感測」(Monostatic dApp) 進行了 Cramér-Rao 下界 (CRLB) 分析，  
探討「數據傳輸量（E3 data rate）」與「感測精度（RMSE）」的關係  

**結果：**

- 數據在 300 Mbps → 範圍誤差約 **3.6 m**  
- 提升至 3 Gbps → 範圍誤差降至 **0.2 m 以下**

**結論：**

- 感測精度與數據頻寬呈正相關  
- 若沒有高頻寬即時數據（如 E3），無法達到高精度感測  
- 感測必須在 **邊緣（DU 附近）進行**，不能外移  

---

### 2. 實驗驗證：測距 dApp（5G Testbed）

在基於 OpenAirInterface (OAI) 的 5G 測試平台上實作了一個「測距 dApp」，並在  
充滿多路徑干擾的室內環境中進行測試。

**實驗平台：**  
- OpenAirInterface（OAI）5G 測試平台  
- 室內多路徑干擾的環境  

---

#### 比較方法

**(1) 傳統 5G 方法：**
- 僅回傳標量資訊，如峰值偵測結果（Peak detection）給核心網的 LMF
- 無完整訊號資訊  

**(2) dApp 方法：**
- 使用 E3 介面獲取完整的複數值通道衝擊響應 CIR（含振幅 + 相位）  
- 採用子空間估計演算法  

---

#### 實驗結果（CDF 分析）

- dApp 方法：  
  - 在 20 次觀測（M=20）下  
  - **超過 90% 的情況下達到亞米級（<1m）精度**

- 傳統方法：  
  - 在相同條件下幾乎完全失效  

---

#### 關鍵發現

1. **完整數據的重要性**  
   傳統方法丟棄相位與振幅資訊，  
   導致精度無法提升（不可逆資訊損失）

2. **增加觀測次數無法補救**  
   即使傳統方法增加到 M=60，效果仍有限  

---

### 3. 總結（Key Findings）

作者成功證明：

1. **E3 介面的必要性**  
   高精度感測需要 Gbps 等級即時數據  
   → 只有 E3 能提供  

2. **dApp 架構的優越性**  
   可在邊緣直接處理完整 PHY 數據  
   → 感測效能大幅提升  

3. **突破傳統架構限制**  
   傳統 E2 + 標量回報方法無法達成此精度  

---

### 最終結論

所提出方法能：

✔ 提供即時高頻寬數據存取  
✔ 顯著提升感測精度（sub-meter）  
✔ 支援可程式化 ISAC 系統  

→ 成功解決論文所提出的核心問題 

---

## 7. 未來展望及未解決問題

明確指出目前雖然提出了架構，但仍有幾個尚未完全解決的開放性  
挑戰（Open Challenges）。作者將這些問題歸類為技術、標準  
化、資源共存以及安全隱私四個面向：

1. **分佈式節點的同步問題（Technical Synchronization）：**  
   對於協作式感測（Collaborative sensing），如何讓分佈在各  
   處的感測節點達成精確同步以進行相干處理（Coherent processing）是一項關鍵技術挑戰。

2. **E3 介面的細節規範：**  
   雖然作者提出了 E3 介面的概念，但其數據格式、存取模式，以及  
   如何與通訊流水線達成物理隔離的具體規格仍需進一步設計。

3. **AI 與 RAN 的資源共存（AI-RAN Coexistence）：**  
   由於感測 dApps、AI 推理模型與通訊協議棧會競爭 O-Cloud 基礎設施上  
   的 GPU 週期、記憶體與 I/O 資源，如何在不降低通訊服務品質（QoS）的  
   前提下進行資源分配，仍是待解決的難題。

4. **多廠商互操作性與標準化：**  
   需要將 3GPP 的感測服務定義與 O-RAN 的可程式化能力（如 dApps, E3）進  
   行橋接，並制定標準化的感測數據表示法，以支援多廠商設備間的互通。

5. **隱私與安全（Privacy and Security）：**  
-隱私風險： ISAC 可能會透過被動感測（Passive sensing）無意中收集到個人資訊。  
-安全威脅： 透過 E3 介面暴露原始 I/Q 數據創造了新的攻擊面，必須建立嚴格的存  
  取控制與數據隔離機制，以防止惡意應用程式竊取敏感數據。

總結來說，作者雖然提出了解決現有 O-RAN 架構不足的方法，但認為系統資源的動態權  
衡、跨廠商標準化以及感測帶來的安全隱私疑慮，仍是未來 6G 發展中需要繼續攻克的課題。

## 7. 論文研讀

1. **如何研讀：**  
   丟至 **Notebooklm** 進行分析，並輸入想問的問題，例如:這篇文章提出的問題是甚麼、  
   這篇文章的解決方法，**Notebooklm**可以製作出講解影片跟心智圖供我快速抓住重點。  
   在整理出筆記後，筆記及論文連結丟入**Chatgpt**請它幫我檢查是否有正確。

2. **花多少時間：**  
   -主要問問題花半小時  
   -研讀**Notebooklm**提供的內容花一個半小時
   -筆記整理花兩小時以上  
   -檢查花一個半小時

3. **採用哪些AI工具來加速學習：**  
   -**Notebooklm**  
   -**Chatgpt**
   
## 8. 列出你可能有興趣的議題與可能解決方法

1. **有興趣的議題：**  
   -智能化網路控制 (AI for RAN)  
   -衛星協定設計、分析

2. **可能解決方法：**  
   -5G 測試平台:環境模擬  
   -Notebooklm:論文整理  
   -Github:參考可使用的開源資料  
