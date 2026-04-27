---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EEG-WM-JEPA is a research project building an EEG foundation model that combines:
- **LeWM (LeWorldModel)** — JEPA-based world model using SIGReg regularization (no EMA/stop-gradient)
- **LUNA-style Channel Mixer** — Learned-query cross-attention for topology-invariant EEG processing
- **Spatiotemporal Masking** — Simultaneous time + spatial masking for self-supervised pretraining

The goal is a universal EEG encoder that works across any electrode montage/equipment, pretrained via self-supervised learning, evaluated on classification benchmarks (TUAB, TUSL, BCI, TDBrain).

## Architecture (3-Stage Pipeline)

### Stage 1: EEG Standardization
- Resample → 200Hz
- Re-reference → Common Average Reference
- Bandpass filter → 0.5–75Hz
- Segment → 2-second windows

### Stage 2: Self-Supervised Pretraining
```
Raw EEG [B, C, T] (C is variable per dataset)
  → Patch Embedder (depthwise 1D Conv, channel-independent)
  → Channel Mixer (3D electrode coords + Fourier encoding + cross-attention → fixed-size [B, N, D])
  → Spatiotemporal Masking (60%, 500ms–1s blocks)
  → Transformer Encoder (RoPE positional embeddings)
  → Predictor (predicts masked region latent vectors)
  → Loss: Prediction Loss (MSE) + SIGReg + Query Specialization Loss
```

### Stage 3: Downstream Evaluation
- Linear probing (encoder frozen) and fine-tuning (encoder trainable)
- Benchmarks: TUAB (normal/abnormal), TUSL (sleep staging), BCI Competition (motor imagery), TDBrain

## Key Technical Decisions

- **SIGReg over RVQ**: Continuous latent space (SIGReg) chosen over discrete tokenization (RVQ) for training stability. These two approaches are mathematically incompatible.
- **Minimal preprocessing**: No ICA or heavy artifact removal. JEPA's latent prediction naturally filters noise.
- **2-second windows**: Matches Laya/LuMamba for fair baseline comparison.
- **Channel Mixer from LUNA/Laya**: Handles variable channel counts (8–256) by projecting to fixed-size representation via learned queries.

## Reference Codebases

- LeWM: https://github.com/lucas-maes/le-wm (SIGReg + Predictor architecture)
- Laya: LeJEPA-based EEG foundation model (Channel Mixer + Patch Embedder reference)
- LuMamba: LeJEPA + Mamba backbone for EEG
- DELTA: RVQ tokenizer for EEG (not used, but studied for comparison)

## Project Structure

```
EEG-WM-JEPA/
├── CLAUDE.md                  # 이 파일 (Claude Code 가이드)
├── main.py                    # 사전학습 entry point (학습 루프, AMP, wandb, ckpt)
├── pyproject.toml             # uv 의존성 정의
├── uv.lock
├── .claude/
│   └── skills/
│       └── hp-tune/           # HP 튜닝 자동화 skill (W&B Bayes sweep + Claude 능동 프루닝)
│           ├── SKILL.md       #   Claude 운영 프로토콜 (폴링/프루닝 룰)
│           ├── sweep.yaml     #   W&B sweep 설정 (lr, sigreg_projections, grad_accum)
│           └── README.md      #   사용자용 시작 가이드
├── configs/
│   └── default.yaml           # 모델/학습/데이터 하이퍼파라미터 (canonical)
├── src/
│   ├── model/
│   │   ├── eeg_jepa.py        # 전체 모델 (Encoder + Predictor + Loss)
│   │   ├── patch_embedder.py  # depthwise 1D Conv patcher
│   │   ├── channel_mixer.py   # LUNA-style 학습 쿼리 cross-attention (가변 채널 → 고정)
│   │   ├── masking.py         # Spatiotemporal block masking
│   │   └── lewm_modules.py    # LeWM Transformer 블록 (RoPE, SIGReg)
│   ├── preprocessing/
│   │   ├── standardize.py     # resample/CAR/bandpass/window
│   │   ├── reve_loader.py     # REVE memmap + 메타데이터
│   │   ├── dataset.py         # 더미/오프라인 dataset
│   │   ├── streaming_dataset.py  # HF Parquet 스트리밍 (현 사용)
│   │   └── mds_dataset.py     # MosaicML Streaming (보류, MDS 변환 포기 후 미사용)
│   ├── training/              # 학습 유틸 (현 비어있음, main.py에 통합되어 있음)
│   └── evaluation/
│       ├── downstream.py      # Linear probing + fine-tuning
│       └── bci_dataset.py     # BCI IV 2a 로더 (MOABB)
├── scripts/
│   ├── preprocess_reve.py            # REVE → parquet 메인
│   ├── preprocess_pipeline.py        # 단일 파일 전처리
│   ├── preprocess_parallel.py        # 멀티프로세스 전처리
│   ├── preprocess_aggressive.py      # OOM 회피용 보수적 변형
│   ├── resume_preprocess.sh          # 중단 재개
│   ├── monitor_preprocess.sh
│   ├── convert_parquet_to_mds*.py    # parquet → MDS 변환 (3종, 보류)
│   ├── eval_bci.py                   # BCI IV 2a 평가
│   └── prepare_downstream_hf.py     # Downstream 데이터 전처리 + HF 업로드 (BCI, APAVA)
├── tests/
│   └── test_forward_pass.py   # 더미 데이터 forward/backward 검증
├── paper/
│   └── main.md                # 논문 초안
└── docs/                       # 문서 (구조는 아래 Documentation 섹션 참조)
```

새 모듈/스크립트/디렉토리를 추가하거나 기존 파일을 이동·삭제하면 **반드시 위 구조도를 함께 갱신**할 것. 한 줄 설명을 함께 적어 새로 합류하는 사람이 한눈에 파악할 수 있게 유지한다.

## Documentation Convention

`docs/`는 두 층으로 구성된다:

### 1. 핵심 프로젝트 매니징 (`docs/` 루트)

항상 최신 상태로 유지해야 하는 5개 파일. 작업 후 관련 항목은 반드시 업데이트할 것.

- **`docs/todo.md`** — 할 일 관리. 완료된 항목 `[x]` 체크 + 날짜 기록. 진행 중인 항목은 상태 표시 (예: "— 진행 중 (54/322)").
- **`docs/history.md`** — 의사결정/기술적 발견의 연대기. Phase 번호로 추가하고 변경 로그 테이블에 한 줄 요약.
- **`docs/issues.md`** — 기술적 이슈 트래커. Issue 번호 부여, 상태 표기 (✅ 해결됨 / ⚠️ 모니터링 중 / ⚠️ 보류 중). **해결된 이슈도 절대 삭제하지 말 것** (의사결정 근거 보존). 이슈가 여러 문서에 걸치면 issues.md를 canonical source로 두고 history.md엔 한 줄 요약만 추가.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fbdeme) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
