# Edge Impulse 快速體驗（edge-impulse-play）

2026-09-01 調查 + 實作導覽。目標：在 **web（Studio）** 上用開源資料走完一次完整 pipeline，理解最後拿到什麼、跟一般 MLOps 差在哪。

## 快問快答

### 可以用嗎？免費嗎？
**可以，免費。** 2025 年 Qualcomm 收購 Edge Impulse 後，反而把免費層升級成 **Developer Plan**（2025-05-06 起為預設免費方案，取代舊的 Community/Professional 分層），不用信用卡。額度：

| 項目 | Developer Plan（免費） |
|---|---|
| 私有專案 | 3 個 |
| 公開專案 | 無限 |
| 協作者 | 每專案 3 人 |
| Compute | 每個 job 上限 60 分鐘、16GB RAM |
| Experiments | 每專案 10 個 |
| 部署授權 | 內部研發/原型/展示 OK，內部生產最多 1,000 台裝置；**禁止對第三方外部發行產品** |
| 沒有的東西 | Organization 資料中心、完整 API 自動化、RBAC/SSO、部分進階 block |

對「試水溫玩一輪」完全夠用；商用出貨才需要 Enterprise。

### 最後會拿到什麼？（deliverables）
走完 pipeline 後，Deployment 頁一鍵匯出，這是它跟一般訓練流程最不一樣的地方——產出不是 model checkpoint，而是**可直接塞進裝置的推論套件**：

1. **C++ library**（零依賴，DSP 前處理 + 模型 + 推論 API 打包成一個 zip，任何 MCU/嵌入式都能編）
2. **Arduino library**（直接匯入 Arduino IDE 的範例專案）
3. **WebAssembly**（瀏覽器/Node.js 直接跑）
4. **Linux `.eim` binary / Docker container**（container 直接開一個 HTTP 推論 server）
5. **完整韌體 binary**（官方支援板直接燒錄）
6. **手機瀏覽器即時推論**（掃 QR code，手機就是推論裝置——不用任何硬體就能體驗閉環）
7. 也可下載原始 **TFLite（int8 量化 / float32）** 模型檔

外加兩個訓練頁就給你的東西：**每個目標 MCU 的 latency / RAM / Flash 估計值**，以及 **EON Compiler**（宣稱同精度下省 RAM/Flash 的模型編譯器）。

### 跟一般 MLOps 差在哪？

| | 一般 MLOps（ClearML/MLflow/W&B + serving） | Edge Impulse |
|---|---|---|
| 定位 | **水平工具鏈**：你自己組 tracking→registry→serving | **垂直一條龍 SaaS**：資料→DSP→訓練→量化→部署全在一個網頁 |
| 部署目標 | Server/GPU/雲端 API | **MCU、手機、瀏覽器、edge Linux**（KB 級 RAM 也行） |
| 前處理 | 自己寫，通常不進版控 | **DSP block 是 pipeline 一級公民**（MFCC/頻譜等），會跟模型一起編譯進 C++ 輸出 |
| 模型 | 任意大，自己管量化 | 內建小模型模板 + int8 量化 + EON Compiler，**訓練時就顯示目標硬體的 latency/RAM/Flash** |
| 寫 code 量 | 多 | 幾乎零（no-code；進階可 expert mode 改 Keras code） |
| 實驗管理 | 完整（任意 metric、任意框架） | 陽春（免費版 10 experiments + EON Tuner AutoML） |
| 類比 | 開放工具棒，彈性高、要自己接 | 像「TinyML 版的 Label Studio + AutoML + 交叉編譯器」合體 |

一句話：**一般 MLOps 管的是「模型的生命週期」，Edge Impulse 管的是「模型變成裝置韌體的生產線」**。它弱在實驗彈性與大模型，強在 DSP+量化+硬體資源估計這段一般工具鏈完全沒有的東西。

## 這個資料夾

- `data/keyword-spotting/` — 已下載並解壓的官方開源資料集（248MB、1,478 筆、34 分鐘音訊）：`helloworld` / `noise` / `unknown` 三類，Edge Impulse exporter 格式（含 `info.labels`，上傳時自動帶 label 與 train/test 切分）
- `WALKTHROUGH.md` — **照著做**：web 端 30–45 分鐘完整 pipeline 步驟

## Sources

- [Developer Plan 公告](https://www.edgeimpulse.com/blog/introducing-the-developer-plan/)、[Pricing](https://www.edgeimpulse.com/pricing)
- [Getting started for beginners](https://docs.edgeimpulse.com/knowledge/guides/getting-started-for-beginners)
- [Keyword spotting 資料集](https://docs.edgeimpulse.com/docs/pre-built-datasets/keyword-spotting)（[zip 直鏈](https://cdn.edgeimpulse.com/datasets/Audio+Classification+-+Keyword+Spotting.zip)）
- [官方公開示範專案（同資料集，可直接 clone）](https://studio.edgeimpulse.com/public/499022/latest)
- [公開專案機制](https://www.edgeimpulse.com/blog/public-projects-launch/)
