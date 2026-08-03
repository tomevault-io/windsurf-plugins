---
trigger: always_on
description: > Contexto del proyecto para Claude Code. El alcance vive en [BRIEF.md](BRIEF.md);
---

# CLAUDE.md — `agente-voz-ghl`

> Contexto del proyecto para Claude Code. El alcance vive en [BRIEF.md](BRIEF.md);
> el detalle técnico en [BRIEF-TECNICO.md](BRIEF-TECNICO.md). Este archivo es el mapa
> que se lee al inicio de cada sesión.

---

## 1. Qué construimos

Un **agente de voz** llamado **Sofía**: la recepcionista de la **Clínica Dental "Sonrisa
Perfecta"**. Contesta el teléfono 24/7, califica al paciente y **agenda una CITA DE
VALORACIÓN** en el calendario real del negocio.

En concreto, Sofía:

1. **Contesta 24/7** con voz natural en español de México.
2. **Califica:** motivo de la llamada, síntoma y urgencia (dolor, hinchazón, sangrado →
   prioriza), tratamiento de interés, datos de contacto. **Nunca diagnostica.**
3. **Agenda** la cita de valoración en el calendario de GoHighLevel.
4. **Llena el CRM:** crea el contacto, abre la oportunidad en el pipeline "Nuevos Pacientes"
   y pone tags de temperatura (hot / warm / cold).
5. **Resume la llamada** al colgar (Claude lee la transcripción) y guarda nota + score en la
   ficha del contacto.
6. **Devuelve llamadas (outbound):** un worker en Modal revisa GHL cada hora y llama a
   no-shows y leads frescos para recalificar y reagendar.

Caso ancla deliberado: el mismo negocio y personaje del curso de GoHighLevel y del agente de
WhatsApp, para que toda la serie sea coherente.

### Principio de diseño

**Un proveedor por capa, a propósito.** No hay base de datos aparte ni capa de automatización
extra. **GoHighLevel es la fuente de la verdad** — contactos, calendario y pipeline en una
sola subcuenta (Location). El backend **no guarda estado propio**: lee y escribe en GHL.

Consecuencia práctica: si quieres saber qué pasó con un paciente, lo ves en GHL, no en un log.

---

## 2. El flujo completo de una llamada

```
1. El paciente marca el número Twilio
        │
        ▼
2. Twilio enruta por Elastic SIP Trunk ──▶ Retell
        │
        ▼
3. Retell: STT (oye) · LLM (decide) · TTS (habla) — orquesta el turno de conversación
        │
        │  durante la llamada, cuando necesita hacer algo, llama tools HTTP:
        ▼
4. Modal (FastAPI, URL pública) — el backend donde viven las tools
        │
        ├─▶ POST /create-lead        → GHL: upsert del contacto
        ├─▶ POST /check-availability → GHL: free-slots del calendario
        ├─▶ POST /book-appointment   → GHL: upsert + cita + oportunidad
        └─▶ POST /update-lead-status → GHL: tags de temperatura + etapa del pipeline
        │
        ▼
5. Cuelga la llamada → Retell dispara `call_ended` a POST /retell-webhook
        │
        ▼
6. Modal llama a Claude con la transcripción → resumen + score estructurado
        │
        ▼
7. GHL: la nota y los custom fields quedan en la ficha del contacto,
   con la cita en el calendario y la oportunidad en el pipeline.
```

Y en paralelo, el ciclo outbound:

```
Modal Cron (cada hora) ─▶ lee leads pendientes y no-shows en GHL
                       ─▶ Retell `create_phone_call` (agente Sofía outbound)
                       ─▶ mismo flujo de tools de arriba
```

---

## 3. El stack y el rol exacto de cada pieza

| Pieza           | Rol exacto                                                                                                                                                                                      |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Retell AI**   | **La voz y los oídos.** STT + TTS en tiempo real y la orquestación del turno de conversación. Aquí viven los dos agentes (Sofía inbound y Sofía outbound) y sus custom tools apuntando a Modal. |
| **Twilio**      | **El número.** Recibe y hace las llamadas. Se conecta a Retell por **Elastic SIP Trunk**. Nada de lógica de negocio vive aquí.                                                                  |
| **Claude**      | **El cerebro.** Razona en llamada (qué responder, qué tool usar) y hace el **análisis post-llamada**: lee la transcripción y devuelve resumen + score estructurado que se escribe en GHL.       |
| **Modal**       | **La cocina.** El backend Python/FastAPI donde viven y se ejecutan las tools, 24/7, con URL pública. También hospeda el worker Cron de outbound.                                                |
| **GoHighLevel** | **El valor guardado.** CRM + calendario + pipeline en una sola Location: contacto, cita y avance del paciente. Fuente única de la verdad.                                                       |

> **Nota sobre el LLM en llamada.** Dentro de Retell usamos **Claude Haiku · temperature
> 0.3–0.4**. Haiku es el modelo rápido de Anthropic: en voz mandan la **latencia** y el
> **costo por minuto**, no el tamaño del modelo. Evitar modelos de razonamiento — añaden
> 1–2 s de silencio por turno y matan la conversación.
>
> Así que **Claude es el cerebro de punta a punta**: razona en llamada y hace el análisis
> post-llamada. El catálogo de modelos de Retell rota — verificar el nombre exacto del
> selector y el precio en `retellai.com/pricing` antes de fijarlo.

### Otros valores de referencia


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Carlos-Dominguez-faber/sofia-agente-voz](https://github.com/Carlos-Dominguez-faber/sofia-agente-voz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
