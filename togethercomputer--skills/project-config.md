---
trigger: always_on
description: This repository contains 12 agent skills for the Together AI platform. Each skill is a self-contained directory following the [Agent Skills specification](https://agentskills.io/specification).
---

# AGENTS.md

This repository contains 12 agent skills for the Together AI platform. Each skill is a self-contained directory following the [Agent Skills specification](https://agentskills.io/specification).

## Skill registry

<skills>
- **together-chat-completions**: Real-time and streaming text generation via Together AI's OpenAI-compatible chat/completions API, including multi-turn conversations, tool and function calling, structured JSON outputs, and reasoning models. Reach for it whenever the user wants to build or debug text generation on Together AI, unless they specifically need batch jobs, embeddings, fine-tuning, dedicated endpoints, dedicated containers, or GPU clusters.
- **together-images**: Text-to-image generation and image editing via Together AI, including FLUX and Kontext models, LoRA-based styling, reference-image guidance, and local image downloads. Reach for it whenever the user wants to generate or edit images on Together AI rather than create videos or build text-only chat applications.
- **together-video**: Text-to-video and image-to-video generation via Together AI, including keyframe control, model and dimension selection, asynchronous job polling, and video downloads. Reach for it whenever the user wants motion generation on Together AI rather than still-image generation or text-only inference.
- **together-audio**: Text-to-speech and speech-to-text via Together AI, including REST, streaming, and realtime WebSocket TTS, plus transcription, translation, diarization, timestamps, and live STT. Reach for it whenever the user needs audio in or audio out on Together AI rather than chat generation, image or video creation, or model training.
- **together-embeddings**: Dense vector embeddings, semantic search, RAG pipelines, and reranking via Together AI. Generate embeddings with open-source models and rerank results behind dedicated endpoints. Reach for it whenever the user needs vector representations or retrieval quality improvements rather than direct text generation.
- **together-fine-tuning**: LoRA, full fine-tuning, DPO preference tuning, VLM training, function-calling tuning, reasoning tuning, and BYOM uploads on Together AI. Reach for it whenever the user wants to adapt a model on custom data rather than only run inference, evaluate outputs, or host an existing model.
- **together-batch-inference**: High-volume, asynchronous offline inference at up to 50% lower cost via Together AI's Batch API. Prepare JSONL inputs, upload files, create jobs, poll status, and download outputs. Reach for it whenever the user needs non-interactive bulk inference rather than real-time chat or evaluation jobs.
- **together-evaluations**: LLM-as-a-judge evaluation framework on Together AI. Classify, score, and compare model outputs, select judge models, use external-provider judges or targets, poll results and download reports. Reach for it whenever the user wants to benchmark outputs, grade responses, compare A/B variants, or operationalize automated evaluations.
- **together-sandboxes**: Remote Python execution in managed sandboxes on Together AI with stateful sessions, file uploads, data analysis, chart generation, and notebook-like runs via the Sandboxes API. Reach for it whenever the user wants managed remote Python execution instead of local execution, raw clusters, or full model hosting.
- **together-dedicated-endpoints**: Single-tenant GPU endpoints on Together AI with autoscaling and no rate limits. Deploy fine-tuned or uploaded models, size hardware, and manage endpoint lifecycle. Reach for it whenever the user needs predictable always-on hosting rather than serverless inference, custom containers, or raw clusters.
- **together-dedicated-containers**: Custom Dockerized inference workers on Together AI's managed GPU infrastructure. Build with Sprocket SDK, configure with Jig CLI, submit async queue jobs, and poll results. Reach for it whenever the user needs container-level control rather than a standard model endpoint or raw cluster.
- **together-gpu-clusters**: On-demand and reserved GPU clusters (H100, H200, B200) on Together AI with Kubernetes or Slurm orchestration, shared storage, credential management, and cluster scaling for ML and HPC jobs. Reach for it when the user needs multi-node compute or infrastructure control rather than a managed model endpoint.

</skills>

## Project structure

```
togetherai-skills/
├── AGENTS.md                     # This file — agent instructions
├── README.md                     # Human-facing docs
├── LICENSE                       # MIT
├── quality/
│   └── trigger-evals/            # Skill trigger eval sets
├── scripts/                      # Repo tooling and generators
└── skills/
    └── together-<product>/       # One directory per skill
        ├── SKILL.md              # Required — frontmatter + instructions
        ├── agents/
        │   └── openai.yaml       # Optional — UI metadata for OpenAI/Codex surfaces
        ├── references/           # Optional — detailed reference docs
        │   ├── models.md
        │   ├── api-reference.md
        │   └── ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [togethercomputer/skills](https://github.com/togethercomputer/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
