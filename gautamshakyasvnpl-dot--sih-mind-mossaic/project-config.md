---
trigger: always_on
description: **Project:** SahAIk — AI-Powered Inclusive Learning & Wellbeing Platform for Neurodivergent Learners (SIH 2026, GGSIPU2605)
---

# AGENTS.md — SahAIk Project Guide (READ FIRST)

**Project:** SahAIk — AI-Powered Inclusive Learning & Wellbeing Platform for Neurodivergent Learners (SIH 2026, GGSIPU2605)
**Spec:** `docs/superpowers/specs/2026-08-21-sahaik-design.md`
**Current phase:** WEEK 1 vertical slice — auth → profiler onboarding → upload PPTX/PDF/DOCX → simplified text + TTS output.

## Golden rules

1. **Ownership map below is law.** Only edit files you own. Never edit `package.json`, `requirements.txt`, or another member's directory. If you need a change outside your ownership, note it in your final report instead.
2. **No code comments** unless explaining a non-obvious invariant. Type hints mandatory (Python), strict TypeScript.
3. **Graceful degradation:** every LLM feature MUST work with no `GEMINI_API_KEY` set — fall back to deterministic logic and set `"used_llm": false`. Never crash because an external API is missing.
4. **Accessibility is not optional:** semantic HTML, labeled controls, keyboard operability, visible focus, contrast ≥ 4.5:1, respect reduced-motion.
5. Run the verification commands for your area before reporting done.

## Commands

Backend (venv already created at `backend/.venv`):
```
backend\.venv\Scripts\python -m pip install -r backend\requirements.txt   # already done
backend\.venv\Scripts\python -m pytest backend/tests -q                   # tests
backend\.venv\Scripts\python -m uvicorn app.main:app --reload --app-dir backend  # run API :8000
```

Frontend (node_modules already installed):
```
cd frontend; npm run dev     # :5173
cd frontend; npm run build   # type-check + production build
```

Env vars (optional): `GEMINI_API_KEY`, `DATABASE_URL` (default `sqlite:///./sahaik.db`), `JWT_SECRET`.

## Week-1 API contract (v1) — implement EXACTLY

Base path `/api`. JSON responses. Errors: `{"detail": "..."}` with proper status codes.

### Auth
- `POST /api/auth/register` `{email, password, display_name}` → `{token, user}` where user = `{id, email, display_name}`
- `POST /api/auth/login` `{email, password}` → `{token, user}`
- `GET /api/auth/me` (Bearer token) → `{id, email, display_name}`
JWT HS256 via python-jose, 7-day expiry. Passwords hashed with stdlib `hashlib.pbkdf2_hmac` (sha256, 200k iters, random salt, store `salt$hash`). Duplicate email → 409.

### Profile (LearnerProfile v1 schema — canonical)
```json
{
  "modality_affinity": "text | audio | visual",
  "chunk_size": "small | medium | large",
  "font_style": "default | dyslexia_friendly",
  "line_spacing": "normal | wide",
  "reduce_motion": false,
  "audio_autoplay": false,
  "pace": "gentle | standard",
  "noise_sensitive": false,
  "onboarding_complete": false
}
```
- `GET /api/profile` → profile (create defaults on first access)
- `PUT /api/profile` body = partial or full profile → merged stored profile

### Consents
- `GET /api/consents` → `{voice: bool, telemetry: bool, memory: bool}`
- `POST /api/consents` same shape → stored (upsert)

### Documents
- `POST /api/documents` multipart `file` (.pptx/.pdf/.docx/.txt, ≤20 MB) → `{id, filename, doc_type, char_count, created_at}`
- `GET /api/documents` → `{items: [doc...]}`
- `GET /api/documents/{id}` → doc; 404 if not owner's
- `DELETE /api/documents/{id}` → 204

### Adaptations
- `POST /api/documents/{id}/adapt` `{"formats": ["simplified_text", "tts_audio"]}` → 
```json
{"document_id": "...", "used_llm": true,
 "results": [
   {"format": "simplified_text", "status": "ok", "content": "...", "explanation": "why this rendering"},
   {"format": "tts_audio", "status": "ok", "content": "/api/audio/<filename>.mp3", "explanation": "..."}
 ]}
```
- `GET /api/audio/{filename}` serves mp3 from `backend/uploads/audio/`
- `GET /api/documents/{id}/adaptations` → latest adaptation result

## Service contracts (exact signatures)

`backend/app/services/extraction.py` (ML lead):
```python
def extract_text(filename: str, data: bytes) -> str:
    """Dispatch by extension: .pptx .pdf .docx .txt. Raise ValueError on unsupported/empty."""
def doc_type(filename: str) -> str: ...
```

`backend/app/services/tts.py` (ML lead):
```python
def synthesize_speech(text: str, out_path: Path) -> Path:
    """Write MP3 to out_path (gTTS). Raise RuntimeError on failure."""
```

`backend/app/services/adapter.py` (AI-Agents lead):
```python
def adapt_document(text: str, profile: dict) -> dict:
    """Returns {"results": [...], "used_llm": bool} per the adaptation response contract.
    simplified_text: LLM simplify respecting chunk_size/pace; fallback = heuristic
    sentence-split + first-N-chunks join. tts_audio always produced locally."""
class ProfilerState(TypedDict): ...  # LangGraph state in agents/profiler.py
def build_profiler_graph(): ...      # StateGraph: answers -> LearnerProfile merge suggestion
```

Routes may import ONLY these functions from services. Backend lead wires thin handlers.

## Ownership map (Week 1)

| Member | Owns |
|---|---|
| frontend-lead | `frontend/src/**` EXCEPT files owned by ux-lead; pages/routes/components/wiring |
| ux-lead | `frontend/src/styles/**`, `frontend/src/context/SensorySettings.tsx`, `docs/accessibility/**`, consent+SUS templates in `docs/user-testing/**` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gautamshakyasvnpl-dot/SIH-MIND-MOSSAIC](https://github.com/gautamshakyasvnpl-dot/SIH-MIND-MOSSAIC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
