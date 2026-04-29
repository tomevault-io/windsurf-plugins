---
trigger: always_on
description: - Keep runs sequential per provider endpoint. Parallelize only across different providers or different OpenAI-compatible base URLs.
---

# AGENTS.md

## Port-to-Port Benchmark Ops

### Core rules
- Keep runs sequential per provider endpoint. Parallelize only across different providers or different OpenAI-compatible base URLs.
- Do not use fail-fast wrappers such as `set -e` for multi-run workers. One failed run must not block later runs.
- Always pass `--log-json runs/<run-stem>.json` for any run that may be judged or compared later.
- Always capture console output to `runs/<run-stem>.log` or a worker log with `tee`.
- A failed run is still benchmark data if the raw JSON exists. Judge it instead of discarding it.
- Default benchmark settings are `--max-turns 50` and `--function-call-timeout-secs 20`.
- If `run_repeat_clean_config.sh` is not executable on this checkout, invoke it as `bash ./run_repeat_clean_config.sh ...`.

### Environment keys
- Do not `source` the repo `.env` wholesale. Extract only the needed key.
- Anthropic:
```bash
ANTHROPIC_API_KEY="$(rg --no-line-number '^ANTHROPIC_API_KEY=' /home/khkramer/src/gb-benchmarks/.env | cut -d= -f2-)"
```
- OpenAI / OpenAI-compatible:
```bash
OPENAI_API_KEY="$(rg --no-line-number '^OPENAI_API_KEY=' /home/khkramer/src/gb-benchmarks/.env | cut -d= -f2-)"
```
- Google:
```bash
GOOGLE_API_KEY="$(rg --no-line-number '^GOOGLE_API_KEY=' /home/khkramer/src/gb-benchmarks/.env | cut -d= -f2-)"
```

### Shared command pattern
- Work from `port-to-port`:
```bash
cd /home/khkramer/src/gb-benchmarks/port-to-port
```
- Natural prompt:
```bash
--task-variant natural
```
- Literal prompt:
```bash
--task-variant literal
```
- Common flags:
```bash
--max-turns 50 --function-call-timeout-secs 20 --log-json runs/<run-stem>.json
```

### Exact run commands

#### Anthropic
- Claude Sonnet 4.6 none:
```bash
ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider anthropic --model claude-sonnet-4-6 \
  --task-variant natural --thinking none \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/claude-sonnet-4-6-natural-none-<ts>.json \
  > runs/claude-sonnet-4-6-natural-none-<ts>.log 2>&1
```
- Claude Sonnet 4.6 low:
```bash
ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider anthropic --model claude-sonnet-4-6 \
  --task-variant natural --thinking low \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/claude-sonnet-4-6-natural-low-<ts>.json \
  > runs/claude-sonnet-4-6-natural-low-<ts>.log 2>&1
```
- Claude Sonnet 4.6 medium:
```bash
ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider anthropic --model claude-sonnet-4-6 \
  --task-variant natural --thinking medium \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/claude-sonnet-4-6-natural-medium-<ts>.json \
  > runs/claude-sonnet-4-6-natural-medium-<ts>.log 2>&1
```
- Claude Haiku 4.5 low:
```bash
ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider anthropic --model claude-haiku-4-5-20251001 \
  --task-variant natural --thinking low \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/claude-haiku-4-5-natural-low-<ts>.json \
  > runs/claude-haiku-4-5-natural-low-<ts>.log 2>&1
```
- Claude Haiku 4.5 medium:
```bash
ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider anthropic --model claude-haiku-4-5-20251001 \
  --task-variant natural --thinking medium \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/claude-haiku-4-5-natural-medium-<ts>.json \
  > runs/claude-haiku-4-5-natural-medium-<ts>.log 2>&1
```

#### Hosted OpenAI
- GPT 5.4 none:
```bash
OPENAI_API_KEY="$OPENAI_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider openai --model gpt-5.4 \
  --task-variant natural --thinking none --max-tokens 4096 \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/gpt-5.4-natural-none-<ts>.json \
  > runs/gpt-5.4-natural-none-<ts>.log 2>&1
```
- GPT 5.4 low:
```bash
OPENAI_API_KEY="$OPENAI_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider openai --model gpt-5.4 \
  --task-variant natural --thinking low --max-tokens 4096 \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/gpt-5.4-natural-low-<ts>.json \
  > runs/gpt-5.4-natural-low-<ts>.log 2>&1
```
- GPT 5.4 medium:
```bash
OPENAI_API_KEY="$OPENAI_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider openai --model gpt-5.4 \
  --task-variant natural --thinking medium --max-tokens 4096 \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/gpt-5.4-natural-medium-<ts>.json \
  > runs/gpt-5.4-natural-medium-<ts>.log 2>&1
```
- GPT 5.2 medium:
```bash
OPENAI_API_KEY="$OPENAI_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider openai --model gpt-5.2 \
  --task-variant natural --thinking medium \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/gpt-5.2-natural-medium-<ts>.json \
  > runs/gpt-5.2-natural-medium-<ts>.log 2>&1
```
- GPT 5.1 low:
```bash
OPENAI_API_KEY="$OPENAI_API_KEY" .venv/bin/python mini-rl-env.py \
  --provider openai --model gpt-5.1 \
  --task-variant natural --thinking low \
  --max-turns 50 --function-call-timeout-secs 20 \
  --log-json runs/gpt-5.1-natural-low-<ts>.json \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pipecat-ai/gb-benchmarks](https://github.com/pipecat-ai/gb-benchmarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
