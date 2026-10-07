---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**local-llm**

Apple Silicon Mac에서 MLX 기반 로컬 LLM을 OpenAI 호환 API 서버로 실행하는 프로젝트. Qwen3.6-27B(기본 멀티모달 VLM)와 SuperGemma4-26B(보조 무검열 모델)를 서버 시작 시 선택해 실행하며, openclaw 및 OpenAI SDK 호환 클라이언트에 로컬 추론 API를 제공한다.

**Core Value:** openclaw 등 OpenAI SDK 호환 클라이언트가 이미지를 포함한 메시지를 보내면 로컬에서 완전히 처리되어 응답이 돌아온다.

### Constraints

- **Tech stack**: Python 3.11 + MLX 0.31+ + FastAPI — 기존 스택 유지, mlx-vlm 추가
- **Memory**: M5 Pro 64GB, 모델 1개만 로드 (15~20GB 점유)
- **API 호환성**: OpenAI SDK 호환 포맷 유지 필수 (클라이언트 변경 없이 동작해야 함)
- **플랫폼**: Apple Silicon 전용 (mlx, mlx-vlm 모두 Apple Silicon 필수)
<!-- GSD:project-end -->

## README 이중 언어 규칙

- `README.md`(영어, 기본)와 `README.ko.md`(한국어)는 항상 동기화 상태를 유지한다.
- README 내용을 수정할 때는 반드시 **두 파일 모두** 같은 변경을 반영할 것.
- 상단의 언어 전환 링크(`한국어 | English`)는 양쪽 파일에서 유지한다.

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Python 3.11 - All application logic, API server, inference integration
- Shell (Bash) - Setup automation and server launch scripts
## Runtime
- Python 3.11.14 (Apple Silicon specific - requires 3.10+)
- Virtual environment: `.venv/` (isolated dependency management)
- pip - Python package management
- Lockfile: No explicit lock file (setup.sh pins mlx-vlm and fastapi versions)
## Frameworks
- **MLX** 0.31.1 - Apple Silicon native ML framework (GPU acceleration via Metal)
- **FastAPI** 0.135.2 - Async HTTP API server for OpenAI-compatible endpoints
- **Uvicorn** 0.42.0 - ASGI server runner for FastAPI
- **pytest** 9.0.2 - Unit testing framework
- **NumPy** 2.4.3 - Numerical computing (MLX dependency)
- **transformers** 5.0.0+ - Model loading and tokenization (mlx-vlm dependency)
- **sentencepiece** - Tokenization library (mlx-vlm dependency)
## Key Dependencies
- mlx-vlm 0.6.8 - Provides load(), generate(), stream_generate(), Qwen MTP, APC prefix caching, and VLM inference support
- mlx 0.31.1 - Core computation engine, mx.clear_cache() for KV cache management
- fastapi 0.135.2 - HTTP API framework for /v1/chat/completions and /v1/models endpoints
- transformers 5.0.0+ - Tokenizer.apply_chat_template() for prompt formatting with enable_thinking support
- pyyaml - Config parsing for model definitions
- jinja2 - Chat template rendering engine
- protobuf - Message serialization (transformers dependency)
- numpy 2.4.3 - Array operations (MLX dependency)
## Configuration
- `HF_HOME` - Hugging Face cache directory (default: `~/.cache/huggingface`)
- `profiles/config-qwen36-27b-262k.json` - Default Qwen3.6-27B config with 262K token context window
- `profiles/config-qwen36-27b-1m.json` - Extended context (1M tokens) with YaRN rope scaling for Qwen3.6-27B
- `setup.sh` - Automated environment setup (Python check, venv creation, pip install)
- `llm-server.sh` - API server launcher with profile switching and thinking mode control
- `llm-chat.sh` - Interactive CLI chat launcher with context profile support
## Platform Requirements
- **Architecture:** Apple Silicon only (arm64 - M1/M2/M3/M4/M5)
- **Memory:** Minimum 24GB RAM (16GB for lighter models like Qwen3.5-9B)
- **Disk:** 20GB+ free (model caching at ~/.cache/huggingface/hub/)
- **Python:** 3.10+ (tested with 3.11.14)
- **macOS:** Native support for sleep prevention via caffeinate command
- Single Apple Silicon Mac (M-series)
- Network accessible via localhost or local network IP
- Tailscale integration supported for remote access (no public internet exposure)
## Logs & Artifacts
- Location: `logs/` directory (auto-created)
- Format: JSONL (one JSON object per line, date-partitioned)
- Contents: Request IP, thinking mode, token counts, duration, content preview
- Filename: `YYYY-MM-DD.jsonl` (new file per calendar day)
- Location: `~/.cache/huggingface/hub/models--mlx-community--Qwen3.6-27B-6bit/`
- Size: ~22.8GB (quantized to 6-bit)
- Auto-downloaded on first run via mlx-vlm
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- Kebab-case for executable Python scripts: `llm-api-server.py`, `llm-proxy.py`, `llm-server.sh`, `llm-chat.sh`
- Snake_case for test files: `test_api_server.py`, `test_proxy.py`
- Avoid underscores in filenames for production code; reserve for tests
- Snake_case for all functions: `parse_request()`, `normalize_messages()`, `strip_thinking()`, `get_log_file()`, `log_entry()`
- Descriptive verbs: `make_completion_response()`, `make_chunk()`, `run_inference()`, `run_inference_streaming()`
- Private/internal functions prefixed with underscore: `_stream_response()`, `_produce()`
- Snake_case for all variables: `model`, `processor`, `model_id`, `gpu_semaphore`, `pending`, `req_id`, `req_data`, `resp_data`
- All-caps for module-level constants: `MAX_QUEUE`, `LOG_DIR`, `DEFAULT_THINKING`, `BACKEND_PORT`, `PROXY_PORT`, `_SENTINEL`
- English names (no Korean) for variables, except in docstrings and comments
- No explicit type hints in most function signatures; only on critical functions: `parse_request(data: dict) -> dict`
- Dataclasses for structured mock data: `@dataclass class MockResponse`
- Dictionary literals for configuration/response objects
## Code Style
- 4-space indentation (Python standard)
- No explicit formatter configured (no .black, .flake8, or .isort files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeeKiYoung/local-llm](https://github.com/LeeKiYoung/local-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
