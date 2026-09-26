---
trigger: always_on
description: Machine-legible reference for the Diction self-hosted gateway. Covers routes, wire formats,
---

# Diction Gateway -- Agent Reference

Machine-legible reference for the Diction self-hosted gateway. Covers routes, wire formats,
environment variables, capabilities, and what the gateway can and cannot do.

## Overview

The gateway is a Go HTTP service that sits between the Diction iOS app and one or more
OpenAI-compatible speech-to-text backends. It handles:

- WebSocket streaming for low-latency transcription
- HTTP transcription fallback
- Optional LLM post-processing (transcript cleanup, voice editing, suggestions)
- Optional trial token auth (Diction One -- not for self-hosters)

> WARNING: TEXT_ROUTES_OPEN=false (default). /v1/text/* routes return 403 until
> you explicitly set TEXT_ROUTES_OPEN=true or AUTH_ENABLED=true. This is a
> deliberate speed bump -- not a security control. Set it before calling those routes.

## Routes

### GET /health

Health check. Returns 200 when the gateway is up.

```
Response: 200 OK
Body: "ok"
```

### GET /v1/models

Lists configured speech backends (OpenAI-compatible + Diction legacy grouping).
Also includes a top-level capabilities object describing what this gateway instance
can do (additive -- never removes data[] or providers[]).

```
Response: 200 OK
Content-Type: application/json

{
  "object": "list",
  "data": [
    { "id": "nvidia/parakeet-tdt-0.6b-v3", "object": "model", "created": 0, "owned_by": "nvidia" }
  ],
  "providers": [
    {
      "id": "parakeet",
      "name": "NVIDIA Parakeet",
      "models": [{ "id": "parakeet-v3", "name": "Parakeet v3", "description": "...", "available": true }]
    }
  ],
  "capabilities": {
    "llm": true,
    "text_process": true,
    "text_suggest": true
  }
}
```

**capabilities object:**

| Field | Type | Meaning |
|-------|------|---------|
| `llm` | bool | LLM_BASE_URL + LLM_MODEL are set and LLM is active |
| `text_process` | bool | /v1/text/process is open (llm=true AND TEXT_ROUTES_OPEN=true, or auth=true) |
| `text_suggest` | bool | /v1/text/suggest is open (same condition as text_process) |
| `text_summarize` | bool | /v1/text/summarize is open (same condition as text_process) |
| `formatting` | bool | the `formatting` context key is honoured on cleanup (llm=true) |

### POST /v1/trial

Issues a trial token for Diction One cloud subscription. Not useful for self-hosters
(requires TRIAL_SECRET to be configured with Diction's server-side secret).

```
Request body: {"device_id": "<UUID>"}
Response 200: {"token": "<hmac-token>", "expires_at": "<RFC3339>"}
Response 503: {"error":"trial_not_configured"}  -- TRIAL_SECRET not set
Response 409: {"error":"trial_already_used"}  -- trial already consumed
```

### POST /v1/audio/transcriptions

Transcribes an audio file. OpenAI-compatible multipart form upload.

```
Request: multipart/form-data
  file=<audio>
  model=<model-id>        (optional, defaults to DEFAULT_MODEL)
  language=<bcp47>        (optional)
  prompt=<string>         (optional, Whisper prompt hint)
  response_format=json|text  (optional, default json)

Response 200:
  Content-Type: application/json
  X-Diction-Whisper-Ms: <int>
  X-Diction-Route-Model: <model-id>
  X-Diction-LLM-Ms: <int>  (only when LLM ran)
  Body: {"text": "<transcript>"}
```

Append `?enhance=true` to request LLM cleanup. If LLM is not configured,
the raw transcript is returned -- transcription never fails due to LLM issues.

### WS /v1/audio/stream

WebSocket streaming transcription. Used by the Diction iOS app for live transcription
as you speak. Not part of any standard API. Binary audio frames in, JSON text frames out.

Not suitable for scripting -- use /v1/audio/transcriptions for batch use.

### POST /v1/text/process

**Plaintext JSON -- no E2E encryption. Bearer token optional.**

Applies LLM post-processing to text with an explicit intent. Used by the iOS app
for voice editing and transcript cleanup outside the transcription flow.

Requires LLM to be configured (LLM_BASE_URL + LLM_MODEL) and either:
- TEXT_ROUTES_OPEN=true, or
- AUTH_ENABLED=true with a valid bearer token

Returns 403 {"error":"text_routes_closed"} if neither condition is met.
Returns 503 {"error":"llm_not_configured"} if LLM env vars are not set.
Returns 400 {"error":"e2e not supported on this gateway"} if X-Diction-E2E header is present.

```
Request:
  POST /v1/text/process?intent=<intent>
  Content-Type: application/json
  Authorization: Bearer <token>  (optional)

  {
    "text": "<text or instruction>",
    "context": "<JSON string, see below>"
  }

context fields (all optional; `context` itself is a JSON *string*, not an object):
  before          string    text before the user's cursor
  after           string    text after the user's cursor
  selected        string    the user's selection, for intent=edit-selected
  customWords     array     user vocabulary. Objects [{"word":"Diction"}] or plain
                            strings ["Diction"] are both accepted. Cap 50.
  tone            string    how the user wants to be written for. Cap 500 chars.
  profile         string    who the user is. Merged with `tone` into one block.
  sessionContext  [string]  accepted and IGNORED by the cleanup prompt (see note below).
  clipboard       string    accepted and IGNORED by the cleanup prompt (see note below).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DictionLabs/Diction](https://github.com/DictionLabs/Diction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
