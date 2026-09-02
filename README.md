# edge-impulse-play

Keyword spotting ("hello world") built end-to-end on [Edge Impulse](https://edgeimpulse.com)'s free tier — open dataset → MFCC → 1D CNN → int8 → deployable C++ package. Zero code, zero hardware, ~90 minutes.

**[▶ Try it on your phone](https://smartphone.edgeimpulse.com/classifier.html?publicProjectId=523486)** — no account; runs locally in your browser (WASM).
**[Public project](https://studio.edgeimpulse.com/public/523486/latest)** — data, pipeline, model; cloneable.

## Results

| Metric | Value |
|---|---|
| Validation accuracy (int8) | 90.8% · AUC 0.97 |
| DSP cost (Cortex-M4F 80MHz) | 154 ms / 15 KB RAM per 1s window |
| Model size | 52 KB generated C++ (EON-compiled, no TFLite interpreter) |
| Known weakness | `noise`→`helloworld` 19.8%, from 4:1:1 window imbalance |

## Why Edge Impulse

- DSP preprocessing is a first-class pipeline stage, compiled into the C++ export with the model
- Latency / RAM / Flash per target MCU shown **at training time**
- One-click exports: C++ lib, Arduino, WASM, Linux `.eim`, Docker, firmware, raw TFLite
- TinyML in one line: hand-crafted DSP features do the heavy lifting so the NN stays KB-scale
- Free tier: 3 private projects, 60 min/job, 1,000 internal devices; no third-party distribution

## Layout

```
tinyml_demo.pdf                        full report (pipeline I/O, results, QR demo)
WALKTHROUGH.md                         step-by-step Studio guide (zh-TW, 30–45 min)
cjr96-project-1-cpp-mcu-v2-impulse-#1/ exported zero-dependency C++ inference package
```

## Dataset

Not included. Download the official [Keyword Spotting zip (~110 MB)](https://cdn.edgeimpulse.com/datasets/Audio+Classification+-+Keyword+Spotting.zip) ([docs](https://docs.edgeimpulse.com/docs/pre-built-datasets/keyword-spotting)), unzip into `data/` (git-ignored), then follow `WALKTHROUGH.md`.

- 1,474 samples / 34 min: `helloworld` · `noise` · `unknown`; `info.labels` auto-labels on upload
- License: public tutorial dataset; `noise`/`unknown` derive from [Google Speech Commands](https://arxiv.org/abs/1804.03209) (CC BY 4.0); no bundled LICENSE — verify before commercial use
- ⚠ 17 `testing/` files duplicate `training/`; Studio dedupes them, leaving the test set without `helloworld`. Move a few samples from training to test, then retrain.
