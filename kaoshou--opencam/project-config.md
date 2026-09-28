---
trigger: always_on
description: 請協助我從零開始規劃並開發一套「簡單、穩定、高可靠度」的桌面螢幕錄影工具。
---

﻿# 高可靠跨平台螢幕錄影工具開發規格

## 1. 專案目標

請協助我從零開始規劃並開發一套「簡單、穩定、高可靠度」的桌面螢幕錄影工具。

本專案概念上類似 oCam 的「螢幕錄影」核心功能，但不需要複製 oCam 的全部功能，也不需要加入影片剪輯、直播、GIF、截圖編輯等額外功能。

本專案的核心目標只有：

> 讓使用者可以簡單選擇錄影範圍、系統聲音與麥克風，然後可靠地完成長時間螢幕錄影。

「錄影可靠性」的重要性高於 UI、功能數量與開發速度。

---

# 2. 主要需求

第一階段至少提供：

- 全螢幕錄影
- 指定螢幕錄影
- 自訂矩形區域錄影
- 可選擇是否錄製系統聲音
- 可選擇是否錄製麥克風
- 可選擇音訊裝置
- 30 FPS
- 60 FPS
- 開始錄影
- 停止錄影
- MKV 安全工作檔
- 正常停止後自動 Remux 為 MP4
- Crash Recovery
- 錄影 Session 狀態保存
- 磁碟剩餘空間監控
- Structured Logging
- 長時間穩定錄影

第一階段不需要：

- 影片剪輯
- GIF
- 直播
- YouTube 上傳
- Webcam Overlay
- OCR
- 即時特效
- 專業串流功能
- OBS 等級場景管理

---

# 3. 開發平台

第一階段主要支援：

- Windows 10
- Windows 11
- x64

但從第一天開始，程式架構必須考慮未來移植至：

- macOS

目前不要求立即完成 macOS 版本。

第一階段只需要完成 Windows 正式可用版本，但禁止將整個系統設計成 Windows-only architecture。

---

# 4. 建議技術棧

優先評估以下架構：

- C#
- .NET 8 或目前適合正式發布的 .NET LTS
- Avalonia UI
- MVVM
- Dependency Injection
- Structured Logging

Windows 螢幕擷取優先：

- Windows.Graphics.Capture
- Direct3D / Direct3D11，如有必要

Windows 音訊：

- WASAPI
- WASAPI Loopback
- 可評估 NAudio，但必須說明採用理由

影片編碼與封裝：

- FFmpeg

錄影中的主要安全容器：

- MKV

最終使用者輸出格式：

- MP4

正常停止錄影後：

```text
MKV → Remux → MP4
```

原則上禁止為了產生 MP4 而重新編碼。

如果經過技術分析後認為有更穩定、更適合長期維護的方案，可以提出調整，但必須先說明：

1. 為什麼修改
2. 修改後的優點
3. 對穩定性的影響
4. 對未來 macOS 移植的影響

不要只因為某個方案比較容易寫，就犧牲可靠性。

---

# 5. 最高優先原則：Recording Reliability

這是本專案最重要的規則。

任何功能、UI、效能或架構決策，都不得犧牲已錄製資料的安全性。

本軟體必須假設以下事件一定可能發生：

- UI crash
- Recorder crash
- FFmpeg crash
- 麥克風突然拔除
- 音訊裝置消失
- 預設音訊裝置改變
- 螢幕拔除
- 螢幕解析度改變
- DPI Scaling 改變
- Windows 鎖定
- Windows 睡眠
- Windows 喚醒
- 磁碟空間不足
- 磁碟寫入錯誤
- 使用者強制關閉程式
- 使用者從工作管理員 End Task
- 系統非正常關機
- 長時間錄影
- Encoder 異常
- Capture thread 異常

我們不能保證程式永遠不 crash。

真正的目標是：

> 即使程式異常，也應盡最大可能保留異常發生以前已成功錄製的內容。

---

# 6. 禁止直接以傳統 MP4 作為主要錄製工作檔

錄影進行中：

優先使用：

```text
recording_xxx.mkv
```

正常停止後：

1. 正常結束 MKV
2. 驗證 MKV 是否可讀
3. Remux 成 MP4
4. 驗證 MP4
5. 確認成功後才依設定處理 MKV

禁止：

```text
錄影開始
↓
recording.mp4
↓
所有 metadata 等到最後才 finalize
```

因為如果程式非正常中止，不應讓整段錄影因此全部損毀。

---

# 7. Recovery 機制

每一次 Recording Session 都應建立獨立工作目錄。

例如：

```text
Recordings/
└── Sessions/
    └── 20260915_004500_xxxxx/
        ├── session.json
        ├── recording.mkv
        ├── recording.log
        └── recovery.json
```

`session.json` 至少應保存：

- Session ID
- 開始時間
- 最後狀態
- Capture source
- Capture region
- Monitor
- Resolution
- FPS
- Audio configuration
- System audio device
- Microphone device
- Encoder
- Output path
- Temporary path
- Recording state

程式啟動時應檢查是否存在：

- Recording
- Interrupted
- Recoverable

等未正常結束的 Session。

如果存在，應提供 Recovery。

例如：

```text
偵測到上次未正常完成的錄影

開始時間：
已錄製時間：
檔案大小：
工作檔位置：

[恢復]
[轉換成 MP4]
[開啟檔案位置]
[保留原始檔]
```

Recovery 過程不得先刪除原始 MKV。

---

# 8. Recorder 與 UI 解耦

請優先評估：

```text
UI Process
```

與：

```text
Recorder Process
```

是否應分離。

理想狀況：

```text
ScreenRecorder.UI

ScreenRecorder.Recorder
```

分開。

UI crash 不應必然導致 Recorder crash。

如果採用 Process Separation，請設計：

- IPC
- Recorder state
- Heartbeat
- Graceful shutdown
- Unexpected disconnect
- Recorder recovery

IPC 可評估：

- Named Pipe
- Local IPC
- 其他適合 .NET Desktop 的方式

請選擇簡單、可靠、容易維護的方式。

不要為了架構漂亮而過度工程化。

---

# 9. Watchdog

評估加入 Watchdog / Health Monitoring。

至少監控：

- Capture 是否仍有 frame
- Encoder 是否仍運作
- Output file 是否持續增加
- Audio capture 是否正常
- Recorder process 是否 alive
- 剩餘磁碟空間

如果發現異常：

優先：

1. 保留現有錄影
2. 嘗試安全停止
3. Flush 可以保存的資料
4. 更新 Session 狀態
5. 寫入 Log
6. 通知 UI

不得因某個非必要功能失敗而讓整個 Recorder crash。

---

# 10. 第一階段核心 Capture 功能

Windows 第一版必須至少支援：

- 全螢幕錄影
- 指定螢幕錄影
- 自訂矩形區域錄影

未來可增加：

- 指定視窗錄影

---

# 11. 自訂區域選擇

使用者點：

```text
選擇錄影區域
```

後：

顯示螢幕 Overlay。

使用者可以：

- 滑鼠拖曳選擇矩形
- 顯示寬 × 高
- 調整選取範圍
- 確認
- 取消

必須正確處理：

- 多螢幕
- 不同 DPI
- Windows Scaling
- Negative monitor coordinates

錄影開始後，區域選擇 Overlay 必須消失。

---

# 12. 音訊

至少支援：

系統聲音：

- 開
- 關

麥克風：

- 開
- 關

並允許使用者選擇：

- System audio device
- Microphone device

至少應支援四種組合：

1. 無聲音
2. 只有系統聲音
3. 只有麥克風
4. 系統聲音 + 麥克風

---

# 13. 音訊裝置異常

如果錄影過程中發生：

- 麥克風被拔除
- Audio device lost
- Default device changed

不得直接讓整個 Recorder crash。

應：

- 記錄錯誤
- 儘可能繼續畫面錄製
- UI 顯示警告

如果技術上可行，可嘗試重新連線。

但：

> Recovery 不得造成比原本錯誤更大的風險。

---

# 14. Audio / Video Synchronization

必須特別處理長時間錄影：

- Audio drift
- Video timestamp
- Dropped frames
- Variable frame timing
- System load

不要假設：

每秒一定剛好收到 30 或 60 frames。

Timestamp 必須有可靠來源。

請設計長時間 A/V sync 測試。

至少測試：

- 30 分鐘
- 1 小時
- 2 小時
- 4 小時

後續穩定版本應測試：

- 8 小時

---

# 15. FPS

第一版提供：

- 30 FPS
- 60 FPS

預設：

```text
30 FPS
```

如果裝置或系統無法穩定維持 FPS：

不應造成 Recorder crash。

---

# 16. Encoder

優先支援 H.264。

請設計 Encoder abstraction，例如：

```text
IEncoder
```

未來可以支援：

- Software H.264
- NVIDIA NVENC
- Intel Quick Sync
- AMD AMF
- Apple VideoToolbox

第一版不要求全部完成。

第一版可以先選擇最穩定方案。

但 Core 不得寫死單一 encoder。

---

# 17. Hardware Encoder

後續版本可自動偵測：

- NVIDIA
- Intel
- AMD

如果 Hardware Encoder 不可用：

自動 fallback。

Fallback 不得造成錄影中止。

第一版可以先不做完整 Hardware Encoder 自動選擇，但架構必須保留。

---

# 18. Disk Space Monitoring

錄影前：

檢查目標磁碟剩餘空間。

錄影中：

定期檢查。

例如設定：

```text
Warning threshold
```

與：

```text
Critical threshold
```

當剩餘空間過低：

UI 必須明確警告。

到 Critical threshold 時：

應考慮：

```text
安全停止錄影
```

而不是：

```text
讓磁碟完全寫滿後 crash
```

所有 threshold 應可設定。

---

# 19. 檔案命名

預設：

```text
Recording_yyyyMMdd_HHmmss.mkv
```

完成：

```text
Recording_yyyyMMdd_HHmmss.mp4
```

避免檔名衝突。

---

# 20. Remux

正常停止後：

```text
MKV → MP4
```

應使用 Stream Copy。

例如概念：

```text
-c copy
```

不要重新 encode。

Remux 完成後必須：

- 檢查 exit code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaoshou/OpenCam](https://github.com/kaoshou/OpenCam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
