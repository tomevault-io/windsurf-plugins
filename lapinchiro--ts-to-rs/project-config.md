---
trigger: always_on
description: TypeScript → Rust conversion codemod CLI tool.
---

# CLAUDE.md

TypeScript → Rust conversion codemod CLI tool.

## Response Language

Always respond to the user in **Japanese**. Commit messages must also be in **Japanese**. Code, comments, and documentation may be in English, but conversational responses and commit messages must be in Japanese.

## Tech Stack

- **Language**: Rust
- **TS parsing**: swc_ecma_parser + swc_ecma_ast
- **CLI**: clap
- **Testing**: cargo test + insta (snapshots)
- **Lint**: clippy
- **Formatting**: rustfmt

## Key Commands

```bash
cargo build                # debug build
cargo build --release      # release build
cargo check                # fast type check
cargo test                 # run all tests
cargo fix --allow-dirty --allow-staged  # auto-fix unused imports etc.
cargo clippy --all-targets --all-features -- -D warnings  # lint
cargo fmt --all --check    # format check
cargo llvm-cov --ignore-filename-regex 'main\.rs' --fail-under-lines 89  # coverage (threshold 89%, excluding main.rs)
cargo llvm-cov --html                  # generate HTML report (target/llvm-cov/html/)
./scripts/check-file-lines.sh        # .rs file line count check (threshold: 1000 lines)
./scripts/hono-bench.sh              # Hono conversion rate benchmark (directory mode)
./scripts/hono-bench.sh --both       # both directory + single-file modes
```

### Hono Benchmark

Measures Hono framework conversion success rate. Run after conversion feature changes to quantify impact.

- **Run**: `./scripts/hono-bench.sh` (verifies `cargo build --release` and auto-clones Hono repo)
- **Analysis**: `scripts/analyze-bench.py` auto-invoked, aggregating error JSON by category
- **History**: `bench-history.jsonl` (JSONL, one line per run). View with: `python3 -c "import sys,json; [print(f\"{json.loads(l)['timestamp'][:10]} clean={json.loads(l)['clean_pct']}% errors={json.loads(l)['error_instances']}\") for l in open('bench-history.jsonl')]"`
- **Error JSON**: `/tmp/hono-bench-errors.json`
- **Error inspection**: `scripts/inspect-errors.py` — エラーの詳細分析ツール（下記参照）

**Note**: "Clean" = zero conversion errors (`--report-unsupported` with 0 errors), separate from whether generated Rust compiles.

### Error Inspection

`/tmp/hono-bench-errors.json` の詳細分析には `scripts/inspect-errors.py` を使用する。アドホックな Python ワンライナーでの解析は禁止。

```bash
python3 scripts/inspect-errors.py                        # カテゴリ別集計
python3 scripts/inspect-errors.py --kind TYPEOF          # kind で部分一致フィルタ
python3 scripts/inspect-errors.py --category TYPEOF_TYPE # カテゴリで完全一致フィルタ
python3 scripts/inspect-errors.py --file client          # ファイル名で部分一致フィルタ
python3 scripts/inspect-errors.py --source               # エラー箇所の TS ソースを表示
python3 scripts/inspect-errors.py --discriminant --source # Discriminant エラーの AST ノード種を推定
python3 scripts/inspect-errors.py --raw                  # フィルタ後の JSON 出力
```

フィルタは組み合わせ可能（例: `--category INDEXED_ACCESS --source`）。不足機能があれば同スクリプトに追加する。

## Architecture

See [README.md](README.md#ディレクトリ構成) for directory structure.

```
TS source → Parser (SWC AST)
  → ModuleGraph (import/export analysis)
  → TypeCollector + TypeConverter (build TypeRegistry)
  → TypeResolver (pre-compute expression types, expected types, narrowing)
  → Transformer (AST + type info → IR)
  → Generator (IR → Rust source code)
  → OutputWriter (file output, mod.rs generation)
```

## Shared Agent Docs

The Claude and Codex environments are intended to coexist.

- Shared guidance for both agents lives under `doc/agent/`
- Codex entrypoint is `AGENTS.md`
- Claude-specific rules remain under `.claude/`
- Codex-specific settings live under `.codex/` and `.agents/skills/`

## Core Principles

- **Ideal implementation**: Pursue the logically most ideal implementation regardless of cost. No compromises, no ad-hoc solutions. "Too much effort" and "good enough for now" are not valid justifications
- **KISS**: Minimal complexity for current requirements. When conflicting with "ideal implementation", prioritize the ideal
- **YAGNI**: Implement only what is needed now. No unrequested features or extensions
- **DRY + Orthogonality**: DRY eliminates duplication of *knowledge*, not *code appearance*. Keep duplication if sharing increases coupling

## Code Conventions

- `unwrap()` / `expect()` only in test code — see `.claude/rules/testing.md`
- `unsafe` prohibited (requires documented reason + user approval)
- `clone()` acceptable initially; leave TODO comment for unnecessary clones
- Public types/functions must have doc comments (`///`)

## Quality Standards

Maintain **0 errors, 0 warnings** for all changes. Run /quality-check upon work completion.

Coverage threshold ratchet: when measured coverage exceeds threshold by 2+ points, raise threshold by 1 point.

## Code of Conduct

- **Ideal implementation primacy** — 本プロジェクトの最上位目標は「理想的な TS→Rust トランスパイラの獲得」。ベンチ数値は defect 発見のシグナルであり最適化ターゲットではない。Structural fix > interim patch。詳細は `.claude/rules/ideal-implementation-primacy.md`
- **Uncertainty-driven investigation** — 不確定要素は一級市民として TODO に `[INV-N]` 形式で記録し、影響範囲が絞れるまで調査を尽くしてから実装に進む。`todo-prioritization.md` Step 0 参照
- **No unilateral conversion feasibility judgments** — "difficult in Rust" is never a valid reason to defer or deprioritize. Applies across all phases: TODO, plan.md, PRD. See `.claude/rules/conversion-feasibility.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lapinChiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
