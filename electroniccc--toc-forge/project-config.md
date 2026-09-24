---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`toc_forge` extracts a table of contents from a PDF using PaddleOCR layout/OCR models and injects the resulting TOC tree back into the PDF as navigable bookmarks (outline). The code is organized as an installable Python package under `toc_forge/`.

## Environment setup

Source the env script before running:
```powershell
.\.set_paddlex_env.ps1
```
This sets `PADDLE_HUB_HOME`, `PADDLE_PDX_CACHE_HOME`, and disables model source checks. The script checks for `PADDLE_PDX_CACHE_HOME` at startup and refuses to run without it.

The project uses a local `.venv`. Install in editable mode with the ONNX CPU runtime (recommended):
```powershell
uv pip install -e ".[onnx-cpu]"
```
Choose exactly one runtime extra for other backends: `.[onnx-gpu]` or `.[paddle-gpu]`. The dependency groups are maintained in `pyproject.toml`; there is no checked-in `requirements.txt`.
Key dependencies: `paddleocr`, `paddlex`, `opencv-python` (cv2), `PyMuPDF` (fitz), `Pillow`, `numpy`, `scikit-learn`, `openai`.

## Running

After installing the package and the runtime extra matching your chosen engine:
```powershell
toc-forge --input <pdf_path> --output <output_dir> [--model_dir ./models] [--debug]
```

Or without installing:
```powershell
python -m toc_forge --input <pdf_path> --output <output_dir> [--model_dir ./models] [--debug]
```

- `--input`: path to the source PDF
- `--output`: output directory (default: `output directory`)
- `--model_dir`: directory containing PaddleOCR models (default: `./models`)
- `--cache_dir`: OCR result cache directory (default: `./.ocr_cache`)
- `--log_dir`: log directory (default: `log`)
- `--debug`: saves intermediate layout/OCR/parsing results as images and JSON
- `--hash`: print the input file's hash (used to locate OCR cache) and exit
- `--api_base_url`: OpenAI-compatible API base URL (also reads `OPENAI_BASE_URL` env var)
- `--api_key`: API key (also reads `OPENAI_API_KEY` env var)
- `--llm_name`: text LLM model name for the `llm` strategy (e.g. `deepseek-v4-flash`)
- `--vllm_name`: vision LLM model name for the `vllm` strategy (e.g. `qwen3.6-35b-a3b`)
- `--no_toc_cache`: for `llm`/`vllm` strategies, re-call the LLM even if a cached TOC tree exists
- `--device`: device for PaddleOCR inference — `cpu`, `gpu`, `gpu:0`, etc. (default: auto-detect)
- `--llm_timeout`: LLM API request timeout in seconds (default: `600`). Increase if using slow reasoning models.
- `--engine`: inference engine for PaddleOCR models — `paddle`, `paddle_static`, `paddle_dynamic`, `onnxruntime`, etc. (default: PaddleX auto). With `onnxruntime`, required ONNX files are automatically downloaded to the separate `{model_name}_onnx/` directory — see "ONNX Runtime engine" below.
- `--disable_mkldnn`: disable MKLDNN for CPU inference (workaround for the paddle 3.3.1 oneDNN executor crash on Windows, `ConvertPirAttribute2RuntimeAttribute`). Sets `PADDLE_PDX_ENABLE_MKLDNN_BYDEFAULT=false` in `cli.py` **before** importing paddle — the `enable_mkldnn` kwarg is a no-op in paddlex 3.5.2/3.7.2 (verified), the env var is the real switch. Default: MKLDNN on (Linux runs it normally). Only affects the paddle engine — ignored by `onnxruntime`.
- `--ocr_model_size`: OCR det/rec model size — `server` (default, higher accuracy) or `mobile` (much faster on CPU, what the GUI uses). With `--engine onnxruntime`, the CLI automatically downloads `inference.onnx` + `inference.yml` to the chosen `{model_name}_onnx` directory (see "ONNX Runtime engine").
- `--toc_detect_max_page`: max pages to scan for TOC detection (default: `25*3=75`, also capped by the total page count). See "Layout detection" below for the batched scan behavior.

Output: `{output}/{input_stem}_bookmarked.pdf` with injected PDF outline.

## Architecture

The pipeline has five stages (`bookmark_pdf` in `toc_forge/pipeline.py:118`):

`bookmark_pdf` returns a `BookmarkResult` object with `pdf_bookmarks_path`,
`time_cost`, `toc_tree`, `input_tokens`, and `output_tokens` fields. Token
counts are zero for local OCR or an LLM cache hit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [electroniccc/toc_forge](https://github.com/electroniccc/toc_forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
