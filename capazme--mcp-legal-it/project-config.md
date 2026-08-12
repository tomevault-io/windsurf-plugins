---
trigger: always_on
description: > MCP server con 218 tool di calcolo legale italiano, consultazione normativa
---

# mcp-legal-it — Project Context

> MCP server con 218 tool di calcolo legale italiano, consultazione normativa
> (Normattiva, EUR-Lex, Brocardi), ricerca giurisprudenziale (Italgiure, CeRDEF,
> TAR/CdS, CGUE), delibere CONSOB.

## Confini del progetto — LEGGERE PRIMA

Questo progetto è **autonomo e indipendente** dal SAPG Tech Desk.

| Progetto | Path | Scopo |
|----------|------|-------|
| **mcp-legal-it** (questo) | `/Users/gpuzio/Desktop/CODE/mcp-legal-it/` | Tool legali generici: calcoli, normativa, giurisprudenza |
| SAPG MCP Server | `~/Desktop/CODE/SAPG TECH DESK/sapg-tech-desk/apps/mcp-server/` | Piattaforma compliance: clienti, checklist, task, documenti |

**I tool Italgiure (`leggi_sentenza`, `cerca_giurisprudenza`, ecc.) sono QUI, non in SAPG.**
Se un agente non vede `leggi_sentenza`, è perché sta guardando il server sbagliato.

## Stack

| Layer | Tecnologia |
|-------|-----------|
| Framework MCP | FastMCP >= 2.0 (`@mcp.tool()`, `@mcp.prompt()`, `@mcp.resource()`) |
| HTTP client | httpx >= 0.27 (async) |
| HTML scraping | BeautifulSoup4 + lxml |
| PDF generation | fpdf2 |
| Python | >= 3.10, venv in `.venv/` |
| Test | pytest + pytest-asyncio |

## Struttura

```
mcp-legal-it/
├── src/
│   ├── server.py              # FastMCP entry point — registra tutti i tool
│   ├── prompts.py             # 23 workflow guidati (@mcp.prompt)
│   ├── resources.py           # 15 risorse statiche (@mcp.resource)
│   ├── lib/
│   │   ├── visualex/          # Normattiva + EUR-Lex scraper
│   │   │   ├── scraper.py     # fetch_article(), fetch_annotations(), fetch_normattiva_full_text()
│   │   │   ├── map.py         # BROCARDI_CODICI, ATTI_NOTI, resolve_atto(), find_brocardi_url()
│   │   │   └── models.py      # Norma, NormaVisitata dataclasses
│   │   ├── brocardi/          # Scraper Brocardi standalone
│   │   │   └── client.py      # fetch_brocardi(), BrocardiResult, Massima, parse_massime_references()
│   │   ├── italgiure/         # Client Italgiure (Cassazione Solr API)
│   │   │   └── client.py      # solr_query(), build_*_params(), format_*()
│   │   ├── consob/            # Client CONSOB (Liferay Portal scraper)
│   │   │   └── client.py      # search_delibere(), fetch_delibera(), format_*()
│   │   ├── cerdef/            # Client CeRDEF (Giurisprudenza Tributaria MEF)
│   │   │   └── client.py      # search_giurisprudenza(), fetch_provvedimento(), format_*()
│   │   ├── giustizia_amm/     # Client Giustizia Amministrativa (TAR/CdS)
│   │   │   └── client.py      # search_provvedimenti(), fetch_provvedimento_text(), format_*()
│   │   ├── cgue/              # Client CGUE (CELLAR SPARQL + EUR-Lex)
│   │   │   └── client.py      # search_giurisprudenza(), fetch_sentenza_text(), format_*()
│   │   └── vies/                  # Client VIES (validazione P.IVA UE)
│   │       └── client.py          # check_vat(), checksum_partita_iva()
│   └── tools/
│       ├── legal_citations.py # cite_law, fetch_law_article, fetch_law_annotations, cerca_brocardi, download_law_pdf
│       ├── italgiure.py       # leggi_sentenza, cerca_giurisprudenza, giurisprudenza_su_norma, ultime_pronunce
│       ├── rivalutazioni_istat.py
│       ├── tassi_interessi.py
│       ├── scadenze_termini.py
│       ├── atti_giudiziari.py
│       ├── fatturazione_avvocati.py
│       ├── parcelle_professionisti.py
│       ├── risarcimento_danni.py
│       ├── diritto_penale.py
│       ├── proprieta_successioni.py
│       ├── investimenti.py
│       ├── dichiarazione_redditi.py
│       ├── varie.py
│       ├── consob.py          # cerca_delibere_consob, leggi_delibera_consob, ultime_delibere_consob
│       ├── cerdef.py          # cerca_giurisprudenza_tributaria, cerdef_leggi_provvedimento, ultime_sentenze_tributarie
│       ├── giustizia_amm.py   # cerca_giurisprudenza_amministrativa, leggi_provvedimento_amm, giurisprudenza_amm_su_norma, ultimi_provvedimenti_amm
│       ├── cgue.py            # cerca_giurisprudenza_cgue, leggi_sentenza_cgue, giurisprudenza_cgue_su_norma, ultime_sentenze_cgue
│       ├── privacy_gdpr.py
│       ├── procure_quotazioni.py  # genera_procura_liti_docx, genera_quotazione_docx
│       └── analisi_fornitori.py   # verifica_partita_iva_vies, genera_report_fornitori
└── tests/
    ├── unit/
    │   ├── test_calculations.py     # Test calcoli numerici
    │   ├── test_legal_citations.py  # Test cite_law, resolve_act, PDF helpers
    │   ├── test_brocardi.py         # Test scraper Brocardi e tool cerca_brocardi
    │   ├── test_consob.py          # Test scraper CONSOB e 3 tool delibere
    │   ├── test_privacy_gdpr.py   # Test 12 tool GDPR/Privacy compliance
    │   ├── test_cerdef.py         # Test CeRDEF scraper e 3 tool tributari (88 test)
    │   ├── test_giustizia_amm.py  # Test GA scraper e 4 tool TAR/CdS (90 test)
    │   ├── test_cgue.py           # Test CGUE SPARQL client e 4 tool (76 test)
    │   ├── test_http_retry.py     # Test retry helper con backoff
    │   ├── test_atti_giudiziari.py     # Test 23 tool atti giudiziari (134 test)
    │   ├── test_scadenze_termini.py    # Test 11 tool scadenze processuali (91 test)
    │   ├── test_fatturazione_avvocati.py # Test 12 tool parcelle DM 55/2014 (100 test)
    │   ├── test_dichiarazione_redditi.py # Test 15 tool IRPEF/fiscale (121 test)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [capazme/mcp-legal-it](https://github.com/capazme/mcp-legal-it) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
