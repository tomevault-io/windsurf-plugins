---
trigger: always_on
description: This repository is for **AI Interview Coach Agent**, a Java code diagnosis and interview-training system built around an Agent Workflow.
---

# AGENTS.md

## Project Identity

This repository is for **AI Interview Coach Agent**, a Java code diagnosis and interview-training system built around an Agent Workflow.

The project must not drift into a generic LeetCode clone, a generic AI chatbot, a broad education platform, or a Spring Boot wrapper around an LLM API. Its core value is the Agent-driven interview-training loop:

```text
agent task -> planner -> tool call -> observation -> error diagnosis / code review -> weakness memory -> training plan
```

Current product distinction:

- Problem-level layered hints are preset problem content shown on the left problem panel.
- RAG V1 is an internal Agent Tool after code execution observation; it retrieves problem knowledge, backend knowledge cards, and this user's historical learning memory from MySQL before AI diagnosis or AC code review.
- AI diagnosis is generated only after a failed submission and explains this user's current error on the right result panel.
- Accepted submissions may receive lightweight code review through the Agent flow; this is not a full accepted answer generator.
- `hint_record` and legacy hint fields are retained for schema compatibility and future expansion, but the current Agent flow no longer writes new AI hint records and the frontend no longer shows a separate right-side "layered hints" tab.

Primary audience:

- Java backend job seekers
- Student developers preparing for backend interviews
- Interviewers reviewing this as a resume project

Primary resume focus:

- Spring Boot backend design
- Code execution service integration
- Agent Workflow, Tool Calling, Observation, RAG Retrieval, and Memory design
- MySQL data modeling
- MyBatis-Plus mapper and SQL-layer design
- Redis problem hot-cache design with MySQL fallback
- SSE streaming
- Agent step / trace recording
- Clear demo flow

## Source Documents

Use these documents as the source of truth:

- `docs/AI-Interview-Coach.md`: project design, database tables, API design, Agent workflow, resume packaging, interview talking points
- `docs/IMPLEMENTATION_PLAN.md`: implementation phases, directory structure, acceptance criteria, prompts, risks, demo script
- `docs/API.md`: current implemented REST/SSE API surface
- `docs/PROJECT_STATUS.md`: current accomplishments, progress assessment, risks, and next-step outline

If this file conflicts with those documents, prefer this file for engineering constraints and MVP discipline, then update the docs only when the user explicitly asks.

## Current Implementation Status

As of Phase 5 product polish, RAG V1, and learning-memory continuity, the project has a demoable end-to-end Agent workflow, real Dashboard data, a unified Hot100 Solution-mode Java submission model, SSE frontend diagnosis, AC code review branch, clearer frontend separation between preset hints and AI diagnosis, MySQL-backed RAG retrieval, and persistent knowledge self-test / mastery records:

```text
POST /api/submissions
  -> persist original Java submission
  -> wrap current Hot100 Solution-mode problems into Main.java for Piston through CodeWrapper adapters
  -> judge Java submission through Piston

POST /api/agent/analyze
  -> run the Agent workflow synchronously as fallback and API-level demo
  -> failed submissions return error type, knowledge point, diagnosis, improvement suggestion, training plan title, and trace steps
  -> accepted submissions return a lightweight codeReview object and trace steps
  -> frontend does not show hintLevel1/2/3 as a separate right-side tab

GET /api/submissions/{submissionId}/diagnosis/stream
  -> create AgentRun
  -> emit AgentStep events through SSE
  -> rejudge submission through CodeExecutionTool
  -> run RagRetrieveTool after Observation to retrieve problem / knowledge-card / user-memory evidence
  -> for failed submissions: classify error through AI with RAG evidence as supporting context
  -> persist diagnosis, weakness memory, and mistake card
  -> index diagnosis and mistake card into RAG memory after learning records are persisted
  -> create deterministic 3-day training plan with optional RAG-preferred knowledge-card items (optional, failure does not block)
  -> for accepted submissions: run CodeReviewTool with RAG evidence and skip weakness/mistake/training-plan writes
  -> emit final AgentAnalyzeVO

Dashboard and knowledge learning:
  -> Dashboard reads MySQL-backed stats, weaknesses, weakness events, mistake cards, recent submissions, error distribution, and latest training plan
  -> training plan items can be marked PENDING / COMPLETED / SKIPPED
  -> manual training-plan regeneration is exposed through UserController and uses deterministic rules, not LLM generation
  -> knowledge-card self-tests persist self_test_record and update user_knowledge_card_mastery
  -> low-score self-tests may write user_weakness_event with sourceType=SELF_TEST

Frontend SSE integration:
  -> fetch + ReadableStream (not EventSource, for future Authorization header)
  -> real-time step display during analysis, including optional RAG_RETRIEVAL
  -> done event shows final diagnosis result
  -> onEnd fallback ensures loading state is cleared

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyl0828/ai-study](https://github.com/wyl0828/ai-study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
