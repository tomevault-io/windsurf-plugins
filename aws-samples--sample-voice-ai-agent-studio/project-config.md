---
trigger: always_on
description: This file provides guidance to AI coding agents working with this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with this repository.

## Project Overview

Nova Sonic Test Harness — an end-to-end test framework for evaluating **Amazon Nova Sonic** (speech-to-speech model) via AWS Bedrock's bidirectional streaming API. It simulates multi-turn conversations between a configurable user simulator (Bedrock LLMs or scripted inputs) and Nova Sonic, logs all interactions, and provides LLM-as-judge evaluation.

## Setup

- **Python 3.12** (managed via mise, see `.mise.toml`)
- **Install dependencies:** `pip install -r requirements.txt`
- **AWS credentials:** Copy `.env.example` to `.env` and fill in credentials
- No microphone/speaker needed — uses silent audio streaming

### Required AWS IAM Permissions

| Service | Permissions | When Needed |
|---|---|---|
| **Amazon Bedrock** | `bedrock:InvokeModel`, `bedrock:InvokeModelWithResponseStream`, `bedrock:InvokeModelWithBidirectionalStream` | Always (Nova Sonic streaming, user simulator, judge) |
| **Amazon Polly** | `polly:SynthesizeSpeech` | Only when `input_mode: "polly"` |
| **Amazon S3** | `s3:PutObject`, `s3:GetObject`, `s3:DeleteObject`, `s3:CreateBucket` | Only for audio-text consistency evaluation |
| **Amazon Transcribe** | `transcribe:StartTranscriptionJob`, `transcribe:GetTranscriptionJob`, `transcribe:DeleteTranscriptionJob` | Only for audio-text consistency evaluation |

### Key Dependencies

| Package | Purpose |
|---|---|
| `boto3` | AWS SDK for Bedrock Converse API (user sim, judge) |
| `aws_sdk_bedrock_runtime` | Bidirectional streaming with Nova Sonic |
| `rx` (RxPY) | Reactive streams for event-driven audio/text |
| `smithy-aws-core` | Low-level AWS signing for streaming |
| `pyyaml` | Config file loading |
| `python-dotenv` | `.env` file support |
| `pandas` | Results management and CSV export |
| `streamlit` | Evaluation dashboard (optional, for `evaluation/evaluation_dashboard.py`) |
| `plotly` | Dashboard charts (optional, used by dashboard) |

## Directory Structure

```
project_root/
├── main.py                              # Entry point — turn loop orchestration
│
├── core/                                # Core pipeline modules
│   ├── sonic_stream_manager.py          # Bidirectional streaming with Nova Sonic (RxPY)
│   ├── session_continuation.py          # Auto-reconnect on 8-min timeout
│   ├── conversation_history.py          # Message replay for new sessions
│   └── config_manager.py               # Config loading, TestConfig dataclass
│
├── clients/                             # External service wrappers
│   ├── bedrock_model_client.py          # User simulator (Bedrock models + scripted)
│   ├── polly_tts_client.py              # Amazon Polly TTS synthesis
│   └── audio_transcription_client.py    # Amazon Transcribe wrapper
│
├── evaluation/                          # Evaluation and judging
│   ├── llm_judge_binary.py              # Binary YES/NO LLM judge
│   ├── evaluation_types.py              # Shared eval dataclasses
│   ├── evaluate_log.py                  # Evaluate existing conversation logs (also via main.py evaluate)
│   ├── evaluate_audio_text.py           # Audio-text consistency / hallucination detection (also via main.py evaluate-audio)
│   └── evaluation_dashboard.py          # Streamlit dashboard for visualizing results
│
├── logging_/                            # Logging and results (trailing _ avoids stdlib collision)
│   ├── interaction_logger.py            # Per-turn JSON + WAV logging
│   ├── event_logger.py                  # JSONL streaming event debug log
│   ├── results_manager.py              # Session directory organization, indexes, batch summaries
│   └── conversation_audio_recorder.py   # Input/output LPCM + stereo WAV recording
│
├── tools/                               # Tool registry and handler modules
│   ├── tool_registry.py                 # Decorator-based tool registration
│   └── order_status_tools.py            # Example: order status tool handlers
│
├── runners/                             # Execution orchestration
│   ├── multi_session_runner.py          # Batch/parallel session execution
│   └── dataset_loader.py               # JSONL and HuggingFace dataset loading
│
├── utils/                               # Shared utilities
│   ├── model_registry.py               # Model alias → Bedrock model ID resolution
│   └── generate_config.py              # LLM-powered config generator CLI
│
├── configs/                             # Test configurations
│   ├── models.yaml                      # Model alias registry (user, sonic, judge)
│   ├── order_status/                    # Order status scenario variants (5 configs)
│   └── *.json                           # Individual test configs
│
├── scenarios/                           # Complex test scenario configs
│   ├── healthcare/                      # 12 healthcare insurance chatbot scenarios
│   └── banking/                         # 8 retail banking chatbot scenarios
│
├── datasets/                            # Test datasets (JSONL)
│   ├── example_customer_support.jsonl   # Sample customer support scenarios
│   ├── healthcare_eval.jsonl            # Healthcare evaluation dataset
│   └── audiomc_sample_10.jsonl          # AudioMC wide-format sample
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-samples/sample-voice-ai-agent-studio](https://github.com/aws-samples/sample-voice-ai-agent-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
