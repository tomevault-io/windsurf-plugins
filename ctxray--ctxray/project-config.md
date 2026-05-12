---
trigger: always_on
description: See how you really use AI — X-ray your AI coding sessions locally.
---

# ctxray

See how you really use AI — X-ray your AI coding sessions locally.

> Renamed from `reprompt` / `reprompt-cli` on 2026-04-04. PyPI: `ctxray`. CLI: `ctxray`.

## Build & Test

```bash
uv venv && uv pip install -e ".[dev]"
uv run pytest tests/ -v                    # run tests
uv run pytest tests/ -v --cov=ctxray       # with coverage
uv run ruff check src/ tests/              # lint
uv run ruff format src/ tests/             # format
uv run mypy src/ctxray/                    # type check (strict)
uv run python -m build                     # build wheel
```

## Architecture

```
src/ctxray/
├── cli.py              # Typer CLI (scan, import, report, search, demo, status, purge, install-hook, install-extension, extension-status, score, compare, check, insights, digest, style, template [save|list|use], privacy, compress, distill, agent, sessions, repetition, lint, wrapped, telemetry, mcp-serve, rewrite, init, projects, build, feedback) + bare `ctxray` dashboard + plugin loading
├── config.py           # pydantic-settings, env vars (CTXRAY_ prefix) + TOML config
├── demo.py             # Built-in demo data generator (no network required)
├── core/
│   ├── models.py       # Prompt dataclass (auto SHA-256 hash)
│   ├── build.py        # Prompt builder — assemble from components (task, context, files, error, constraints)
│   ├── check.py        # Unified diagnostic — score + lint + rewrite in one pass
│   ├── dedup.py        # Two-layer dedup: exact hash + TF-IDF cosine
│   ├── analyzer.py     # TF-IDF hot terms + K-means clustering
│   ├── library.py      # Pattern extraction + keyword categorization
│   ├── recommend.py    # Prompt recommendations based on history + effectiveness
│   ├── pipeline.py     # Orchestrator: scan → dedup → store → analyze → cluster
│   ├── prompt_dna.py    # PromptDNA dataclass (30+ features per prompt)
│   ├── extractors.py    # Tier 1 feature extraction (regex, <1ms)
│   ├── scorer.py        # Research-calibrated scoring (0-100)
│   ├── segmenter.py     # Three-pass prompt segmentation
│   ├── insights.py      # Personal insights vs research-optimal
│   ├── digest.py        # Two-window comparison for weekly digest
│   ├── style.py         # Personal prompting style fingerprint
│   ├── lang_detect.py   # Language detection (zh/ja/ko/en) via Unicode ranges
│   ├── extractors_zh.py # Chinese feature extraction (jieba + Chinese regex)
│   ├── persona.py       # 6 prompt personas (Architect/Debugger/Explorer/Novelist/Sniper/Teacher)
│   ├── wrapped.py       # WrappedReport dataclass + build_wrapped(db) aggregation
│   ├── privacy.py       # Privacy metadata registry + exposure summary per adapter
│   ├── privacy_scan.py  # Sensitive content detection (API keys, tokens, PII) via regex
│   ├── compress.py      # 4-layer prompt compression (char norm + phrase simplify + filler delete + structure cleanup)
│   ├── suggestions.py   # Command journey suggestions ("→ Try:" hints for 7 core commands)
│   ├── conversation.py  # ConversationTurn, Conversation, DistillResult dataclasses
│   ├── distill.py       # 6-signal importance scoring + filtering + summary generation
│   ├── agent.py         # Agent workflow analysis: error loop detection, tool distribution, efficiency
│   ├── session_quality.py # Session-level composite scoring (0-100) + frustration signals
│   ├── repetition.py    # Cross-session prompt repetition detection via TF-IDF clustering
│   ├── rewrite.py       # Rule-based prompt rewriting (4 layers: compress, front-load, echo, hedging)
│   ├── lint.py          # Configurable prompt linter with .ctxray.toml / pyproject.toml support
│   └── cost.py          # Token cost estimation (locale-aware counting, multi-model pricing)
├── adapters/
│   ├── base.py         # BaseAdapter ABC + parse_conversation() default
│   ├── claude_code.py  # Claude Code JSONL parser (full conversation + tool names)
│   ├── codex.py        # Codex CLI JSONL rollout parser (full conversation + shell/function calls)
│   ├── openclaw.py     # OpenClaw JSON parser (supports ~/.openclaw/ + legacy ~/.opencode/)
│   ├── cursor.py       # Cursor IDE .vscdb parser (cursorDiskKV + legacy ItemTable)
│   ├── aider.py        # Aider markdown chat history parser (.aider.chat.history.md)
│   ├── gemini.py       # Gemini CLI JSON session parser (~/.gemini/tmp/)
│   ├── cline.py        # Cline VS Code agent task parser (globalStorage/saoudrizwan.claude-dev/)
│   ├── chatgpt.py      # ChatGPT conversations.json export parser
│   └── claude_chat.py  # Claude.ai web chat export parser (JSON/ZIP)
├── embeddings/
│   ├── base.py         # BaseEmbedder ABC
│   ├── tfidf.py        # Default (sklearn, zero config)
│   ├── ollama.py       # Optional: pip install ctxray[ollama]
│   ├── local_embed.py  # Optional: pip install ctxray[local] (sentence-transformers)
│   └── openai_embed.py # Optional: pip install ctxray[openai]
├── bridge/
│   ├── protocol.py    # Native Messaging stdio protocol (4-byte length-prefixed JSON)
│   ├── handler.py     # Message handler (ping, sync_prompts, get_status)
│   ├── host.py        # Entry point launched by Chrome/Firefox as subprocess
│   └── manifest.py    # Manifest generator for Chrome/Firefox/Chromium
├── commands/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ctxray/ctxray](https://github.com/ctxray/ctxray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
