---
trigger: always_on
description: This file documents the codebase architecture, active conventions, and things to know before making changes.
---

# CLAUDE.md — CSR Conflict Resolution Simulator

This file documents the codebase architecture, active conventions, and things to know before making changes.

## Quick orientation

The app is an AI-powered customer service training tool. The LLM plays a difficult customer; a human trainee plays the CSR. The backend assembles a system prompt from modular files and routes LLM calls. The frontend is a React SPA with a 4-step session wizard and a split-pane chat + workflow portal.

**Entrypoints:**
- Backend: `backend/main.py` (FastAPI app)
- Frontend: `frontend/src/App.jsx` (top-level state machine and router)
- LLM logic: `backend/services/llm_service.py`

---

## Prompt system architecture

The system prompt is assembled from independent, composable files. **No prompt logic lives in Python strings** (except the COACHING_SIGNAL per-turn injection, which is dynamic).

### Prompt directory layout

```
backend/prompts/
├── scenarios/       # WHO the customer is and WHAT happened — factual context only
│   ├── flight_cancellation.txt
│   ├── baggage_delay.txt
│   ├── loan_delay.txt
│   └── refund_request.txt
│
├── emotions/        # HOW the customer feels and behaves — no scenario facts
│   ├── angry.txt
│   ├── confused.txt
│   ├── demanding.txt
│   └── anxious.txt
│
├── shared/          # Rules that apply to all prompts
│   ├── system_rules.txt     # Role constraint, character integrity, natural speech
│   ├── behavior_rules.txt   # Info reveal, conversation progression, confirmation ack
│   └── output_format.txt    # 2-4 sentence limit, conversational style
│
├── formats/         # Mode-specific JSON response schemas
│   ├── plain.txt            # Non-training: customer_response + feedback: null
│   ├── training.txt         # Training: customer_response + full feedback block
│   └── stream.txt           # Streaming: plain text, no JSON
│
├── evaluation/      # Evaluation and coaching content
│   ├── coaching.txt         # Full scoring rubric for empathy + active listening
│   └── session_report.txt   # End-of-session report prompt
│
└── openers/         # First message sent to start the scenario
    ├── flight_cancellation.txt
    ├── baggage_delay.txt
    ├── loan_delay.txt
    └── refund_request.txt
```

### Assembly order in `build_system_prompt()`

```python
full_prompt = "\n\n".join([
    system_rules,      # shared/system_rules.txt
    scenario_text,     # scenarios/{scenario}.txt  (PORTAL_DATA block stripped)
    emotion_text,      # emotions/{persona}.txt
    behavior_rules,    # shared/behavior_rules.txt
    output_format,     # shared/output_format.txt
    response_rules,    # formats/{mode}.txt
])
# training mode only:
full_prompt += "\n\n" + coaching_rules   # evaluation/coaching.txt
```

### Separation of concerns — strict rule

| What belongs | Where |
|---|---|
| Customer name, situation facts, case details, resolution criteria | `scenarios/` |
| Emotional intensity, escalation triggers, de-escalation behaviour | `emotions/` |
| Role constraint, character integrity, no-meta-phrases rule | `shared/system_rules.txt` |
| Info reveal rules, conversation progression, email acknowledgment | `shared/behavior_rules.txt` |
| Sentence count, conversational style | `shared/output_format.txt` |
| JSON schema for LLM response | `formats/` |
| Scoring rubric (empathy + active listening) | `evaluation/coaching.txt` |

**Never put emotional descriptors in scenario files.** "Communication Style" in the Identity Layer describes speaking register (direct, methodical, formal), not emotional state (frustrated, anxious, impatient). Emotional state comes entirely from the emotion file.

**Never put scenario facts in emotion files.** `angry.txt` must not reference "refund", "flight", or any other scenario-specific noun.

### Per-turn dynamic injections (not in `build_system_prompt`)

Two things are appended dynamically in `call_llm()` and `stream_llm_response()`, not in the base prompt:

1. **COACHING SIGNAL** — the `nextStep` from the previous feedback turn, injected via `append_coaching_signal()` in `llm_service.py`. This is what nudges the customer to react differently to improved CSR behaviour.
2. **CSR_MESSAGE_TO_EVALUATE** — the literal CSR message the coach must evaluate, appended only in training mode.

---

## Scenario and persona registration

### Adding a new scenario

1. Create `backend/prompts/scenarios/{scenario_key}.txt` — scenario facts only.
2. Create `backend/prompts/openers/{scenario_key}.txt` — the first message sent to start the conversation.
3. Create `backend/workflows/{scenario_key}.json` — portal config (steps, screens, policy, customer data).
4. Add an entry to `SCENARIOS` in `backend/services/llm_service.py`:
   ```python
   SCENARIOS = {
       "your_scenario": {
           "file": "your_scenario",
           "opener": "your_scenario",
       },
   }
   ```
5. Add to `VALID_SCENARIOS` and `SCENARIO_LABELS` in `backend/main.py`.
6. Add to the `DOMAINS` array in `frontend/src/components/ModeSelector.jsx`.
7. Create workflow step components in `frontend/src/components/workflow/{your_scenario}/`.
8. Register the scenario in `frontend/src/components/workflow/utils/screenMaps.js`.

### Adding a new persona/emotion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cmu-teleperformance-research/teleperformance](https://github.com/cmu-teleperformance-research/teleperformance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
