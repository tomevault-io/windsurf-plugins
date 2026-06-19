---
trigger: always_on
description: Creazione specifiche OpenAPI 3.0+ conformi alle Linee Guida AgID ModI (Modello di Interoperabilità) con arricchimento semantico JSON-LD. Include workflow completo con API Design Canvas, generazione automatica, validazione conformità e mappatura ontologie italiane (CPV, DCAT-AP_IT, CLV, COV). Usa questa skill quando l'utente chiede di creare specifiche OpenAPI per la PA italiana, menziona linee guida AgID, ModI, interoperabilità PA, o richiede conformità agli standard di interoperabilità italiani
---


# Skill: OpenAPI Conforme alle Linee Guida AgID ModI

## Trigger
Usa questa skill quando:
- L'utente chiede di creare una specifica OpenAPI per la PA italiana
- Si menzionano "linee guida AgID", "ModI", "interoperabilità PA"
- Si deve documentare un'API REST per enti pubblici italiani
- Si richiede conformità agli standard di interoperabilità italiani

## MCP Tools Disponibili

1. **API Canvas Design** — Progettazione iniziale (scopo, stakeholder, risorse, operazioni)
2. **OAS Checker MCP** — Validazione conformità AgID (naming, sicurezza, RFC 7807)
3. **schema-gov-it MCP** — Arricchimento semantico (ontologie italiane, vocabolari controllati)

## Workflow in 7 Fasi

### FASE 1: Progettazione con Canvas API
Raccogliere requisiti prima di scrivere YAML:
- **Chi**: stakeholder (cittadini, imprese, altre PA)
- **Cosa**: casi d'uso principali
- **Come**: risorse REST e operazioni
- **Perché**: obiettivi di business
- **Sicurezza**: JWT / OAuth2 / mTLS, ruoli
- **NFR**: volumi, tempi di risposta, rate limiting

Output: documento di design condiviso.

### FASE 2: Generazione Specifica OpenAPI
Trasformare il design in specifica conforme ModI:
1. Struttura base (info, servers, security) — vedi regole sezioni 1-4
2. Per ogni risorsa: path REST, operazioni CRUD, schemi dati
3. Pattern ModI: paginazione, filtri, errori RFC 7807, health check `/status`
4. Esempi request/response per OGNI operazione (vedi regole esempi)

**Regole esempi (obbligatorie)**:
- `example` per ogni `requestBody` (POST, PUT, PATCH)
- `example` per ogni response `2xx` con body
- Esempi errore nei `components/responses`
- Valori realistici: nomi italiani, CF validi, date ISO 8601 con timezone, UUID plausibili
- Coerenza request↔response: il dato inviato in POST deve tornare nella response
- Collezioni: almeno 2 item diversificati
- Enum: coprire almeno 2 valori diversi
- Usare `examples` (plurale) per operazioni con varianti significative (es. polling asincrono)

### FASE 3: Arricchimento Semantico con schema-gov-it MCP

Per ogni schema:

1. **Cerca tipo ontologico**: `search_concepts(keyword)` → `inspect_concept(uri)`
   - Se trovato → `x-jsonld-type` con prefisso compatto
   - Se non trovato → segnala gap, suggerisci tipo più vicino
2. **Mappa proprietà**: `list_properties(ontologyUri)` → `get_property_details(propertyUri)`
   - Definisci prefissi PRIMA, poi `prefisso:nome`
   - Se non trovata → prova sinonimi, ontologie alternative, segnala gap
3. **Verifica vocabolari controllati**: `list_vocabularies()` → `browse_vocabulary(schemeUri, keyword)`
   - Se trovato → referenzia con URI compatta + codice `skos:notation`
   - Se non trovato → segnala gap, suggerisci codifica locale documentata
4. **Arricchisci descrizioni**: se assente/solo in inglese/generica → proponi testo migliorato in italiano
5. **Report di copertura**: N proprietà mappate / N totali, gap identificati, descrizioni arricchite

**Efficienza MCP**: usare SEMPRE `keyword` e `limit`. Non scaricare interi vocabolari. Preferire `search_concepts` a esplorazioni generiche.

### FASE 3.5: Validazione Semantica via MCP (OBBLIGATORIA)

> **REGOLA CRITICA**: Non usare MAI una URI ontologica copiata da template senza verificarla con schema-gov-it MCP. I template sono un PUNTO DI PARTENZA, la fonte di verità sono le ontologie via MCP.

Per ogni mappatura:
1. `search_concepts(keyword="nome proprietà")`
2. Se trovata: `get_property_details(propertyUri)` → verifica domain/range
3. Se domain/range compatibili → usa nel context con prefisso
4. Se domain/range incompatibili → segnala, cerca alternativa
5. Se non trovata → segnala gap, cerca in ontologie alternative

Per ogni `x-jsonld-type`: `inspect_concept(uri)` → verifica che la classe esista.
Per ogni vocabolario: `list_vocabularies()` → verifica che il ConceptScheme esista.

Produrre report:
```
REPORT VALIDAZIONE SEMANTICA
URI verificate OK: N | URI inesistenti: N | Domain/range incompatibili: N | Gap: N
```

### FASE 4: Validazione con oas-checker-mcp
Verificare conformità AgID. Errori critici: Problem schema mancante, path non kebab-case, security scheme assente, /status mancante.

### FASE 5: Iterazione
Correggere errori → ri-validare → raffinare semantica → re-iterare fino a score ≥ 95%.

### FASE 6: Generazione Esempi JSON-LD
SOLO per schemi con `x-jsonld-context` contenente mappature di proprietà:
- `{schema}.json`: risposta JSON standard
- `{schema}.jsonld`: risposta JSON-LD con `@context` espanso

NON generare `.jsonld` per schemi senza `x-jsonld-context`. Proprietà senza mappatura non appaiono nel `.jsonld` — documentare gap nel README.

**Validazione** (eseguire via Bash):
```bash
test -d jsonld/.venv || (python3 -m venv jsonld/.venv && jsonld/.venv/bin/pip install -q pyld)
cd jsonld/ && .venv/bin/python3 -c "
from pyld import jsonld
import json, glob, sys
errors = 0

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mfortini/skill-openapi-modi](https://github.com/mfortini/skill-openapi-modi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
