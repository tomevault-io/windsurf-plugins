---
trigger: always_on
description: AdClaim answers marketing-compliance questions ("Do influencers have to disclose paid
---

# AGENTS.md — AdClaim

## Project summary
AdClaim answers marketing-compliance questions ("Do influencers have to disclose paid
posts?", "Can I say 'clinically proven'?") with a grounded, cited answer drawn only from
official public-domain US FTC advertising guidance, or a clean refusal when the question is
out of scope. The portfolio point is the eval: a faithfulness-verifier agent (CrewAI) is
measured, via Ragas plus custom refusal metrics, against a plain retrieve-then-answer
baseline to show it measurably cuts ungrounded answers and improves refusal accuracy.

## Stack
Python 3.13, CrewAI (orchestration), Chroma (local vector store), an embedding model
(OpenAI `text-embedding-3-small` or an open alternative), one or two chat LLMs, Ragas (RAG
eval), pytest, python-dotenv. No UI.

Full spec: `docs/agent-handoff.md`. Read it before resuming work.

## Phase checklist
- [x] Task 0 — continuity setup (this file, git init)
- [x] Phase 1 — scaffold + hygiene
- [x] Phase 2 — corpus ingest + index
- [x] Phase 3 — CrewAI crew
- [x] Phase 4 — eval harness
- [x] Phase 5 — README

## Resuming work (any agent)
1. Read `AGENTS.md` (this file) and `docs/agent-handoff.md` (full spec).
2. Run `git log --oneline` and `pytest` to see what's done and what's green.
3. Check `docs/sprint-handoff.md` for the active sprint contract (agent-agnostic:
   Claude Code, Codex, or any coding agent); resume there, else at the first unchecked
   phase above.

## Macro planning docs (managed by the Cowork macro layer; do not edit from sprints)
- `docs/planning/product-loop-map.md` — chunk sequence + status
- `docs/planning/loops/` — canonical loop specs
- `docs/planning/operating-context.md` — two-layer workflow conventions
- Sprint evidence goes to `docs/sprints/output/<sprint>-output.md`.

## Notes / API surprises
- Confirmed installable versions (Python 3.13): crewai 1.15.2, chromadb 1.1.1, ragas 0.4.3,
  openai 2.45.0, langchain-openai 0.3.35, langchain-core 0.3.86, langchain 0.3.30,
  langchain-community 0.3.27, langchain-text-splitters 0.3.11. Ragas 0.4.3 still imports
  `langchain_community.chat_models.vertexai`, removed in langchain-community 0.4.x, so the
  whole langchain-* stack is pinned to the last mutually compatible 0.3.x line. crewai has no
  langchain dependency (uses litellm), so this pin only constrains ragas's evaluator LLM
  wrapper.
- Confirmed CrewAI API (1.15.2): `Agent`/`Task`/`Crew`/`Process.sequential`, custom tools via
  `crewai.tools.BaseTool` subclass with `args_schema`, structured task output via
  `Task(output_pydantic=...)`.
- Confirmed Ragas API (0.4.3): `from ragas import evaluate, SingleTurnSample,
  EvaluationDataset`; `from ragas.metrics import Faithfulness, AnswerRelevancy,
  ContextPrecision, ContextRecall` instantiated with `llm=LangchainLLMWrapper(ChatOpenAI(...))`.
  This path is deprecated in favor of `ragas.metrics.collections` (which wants an
  `InstructorBaseRagasLLM` + explicit embeddings) but is still the documented, working
  pre-1.0 API and is what Phase 4 uses.
- Confirmed Chroma API (1.1.1): `chromadb.PersistentClient(path=...)`,
  `get_or_create_collection`, `add`/`upsert`/`query` with explicit `embeddings=` /
  `query_embeddings=` (we compute OpenAI embeddings ourselves rather than using Chroma's
  built-in embedding function).
- Phase 2 corpus already downloaded to `data/corpus/` (8 FTC documents + `manifest.json`,
  retrieved 2026-07-10) ahead of the Phase 1 close-out sprint that formally gates it; left
  uncommitted per that sprint's out-of-scope rule ("corpus downloads") rather than folded into
  the Phase 1 commit. The next Phase 2 loop should pick these files up as-is instead of
  re-downloading. One provenance wrinkle worth knowing: the first Endorsement Guides PDF found
  by search was the 2022 *proposed* rule, not the current one; the manifest correctly points at
  `p204500_endorsement_guides_in_2023.pdf`, the adopted final rule.
- Phase 2 complete: `python -m adclaim.ingest` builds 1,012 chunks into Chroma collection
  `adclaim_ftc_guidance` with stable IDs (`<doc-slug>#<chunk-idx>`), explicit OpenAI
  `text-embedding-3-small` embeddings, and required metadata `{source_title, source_url,
  section}` on every chunk. Per-document counts: Endorsement Guides PDF 248, Endorsement FAQ
  159, .com Disclosures 136, Advertising FAQs 90, Health Products Compliance Guidance 190,
  Green Guides 107, Made in USA 52, CAN-SPAM 30. Metadata verification reported
  `1012 chunks, 0 missing source_title/source_url/section`; the influencer disclosure smoke
  query returned the Endorsement Guides FAQ as the top hit. `data/index/` remains ignored.
- Phase 3 complete: `src/adclaim/crew.py` builds a sequential crew (planner, retriever with
  the `ftc_guidance_search` BaseTool, answerer, and an optional faithfulness verifier) via
  `build_crew(question, use_verifier=...)`; `python -m adclaim.ask "question"` prints the
  `AskResult` JSON (`answer`, `citations[]` of `{source_title, section}`, `refused`) to stdout
  and `verifier: on|off` to stderr, non-zero exit on failure. The refusal contract is enforced
  in plain Python, not by the LLM's self-report: the verifier task outputs a separate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whoisbe/AdClaim](https://github.com/whoisbe/AdClaim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
