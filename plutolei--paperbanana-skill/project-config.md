---
trigger: always_on
description: Use when user needs academic diagrams, methodology figures, statistical plots, or presentation slides from text descriptions or data files. Also use for evaluating generated figures against references.
---


# PaperBanana - Academic Illustration Generator

Multi-agent pipeline (Retriever → Planner → Stylist → Visualizer → Critic) for publication-quality academic diagrams, statistical plots, and presentation slides.

**API key:** Set provider keys in PaperBanana project's `.env` file.
**Timeout:** 300000 (5 min) for all generation commands.

---

## Commands

All commands run from project root: `cd <paperbanana_dir> && python -m paperbanana.cli <cmd>`

### Command Selection Decision Tree

Route user requests to the right subcommand **before** looking up parameters:

| User intent | Signal words | Subcommand |
|-------------|--------------|------------|
| 方法论/架构/流程图 from text or PDF | "method figure", "架构图", "流程图", "methodology", "pipeline diagram", "论文配图" | `generate` |
| Statistical plot from data file | "plot", "curve", "bar chart", "scatter", "heatmap", has CSV/JSON | `plot` |
| Single presentation slide | "slide", "一张幻灯片", "封面图", single prompt file | `slide` |
| Batch slide generation | "all slides", "批量生成", "N 张幻灯片", `prompts/` directory | `slide-batch` |
| Compare generated vs human reference | "evaluate", "对比", "与参考图对比" | `evaluate` |
| Manage reference dataset | "download dataset", "清缓存" | `data` |
| First-time provider config | "setup", "配置 API key" | `setup` |

**Ambiguous input**: If user provides just a description with no subcommand signal, default to `generate` (see Argument Parsing table for details).

**Out-of-scope**: Pure code generation (matplotlib/seaborn script) is NOT paperbanana's job — those go to `matplotlib` / `scientific-visualization` skills. Paperbanana is for AI-driven image generation + critique loops.

> **Note (upstream sync pending):** Upstream `paperbanana` CLI also adds subcommands (`plot-batch` #123, `sweep` #118) not yet reflected in this table. See the [llmsresearch/paperbanana CHANGELOG](https://github.com/llmsresearch/paperbanana) for the authoritative CLI surface.

### `generate` — Methodology Diagrams

```bash
python -m paperbanana.cli generate --input '<file>' --caption '<caption>' --optimize --verbose
```

When user provides inline text (no file): write to temp file, use as `--input`.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--input` / `-i` | — | Path to methodology text file or PDF (`.pdf` requires `pip install 'paperbanana'`) |
| `--caption` / `-c` | — | Figure caption / communicative intent |
| `--output` / `-o` | auto | Output image path |
| `--vlm-provider` | `gemini` | VLM provider: `gemini`, `anthropic`, `openai`, `bedrock`, `openrouter`, `ollama`, `claude_code`, `litellm` |
| `--vlm-model` | auto | VLM model name |
| `--image-provider` | auto | Image gen provider: `google_imagen`, `openai`, `bedrock`, `openrouter` |
| `--image-model` | auto | Image gen model name |
| `--iterations` / `-n` | `3` | Max critic rounds |
| `--auto` | off | Loop until critic is satisfied (safety cap via `--max-iterations`) |
| `--max-iterations` | `30` | Safety cap for `--auto` mode |
| `--optimize` | off | Preprocess inputs (parallel enrichment + caption sharpening) |
| `--continue` | off | Continue from the latest run |
| `--continue-run` | — | Continue from a specific run ID |
| `--feedback` | — | User feedback for the critic when continuing a run |
| `--aspect-ratio` / `-ar` | auto | Target aspect ratio: `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `9:16`, `16:9`, `21:9` |
| `--format` / `-f` | `png` | Output format: `png`, `jpeg`, `webp` |
| `--dry-run` | off | Validate inputs without making API calls |
| `--exemplar-retrieval` | off | Enable external exemplar retrieval before planning |
| `--seed` | — | Random seed for reproducible generation |
| `--verbose` / `-v` | off | Show detailed agent progress and timing |
| `--auto-download-data` | off | Auto-download expanded reference set (~257MB) on first run |
| `--venue` | — | Academic venue style: `neurips`, `icml`, `acl`, `ieee`, `custom` |
| `--pages` | — | Page range for PDF input (e.g., `3-5`) |
| `--config` | — | Path to config YAML file |

> **Venue styles:** `--venue neurips` applies NeurIPS-specific methodology and plot style guides from `data/guidelines/`. Each venue has distinct color palettes, layout conventions, and typography expectations.

> **PDF input:** `--input paper.pdf --pages 3-5` extracts text from the specified pages as source context.

> **Exemplar advanced flags:** `--exemplar-retrieval` enables retrieval; see `generate --help` for additional config flags (`--exemplar-endpoint`, `--exemplar-mode`, `--exemplar-top-k`, `--exemplar-timeout`, `--exemplar-retries`).

### `plot` — Statistical Plots

```bash
python -m paperbanana.cli plot --data '<data.csv>' --intent '<intent>' --optimize --verbose
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--data` / `-d` | — | Path to data file (CSV or JSON) **[required]** |
| `--intent` | — | Communicative intent for the plot **[required]** |
| `--output` / `-o` | auto | Output image path |
| `--vlm-provider` | `gemini` | VLM provider |
| `--iterations` / `-n` | `3` | Refinement iterations |
| `--format` / `-f` | `png` | Output format |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlutoLei/paperbanana-skill](https://github.com/PlutoLei/paperbanana-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
