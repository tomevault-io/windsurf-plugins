---
trigger: always_on
description: Currents is an open-source Pinterest alternative on the AT Protocol. Users save images into Collections. Two services:
---

# Currents — Claude Code Instructions

## Project overview

Currents is an open-source Pinterest alternative on the AT Protocol. Users save images into Collections. Two services:

- **`appview/`** — Go HTTP server, AT Protocol OAuth (indigo SDK), PostgreSQL + pgvector
- **`inference/`** — Python FastAPI server, image/text embeddings via SigLIP2 (`google/siglip2-base-patch16-naflex`)

## Coding philosophy

Code minimalism is a core pillar of this project. Write the least code that correctly solves the problem. No abstractions for their own sake, no extra configurability, no defensive handling of cases that can't happen. If something can be deleted without breaking behavior, delete it (always ask before doing it).

## Key conventions

- The inference server targets Apple Silicon (`DEVICE = "mps"`). When editing, preserve MPS/CPU fallback compatibility — avoid CUDA-only APIs.
- Embeddings from `/embed/image` and `/embed/text` share the same vector space (multimodal retrieval).
- The Go appview uses PostgreSQL with the pgvector extension for storing and querying embeddings.
- The appview indexes AT Protocol records via a TAP WebSocket listener (`appview/tap.go`). TAP is configured in docker-compose to filter `is.currents.*` collections and signal on `is.currents.actor.profile`. The `collection`, `save`, and `user` tables are populated by this listener, not by the HTTP handlers.
- The `collection` and `save` tables have no FK constraints on `author_did` or `collection_uri` — events arrive for all network users and ordering isn't guaranteed.
- **Visual identity** deduplicates images across saves. Every save gets a `visual_identity_id` FK pointing to a `visual_identity` row. Resolution happens in `appview/tap.go` (`handleSaveUpsert`): resave-of-known-save → same CID already in DB → novel image (fetch blob + inference). The inference server returns both the SigLIP2 embedding and image metadata used to populate width, height, and dominant-color palette. `visual_identity` holds the canonical blob reference, embedding (HNSW-indexed), dominant-color palette, and save count. `save_count` is maintained by a DB trigger; canonical re-election happens in Go (`DeleteSave`). If the inference server is unreachable, `visual_identity_id` is left NULL and the event is acked — backfill manually later.
- **Actor profiles** are indexed into the `user` table from two sources: `is.currents.actor.profile` TAP events (all active Currents users) and the OAuth callback (`ensureUserProfile` in `appview/auth.go` for the logged-in user). Avatar URLs are stored as `{CDN_URL}/img/{did}/{cid}` — routed through the appview's image proxy. The `user` table upserts on conflict, so profiles stay current.
- **XRPC endpoints** live in `appview/xrpc.go`. The appview's service DID is `did:web:{hostname}` (served at `/.well-known/did.json`). `getActorCollections`, `getProfile`, `getCollectionSaves`, `getSaves`, `searchSaves`, `getRelatedSaves`, and `getFeed` use optional auth — session cookie for the first-party web client, Bearer JWT for PDS-proxied calls (`atproto-proxy`), unauthenticated otherwise. The `CDN_URL` env var sets the base URL embedded in image URLs in XRPC responses. Image URLs are always built from the save's own `author_did` + `pds_blob_cid` — the visual identity is internal only and never surfaced in XRPC responses.
- **Save flow**: saves are AT Protocol records written to the user's PDS, then indexed asynchronously by the TAP listener. The frontend never writes to the appview DB directly. For new saves, both the web uploader and the browser-extension clipper POST the image to the appview's `POST /save` endpoint (`CreateSave`), which uploads the blob to the user's PDS and writes the record. For resaves (saving an image that already exists on another user's PDS), the frontend calls `POST /resave` on the appview, which fetches the blob from the original author's PDS, uploads it to the viewer's PDS, and creates the record — avoiding CORS issues with cross-origin blob fetches. In both cases the TAP listener picks up the new record and populates the `save` table. Viewer save state in XRPC responses (`viewer.saves`) is hydrated by matching `pds_blob_cid` (content-addressed, so the same image bytes produce the same CID regardless of which PDS hosts the blob).
- **Starred collections** are stored in the `starred_collection` table (viewer_did + collection_uri PK). The `getActorCollections` and `getCollectionSaves` endpoints hydrate viewer state (`starred`) when the request is authenticated.
- **`searchSaves`** embeds the query text via the inference server (`/embed/text`) and runs pgvector ANN search (`<=>` cosine distance) over `visual_identity.embedding`. Offset-based pagination.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matteomarjanovic/currents](https://github.com/matteomarjanovic/currents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
