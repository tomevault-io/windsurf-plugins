---
trigger: always_on
description: VITS ベースの高品質ニューラル TTS。**6言語マルチリンガルモデル**を学習済み、**8言語の G2P コード**を全7ランタイム (Python/C#/Rust/Go/JS-WASM/C++/CLI) に実装済み。
---

# Piper TTS - プロジェクト概要

VITS ベースの高品質ニューラル TTS。**6言語マルチリンガルモデル**を学習済み、**8言語の G2P コード**を全7ランタイム (Python/C#/Rust/Go/JS-WASM/C++/CLI) に実装済み。

**ブランチ**: `dev` (v1.12.0 リリース済み)。v1.12.0 Breaking changes: [docs/migration/v1.11-to-v1.12.md](docs/migration/v1.11-to-v1.12.md)。

---

## 現在の状態

### 学習済みモデル (6言語マルチリンガル)

| モデル | 学習基盤 | 状態 | パス |
|-------|--------|------|------|
| **6lang base (HiFi-GAN)** | 75 epoch (v1.11 系) | 完了 (2026-03-16) | `/data/piper/output-multilingual-6lang/` |
| **6lang base MB-iSTFT** | 75 epoch スクラッチ | 完了 (2026-04-16) | `/data/piper/output-multilingual-6lang-mb-istft/multilingual-6lang-mb-istft-scratch-75epoch.onnx` |
| **つくよみちゃん 6lang-v2** | 6lang ベースから FT 500 epoch | 完了 (2026-03-16) | `/data/piper/output-tsukuyomi-finetune-6lang-v2/tsukuyomi-6lang-v2-fixed.onnx` |
| **つくよみちゃん MB-iSTFT** | 6lang MB-iSTFT ベースから FT 500 epoch | 完了 (2026-05-02) | `/data/piper/output-tsukuyomi-mb-istft-finetune/tsukuyomi-mb-istft-500epoch.onnx` |
| **CSS10 JA 6lang** | 6lang ベースから 50 epoch | 完了 (2026-03-16) | `test/models/multilingual-test-medium.onnx` |

### データセット

| データセット | 発話数 | 話者数 | シンボル | パス |
|-------------|-------|-------|---------|------|
| `dataset-multilingual-6lang-filtered` | **508,187** | 571 | 173 | `/data/piper/dataset-multilingual-6lang-filtered/` |
| `dataset-tsukuyomi-finetune-6lang` | 100 | 1 | 173 | `/data/piper/dataset-tsukuyomi-finetune-6lang/` |

**6lang 内訳:** ja 60,148 (20話者, MOE-Speech) / en 74,912 (310, LibriTTS-R) / zh 63,223 (142, AISHELL-3) / es 168,374 (63, CML-TTS) / fr 107,464 (28, CML-TTS) / pt 34,066 (8, CML-TTS)。言語コード: ja=0, en=1, zh=2, es=3, fr=4, pt=5。

**前処理:** `src/python/piper_train/tools/prepare_multilingual_dataset.py`

---

## 開発環境セットアップ

clone 直後 1 回だけ実行 (commit 時の lint drift / format drift を CI 待ち前に検出):

```bash
pip install pre-commit  # または: uvx pre-commit
pre-commit install      # .git/hooks/pre-commit を生成
```

これで `git commit` 時に ruff (check + format) と PUA consistency が自動実行され、
drift があれば commit がブロックされます (auto-fix も適用)。手動全件確認は
`pre-commit run --all-files`。

**Pre-push 段階 (opt-in、推奨):**

```bash
pre-commit install --hook-type pre-push   # .git/hooks/pre-push を追加生成
```

`git push` 直前に 4 つの contract gate (loanword / PUA / language parity /
CHANGELOG unreleased) を full-repo で実行 (~20-30 秒)。`SKIP=...` で commit
時に gate を bypass した場合でも push 前に drift を検出できる。bypass は
`git push --no-verify`。

> **Why:** PR #401 で ruff format 漏れが 2 度起きた (commits 5d9c60e6 / 8dc19e3b)。
> CI 検出 (~12 秒) → 修正 → commit → push の cycle of shame を断つため、
> ローカル commit 時点で format drift を fail-fast。CI 側にも
> `.github/workflows/pre-commit.yml` で同じ hook を gate 化済み (install 忘れ
> しても PR 時に検出)。

> **ruff version pin:** `.pre-commit-config.yaml` の `rev: vX` と
> `.github/workflows/python-lint.yml` / `.github/workflows/ci.yml` の
> `(uv) pip install ruff==X`、加えて `pyproject.toml` の 3 dependency
> group エントリ (合計 6 箇所) は同期必須。
> drift = local-clean / CI-fail mismatch (PR #401 の根本原因の一つ)。
> `scripts/check_ruff_version_sync.py` + `ruff-version-sync.yml` で
> 6 箇所一致を CI gate (本 gate がなければ Dependabot uv-workspace PR が
> pyproject.toml だけ bump → drift → 後追い PR が発生する)。

---

## 学習テンプレート

> 6lang ベース学習は完了済み。新規 FT を行う際は Template B を使う。

### Template A: 事前学習 (Multi-speaker pretraining)

```bash
export WANDB_API_KEY=$(grep WANDB_API_KEY /data/piper/.env | cut -d= -f2) && \
NCCL_DEBUG=WARN NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 \
nohup /data/piper/.venv/bin/python -m piper_train \
    --dataset-dir <DATASET_DIR> \
    --prosody-dim 16 \
    --accelerator gpu --devices 4 --precision 32-true \
    --max_epochs <EPOCHS> --batch-size 20 --samples-per-speaker 2 \
    --checkpoint-epochs 5 --quality medium \
    --base_lr 2e-4 --disable_auto_lr_scaling \
    --ema-decay 0.9995 --max-phoneme-ids 400 --no-wavlm \
    --audio-log-epochs 5 \
    --default_root_dir <OUTPUT_DIR> > training.log 2>&1 &
```

### Template B: シングルスピーカー ファインチューニング

```bash
export WANDB_API_KEY=$(grep WANDB_API_KEY /data/piper/.env | cut -d= -f2) && \
NCCL_DEBUG=WARN NCCL_P2P_DISABLE=1 NCCL_IB_DISABLE=1 \
nohup /data/piper/.venv/bin/python -m piper_train \
    --dataset-dir <FINETUNE_DATASET> \
    --prosody-dim 16 \
    --accelerator gpu --devices 1 --precision 32-true \
    --max_epochs 500 --batch-size 4 --samples-per-speaker 4 \
    --checkpoint-epochs 50 --quality medium \
    --base_lr 2e-5 --disable_auto_lr_scaling \
    --ema-decay 0.9995 --max-phoneme-ids 400 --no-wavlm \
    --val-every-n-epochs 50 --audio-log-epochs 50 \
    --resume-from-multispeaker-checkpoint <BASE_CHECKPOINT> \
    --default_root_dir <OUTPUT_DIR> > training.log 2>&1 &
```

### A vs B 主要差分

B (FT) は A から `--devices 1`、`--base_lr 2e-5` (1/10 で catastrophic forgetting 防止)、`--batch-size 4`、`--max_epochs 500`、`--freeze-dp` 自動有効、`--audio-log-epochs 50` (Validation 頻度に合わせ) を変更。HiFi-GAN ckpt からの resume/FT は v1.12.0 で明示エラー (MB-iSTFT base から再 FT)。

---

## 実装済み機能

### Decoder & 生成


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayutaz/piper-plus](https://github.com/ayutaz/piper-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
