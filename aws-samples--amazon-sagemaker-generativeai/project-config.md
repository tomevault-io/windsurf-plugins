---
trigger: always_on
description: Navigation guide for the `amazon-sagemaker-generativeai` repo. The authoritative content overview is the top-level `README.md` (incl. the Model Support Matrix); this file captures what isn't obvious from the README and helps you orient quickly.
---

# CLAUDE.md

Navigation guide for the `amazon-sagemaker-generativeai` repo. The authoritative content overview is the top-level `README.md` (incl. the Model Support Matrix); this file captures what isn't obvious from the README and helps you orient quickly.

## What this repo is

A collection of standalone, mostly-Jupyter examples showing Generative AI workflows on Amazon SageMaker. It is **not** a single codebase — each folder under a numbered prefix is an independent example or family of examples with its own dependencies. Don't expect shared utilities, a unified test suite, or cross-folder imports.

## Navigation map

| Folder                             | What lives here                                                                                                                                                                                                                         | Notes                                                                                     |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `0_model_customization_recipes/`   | Config-driven SFT (`supervised_finetuning/`) and preference optimization (`preference_optimization/`) recipes for 20+ FMs (Llama, Qwen, GPT-OSS, DeepSeek, Phi, Gemma)                                                                  | Largest curated catalog. Strategies: QLoRA, Spectrum, Full FT                             |
| `1._getting_started/`              | **Placeholder** — README literally says "placeholder"                                                                                                                                                                                   | Skip unless asked to populate it                                                          |
| `2_end_to_end_genai_on_sagemaker/` | End-to-end MLOps. `2_model_customization/` and `3_inference/` are placeholders; substance is in `4_mlops/` (SM Pipelines + Unified Studio + DataZone, Abalone XGBoost demo)                                                             | Two of the three subfolders are empty                                                     |
| `3_distributed_training/`          | Distributed training deep-dives. Subfolders: `models/`, `reinforcement-learning/` (DPO + GRPO), `nvidia-nemo/`, `diffusers/`, `spectrum_finetuning/`, `unsloth/`, `time-series-forecasting/`, `distributed_training_sm_unified_studio/` | Largest folder. Most diverse stack — DDP, FSDP/FSDP2, DeepSpeed, Ray, vLLM, NeMo, Unsloth |
| `4_rag/`                           | One example: `voyageai-embedding-RAG/` (Voyage AI + OpenSearch KNN + Claude 3 via Bedrock)                                                                                                                                              | Single notebook                                                                           |
| `5_agents/`                        | Agent frameworks: `deepseek_crewai_based_agent/`, `langgraph_model_context_protocol/`, `ml-models-as-agent-tools/`, `sagemaker-strands-agentcore/`, `sagemaker-with-strands-agents/`                                                    | CrewAI, LangGraph + MCP, Strands, Bedrock AgentCore                                       |
| `6_use_cases/`                     | Task/industry recipes: RAG chatbot, text-to-SQL, phishing classification, function-calling SFT+DPO, summarization, summarization-to-image, job governance                                                                               | Smaller and older models (Flan-T5, Falcon, CodeLlama)                                     |
| `7_inference/`                     | `post_training_quantization/` (GPTQ/AWQ); `sagemaker-genai-hosting-examples/` is empty                                                                                                                                                  | Quantization + benchmarking only                                                          |
| `llm-performance-evaluation/`      | `deepseek-r1-distilled/` benchmarking with Ray                                                                                                                                                                                          | Single example                                                                            |
| `x_archive/`                       | 18 legacy Llama-2-era examples                                                                                                                                                                                                          | **Don't use as reference for new work** unless explicitly asked                           |

## Key things to know before changing anything


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/amazon-sagemaker-generativeai](https://github.com/aws-samples/amazon-sagemaker-generativeai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
