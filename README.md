📘 WakeWordApp
離線、可訓練、使用者自訂喚醒詞的 Android App

WakeWordApp 是一個讓使用者「錄製自己的喚醒詞」並在本地訓練模型的 Android App。
整個流程完全離線，不上傳任何語音資料，並能在背景持續偵測喚醒詞。

🚀 功能特色
完全離線運作（無需網路、不傳雲端）

使用者自訂喚醒詞（錄 3～5 次即可）

本地訓練模型（TFLite / Vosk）

前景服務常駐偵測

低耗電、低延遲

可擴充動作觸發（開 App、執行指令等）

🧱 系統架構
WakeWordApp 由以下模組組成：

Android App Layer  
UI、錄音、前景服務、權限管理

DSP Layer  
音訊前處理、MFCC、降噪

Model Layer  
TFLite / Vosk 模型推論

Training Module  
使用者錄音 → 特徵抽取 → 本地訓練

Wake Word Engine (KWS)  
閾值判斷、連續偵測

Action Trigger Layer  
喚醒後執行動作（Toast、開啟 App…）

詳細架構請見：
📄 Docs/WakeWordApp.md

📂 專案結構（建議）
Code
WakeWordApp/
├── app/                # Android App 原始碼
├── dsp/                # 音訊處理 (MFCC / Filter)
├── model/              # TFLite / Vosk 模型
├── training/           # 本地訓練模組
├── docs/               # 文件
│   └── WakeWordApp.md
└── README.md
🛠 技術堆疊
Android (Kotlin)

TFLite / Vosk

MFCC / DSP

前景服務 (Foreground Service)

本地模型訓練

SQLite / JSON 儲存

📦 安裝與建置
Clone 專案：

Code
git clone https://github.com/kimbhom/WakeWordApp.git
用 Android Studio 開啟 app/

Build & Run

🧪 使用流程
使用者錄製喚醒詞（3～5 次）

App 進行特徵抽取（MFCC）

本地訓練模型

前景服務啟動

偵測到喚醒詞 → 執行動作

📜 License