---
trigger: always_on
description: <!-- CLAUDE.md — Claude Code project instructions + research journal -->
---

<!-- CLAUDE.md — Claude Code project instructions + research journal -->
<!--
  This file serves dual purposes:
  1. Project instructions for my Claude Code AI assistant
  2. A complete research journal documenting experimental decisions and findings

  Workflow: I designed the research architecture, experiments, and made all
  technical decisions. Claude Code assisted with code implementation and
  formatting. The 7 core conclusions and all ablation study designs are mine.
-->

# CardboardBox_detect — 專案說明文件

## 專案最終目標

使用筆電視訊鏡頭，即時執行兩階段 YOLO 推論：
1. **YOLO1** 偵測畫面中的紙箱，框出 The Box（class 0）
2. **Crop Program** 依照 YOLO1 輸出的 BBox 裁切出紙箱 ROI
3. **YOLO2** 在 ROI 內偵測瑕疵：Stain（class 0）、Puncture（class 1）

用戶拿著紙箱出現在鏡頭前，系統以肉眼可感受的 Realtime 同時框出 The Box / Stain / Puncture。

**核心 Feature：使用 Synthetic Data Generation（合成資料）來解決真實標記資料不足的問題。**

**方法論：Zero-shot Sim-to-Real Transfer** — 訓練集 100% 使用合成資料，透過 Domain Randomization
（隨機相機、光照、地面材質、瑕疵位置/大小/貼圖）使模型直接泛化至真實場景。

---

## 兩階段 YOLO 模型規格

| 模型 | 架構 | 輸入尺寸 | 偵測目標 | Class ID |
|------|------|----------|----------|----------|
| YOLO1 | YOLOv8n | 640×640（全幀） | 紙箱 (The Box) | 0 |
| YOLO2 | YOLOv8n | 640×640（紙箱 ROI） | Stain、Puncture | Stain=0, Puncture=1 |

> **YOLO2 的訓練輸入是 YOLO1 的輸出 crop，推論流程必須與訓練完全鏡像。**

---

## 目錄結構（實際）

```
CardboardBox_detect/
├── 01_Assets/                              # 素材庫
│   ├── Ground/                             # 地面 PBR 材質 (29 種，各含 Color/Normal/Roughness .jpg)
│   ├── HDRIs/                              # 環境光照 (.exr，20 個)
│   ├── decals/
│   │   ├── defects/                        # 瑕疵 Decal 貼圖
│   │   │   ├── Stain/                      # 污漬 (17 張 .png，帶 Alpha，2026-04-18 刪除 3 張並重新編號)
│   │   │   ├── Puncture/                   # 破孔 (7 張)
│   │   │   ├── Ink/                        # 墨水 (12 張)
│   │   │   ├── Scratch/                    # 刮痕 (16 張)
│   │   │   └── Surface_Damage/             # 表面損傷 (9 張)
│   │   └── interference/                   # 干擾貼圖
│   │       ├── Barcode/
│   │       ├── Colored_Labels/
│   │       └── Shipping Label/
│   └── real_refs/
│       └── 0408/                           # 真實參考照片 (Val / Test 專用，不入訓練)
│           ├── Roboflow_Ready/             # 134 張原始 Roboflow 匯出（含 hash 檔名，備用）
│           ├── Cardboard_project.yolov8 (1)/  # 最新 Roboflow 下載（含 labels，來源）
│           ├── labels/                     # 處理後 YOLO labels（class remap 版）
│           ├── labels_origin/              # Roboflow 原始 labels 備份（勿修改）
│           ├── clean_box/                  # 62 張  ← images/ + labels/ 已整理
│           ├── stain/                      # 24 張  ← images/ + labels/ 已整理
│           ├── puncture/                   # 20 張  ← images/ + labels/ 已整理
│           ├── both/                       # 8 張   ← images/ + labels/ 已整理
│           └── empty_background/           # 22 張  ← 備用（Hybrid Synthetic 實驗）
│
├── 02_Scripts/                             # 流水線腳本 (依序執行)
│   ├── 00_scene_health_check.py            # Blender 場景健檢，輸出 scene_spec.md / .json
│   ├── 01_orchestrator.py                  # 劇本生成器：輸出 JSON 到 06_Raw_Output/configs/
│   ├── 02_blender_render.py                # Blender 內部腳本：三 Pass 算圖 + Mask 輸出
│   ├── 03_render_operator.py                      # 外部守衛：驗證 configs → 啟動 Blender → 自動重啟
│   ├── 04_annotator_yolo1.py               # Box Mask → YOLO1 全圖 BBox labels + QA 視覺化（無增強）
│   ├── 05_cropper_yolo2.py                 # 裁切紙箱 ROI → 增強 → YOLO2 labels + QA 視覺化
│   ├── 06_build_dataset.py                 # (已部分取代) val/test 實拍圖現由內嵌腳本直接處理
│   ├── 07_augment_test.py                  # 影像增強視覺測試工具（比較原圖/YOLO1/YOLO2 效果）
│   ├── 08_train.py                         # 統一訓練腳本：--target yolo1|yolo2（RTX 4050 最佳化）
│   ├── 09_predict_yolo1.py                 # YOLO1 單段推論：跑 04_Dataset_YOLO1/images/test/ → predict_yolo1/
│   ├── 10_predict_yolo2.py                 # YOLO2 單段推論：跑 05_Dataset_YOLO2/images/test/（已 crop）→ predict_yolo2/
│   ├── 11_realtime_demo.py                 # Webcam 雙段即時：YOLO1→ROI crop→YOLO2（S 截圖到 predict_snapshots/）
│   ├── 12_qa_real_dataset.py               # 實拍 val/test 標注視覺化（人眼 QA）
│   ├── scene_spec.md                       # 00_scene_health_check 輸出的場景規格文件
│   ├── scene_spec.json                     # 同上，JSON 格式
│   │
│   └── [參考用，勿修改]
│       ├── object_orchestrator_v7.py       # 舊版劇本生成器（路徑/格式已不適用，邏輯參考）
│       ├── hsd_pipeline_mvp_v15.py         # 上一代 Blender 渲染腳本（02_blender_render.py 的原型）
│       └── auto_annotator_v6_0407.py       # 上一代 OpenCV 標註腳本（mask 處理邏輯參考）
│
├── 03_Blender_Project/
│   └── AICV_MotherFile_Cardboard_detect.blend   # Blender 主場景（不入版控）
│
├── 04_Dataset_YOLO1/                       # YOLO1 訓練資料集
│   ├── images/train/                       # 合成全圖 640×640
│   ├── images/val/                         # 真實 + 合成 clean
│   ├── images/test/                        # 真實照片
│   ├── labels/train/                       # class 0：The Box
│   ├── labels/val/
│   └── labels/test/
│
├── 05_Dataset_YOLO2/                       # YOLO2 訓練資料集（紙箱 ROI）
│   ├── images/train/                       # 合成 ROI（裁切自 YOLO1 Box BBox + 5% padding）
│   ├── images/val/
│   ├── images/test/
│   ├── labels/train/                       # class 0：Stain，class 1：Puncture
│   ├── labels/val/
│   └── labels/test/
│
├── 06_Raw_Output/                          # Blender 原始輸出（不入版控）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbKurt11/SDG_CardboardBox_detect](https://github.com/bbKurt11/SDG_CardboardBox_detect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
