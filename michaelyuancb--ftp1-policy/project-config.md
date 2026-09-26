---
trigger: always_on
description: 本项目实现了 **FTP1 (Robot Tactile Action 1)** 模型，这是一个基于扩散的多模态策略模型，扩展自 Pi0.5 架构，添加了异构触觉传感器支持。
---

# OpenPI 模型架构文档

## 概述

本项目实现了 **FTP1 (Robot Tactile Action 1)** 模型，这是一个基于扩散的多模态策略模型，扩展自 Pi0.5 架构，添加了异构触觉传感器支持。

训练入口脚本: `scripts/zarr_train_ftp1_pytorch.py`

---

## 模型架构总览

```
FTP1Pytorch
├── paligemma_with_expert (FTP1PaliGemmaWithExpertModel)
│   ├── paligemma (PaliGemmaForConditionalGeneration)  -- VLM Expert (Gemma 2B)
│   │   ├── vision_tower (SiglipVisionModel)           -- 图像编码器
│   │   └── language_model (GemmaForCausalLM)          -- 语言模型
│   ├── gemma_expert (GemmaForCausalLM)                -- Action Expert (Gemma 300M)
│   └── gemma_tactile_expert (GemmaForCausalLM)        -- Tactile Expert (Gemma 300M, 可选)
├── hpt_tactile_encoder (FTP1HptTactileEncoder)       -- 异构触觉编码器
│   ├── tokenizers (ModuleDict)                        -- 不同传感器类型的编码器
│   │   ├── TactileDataEncoder (binary/state)          -- 二值/状态类型编码
│   │   ├── MatrixCNNEncoder                           -- 矩阵类型CNN编码
│   │   └── ImageTactileEncoder (ViT)                  -- 图像类型ViT编码
│   └── shared_image_chunk_encoder                     -- T3预训练共享块编码器
├── state_encoder (FourierStateEncoder)                -- 状态傅里叶编码器
├── action_in_proj (Linear)                            -- 动作输入投影
├── action_out_proj (Linear)                           -- 动作输出投影
├── time_mlp_in (Linear)                               -- 时间步MLP输入
└── time_mlp_out (Linear)                              -- 时间步MLP输出
```

---

## 模型架构图
 输入数据
      ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
      │     Images      │ │    Language     │ │     State       │ │    Tactiles     │ │ Actions + Noise │
      │   (H,W,3) x N   │ │    Prompt       │ │   (B,T,dim)     │ │   (dict)        │ │  (B,T,act_dim)  │
      └────────┬────────┘ └────────┬────────┘ └────────┬────────┘ └────────┬────────┘ └────────┬────────┘
               │                   │                   │                   │                   │
               ▼                   ▼                   │                   ▼                   ▼
  ┌────────────────────────────────────────────┐      │    ┌──────────────────────────────────────────────────────┐
  │                                            │      │    │                                                      │
  │           embed_prefix()                   │      │    │                 embed_suffix()                       │
  │                                            │      │    │                                                      │
  │  ┌──────────────────────────────────────┐  │      │    │  ┌────────────────────┐    ┌─────────────────────┐   │
  │  │        SiglipVisionModel             │  │      │    │  │   action_in_proj   │    │    time_mlp_in      │   │
  │  │  ┌────────────────────────────────┐  │  │      │    │  │    (nn.Linear)     │    │    time_mlp_out     │   │
  │  │  │ transformers_replace/models/   │  │  │      │    │  │                    │    │    (nn.Linear)      │   │
  │  │  │ siglip/modeling_siglip.py      │  │  │      │    │  │  32 ──► 1024       │    │                     │   │
  │  │  └────────────────────────────────┘  │  │      │    │  └─────────┬──────────┘    └──────────┬──────────┘   │
  │  │  Patches:14x14, Layers:27, Dim:1152  │  │      │    │            │     ┌──────────────────────┘            │
  │  └──────────────────┬───────────────────┘  │      │    │            │     │  (AdaRMS conditioning)            │
  │                     │                      │      │    │            ▼     ▼                                   │
  │                     ▼                      │      │    │  ┌─────────────────────┐                             │
  │  ┌──────────────────────────────────────┐  │      │    │  │  Suffix Embeddings  │                             │
  │  │     PaliGemma Multi-modal Projector  │  │      │    │  │  (B, T, 1024)       │                             │
  │  └──────────────────┬───────────────────┘  │      │    │  └─────────────────────┘                             │
  │                     │                      │      │    │                                                      │
  │                     ▼                      │      │    │  models_pytorch/ftp1_pytorch.py                     │
  │  ┌──────────────────────────────────────┐  │      │    └──────────────────────────────────┬───────────────────┘
  │  │         Gemma Embedding Layer        │◄─┼──────┘                                       │
  │  └──────────────────┬───────────────────┘  │                                              │
  │                     │                      │                                              │
  │                     ▼                      │                                              │
  │  ┌──────────────────────────────────────┐  │                                              │
  │  │       Prefix Embeddings              │  │                                              │
  │  │       (B, seq_len, 2048)             │  │                                              │
  │  └──────────────────────────────────────┘  │                                              │
  │                                            │                                              │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelyuancb/ftp1-policy](https://github.com/michaelyuancb/ftp1-policy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
