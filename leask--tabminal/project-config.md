---
trigger: always_on
description: Last updated: 2026-03-27
---

# Tabminal Agent Notes

Last updated: 2026-03-27

This file is for future coding agents working in this repo.
Keep it current, specific, and operational. The main goal is to preserve the
current product contracts so later work does not regress multi-host behavior,
ACP agent UX, or mobile ergonomics.

## 1) Project Snapshot

- Runtime: Node.js `>= 22`, ESM project.
- Backend entry: `src/server.mjs`
- Frontend entry: `public/app.js`
- PWA shell:
  - `public/index.html`
  - `public/sw.js`
- Native app workspace:
  - `apps/Apple`
  - `apps/ghostty-vendor`

Current product shape:

- Persistent terminal sessions remain the core product.
- There are now two AI surfaces:
  - terminal-native assistant in `src/terminal-session.mjs`
  - ACP agent workspace in `src/acp-manager.mjs` + `public/app.js`
- One web UI can connect to multiple Tabminal hosts.
- The workspace bar now mixes file tabs, agent tabs, and pinned terminal tabs.

Important persistence files under `~/.tabminal`:

- `config.json`
- `cluster.json`
- `agent-tabs.json`
- `agent-config.json`
- `auth-sessions.json`

## 2) Non-Negotiable Contracts

### 2.1 Host model and state isolation

- UI term is `Host`, not `Server`, for user-facing labels.
- Every session belongs to exactly one host.
- Session state, editor state, file tree state, ACP agent tabs, and workspace
  tabs are host-isolated.
- Do not merge runtime state across hosts.

Relevant code:
- `public/app.js`
- `public/modules/session-meta.js`

### 2.2 Auth model

- Main host (`id = 'main'`) controls the global login modal.
- Only main-host `401/403` should trigger app-wide login UI.
- Sub-host auth failures must stay local to that host.
- Sub-hosts may require Cloudflare Access login without password change.

Relevant code:
- `public/app.js`
  - `ServerClient.handleUnauthorized`
  - `ServerClient.handleAccessRedirect`

### 2.3 Token storage

- Auth state is browser-owned.
- All hosts persist auth state in browser `localStorage`.
- Key format:
  - `tabminal_auth_state:<hostId>`
- Stored auth state currently includes:
  - short-lived access token
  - access token expiry
  - refresh token
  - refresh token expiry
- Legacy password-hash token storage is no longer supported or migrated.
- Removing a host should also remove any stale local auth state keyed to that
  host.

Relevant code:
- `public/app.js`
- `src/persistence.mjs`

### 2.4 Host registry persistence

- Frontend is not the source of truth for host list persistence.
- Source of truth is backend `GET/PUT /api/cluster`.
- File format:
  - `{ "servers": [{ id, baseUrl, host, token }, ...] }`
- On page load, host list restores only after main-host auth succeeds.

Relevant code:
- `public/app.js`
- `src/server.mjs`
- `src/persistence.mjs`

### 2.5 Deduplication and self-host skip

- Host uniqueness key is normalized `hostname[:port]` in lowercase.
- Path is intentionally not part of the dedupe key.
- Hydration skips entries that resolve to the current main node to avoid
  self-loop duplicates.
- Path-based multi-host routing such as `/a/*` and `/b/*` is not supported by
  the current client assumptions.

Relevant code:
- `public/modules/url-auth.js`
- `public/app.js`

### 2.6 Session creation ownership

- Backend does not auto-create a default session anymore.
- Frontend is responsible for usability:
  - create one main-host session if init returns none
  - recreate one main-host session if user closes the last session

Relevant code:
- `public/app.js`
- `src/server.mjs`

### 2.7 Polling and heartbeat

- Frontend heartbeat cadence: `1000ms`
- Reconnect retry throttle per host: `5000ms`
- Do not weaken these without measuring UX fallout.

Relevant code:
- `public/app.js`

### 2.8 Cloudflare Access handling

- Non-main host requests use `credentials: 'include'`.
- Non-main host fetches default to `redirect: 'manual'`.
- Access redirects should show reconnect/login state, not generic password
  failure.
- Reconnect UI may open the host root in a new tab for Access login.

Relevant code:
- `public/app.js`
- `public/modules/url-auth.js`

### 2.9 Runtime version and PWA coherence

- Backend exposes unauthenticated `GET /api/version` for bootstrap versioning.
- Backend heartbeat returns runtime boot id.
- `index.html` must fetch `/api/version` before choosing app-shell asset
  versions.
- Version bootstrap currently uses a `3s` timeout; timeout/error may fall back
  to the last stored boot id, then to a cold key.
- Frontend appends `?rt=<bootId>` and reloads on runtime change.
- `index.html` versions `styles.css` and `app.js` with runtime key.
- Service worker is versioned the same way.

Relevant code:
- `src/server.mjs`
- `public/app.js`
- `public/index.html`
- `public/sw.js`

## 3) ACP Design Contracts

### 3.1 ACP architecture shape

- Tabminal embeds an ACP supervisor in the backend.
- ACP agents are not implemented in-repo; Tabminal launches or attaches to
  external ACP runtimes.
- Built-in definitions currently include:
  - Gemini CLI
  - Codex CLI
  - Claude Agent
  - GitHub Copilot
  - ACP Test Agent when `TABMINAL_ENABLE_TEST_AGENT=1`

Relevant code:
- `src/acp-manager.mjs`
- `src/acp-test-agent.mjs`

### 3.2 ACP workspace model

- Agent tabs share the same workspace strip as file tabs and pinned terminal
  tabs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leask/Tabminal](https://github.com/Leask/Tabminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
