---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes: APIs, conventions, and file structure may all differ from your training data. Before writing Next.js code, prefer checking the project's actual `app/` router structure and installed Next version docs. If `node_modules/` is present locally, you may also consult the bundled docs under `node_modules/next/dist/docs/`.
<!-- END:nextjs-agent-rules -->

# AI Agents & Token Architecture

RemoteDevsBR relies heavily on Large Language Models (LLMs) to power the "Free Tools" tier (which acts as the top of our funnel) and to power the forthcoming AI-driven candidate matching engine.

This document outlines the architecture, prompts, and token management strategies used for these agents.

## 1. Resume Analyzer Agent

The core growth engine for the platform. Developers upload a PDF, we extract the text, and an LLM processes it into structured feedback.

**Location:** `supabase/functions/analyze-resume/`

**Provider:** OpenAI-compatible chat completions endpoint via `_shared/ai.ts` (defaults to OpenRouter at `https://openrouter.ai/api/v1`). Configured with the `OPENAI_API_KEY` and optional `OPENAI_BASE_URL` edge secrets.

**Model Choice (current):** Free-tier OpenRouter models with ordered fallback. Each AI tool passes `models: FREE_MODELS` from `_shared/ai.ts`: `nvidia/nemotron-3-ultra-550b-a55b:free` (best), then `dots-studio/dots-3-note-preview:free`, `nvidia/nemotron-3-super-120b-a12b:free`, `google/gemma-4-31b-it:free`, `google/gemma-4-26b-a4b-it:free` (worst). On HTTP 429/403/404/5xx the shared client (`_shared/ai.ts`) retries with backoff, then falls through to the next model.
* **Why free tier?** Zero cost while the account carries no OpenRouter credits. Resume analysis requires reading ~1,000-2,000 tokens of raw text and generating a few hundred tokens of structured JSON. Free-tier `:free` models impose daily request limits, lower rate limits, and can be temporarily saturated upstream; revisit paid Flash-tier models if throttling or quality regresses.

**Token Flow:**
1. **Input (Prompt + Payload):** ~1,500 - 3,000 tokens depending on resume length.
2. **Output (JSON):** ~400 - 600 tokens containing:
   * `overall_score` (0-100)
   * `top_strengths` (Array of strings)
   * `top_gaps` (Array of strings)
   * `suggested_roles` (Array of strings)
   * `detailed_feedback` (Long-form string masked behind an email gate)

**Prompt Strategy:**
We use a zero-shot prompt with strict JSON schema enforcement to ensure the output maps cleanly to our React frontend state and Postgres tables.

## 2. Token & Cost Economics

Because the Resume Analyzer is a **Free Tool** used to drive top-of-funnel acquisition, managing token costs is paramount.

* **Cost Optimization:** We use a free-tier model via OpenRouter to keep per-request costs at zero. Avoid hardcoding cost-per-resume assumptions in product logic; treat pricing as a deploy-time/config concern that can change with provider/model.
* **Gate Strategy:** While the compute is free, the *value* of the full report is high. We present the user with the partial analysis (`overall_score` + `top_strengths`), but require them to create an account and complete their profile to unlock the `detailed_feedback`. This converts cheap LLM tokens into high-value structured profile data.

## 3. Future Agent: Recruiter Matchmaker (Phase 6)

The upcoming Phase 6 of the strategic sequence involves building an AI-driven matching engine.

**Goal:** Allow recruiters to describe their ideal candidate in natural language (e.g., "I need a Senior React dev who knows AWS and has fluent English, preferably with Fintech experience").

**Proposed Architecture:**
1. **Embeddings Agent:** Converts candidate stack, goals, and bio into vector embeddings stored in Postgres using `pgvector`.
2. **Retrieval-Augmented Generation (RAG):** When a recruiter searches:
   * The query is embedded.
   * Supabase performs a vector similarity search to retrieve the top 50 candidates.
   * An LLM (Flash-tier by default, Pro-tier if needed) synthesizes the top 5 matches, providing a brief explanation for *why* they are a good fit.

**Token Considerations for Matching:**
* Candidate data must be pre-summarized before embedding to reduce dimensionality and noise.
* Real-time generation for recruiters can afford higher latency/cost models if needed (since recruiters are paying subscribers), but RAG with a free-tier model is likely sufficient.

## 4. Edge Function Security & Best Practices

* **No direct client-to-LLM calls:** All AI requests route through Supabase Edge Functions. The `OPENAI_API_KEY` is securely stored in Supabase Vault/Secrets and is never exposed to the frontend.
* **Rate Limiting:** We currently rate-limit resume uploads per anonymous session to prevent abuse of the API key.
* **JSON Validation:** LLM responses are parsed and sanitized before being inserted into the `resume_analyses` table to prevent injection or schema-breaking errors on the frontend.

---

## 5. Project Map (Routes & Endpoints)

This serves as a quick-reference map for all major touchpoints within the architecture.

### Developer Frontend Routes
* `/` - Landing Page & Top-of-Funnel pitch.
* `/auth` - Signup/Login flow.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [william-keller/remotedevsbr](https://github.com/william-keller/remotedevsbr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
