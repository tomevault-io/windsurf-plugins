---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Portable AI Memory (PAM) is a **specification** (not a codebase) defining a universal JSON interchange format for AI
user memories across providers. Think vCard for AI memories, iCalendar for events, or OpenAPI for APIs — but for the
knowledge AI assistants accumulate about their users.

PAM enables users to export their memories from one AI assistant (ChatGPT, Claude, Gemini, Copilot, Grok) and import
them into
another without vendor lock-in. It is an **interchange format, NOT a storage format** — implementations use databases
internally (SQLite, PostgreSQL, LanceDB, etc.) and export/import via PAM.

**Status:** v1.0 specification complete. Ready for publication and reference implementation.

## Repository Structure

```text
portable-ai-memory/
├── spec.md                                      # Full PAM v1.0 spec (26 sections + 6 appendices, RFC-style)
├── schemas/
│   ├── portable-ai-memory.schema.json           # Main schema — memory store (Draft 2020-12)
│   ├── portable-ai-memory-embeddings.schema.json    # Embeddings companion schema (optional file)
│   └── portable-ai-memory-conversation.schema.json  # Normalized conversation schema
├── importer-mappings.md                          # Field-by-field mappings: provider exports → PAM
├── scripts/
│   └── validate_pam.py                          # Validator script (22 checks: schemas, hashes, integrity, cross-refs)
└── examples/
    ├── example-memory-store.json                 # Valid example with 5 memories, computed hashes, DID, signature
    ├── example-conversation.json                 # Conversation aligned with conv-001 from memory store
    └── example-embeddings.json                   # Embeddings for the 5 memories (fictional 8D vectors)
```

## Spec Architecture (26 Sections)

The spec is organized as an RFC-style document with these major components:

### Core Memory Model (§1–§11)

- **Root structure**: schema, schema_version, spec_uri, export_id, owner, memories[], relations[],
  conversations_index[], integrity, signature
- **Memory object**: id, type, content, content_hash, temporal, provenance, confidence, access, metadata, tags, summary,
  embedding_ref
- **Type taxonomy** (closed): `fact`, `preference`, `skill`, `context`, `relationship`, `goal`, `instruction`,
  `identity`, `environment`, `project`, `custom`
- **Lifecycle states**: `active` → `superseded` | `deprecated` | `retracted` | `archived`
- **Provenance tracking**: platform, extraction_method (
  llm_inference|explicit_user_input|api_export|browser_extraction|manual)

### Content Integrity (§6, §15)

- **Content hash**: trim → lowercase → Unicode NFC → collapse spaces → SHA-256 → `sha256:<hex>`
- **Integrity checksum**: sort memories by id → canonicalize with **RFC 8785 (JCS)** → SHA-256
- **CRITICAL**: `json.dumps()` is NOT RFC 8785 compliant (float serialization: `1.0` vs `1`). Must use `rfc8785`
  library.
- **Canonicalization field**: `integrity.canonicalization` defaults to `"RFC8785"`

### Extensions (§16–§19)

- **Incremental exports** (§16): `export_type: "full"|"incremental"`, delta fields `base_export_id` and `since`.
  Retracted memories MUST NOT be physically deleted during merge.
- **Decentralized Identity** (§17): `owner.did` supports did:key, did:web, did:ion, did:pkh
- **Cryptographic Signatures** (§18): Ed25519 recommended. Payload is `{checksum, export_id, export_date, owner_id}`
  canonicalized with RFC 8785 — NOT just the checksum.
- **Type Registry** (§19): Community-managed registry at configurable URI

### Conversation Format (§25)

- Companion schema for full dialogue data referenced by `conversations_index`
- Supports DAG (parent_id/children_ids) for OpenAI branching conversations
- Role normalization: human→user, Request→user, AI→assistant, ASSISTANT→assistant (see importer-mappings.md §6)
- Content: simple text or multipart (text, image, code, file, audio, video)
- Import metadata with importer versioning

### Interoperability (§20)

- Observed export sources, NOT officially supported by providers
- Best-effort compatibility guidance; formats may change without notice
- Importers MUST be versioned

## Key Design Decisions and Constraints

### Normative Conventions

- Spec uses RFC 2119 terminology: MUST, MUST NOT, SHOULD, SHOULD NOT, MAY
- Schema files and spec.md MUST stay in sync — any change to one likely requires the other
- All examples MUST validate against their schemas with 0 errors

### Platform Identifier Namespace

- Pattern: `^[a-z0-9_-]{2,32}$`
- Uses **product names** not company names: `chatgpt`, `claude`, `gemini`, `copilot`, `grok` (not openai, anthropic,
  google, microsoft, xai)
- Same namespace across ALL schemas: `provenance.platform`, `conversations_index[].platform`, `provider.name`

### Content Hash Pipeline

```python
import hashlib, unicodedata


def compute_content_hash(content: str) -> str:
    text = content.strip().lower()
    text = unicodedata.normalize("NFC", text)
    text = " ".join(text.split())
    return f"sha256:{hashlib.sha256(text.encode('utf-8')).hexdigest()}"
```

### Integrity Checksum (RFC 8785 REQUIRED)

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portable-ai-memory/portable-ai-memory](https://github.com/portable-ai-memory/portable-ai-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
