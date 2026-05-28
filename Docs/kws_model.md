# 🔔 WakeWordApp — KWS Engine 判斷邏輯（kws_engine）

本文件說明 WakeWordApp 在推論階段如何判斷「喚醒詞是否被觸發」。
這是整個 KWS 系統的核心邏輯，負責把模型輸出的分數轉換成「是否喚醒成功」。

---

## 1. 🎯 KWS Engine 的目標

- 持續監聽麥克風輸入  
- 將音訊切成小片段（sliding window）  
- 每片段丟進模型做推論  
- 根據模型輸出的分數判斷是否喚醒  
- 避免誤判（false positive）  
- 保持低延遲、低耗電  

---

## 2. 🪟 Sliding Window（滑動窗）

WakeWordApp 使用固定長度的音訊片段做推論：

```text
Window size：1 秒
Hop size：100ms（或 10% overlap）
```

流程：

1. 收集 1 秒音訊  
2. 做 MFCC  
3. 丟進模型  
4. 每 100ms 更新一次結果  

---

## 3. 📊 模型輸出格式

模型輸出兩個分數：

```text
wakeword_score
nonwake_score
```

我們只需要：

```text
wakeword_score
```

---

## 4. 🎚 閾值判斷（Threshold）

```text
若 wakeword_score > threshold
則視為「可能喚醒」
```

預設：

```text
threshold = 0.7
```

---

## 5. 🔁 連續 N 次判定（避免誤判）

```text
連續 N 次超過 threshold 才算喚醒成功
```

預設：

```text
N = 3
```

---

## 6. 🔕 Debounce（冷卻時間）

```text
喚醒成功後，進入 1 秒冷卻時間
```

---

## 7. 🧠 KWS Engine 內部狀態（簡化版）

```text
score_history      # 最近 N 次的分數
consecutive_count  # 連續超過 threshold 的次數
cooldown_timer     # 冷卻時間
```

---

## 8. 🧩 簡化流程（Pseudo Code）

```python
for each frame:
    score = model.predict(mfcc)

    if cooldown_timer > 0:
        cooldown_timer -= 1
        continue

    if score > threshold:
        consecutive_count += 1
    else:
        consecutive_count = 0

    if consecutive_count >= N:
        trigger_wakeword()
        consecutive_count = 0
        cooldown_timer = COOLDOWN_FRAMES
```

---
