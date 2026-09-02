# Web 端完整 Pipeline 實作（30–45 分鐘）

前提：已登入 [studio.edgeimpulse.com](https://studio.edgeimpulse.com)。任務：keyword spotting——讓模型聽懂「hello world」，最後用**手機**即時推論收尾。

> 偷懶路線：打開[官方公開專案](https://studio.edgeimpulse.com/public/499022/latest)按右上 **Clone this project**，資料+設定+訓練好的模型全部複製過來，5 分鐘看完全貌。但要「體驗 pipeline」建議走下面完整版。

## Step 1 — 建專案（2 分鐘）

Studio 首頁 → **Create new project** → 名稱如 `kws-play`，選 Private（免費額度 3 個）。

## Step 2 — 上傳開源資料（5–10 分鐘）

左側 **Data acquisition** → 右上 **Upload data**（上傳圖示）：

- Upload mode: **Select a folder** → 選本機的 `D:\tmptask\edge-impulse-play\data\keyword-spotting`
- Upload into category: **Automatically split between training and testing**（zip 內含 `info.labels`，會自動認 label 與官方的 1,168/310 切分）
- 傳完在 Data acquisition 應看到約 34 分鐘音訊、三個 label：`helloworld` / `noise` / `unknown`，右上圓餅圖 train/test 約 80/20。可點任一筆播放看波形。

## Step 3 — Create impulse：定義 pipeline（3 分鐘）

左側 **Create impulse**。這頁就是它的 pipeline 抽象：`輸入視窗 → Processing block(DSP) → Learning block → 輸出`。

- Time series data：Window size **1000 ms**、Window increase **500 ms**（預設即可）
- **Add a processing block** → **Audio (MFCC)**（語音專用倒頻譜特徵；環境音則選 MFE）
- **Add a learning block** → **Classification**
- **Save impulse**

## Step 4 — 產生特徵（5 分鐘）

左側出現 **MFCC** → 參數用預設 → **Save parameters** → **Generate features**。

看兩個東西：

- **Feature explorer**：全部樣本降維散點圖。三色是否大致分群？`helloworld` 跟 `unknown` 有多少重疊，大概就是等下錯誤率的來源。
- **On-device performance**：這個 DSP 在目標 MCU 上的耗時/RAM 估計——一般 MLOps 完全沒有的資訊。

## Step 5 — 訓練（5–10 分鐘）

左側 **Classifier**：

- 預設：100 epochs、learning rate 0.005、一個小 CNN（可按 ⋮ 切 expert mode 看/改 Keras code）
- **Start training**。免費版單 job 上限 60 分鐘，這個資料集幾分鐘就完。

結果頁重點：

- **Confusion matrix** + accuracy（validation）；預期 90%+
- **Model version**：切 **Quantized (int8)** vs **Unoptimized (float32)** 比精度
- **On-device performance**：右上可換目標裝置（如 Cortex-M4/M7），即時顯示 inference time / RAM / Flash——這就是「訓練時就知道塞不塞得進 MCU」的體驗點

## Step 6 — 用測試集驗證（3 分鐘）

左側 **Model testing** → **Classify all**：跑 Step 2 保留的 310 筆 test set，拿到最終 accuracy 與逐筆結果。這一步等同一般流程的 hold-out evaluation，但零 code。

## Step 7 — 手機即時推論：不用硬體的閉環（5 分鐘）⭐

左側 **Deployment** → 搜尋選 **Mobile phone**（或 Dashboard 的 QR code）→ 手機掃 QR code → 瀏覽器開啟後給麥克風權限 → 對手機說「hello world」。

模型（int8 + WebAssembly）是在**手機瀏覽器本地**跑的，不是傳回雲端——這就是「edge」的意思，也是體驗上最有感的一步。

## Step 8 — 匯出 deliverables（5 分鐘）

回 **Deployment**，各選一次看產物：

1. **C++ library** → 勾 **EON Compiler**、選 **Quantized (int8)** → Build → 下載 zip。打開看：`edge-impulse-sdk/`（DSP+推論引擎）、`model-parameters/`、`tflite-model/`——這包丟給任何嵌入式工程師就能整合。
2. **WebAssembly** → 拿到瀏覽器可跑的 `.wasm + js`。
3. Dashboard 底部也能直接下載 **TFLite (int8/float32)** 模型檔，接回你自己的 Python 流程。

## 加碼（有興趣再玩）

- **EON Tuner**（左側）：AutoML，同時搜 DSP 參數+模型結構，以「目標裝置的 latency/RAM 上限」為約束——這是它相對一般 AutoML 最獨特的設計。
- **Versioning**：對整個專案（資料+DSP+模型）拍快照，類似 pipeline 級的 git tag。
- Dashboard → **Make this project public**：把整包 pipeline 發佈成可 clone 的公開連結（不佔私有額度）。

## 常見坑

- 上傳選「folder」而不是逐檔選，`info.labels` 才會被吃到；否則 label 會變成從檔名猜。
- 手機推論頁沒聲音反應：確認瀏覽器麥克風權限，且要用 https 的 QR 連結原頁。
- 免費版每專案 10 個 experiments：亂試超過會被擋，刪舊的即可。
