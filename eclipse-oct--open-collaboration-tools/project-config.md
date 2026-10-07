---
trigger: always_on
description: TypeScript monorepo implementing the Open Collaboration Protocol (OCP): live-sharing of an IDE
---

# Open Collaboration Tools

TypeScript monorepo implementing the Open Collaboration Protocol (OCP): live-sharing of an IDE
workspace between peers, relayed by a server that never sees plaintext. npm workspaces, Node >=
20.10.0, TypeScript 5.8 project references, ESLint 9 flat config, Vitest 3.

## Commands

Run from the repo root. The whole loop is fast — there is no reason to skip verification.

```sh
npm ci                                       # 8s (fresh clone, warm npm cache)
npm run build                                # 5s clean, 2-4s incremental — tsc -b + per-package builds
npm run lint                                 # 3s — eslint, flat config, no path args needed
npm run check-nls                            # <1s — VS Code manifest %placeholders% resolve
npx vitest run                               # 5s — full suite, 46 tests in 7 files
npx vitest run <file>                        # 1s — single file
npx vitest run -t "<test name>"              # 2s — single test by name
npm run start                                # server on :8100; GET /api/meta returns 200 JSON
npm run dev                                  # Monaco playground on :5173 (Vite)
npm run build:clean                          # when project-reference builds go stale
```

- `npm test` is bare `vitest`, which runs once and exits when stdin is not a TTY (verified), and
  stays in watch mode in an interactive terminal. Use `npx vitest run` to be explicit either way.
- There is no root typecheck script — `npm run build` is the typecheck.
- Workspace-scoped commands take the **package** name, not the directory name. The VS Code extension
  lives in `packages/open-collaboration-vscode` but is named `open-collaboration-tools`:
  `npm run check-types --workspace=open-collaboration-tools`.

## Why and where

One protocol, several clients. Everything below `packages/`:

- `open-collaboration-protocol/` — the wire protocol: message types, msgpack encoding, compression,
  and the end-to-end encryption layer. Every other package depends on it.
- `open-collaboration-yjs/` — binds OCP's sync channel to Yjs CRDT documents and awareness.
- `open-collaboration-server/` — Express 5 + Inversify relay: rooms, peers, auth endpoints, JWT
  credentials. Holds session state **in memory**, so it does not scale horizontally.
- `open-collaboration-vscode/` — the VS Code extension (published as `typefox.open-collaboration-tools`).
- `open-collaboration-monaco/` — single Monaco editor client; `npm run dev` serves its example.
- `open-collaboration-service-process/` — stdio JSON-RPC bridge so non-TypeScript IDEs can join.
- `open-collaboration-agent/` — CLI that joins a room as an AI participant over ACP.

`.vscode/launch.json` carries the real debug loop: "Launch Server", "Run VS Code Extension", then
"Spawn Second Extension Instance" for a two-peer session against `localhost:8100`.

## Conventions

- **Every `.ts`/`.tsx` file under `src/` or `test/` starts with the MIT header** — ESLint fails
  without it, but only says "missing header", not what the header is. Copy it verbatim from a
  neighbouring file. The rule checks the block's shape, not the year.
- **Relative imports carry a `.js` extension** — all packages are ESM under NodeNext, so a sibling
  module is imported with the `.js` suffix even though the file on disk ends in `.ts`.
  The one exception is `open-collaboration-vscode`, which has no `"type": "module"` and so is
  CommonJS; extensionless relative imports are correct there. Match the file you are editing.
- `tsconfig.build.json` lists project references, and `open-collaboration-vscode` is deliberately not
  among them — that package is typechecked by its own `check-types` script instead. A new package
  needs an entry there (or its own build script) or it is silently never built.
- Packages depend on each other by published version range (`"open-collaboration-protocol": "~0.3.3"`),
  not by `workspace:*`. A change to the protocol's shape reaches six consumers — build the whole repo,
  not just the package you edited.
- User-facing strings in the VS Code extension go through l10n: `%key%` in `package.json` resolved by
  `package.nls.json`, and `vscode.l10n.t()` in code exported to `l10n/bundle.l10n.json`. Adding a
  command or setting without its `package.nls.json` entry ships a literal `%oct.foo%` to users.
- ESLint does not lint `**/scripts/**` — build scripts are exempt from the header and style rules.

## Boundaries and definition of done

- Never hand-edit generated output: `lib/`, `dist/`, `bundle/`, `out/`, `*.tsbuildinfo`. Regenerate
  with `npm run build`.
- `test-collab-project/` is a scratch workspace for manual session testing; its contents are
  gitignored. `.vscode/extension-host-data-*/` likewise.
- Never commit secrets. `.env` is gitignored and is also the default entry in the extension's
  `oct.files.exclude`, which keeps it out of shared sessions.
- **The server must not be able to read session content.** Peer payloads are encrypted with a room
  symmetric key wrapped per-peer; the server relays opaque bytes. Do not add server-side inspection,
  logging, or transformation of message content.
- Done means `npm run lint`, `npm run check-nls`, `npm run build`, and `npx vitest run` all pass,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eclipse-oct/open-collaboration-tools](https://github.com/eclipse-oct/open-collaboration-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
