---
trigger: always_on
description: - Static site with Cloudflare Pages Functions under `functions/api`.
---

# AGENTS.md

## Working in this repo
- Static site with Cloudflare Pages Functions under `functions/api`.
- Preserve API response shapes; frontend expects the current JSON fields.
- Avoid adding production dependencies without explicit approval.
- Unless the user explicitly says not to publish, finish tasks by committing the completed work, rebasing onto the latest `origin/main`, and pushing to the remote so Cloudflare Pages deploys it.
- Current content storage is documented in `notes/content-storage.md`. Start there before changing Read Later, shared items, Dharma talk shares, assets, lists, or push device storage.
- For UI/UX consultation, run `claude -p` outside the sandbox. In the sandbox the Claude CLI cannot access its authenticated session and may fail with `Not logged in`; request escalated execution rather than skipping the consultation.

## Coordinator Context

When acting as this repo's represented coordinator in Jeff's agent organization, also follow `/Users/jeffharris/code/agents/AGENTS.md` and its shared delegation and notification protocols. Ordinary repo implementation work should follow this repo's local instructions.

## Dharma audio project
- Public feed merge lives in `tools/dharma-feed/` and publishes static files under `dharma/{corpus}/`.
- Feed-specific agent orientation lives in `tools/dharma-feed/AGENTS.md`; read it before changing sources or generated feed artifacts.
- Local transcription/search tooling lives in `tools/dharma-transcripts/`.
- Future-agent runbook for adding Matthew sources, using QMD, private Dharma Seed keys, automation, and adding other Dharma teachers lives at `tools/dharma-transcripts/docs/dharma-content-agent-runbook.md`.
- The transcript and feed tooling is now corpus-configurable. Matthew Brensilver uses `tools/dharma-transcripts/config/brensilver-corpus.json`; Rob Burbea uses `tools/dharma-transcripts/config/burbea-corpus.json`; Alan Watts uses `tools/dharma-transcripts/config/watts-corpus.json`.
- Transcript artifacts are intentionally written to `.local-corpus/brensilver/`, which is gitignored and should not be deployed unless Jeff explicitly asks.
- Watts transcript artifacts are intentionally written to `.local-corpus/watts/`, also gitignored; public generated artifacts live under `dharma/watts/`.
- Burbea transcript artifacts are intentionally written to `.local-corpus/burbea/`, also gitignored; public generated artifacts live under `dharma/burbea/`.
- The transcript workflow is split deliberately: `whisper-1` raw segments, GPT-5.4 mini transcript correction, then a separate external reference extraction pass.
- Corrected transcripts also pass through local silence-hallucination cleanup; removed rows are preserved as `suppressed_segments` in corrected JSON.
- Reference extraction writes `references/{talk_id}.json`; references are holistic jumpable moments, not exact quote spans. Low-confidence or unsupported attributions should stay marked `needs_review` rather than being promoted into indexes.
- Episode metadata writes `episode-metadata/{talk_id}.json` and `chapters/{talk_id}.json`; generated artwork writes `artwork/prompts`, `artwork/images`, and `artwork/manifests`.
- The public feed builder can merge local episode metadata into `dharma/{corpus}/feed.xml`, write `dharma/{corpus}/chapters/{safe_id}.json`, and generate canonical `dharma/{corpus}/talks/{safe_id}/` pages. Use explicit `--artwork-base-url` and `--chapters-base-url` for new workflows; `--media-base-url` remains a legacy alias for both.
- Generated Dharma artifact policy lives in `tools/dharma-feed/README.md`. Git/Pages owns feed XML, JSON indexes, HTML pages, chapter JSON, and stable corpus images. Per-episode artwork can be same-site for preview builds, but production should prefer media/R2 artwork URLs.
- Use `--prune-generated=report` to dry-run stale generated talk pages, chapter JSON, and per-episode artwork. It reports only; it does not delete files.
- `scripts/build-dharma-feed.py <corpus>` seeds from existing `dharma/{corpus}/talks.json` when present, so historical archived talks are preserved if an upstream source feed shrinks.
- When adding or changing a feed source, do not stop after a feed rebuild. Run local ingestion next so newly discovered talks get transcripts, references, episode metadata, artwork, QMD indexing, and regenerated public feed artifacts.
- Preferred one-command local ingestion: `scripts/run-dharma-ingestion.sh <corpus>`. It refreshes source feeds first, then runs `run-corpus`. Set `<CORPUS>_AUTO_PUBLISH=1` only for a scheduled job that should commit and push generated `dharma/{corpus}/` artifacts automatically.
- For full corpus work, use `run-corpus --limit 20 --feed-every 20`; it processes pending talks through correction, reference extraction, episode metadata, artwork, markdown, QMD, and public feed rebuilds.
- The `/dharma/brensilver/` landing page is generated by the feed builder. Do not hand-edit `dharma/brensilver/index.html`; update `tools/dharma-feed/src/dharma_feed/build.py`.
- The local transcript viewer is generated at `.local-corpus/brensilver/viewer/index.html` and uses original online audio URLs plus local transcript/reference data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeffsharris/jeffharr.is](https://github.com/jeffsharris/jeffharr.is) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
