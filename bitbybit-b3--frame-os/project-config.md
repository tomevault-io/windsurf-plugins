---
trigger: always_on
description: > instructions for any AI coding agent (cursor, claude code, openai codex, copilot, gemini cli, whatever) working in this repo. **read this BEFORE generating any code.** if there's a conflict between this file and your default behavior, this file wins.
---

# AGENTS.md

> instructions for any AI coding agent (cursor, claude code, openai codex, copilot, gemini cli, whatever) working in this repo. **read this BEFORE generating any code.** if there's a conflict between this file and your default behavior, this file wins.

---

## what we're building

**FrameOS** — a desktop-native, chat-first AI video editor. tauri 2 rust desktop talking to a python fastapi backend that runs in k8s (kind locally). all AI is gemini's media stack: gemini 2.5/3.x + veo 3.1 + imagen 4 + nano banana + lyria 3.

killer feature: **auto-edit on upload** — drop raw footage, the backend automatically fires three jobs in parallel (groq whisper transcript, ffmpeg + gemma 3 frame captions into qdrant, gemini 2.5 pro multimodal timeline build). asset is gated until the timeline is ready; one click and the editable JSON timeline materializes in ~45-60 seconds. refine with chat tool calls or mouse, export MP4.

bonus reveal: **crew mode** — toggle the Crew button in the chat header to flip from single-agent chat to a multi-agent pipeline. a planner agent decomposes the user goal, then dispatches to specialists (editor for timeline mutations, captioner for copy). each agent streams its own thinking deltas live as a separate card. backed by `/crew` SSE endpoint + `apps/backend/app/core/crew.py`. crewai-style orchestration without pulling the heavy pypi lib (would violate the gemini-only model-calls rule).

---

## read THESE before writing code

read in this order. they're load-bearing.

1. **`README.md`** — the user's vision in their own voice. constantly evolving. **re-read every time you start a task.**
2. **`docs/PLAN.md`** — shipped-state snapshot + what's left. start here on a fresh checkout.
3. **`docs/DEMO.md`** — recorded-walkthrough script + judging hooks + crew mode reveal in step 5.5 + "do not click X" list.
4. **`docs/DESIGN.md`** — the spec. architecture, feature tiers, auto-edit pipeline, demo script, 20-hr timebox. canonical source for "what we're building" — note: the original spec has an Auto-Edit button but that's been killed; pipeline auto-fires on upload now.
5. **`docs/NOT-DOING.md`** — explicit cuts. if you're tempted to add something, check here first. if your idea is in this file, don't build it without asking the human.
6. **`docs/PRODUCT-DESIGN.md`** — UI / UX brief. design language, visual identity, component inventory, interaction patterns. read this if you're touching the frontend or generating mockups. structured so it can be pasted into claude.ai/design / v0 / figma make / lovable directly.

---

## the stack (locked)

| layer | tech |
|---|---|
| desktop shell | **tauri 2** (rust) |
| desktop UI | vite + react + typescript + tailwind + shadcn/ui + zustand + **remotion player** |
| desktop chat | vercel ai sdk v5 with custom transport hitting backend `/chat` |
| backend service | **python 3.12 + fastapi + pydantic v2 + google-genai** |
| video processing | ffmpeg-python (server-side, in the backend container) |
| storage | **SeaweedFS** in-cluster, S3-compatible (minio CE was archived apr 2026 — seaweedfs is the production successor used by Kubeflow Pipelines) |
| local orchestration | **kind** + **skaffold** + plain k8s yaml (no helm) |
| ci | github actions; `tauri-apps/tauri-action` for cross-platform builds |
| external APIs | gemini 2.5/3.x · veo 3.1 fast · imagen 4 fast · nano banana · lyria 3 realtime · groq whisper · pexels videos |

if you want to swap any of these, **ask the human first**. do not silently introduce a new dependency.

---

## one-time local setup

```bash
# prerequisites (macOS — adapt for linux)
brew install kind kubectl skaffold docker rustup uv pnpm
rustup default stable
docker login   # if your kind config pulls private images

# clone + enter
git clone <this repo>
cd frameos

# spin up the cluster
kind create cluster --name frameos --config infra/kind-config.yaml
kubectl create namespace frameos

# load secrets (replace with your real keys)
kubectl -n frameos create secret generic api-keys \
  --from-literal=GEMINI_API_KEY=$GEMINI_API_KEY \
  --from-literal=GROQ_API_KEY=$GROQ_API_KEY \
  --from-literal=PEXELS_API_KEY=$PEXELS_API_KEY

# apply manifests
kubectl apply -f infra/k8s/

# install backend + desktop deps
cd apps/backend && uv sync && cd ../..
cd apps/desktop && pnpm install && cd ../..
```

---

## day-to-day commands

```bash
# terminal 1 — backend live-reload in the cluster
cd infra && skaffold dev

# terminal 2 — desktop app
cd apps/desktop && pnpm tauri dev

# terminal 3 — tail backend logs
kubectl -n frameos logs -f deploy/backend

# run tests
cd apps/backend && uv run pytest -x                     # python
cd apps/desktop && pnpm exec playwright test            # e2e (chromium against vite)
cd apps/desktop/src-tauri && cargo test                 # rust

# lint + format
cd apps/backend && uv run ruff check . && uv run ruff format .
cd apps/desktop/src-tauri && cargo fmt && cargo clippy -- -D warnings

# typecheck
cd apps/desktop && pnpm exec tsc --noEmit
```

---

## coding standards — Google style guides


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BitByBit-B3/frame-os](https://github.com/BitByBit-B3/frame-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
