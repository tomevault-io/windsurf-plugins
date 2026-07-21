---
trigger: always_on
description: This document helps AI coding agents (GitHub Copilot, Claude, Cursor, etc.) understand the cdxgen codebase conventions, architecture, and contribution rules so they can produce code that fits naturally with the existing style.
---

# AGENTS.md — cdxgen contributor guide for AI agents

This document helps AI coding agents (GitHub Copilot, Claude, Cursor, etc.) understand the cdxgen codebase conventions, architecture, and contribution rules so they can produce code that fits naturally with the existing style.

---

## Project overview

**cdxgen** is a universal, polyglot CycloneDX Bill-of-Materials (BOM) generator. It produces SBOM, CBOM, OBOM, SaaSBOM, CDXA, and VDR documents in CycloneDX JSON format. It is distributed as an npm package (`@cyclonedx/cdxgen`), a container image, and a Deno/Bun-compatible script.

Primary entry points:

- **CLI** — `bin/cdxgen.js` (calls into `lib/cli/index.js`)
- **Audit CLI** — `bin/audit.js` (`cdx-audit` predictive supply-chain audit with `console`, `json`, and `sarif` reporters)
- **Conversion CLI** — `bin/convert.js` (`cdx-convert` CycloneDX → SPDX export)
- **Library** — `lib/cli/index.js` exports `createBom`, `submitBom`
- **HTTP server** — `lib/server/server.js` (started via `bin/repl.js` or `cdxgen --server`)
- **REPL** — `bin/repl.js`

Companion binaries:

- cdxgen opportunistically uses the optional companion package/repo **`cdxgen-plugins-bin`** for heavyweight native helpers such as **Trivy**, **osquery**, **SourceKitten**, **dosai**, and **golem**.
- Dynamic SBOM generation via **tracebom** depends only on `@cdxgen/safer-exec` + `@cdxgen/cdx-proto` from optional deps. No atom, no plugins-bin, no hbom.
- Container/rootfs inventory flows through `lib/managers/binary.js` and may combine native parsing with plugin-enriched Trivy output.
- Live-OS OBOM flows use the bundled osquery binary plus the query packs under `data/queries*.json`. Linux query packs also include hardening-focused snapshots such as `sysctl_hardening` and `mount_hardening`.
- When the optional `trustinspector` helper is present, host-path trust enrichment should be treated as batched rather than artificially capped. Keep Authenticode, WDAC, code-signing, and notarization properties intact across large inventories.
- GTFOBins enrichment is expected on Linux live-runtime osquery components in the same way LOLBAS enrichment is expected on Windows runtime components.
- Go Evinse flows use the bundled `golem` helper when available. Keep `cdx:golem:*` metadata and component properties safe, compact, and policy-friendly: counts, categories, scopes, source locations, data-flow rule IDs, taint-kind labels, crypto algorithm names/OIDs, and module facts are acceptable, but do not emit raw generator commands, environment values, HTTP parameter values, raw key material, embedded file contents, generated source contents, or secrets.

---

## Module system and runtime

- The package is **pure ESM** (`"type": "module"` in `package.json`). There is no CommonJS source except the generated `index.cjs` shim.
- The project targets **Node.js ≥ 20** with optional support for Bun and Deno (see `devEngines` in `package.json`).
- Detect the runtime with the helpers exported from `lib/helpers/utils.js`:
  ```js
  export const isNode = globalThis.process?.versions?.node !== undefined;
  export const isBun = globalThis.Bun?.version !== undefined;
  export const isDeno = globalThis.Deno?.version?.deno !== undefined;
  ```

---

## Import conventions

### Always use the `node:` protocol for built-ins

```js
// correct
import { readFileSync } from "node:fs";
import path from "node:path";
import process from "node:process";

// wrong — missing node: prefix
import { readFileSync } from "fs";
```

### Import ordering (enforced by Biome)

Biome (`biome.json`) enforces this exact three-group order, with a blank line between each group:

```
1. Node built-ins   (node:*)
<blank line>
2. npm packages     (packageurl-js, semver, undici, …)
<blank line>
3. Local modules    (../../helpers/utils.js, …)
```

---

## Code style (Biome)

The linter and formatter is **Biome** (not ESLint/Prettier).

| Setting   | Value                                                                                   |
| --------- | --------------------------------------------------------------------------------------- |
| Indent    | 2 spaces                                                                                |
| Formatter | enabled for `lib/**` and root JS/JSON (excludes `test/`, `data/`, `contrib/`, `types/`) |
| Linter    | enabled for the same scope                                                              |

Run locally:

```bash
pnpm run lint        # check + auto-fix
pnpm run lint:check  # check only (only for CI)
pnpm run lint:errors # errors only
```

Key rules to be aware of (see `biome.json`):

- `noUndeclaredVariables` — error. Don't leave variables undeclared.
- `noConstAssign` — error.
- `useDefaultParameterLast` — error (default params must come last).
- `useSingleVarDeclarator` — error (one binding per `const`/`let`).
- `noUnusedVariables` — warn (use `_prefix` for intentionally unused vars).
- `noParameterAssign` — **off** (reassigning parameters is allowed).
- `noForEach` — **off** (`.forEach()` is acceptable).
- `noDelete` — **off** (`delete obj.prop` is acceptable).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdxgen/cdxgen](https://github.com/cdxgen/cdxgen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
