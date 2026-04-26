---
trigger: always_on
description: A mobile-first read-later app combining incremental reading, user knowledge modeling, and algorithmic article selection. Named after Francesco Petrarca, pioneer of systematic reading methods. Built for one power user (Stian), not a social platform.
---

# Petrarca — Intelligent Read-Later App

A mobile-first read-later app combining incremental reading, user knowledge modeling, and algorithmic article selection. Named after Francesco Petrarca, pioneer of systematic reading methods. Built for one power user (Stian), not a social platform.

**Frontend**: Expo SDK 54 (React Native), 2-tab layout (Feed / Library) + ✦ drawer. **Backend**: Hetzner VM — nginx (:8083 content, :8084 web), research-server.py (:8090), 4-hour cron pipeline. **Data**: SQLite (petrarca.db, canonical), JSON fallback.

## Design Principles & North Star

These principles are the intellectual foundation of the project. They override implementation convenience. Read the linked docs when making design decisions beyond bug fixes.

1. **"Hooks, not facts."** Reading success = building frameworks (Caesar, Alexander, Charlemagne), not fact-drilling. The system builds scaffold knowledge that makes everything you read richer. *(research/design-vision.md)*

2. **"I'll manage your memory."** The fear of forgetting stops reading nonfiction. Like Michel Thomas: the system takes responsibility for retention so the user can focus on reading. Frame knowledge maps as positive progress, never anxiety-inducing gaps. *(memory/feedback_michel_thomas.md)*

3. **Comprehension before memory.** Ensure understanding before testing (Matuschak's key finding). SRS-style flashcards fail for conceptual knowledge — we need elaborative retrieval, connection-based resurfacing, spreading activation. *(research/andy-matuschak-research.md, research/beyond-flashcards-knowledge-retention.md)*

4. **Atomic claims are the fundamental unit.** Not articles, not words. Claims enable cross-article tracking, knowledge state management, and novelty detection. Delta-only reports show what's new since last read. *(research/novelty-system-architecture.md)*

5. **Curriculum as bridge.** Don't match claims directly across sources. Curriculum nodes (50-80 per domain, Opus-generated) are the organizing principle — they connect books, articles, and review into a coherent structure. *(research/overlapping-curricula-vision.md)*

6. **Temporal hooks are the key retention mechanism.** Priority: (1) anchor to known events, (2) same-moment connections, (3) causal chains, (4) cross-domain surprises only if the reader knows the other domain. *(memory/feedback_temporal_hooks.md)*

7. **Facts first, then concepts.** Dates, key figures, key events are load-bearing scaffolding. Generate factual questions DETERMINISTICALLY from structured `key_facts` data. Only use LLM for rich answers and analytical questions. Never delegate to LLM what can be computed from structure. *(memory/feedback_rules.md § Knowledge System Design)*

8. **Books encode, system maintains.** Books provide initial encoding through narrative/structure. Maintaining and integrating knowledge is the system's job. Not SRS — but real decay exists. Voice dumps as knowledge elicitation. *(memory/feedback_knowledge_encoding.md)*

9. **Curiosity zone at 70% novelty.** Articles with ~70% novel claims + ~30% familiar context are most engaging (zone of proximal development). "Most novel" ≠ "most interesting." *(research/experiment-results-report.md)*

10. **Dim familiar, don't hide.**

11. **Cards are mini-encyclopedias.** Microlearning card content MUST include primary sources (who wrote about this), material evidence (what survives, where to visit), and cultural artifacts (art, opera, literature). Follow-up queries go SIDEWAYS (geography, counter-narratives, structural causes) — not deeper into what the card already said. Familiar paragraphs at 0.55 opacity, not hidden. Constrained highlighting (150 words) improves comprehension 11-19% over highlighting everything (CHI 2024). *(research/knowledge-diff-interfaces.md)*

**Flag design drift proactively.** If implementation diverges from these principles, raise it before the user discovers it.

## Where to Look

| Working on... | Read first |
|--------------|-----------|
| **Any feature work** | `research/implementation-status.md` (architecture, screens, scripts, endpoints, algorithms) |
| **UI changes** | `design/DESIGN_GUIDE.md` — MUST READ. "The Annotated Folio" Renaissance visual language. |
| **Review/retention** | `research/review-system-architecture.md`, `research/reading-companion-process-design.md` |
| **Microlearning cards** | `review_engine.py` (MICROLEARNING_PROMPT, _run_microlearning_research), `curriculum_db.py` (generate_review_stream ML mixing) |
| **Curriculum/entities** | `research/curriculum-system-audit.md` (audit + code paths), `research/overlapping-curricula-vision.md`, `research/entity-profiles-design.md` |
| **Synthesis** | `research/synthesis-pipeline-design.md`, `memory/feedback_synthesis_design.md` |
| **Books** | `research/book-companion-handoff.md`, `research/book-companion-experiments.md` |
| **Voice recall** | `voice-elicitation.tsx` — Know Nothing + Skip, book/chapter/curriculum recall, auto-loads more. Server: `review_engine.py` `run_voice_elicitation()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/houshuang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
