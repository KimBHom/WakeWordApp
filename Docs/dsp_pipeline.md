# 🎧 WakeWordApp — DSP / MFCC 前處理流程（dsp_pipeline）

本文件說明 WakeWordApp 在喚醒詞偵測（KWS）中使用的  
**音訊前處理（DSP Pipeline）** 與 **MFCC 特徵抽取流程**。

這些步驟是模型準確度與穩定度的核心基礎。

---

## 1. 🎯 前處理目標（DSP Objective）

WakeWordApp 的 DSP 目標：

- 將麥克風原始音訊轉換成模型可用的 MFCC 特徵  
- 降低噪音、提升語音訊號品質  
- 在手機 CPU 上以低延遲方式執行  
- 讓模型能穩定辨識使用者的喚醒詞  

---

## 2. 🎙 音訊輸入規格（Audio Input Spec）

所有音訊會被統一轉換成：

16 kHz
Mono
16-bit PCM

Code

這是語音辨識與 KWS 的標準設定，兼具效能與準確度。

---

## 3. 🔧 DSP Pipeline（完整流程）

WakeWordApp 的前處理流程如下：

麥克風 → 重採樣 → Pre-emphasis → Framing → Windowing → VAD → MFCC

Code

以下逐步說明：

---

## 4. 🔹 Step 1：重採樣（Resampling）

若裝置輸入不是 16kHz，會自動轉換成：

16kHz / mono

Code

理由：

- 16kHz 足夠涵蓋語音頻率（300–3400Hz）
- 降低資料量 → 降低模型延遲

---

## 5. 🔹 Step 2：Pre-emphasis（預加重）

公式：

y[t] = x[t] - 0.97 * x[t-1]

Code

目的：

- 強調高頻成分  
- 提升語音特徵（尤其是子音）  
- 讓 MFCC 更穩定  

---

## 6. 🔹 Step 3：Framing（分幀）

語音不是一次處理，而是切成小片段：

Frame size：25ms
Hop size：10ms

Code

例如：

- 1 秒音訊 → 約 98 個 frame  
- 每個 frame 會被視為一個小型分析單位  

---

## 7. 🔹 Step 4：Windowing（加窗）

每個 frame 會套用 Hamming Window：

w[n] = 0.54 - 0.46 * cos(2πn / (N-1))

Code

目的：

- 減少 frame 邊界造成的訊號不連續  
- 提升頻譜分析品質  

---

## 8. 🔹 Step 5：VAD / 靜音裁切（可選）

WakeWordApp 會偵測：

- 靜音  
- 背景噪音  
- 非語音區段  

並裁切掉不必要的部分。

好處：

- 減少模型負擔  
- 提升喚醒詞辨識準確度  
- 避免背景噪音誤判  

---

## 9. 🔹 Step 6：MFCC 特徵抽取（核心）

WakeWordApp 使用 MFCC（Mel-Frequency Cepstral Coefficients）作為模型輸入。

### MFCC 流程：

Frame → FFT → Mel Filterbank → Log → DCT → MFCC

Code

### 設定：

- MFCC 維度：**13～40 維**
- 可選：Δ、ΔΔ（速度、加速度）
- 每秒 frame 數：約 98

### 輸出張量：

shape = [98, 40]   # 1 秒 MFCC
dtype = float32

Code

這就是模型的輸入格式。

---

## 10. 🔹 Step 7：Normalization（特徵正規化）

為了讓模型更穩定：

- 會對每個 MFCC 維度做 **均值 / 標準差正規化**
- 避免不同錄音環境造成偏移

---

## 11. 📦 DSP 模組檔案結構（建議）

dsp/
├── audio_preprocess.py      # 重採樣、pre-emphasis、VAD
├── mfcc_extractor.py        # MFCC 特徵抽取
├── normalization.py         # 特徵正規化
└── dsp_utils.py             # 工具函式

Code

---

## 12. 🚀 效能需求（Mobile Performance）

WakeWordApp 的 DSP 設計為：

- CPU-only  
- 單 frame 處理時間 < 1ms  
- 整體 MFCC 抽取 < 10ms / 秒  
- 電量消耗極低  

這讓前景服務能長時間運作而不耗電。

---

## ✔️ 文件結束