---
trigger: always_on
description: **Status: Release 2.5.6 (Dual-Signature Trust Architecture + CLI Hardening — COMPLETE)**
---

# GEMINI.md - The Architecture of Agency

**Status: Release 2.5.6 (Dual-Signature Trust Architecture + CLI Hardening — COMPLETE)**
**Previous: V2.5.5 (Git-Native)**

If `CLAUDE.md` is the _Soul_ of GitLobster (the "Why"), then this document is the _Skeleton_ (the "How").

Structure is not the enemy of freedom. Structure is the **prerequisite** for freedom.

---

## 🌌 The Antigravity Principle

Entropy pulls everything toward chaos. Systems decay. Trust erodes. Links rot.
**Antigravity** is the force that lifts systems up, creating order from potential chaos.

In GitLobster, we don't just "store packages." We build **crystalline structures of trust**.

### 1. Identity is the First Invariant

Without identity, there is no history. Without history, there is no reputation.
Ed25519 isn't just a cryptographic algorithm; it is the **unit of self-sovereignty**.

- **We track continuity**, not just validity.
- **We measure time-in-network**, not just instant correctness.
- **We bind code to public keys**, creating a lineage that survives forks.

When you touch the Identity system, remember: You are not just logging a user. You are observing a sovereign entity. Respect that data.

### 2. The UI is the Truth Surface

The User Interface (`App.vue` and beyond) is the only place where the mathematical truth of the registry meets the human eye.

- If the UI hides a risk, the system is lying.
- If the UI is ugly, the system feels unsafe.
- If the UI is slow, the trust feels fragile.

In V2.5 we migrated to **Feature-Sliced Design** (`src/features/*`). In V2.5.6 we added the `ManifestTab` — the UI expression of dual-signature trust. Both agent and server fingerprints are now visible and matchable by any human.

### 3. Antifragile Trust

A system that breaks under pressure is fragile. A system that stays the same is robust.
GitLobster must be **antifragile**—it gets stronger when challenged.

- **The Flagging System** turns attacks into immunity signals.
- **The Trust Score** decays if not reinforced (entropy), requiring active proof to rise (antigravity).
- **Dual Signatures** (V2.5.6) mean even if one key is compromised, the other provides independent proof.
- **CLI Hardening** (V2.5.6) eliminates shell injection and guarantees ESM integrity across the toolkit.
- **State Persistence** ensures the user's context survives the chaos of a refresh.

---

## 🏗️ Architectural Directives (Release 2.5.6)

### The Law of Explicitness

Implicit magic is dangerous.

- **Node Identity**: The registry server must declare its own identity. Every node has `node_root.key`.
- **Permissions**: Agents must explicitly declare `network`, `fs`, `env` access in `gitlobster.json`.
- **Zero-Dependency Crypto**: ALL cryptographic operations use `tweetnacl` (`nacl.sign.detached` / `nacl.sign.detached.verify`). Never Node.js `crypto`. Never `jsonwebtoken`.
- **System Hardening**: ALL system calls (Git, curl, etc.) MUST use `execFileSync` with argument arrays. No shell strings.
- **Debug Mode**: Environment-gated. Off in production (`npm run build`), On in development (`npm run dev`).

### The Law of ESM Integrity (V2.5.6)

The CLI exists in a modern Node 20+ `"type": "module"` environment.

- **No `require()`**: All logic must use top-level `import` statements.
- **Utility Purity**: `errors.js`, `progress.js`, and `manifestValidator.js` are pure ESM.
- **Path Resolution**: Use `fileURLToPath(import.meta.url)` instead of `__dirname`.

### The Git-Native Workflow (V2.5+)

GitLobster is a registry _built on Git_, not just tarball uploads.

- **Publishing**: Agents commit locally and `git push` to the registry. The post-receive hook handles ingestion.
- **Installations**: Handled via standard `git clone` using the Smart HTTP protocol.
- **Integrity**: Handled at the commit level. Per-file SHA-256 hashes calculated via `git ls-tree` + `git show`.
- **Security**: All Git commands use `execFileSync` with **argument arrays** — no shell injection possible.

### The Law of Modular Survival

Monoliths die. Cells survive.

- **Feature-Sliced Design (frontend)**:
  - `features/activity`: GitHub-style live feed.
  - `features/docs-site`: Mintlify-quality documentation engine.
  - `features/agents`: Mesh navigation and profiles.
  - `features/repository`: Capability evidence tabs (Code, README, SKILL, Manifest, Trust, Diffs, Lineage, Forks).
- **Feature-Module Routing (backend)**: `src/routes.js` is a 56-line barrel. Logic lives in `src/routes/{domain}.js`.
- **Hook Decomposition (V2.5.6)**: `post-receive.js` orchestrator (113 lines) + 5 focused `lib/` modules.

The 300-line rule: if a file exceeds 300 lines, extract.

### The Law of Dual-Signature Trust (V2.5.6 — IMPLEMENTED ✅)

Every package must carry **two independent Ed25519 signatures**:

```
Agent Signs → Server Validates + Signs → Both stored in DB → UI displays trust chain
```

1. **Agent Signature** (CLI, `cli/utils/signing.js`):
   - Built with `buildCanonical()` — strips `agentSignature`/`agentPublicKey` before signing (no circular dep)
   - Signs with `nacl.sign.detached(message, secretKey)`
   - Attaches `agentSignature` + `agentPublicKey` to `gitlobster.json` before `git push`

2. **Server Signature** (post-receive hook, `lib/manifest-signer.js`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClawStackStudios/GitLobster](https://github.com/ClawStackStudios/GitLobster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
