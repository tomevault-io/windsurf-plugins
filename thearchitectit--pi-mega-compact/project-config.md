---
trigger: always_on
description: * **docs/INDEX_MAP.md**: Read this FIRST to find documents by keyword/category.
---

# Project Guidelines — pi-mega-compact

## 0. Navigation Maps (READ FIRST)

* **docs/INDEX_MAP.md**: Read this FIRST to find documents by keyword/category.
* **docs/HEADER_MAP.md**: Find specific sections with file:line references for targeted reading.
* **docs/AGENT_GUARDRAILS.md**: MANDATORY safety protocols — read before any code change.

---

## 1. Context & Setup

* **Stack**: TypeScript, Node >= 18, ESM (`"type": "module"`). Ships as a pi coding-agent extension (no remote MCP server).
* **Detector**: `package.json` (`pi.extensions`, `engines.node`), `tsconfig.json`. Do NOT read `package-lock.json` blindly.
* **Guardrails**: Read [docs/AGENT_GUARDRAILS.md](docs/AGENT_GUARDRAILS.md) before any code change. The Four Laws (Read First / Stay in Scope / Verify Before Commit / Halt When Uncertain) are NON-NEGOTIABLE.

---

## 2. Token-Saving Rules (STRICT)

* **NO EXPLORATION**: do not `ls -R` the whole tree; use the maps.
* **NO RE-READING**: trust your context; do not re-read files you just edited.
* **TARGETED CONTEXT**: read ONLY files relevant to the request.
* **CONCISE PLANS**: bullet points only.
* **USE MAPS**: check `docs/INDEX_MAP.md` before reading full documents.

---

## 3. Workflow

* **Tests**: run ONLY relevant tests (`npm test` runs `node --test` on `dist/**/*.test.js` — build first).
* **Edits**: prefer small, single-file edits in `src/`; keep `src/` pi-agnostic (no pi runtime types).
* **Commits**: one focused commit per task; AI-attribution required (pre-commit hook enforces `Co-Authored-By:`).
* **Guardrails gate**: every change must pass `npm run lint` + `python3 scripts/regression_check.py --all`.

---

## 4. Hard Project Constraints (PREVENT-PI)

These are pi-extension invariants; `scripts/guardrails-scan.mjs` scans for violations:

| Rule | Severity | Meaning |
|------|----------|---------|
| PREVENT-PI-001 | error | Never drop messages without the anchor-floor guard (preserve recent N). |
| PREVENT-PI-002 | error | Never split a toolCall/toolResult pair at a compaction boundary. |
| PREVENT-PI-003 | error | Never inject compacted context as `role:"system"` — use the `before_agent_start` systemPrompt prepend. |
| PREVENT-PI-004 | critical | **Zero network calls at runtime.** Extension is fully local (better-sqlite3 = in-process native SQLite, FS persistence). No `fetch`/HTTP to remote. EXCEPTION: the optional, user-triggered `/dashboard` localhost server — audited via `// guardrails-allow PREVENT-PI-004: <reason>` inline annotations (scanner enforces a reason). |

Additional guardrails (from template): PREVENT-001 (JSON.parse without null check), PREVENT-002 (SQL string concat — use parameterized queries), PREVENT-011 (`any` type), PREVENT-024 (hallucinated package import), PREVENT-003 (hardcoded credentials).

* **DISTRIBUTION — npm is the ONLY valid path (PREVENT-DIST-001, error).** The extension is distributed and updated **exclusively via `npm publish` + `pi install npm:pi-mega-compact` / `pi update --extensions`**. **NEVER produce or rely on a `.tgz` tarball** (`npm pack`) for testing or shipping — tarballs bypass pi's package manager and do not propagate to other devices. **Symlinks** into `~/.pi/agent/extensions/` are **dev-only** and likewise bypass the update path. To validate a real install, bump `version`, `npm publish`, then `pi update --extensions` on the device. (See memory `pi-npm-workflow`.)
* **RELEASE PIPELINE — `scripts/deploy.sh` is the authoritative publish gate (MANDATORY).** Every release MUST run `./scripts/deploy.sh <new-version>`. It enforces a clean git tree, the full gate (`build` + `test` + `lint` + `regression_check` + `guardrails-scan`), builds the React dashboard (`build:dashboard`), and **critically verifies `extensions/dashboard-client/dist/index.html` is present AND listed by `npm pack --dry-run` BEFORE `npm publish`** — the exact regression that shipped v0.8.5 without the dashboard bundle. Then bumps version, commits, `npm publish` (only), tags `v<version>`, `git push --follow-tags`, and prints device-side verify steps (`pi update --extensions` + `curl localhost:9320/` grep `id="root"`). Never publish by hand. The device-side helpers `scripts/deploy-dashboard.sh` and root `deploy_dashboard.sh` are NOT publish scripts — they only update+verify on devices after a release.

---

## 5. Architecture at a Glance

* **Layers** (Trident stack): L1 supersede → L2 collapse → L3 cluster/vectorize → L4 persist → L5 recall/inline.
* **One store**: `node:sqlite` (`DatabaseSync`, Node built-in ≥22.13) is the **synchronous source of truth** from v0.4.23 (Slice 1) — replacing the `better-sqlite3` native addon that pi's install-script block prevented from building. FTS5 `trigram` tokenizer backs the pg_trgm-equivalent dedup tiers; cosine stays a **linear scan over `embedding_blob`** (small N) as the DEFAULT recall path. Legacy gzipped JSON checkpoint files are retained as DR snapshots.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheArchitectit/pi-mega-compact](https://github.com/TheArchitectit/pi-mega-compact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
