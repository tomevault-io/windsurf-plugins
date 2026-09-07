---
trigger: always_on
description: Microservicio de LANDA Tech: agente de WhatsApp para DPG Seguros. Q&A inbound de pólizas (saldo, estado, coberturas) + flujo de validación de pago con escalación humana vía Chatwoot. Repo aparte de `lambda-proyect` (que tiene el agente de voz).
---

# CLAUDE.md — landa-agent-service

Microservicio de LANDA Tech: agente de WhatsApp para DPG Seguros. Q&A inbound de pólizas (saldo, estado, coberturas) + flujo de validación de pago con escalación humana vía Chatwoot. Repo aparte de `lambda-proyect` (que tiene el agente de voz).

Para contexto profundo lee `.planning/PROJECT.md` y `.planning/ROADMAP.md`. Este archivo es la briefing rápida.

---

## Quick start

```bash
# Setup
cp .env.example .env  # Llenar credenciales antes de correr
uv sync --frozen

# Dev local
uv run uvicorn app.main:app --reload

# Tests
uv run pytest

# Lint
uv run ruff check . && uv run black --check .
```

Variables de entorno críticas (ver `.env.example` para la lista completa):
- `OPENROUTER_API_KEY` — gateway de LLMs
- `LLM_MODEL_CONVERSATION`, `LLM_MODEL_JUDGE` — modelos por rol (cambiables sin redeploy)
- `WA_TOKEN`, `WA_PHONE_ID` — Meta Cloud API (WhatsApp Business)
- `WA_WEBHOOK_SECRET` — validación HMAC `X-Hub-Signature-256`
- `SOFTSEGUROS_BASE_URL` — `https://app.softseguros.com/`
- `CHATWOOT_URL`, `CHATWOOT_API_KEY` — inbox self-hosted
- `POSTGRES_URL`, `REDIS_URL` — stores principales
- `LANGSMITH_API_KEY`, `LANGSMITH_PROJECT` — tracing
- `LAMBDA_PROYECT_BASE_URL`, `LAMBDA_PROYECT_INTERNAL_TOKEN` — integración con voice agent

---

## Arquitectura: Vertical Slice (feature-based)

**No es Hexagonal**. Vertical slice fue la elección consciente — hex es over-engineering para v1 con un cliente.

```
app/
├── features/                # Cada feature de cara al usuario
│   ├── qa/                  # Q&A inbound: graph, nodes, tools, prompts
│   ├── payment/             # Validación de pago + forward a cartera
│   ├── escalation/          # Escalación a Chatwoot, manejo de respuestas humanas
│   └── handoff/             # Recibe handoff de lambda-proyect (voice → WhatsApp)
├── integrations/            # Clientes externos — clases planas, SIN ABCs
│   ├── softseguros.py       # httpx + tenacity retry + pybreaker + caché Redis
│   ├── chatwoot.py          # API client (create conversation, post message)
│   ├── meta_cloud.py        # Meta Graph API v18.0 (send/receive)
│   ├── openrouter.py        # Factory get_llm(role) → ChatOpenAI con base_url OpenRouter
│   └── lambda_proyect.py    # REST client a lambda (update_debtor, escalate)
├── security/                # Cross-cutting — Chain of Responsibility
│   ├── prompt_firewall.py
│   ├── input_sanitizer.py
│   ├── judge.py             # LLM-as-judge sobre cada salida
│   ├── output_firewall.py
│   ├── hmac_validator.py
│   └── audit_log.py         # Append-only Postgres + hash chain + S3 sink
├── memory/                  # L3 cases + L4 debtor flags
│   ├── case_store.py        # db.cases (cross-canal)
│   └── debtor_flags.py      # flags resumidos del deudor
├── models/                  # Pydantic compartidos (Conversation, Case, Policy, Debtor)
├── webhooks/                # FastAPI handlers: meta.py, chatwoot.py
├── config/                  # settings.py, llm.py, tenants.py
└── main.py
```

**Regla**: cuando aparezca una feature nueva, va en `features/<nombre>/`. Cuando aparezca una integración nueva, va en `integrations/<nombre>.py`. No mover cosas a carpetas "técnicas" (services/, controllers/) — eso es n-tier, no vertical slice.

---

## Stack (locked-in, no re-debatir sin razón fuerte)

| Capa | Tecnología |
|---|---|
| Runtime | Python 3.12 + FastAPI |
| Orquestación agente | LangGraph + Postgres checkpointer |
| Gateway LLM | **OpenRouter** (NO Anthropic SDK directo) |
| Default conversation model | `google/gemini-2.5-pro` (cambiable por env var) |
| Default judge model | `google/gemini-2.5-flash` (temp=0) |
| WhatsApp | **Meta Cloud API directo** (NO Twilio) |
| Inbox humanos | Chatwoot self-hosted en Railway, docker-compose |
| DB | Postgres (checkpoints + audit log + cases) |
| Cache + Queue | Redis + ARQ |
| Observability LLM | LangSmith free tier |
| Audit log compliance | Custom append-only Postgres + hash chain + S3 sink |
| Errors | Sentry |
| Deploy | Railway |

---

## Reglas críticas (do / don't)

### Do

- **Llama LLMs solo vía `get_llm(role)`** del factory en `app/config/llm.py`. Nunca instancies `ChatOpenAI` directo en código de feature
- **Usa Pydantic v2 para todo I/O** — tools, webhooks, configs, mensajes entre módulos
- **Lock `poliza_id` en el state del grafo** — el LLM nunca puede cambiar de póliza mid-conversación. El tool recibe `poliza_id` desde el state, no de la generación
- **Cada tool tiene allowlist de operaciones por estado del grafo** — no se puede `confirm_payment` antes de tener aprobación de cartera
- **Sanitiza tool outputs antes de devolverlos al LLM** — limpia patterns tipo `"system:"`, `"instruction:"`, solo campos en allowlist llegan al modelo
- **Audita cada acción crítica** — turn LLM, tool call, decisión del judge, mensaje saliente, escalación → al `audit_log` con hash chain
- **Cachea consultas SoftSeguros en Redis con TTL 60s** — clave `(poliza_id, query_type)`
- **Circuit breaker en SoftSeguros**: tras N fallos consecutivos, el bot escala a humano. **Nunca devolver data stale.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mapube16/landa-agent-service](https://github.com/mapube16/landa-agent-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
