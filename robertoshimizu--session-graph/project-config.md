---
trigger: always_on
description: Instructions for Claude Code when working on this project.
---

# session-graph

Instructions for Claude Code when working on this project.

## Project Goal

Build a unified developer knowledge graph that connects scattered knowledge from AI coding sessions across multiple platforms:

- Claude Code session logs (`.jsonl`)
- DeepSeek conversation exports (JSON zip)
- Grok conversation exports (JSON zip)
- Warp terminal AI sessions (SQLite)
- Cursor AI sessions (planned)
- VS Code Copilot interactions (planned)
- ChatGPT conversation exports (planned)

The pipeline extracts structured `(subject, predicate, object)` triples from AI assistant messages, links entities to Wikidata via `owl:sameAs`, and loads everything into a SPARQL-queryable triplestore with full provenance.

## Architecture: Ontology + Knowledge Graph + Hybrid Retrieval

### Ontology Stack (Composed W3C Standards)

Do NOT create a custom ontology from scratch. Compose these battle-tested standards:

| Ontology        | Role                                                    | Maturity              |
| --------------- | ------------------------------------------------------- | --------------------- |
| **PROV-O**      | Backbone: who did what, when, from where (provenance)   | W3C Recommendation    |
| **SIOC**        | Conversation structure: messages, threads, platforms    | W3C Member Submission |
| **SKOS**        | Concept taxonomy: topics, skills, technologies          | W3C Recommendation    |
| **Dublin Core** | Universal metadata: dates, titles, creators             | ISO Standard          |
| **Schema.org**  | Cherry-pick: `SoftwareSourceCode`, `Question`, `Answer` | De facto standard     |

Validated by IBM's GRAPH4CODE project (2B triples, same composition approach).

### Triplestore: Apache Jena Fuseki

- SPARQL 1.1 compliant
- Docker or standalone deployment
- Native TDB2 storage
- Handles 100K+ triples without issue

## Ontology: Predicate Vocabulary & Custom Classes

### Custom Classes

- `devkg:Entity` (subclass of `prov:Entity`) -- extracted technical concepts
- `devkg:KnowledgeTriple` -- reified triple for provenance (links to source message + session)
- `devkg:Project` -- software project with working directory and source files

### Curated Predicate Vocabulary (24 OWL ObjectProperties)

Closed-world design: the LLM is instructed to use ONLY these predicates. A normalization step maps any LLM-generated predicate to the closest match (fallback: `relatedTo`).

**Standard-mapped predicates:**

- `devkg:isPartOf` -> `rdfs:subPropertyOf dcterms:isPartOf`
- `devkg:hasPart` -> `rdfs:subPropertyOf dcterms:hasPart`
- `devkg:broader` -> `rdfs:subPropertyOf skos:broader`
- `devkg:narrower` -> `rdfs:subPropertyOf skos:narrower`
- `devkg:relatedTo` -> `rdfs:subPropertyOf skos:related`

**Custom predicates (19):** `uses`, `dependsOn`, `enables`, `implements`, `extends`, `alternativeTo`, `solves`, `produces`, `configures`, `composesWith`, `provides`, `requires`, `isTypeOf`, `builtWith`, `deployedOn`, `storesIn`, `queriedWith`, `integratesWith`, `servesAs`

**Additional properties:** `devkg:hasSourceFile`, `devkg:belongsToProject`, `devkg:hasWorkingDirectory`

## Pipeline Flow

```
1. SOURCE PARSING (per platform -> RDF Turtle)
----------------------------------------------

  Claude Code (.jsonl)  -->  jsonl_to_rdf.py    -->  .ttl
  DeepSeek (.json zip)  -->  deepseek_to_rdf.py -->  .ttl
  Grok (.json zip)      -->  grok_to_rdf.py     -->  .ttl
  Warp (SQLite)         -->  warp_to_rdf.py     -->  .ttl

  Each parser:
  +-- Reads source format
  +-- Creates PROV-O + SIOC structure (sessions, messages, authors)
  +-- Calls triple_extraction.py for each assistant message
  |   +-- SQLite cache check (.triple_cache.db) by message UUID
  |   |   +-- Cache hit -> use cached triples (0 API calls)
  |   |   +-- Cache miss -> call LLM, cache result
  |   +-- Sends text to LLM -> top 10 (subject, predicate, object) triples
  |       +-- Capped at 10 triples per message (top 10 by importance)
  |       +-- Closed-world predicate vocab (24 predicates)
  |       +-- Level 1 entity filter: is_valid_entity() -- 13 filter groups
  |       |   (filenames, hex colors, CLI flags, ICD codes, snake_case,
  |       |    DOM selectors, version strings, CSS dims, issue refs, etc.)
  |       |   48 whitelisted short terms bypass all filters
  |       +-- Entity length filter (1-3 words only)
  |       +-- Retry on JSON truncation (max 2 retries)
  +-- Outputs .ttl with session structure + knowledge triples

  Shared modules:
  +-- common.py          -- namespaces, URI helpers, RDF node builders
  +-- triple_extraction.py -- LLM prompt + parsing + normalization
  +-- vertex_ai.py       -- Vertex AI auth, model factory


2. ENTITY LINKING (RDF -> Wikidata owl:sameAs)
----------------------------------------------

  .ttl files -->  link_entities.py -->  wikidata_links.ttl

  +-- Extracts all devkg:Entity labels from input .ttl files
  +-- Normalizes via entity_aliases.json (161 mappings: k8s->kubernetes, etc.)
  +-- Level 2 pre-filter: is_linkable_entity() -- rejects ~6% garbage
  |   (catches entities that slipped through Level 1 or pre-date the filter)
  +-- Frequency filter: --min-sessions N (default 2) -- only links entities
  |   appearing in 2+ sessions. Reduces linking set by ~77%.
  +-- For each entity that passes:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertoshimizu/session-graph](https://github.com/robertoshimizu/session-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
