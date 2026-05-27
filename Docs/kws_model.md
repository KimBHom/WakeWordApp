🔊 WakeWordApp — KWS 模型架構說明
本文件說明 WakeWordApp 所使用的 喚醒詞偵測模型（KWS Model），包含資料流程、特徵工程、模型架構、訓練方式與推論邏輯。

1. 🎯 模型目標（Model Objective）
WakeWordApp 的模型目標是：

在 低延遲、低耗電 的情況下

持續偵測使用者自訂的喚醒詞

完全 離線運作

可由使用者 本地訓練 自己的喚醒詞模型

模型需能在手機 CPU 上即時運作（無需 GPU）。

2. 🔄 整體資料流程（Data Flow）
Code
麥克風 → 音訊前處理 → MFCC 特徵 → 模型推論 → KWS Engine → 觸發動作
(1) 音訊前處理（DSP Layer）
16kHz / mono

Pre-emphasis

Framing（25ms frame, 10ms hop）

Hamming window

Noise reduction（可選）

(2) 特徵工程（MFCC）
13～40 維 MFCC

加上 Δ、ΔΔ（可選）

1 秒音訊 → 約 98 個 frame

(3) 模型輸入（Input Tensor）
Code
shape = [98, 40]   # 1 秒 MFCC
dtype = float32
3. 🧠 模型架構（Model Architecture）
WakeWordApp 支援兩種模型：

A. TFLite 小型 CNN（預設）
適合手機即時推論，架構如下：

Code
Input (98x40 MFCC)
↓
Conv2D (filters=16, kernel=3x3)
↓
ReLU
↓
MaxPool
↓
Conv2D (filters=32)
↓
ReLU
↓
Flatten
↓
Dense (64)
↓
Dense (2) → [wakeword, non-wakeword]
↓
Softmax
優點：

小、快、省電

可本地訓練

適合自訂喚醒詞

模型大小： 50KB～200KB
推論速度： < 5ms（手機 CPU）

B. Vosk / Kaldi（可選）
適合語音辨識或多喚醒詞情境。

優點：

穩定

支援多語言

可擴充語音命令

缺點：

模型較大（5MB～50MB）

不適合本地訓練

WakeWordApp 預設使用 TFLite CNN 作為 KWS 模型。

4. 🏋️‍♂️ 本地訓練流程（User Training）
使用者錄製 3～5 次喚醒詞後，App 會：

(1) 音訊切片
自動偵測語音區段

去除靜音

(2) 特徵抽取
將每段語音轉成 MFCC

(3) 建立訓練資料
Code
wakeword: 使用者錄音
non-wakeword: 系統內建背景噪音 + 靜音
(4) 訓練小型 CNN
5～20 epoch

Adam optimizer

Loss: categorical crossentropy

(5) 匯出 TFLite 模型
Code
model.tflite
labels.txt
5. ⚡ 推論邏輯（Inference Logic）
WakeWordApp 使用 滑動視窗（sliding window） 持續偵測：

Code
每 100ms 取 1 秒音訊 → 產生 MFCC → 模型推論
模型輸出：

Code
[ wakeword_score , nonwake_score ]
KWS Engine 判斷：

若 wakeword_score > 閾值（預設 0.7）

且連續 N 次（預設 3 次）
→ 判定為喚醒成功

6. 📦 模型檔案結構
Code
model/
├── model.tflite        # KWS 模型
├── labels.txt          # 類別標籤
└── config.json         # 閾值、特徵設定
7. 📱 手機效能需求（Mobile Performance）
CPU：任何 ARMv8 以上

RAM：< 5MB

推論速度：5ms～10ms

電量消耗：< 1% / 小時（前景服務）

8. 🔧 可替換模型（Future Options）
WakeWordApp 架構支援：

Google Speech Commands CNN

DS-CNN（Depthwise CNN）

CRNN（CNN + GRU）

TinyML Edge Impulse 模型

未來可直接替換 model.tflite。

✔️ 文件結束
