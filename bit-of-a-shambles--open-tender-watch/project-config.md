---
trigger: always_on
description: The **Open Tender Watch** is a Rails 8 application that monitors Portuguese (and broader European) public procurement data to detect corruption risk, abuse patterns, and conflicts of interest. It is designed for journalists, auditors, and civic watchdogs.
---

# Open Tender Watch — Context & Guidelines

## Project Overview

The **Open Tender Watch** is a Rails 8 application that monitors Portuguese (and broader European) public procurement data to detect corruption risk, abuse patterns, and conflicts of interest. It is designed for journalists, auditors, and civic watchdogs.

The approach is **risk scoring, not accusation**. The system surfaces cases for audit or journalistic review with explicit confidence levels — it does not produce conclusions.

---

## Domain Model

- **Entity**: Represents both public bodies (adjudicantes) and private companies (adjudicatários). Identified by `tax_identifier` (e.g. NIF/NIPC), scoped to `country_code` — the same numeric ID can exist in different countries.
- **Contract**: A public procurement record with metadata (object, price, dates, procedure type, CPV, location). Linked to a contracting entity and a data source.
- **ContractWinner**: Join table between `Contract` and `Entity`. A contract can have multiple winners with a `price_share`.
- **DataSource**: DB-driven registry of configured data adapters per country. Each record specifies `adapter_class`, `country_code`, `source_type`, and JSON `config`.

---

## Key Data Sources

### Portugal

| Source | What it provides | Notes |
|---|---|---|
| **Portal BASE** | Central public contracts portal — contracts, announcements, modifications, impugnations | Primary source. Data published in OCDS format on dados.gov.pt via IMPIC. API access available for bulk extraction (registration required). Data quality is the responsibility of contracting entities — late or incomplete entries are themselves risk signals. |
| **Portal da Transparência SNS** | Health-sector contracts via OpenDataSoft v2.1 | ~43,000 records. No API key required. `PublicContracts::PT::SnsClient`. |
| **dados.gov.pt** | Open data platform, includes BASE mirrors | Use for bulk OCDS downloads |
| **TED** | EU-level procurement notices | Valuable for contracts at EU thresholds and cross-checking publication consistency |
| **Registo Comercial** | Company registrations, shareholders, management | Scraped from publicacoes.mj.pt |
| **Entidade Transparência** | Public entities, mandates, persons in public roles | https://entidadetransparencia.pt/ — useful for conflict-of-interest checks |
| **RCBE** | Beneficial ownership register | Access requires authentication by legal person number; CJEU ruling limited open access — treat as a constrained layer |
| **AdC** | Competition Authority — cartel cases, sanctions | Cross-reference supplier NIFs against published AdC cases |
| **Tribunal de Contas** | Audit reports, financial liability decisions | Used to corroborate red flags |
| **Mais Transparência / Portugal2020** | EU-funded contract data | Useful for prioritising EU-funded tenders |
| **ECFP/CNE** | Political party donations | Future source for conflict-of-interest detection |

### EU / Cross-border

| Source | What it provides |
|---|---|
| **TED** | EU procurement notices via REST API and bulk XML packages |

---

## Red Flag Catalogue

Indicators are grouped into three tracks following [OECD methodology](https://www.oecd.org/en/publications/preventing-corruption-in-public-procurement_9789264059765-en.html) (see also [OCP Red Flags for Integrity](https://www.open-contracting.org/resources/red-flags-for-integrity-giving-green-light-to-open-data-solutions/)):

### Track A — Rule-based red flags (high explainability)

| # | Indicator | Data fields | Notes |
|---|---|---|---|
| A1 | Repeat direct awards / prior consultations to same supplier by same authority | `procedure_type`, `contracting_entity_id`, winner NIF, `publication_date` | OECD: repeat awards and concentration over 3 years |
| A2 | Contract published after execution starts | `publication_date`, `celebration_date`, `execution_start_date` | OECD: contract data earlier than adjudication date. **Implementation note:** Portuguese law requires signing first then publishing. Only gaps > 10 days are flagged (`MIN_PUBLICATION_DELAY_DAYS = 10`). Before this threshold was introduced, A2 fired on ~85% of all contracts (any gap). |
| A3 | Execution begins before publication in BASE | `publication_date` vs actual start | OECD: contracts implemented before publication |
| A4 | Amendment inflation — large or frequent modifications | `base_price`, amendment value, amendment count | BASE publishes modifications above thresholds |
| A5 | Contract value just below procedural thresholds | `base_price`, `procedure_type` | Threshold-splitting / fragmentation |
| A6 | Single bidder / low-competition procedure | bidder count, `procedure_type` | BASE publishes procedure type and bidder details |
| A7 | Buyer uses direct award far above peer median for same CPV | `procedure_type`, `cpv_code`, `contracting_entity_id` | Peer comparison by CPV and region |
| A8 | Long execution duration (> 3 years) | contract duration | OECD: execution length over 3 years as risk feature |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bit-of-a-shambles/open-tender-watch](https://github.com/bit-of-a-shambles/open-tender-watch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
