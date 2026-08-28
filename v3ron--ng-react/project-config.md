---
trigger: always_on
description: This repository implements **ng-react**: a modular application framework for React and
---

# AGENTS.md — ground truth for contributors and sub-agents

This repository implements **ng-react**: a modular application framework for React and
React Native that brings Angular 2+'s *guarantees* (module boundaries, DI, deterministic
lifecycle) to React — deliberately **not** its mechanisms (no decorators, no
`reflect-metadata`, no hierarchical injectors).

**Read `docs/spec/01-kernel-and-module-system.md` before writing any code.** It is
normative. Every requirement in it is numbered (`M1`, `D3`, `C8`, `A2`, `H4`, `R2`, …) and
those numbers are the contract between issues, code, and tests.

---

## 1. Repository layout

```
.
├── AGENTS.md                    # this file
├── docs/spec/                   # normative specs (01 = kernel & module system)
├── package.json                 # workspace root; scripts run from here
├── pnpm-workspace.yaml
├── tsconfig.base.json           # shared compiler options; every project extends it
├── tsconfig.json                # solution file (project references)
├── vitest.config.ts             # single root config, three named projects
├── eslint.config.js             # flat config
├── .fallowrc.json               # fallow (dead code / dupes / health)
├── packages/
│   └── ng-react/                # @ng-react/kernel — the library
│       ├── package.json         # exports "." -> ./src/index.ts (source exports, no build step to consume)
│       ├── tsconfig.json        # composite; emits to dist/ for `pnpm build`
│       └── src/
│           └── index.ts         # THE public API barrel — see §4
└── apps/
    └── react/                   # @ng-react/demo-react — Vite + React 19 demo & acceptance app
```

The workspace has since grown the packages the staged plan created. As of day 3:

```
packages/
├── ng-react/                    # @ng-react/kernel — the library
├── eslint-config-modules/       # @ng-react/eslint-config-modules — the B1-B3 lint preset (stage 7)
├── create-module/               # @ng-react/create-module — the B4 generator (stage 7)
├── auth/  orders/  payments/  debug/   # @app/<id> — the demo's feature modules (stage 8)
└── nav/                         # @app/nav — the PoC navigation module (criterion 10)
```

The five `@app/*` packages are what every acceptance test is written against, and each was
**generated with `pnpm create-module`** rather than hand-written — if a generated package needs a
hand-edit to pass `pnpm verify`, that is a generator bug and belongs in `packages/create-module`.
Create new packages only in the task that owns them.

---

## 2. Toolchain — fixed facts

| Fact | Value | Why it matters |
|---|---|---|
| Package manager | **pnpm 11** workspaces | `npm`/`yarn` will corrupt the lockfile. Never run them. |
| Node | >= 22 | |
| TypeScript | **5.9.3** — pinned | TS 7 is out but `typescript-eslint@8` does not support it yet. **Do not bump TypeScript.** |
| Bundler (app) | Vite 8 + `@vitejs/plugin-react` | |
| Test runner | **Vitest 4**, `globals: true` | |
| Lint | ESLint 10 flat config + `typescript-eslint` 8 | |
| Dead code | `fallow` 3 | `pnpm fallow` |
| Boundary lint | `@ng-react/eslint-config-modules` (this repo) | Wired into root `eslint.config.js` **by package name**, so the workspace `exports` map is exercised. |
| React | 19.2 — a **required** peer dependency of `@ng-react/kernel` | |

> **`pnpm-workspace.yaml` has `allowBuilds: unrs-resolver: true`.** `unrs-resolver` is the
> native resolver behind `eslint-plugin-import-x`, which backs `import-x/no-cycle` (B3).
> pnpm will not run its postinstall build without this approval, and without the build,
> `pnpm lint` fails outright with `node with invalid interface loaded as resolver`. It is a
> dev-only transitive dependency. Do not remove the entry.

### Commands (always run from the repo root)

```bash
pnpm install          # after touching any package.json
pnpm typecheck        # tsc across all projects
pnpm lint             # eslint
pnpm test             # vitest run — all three projects
pnpm fallow           # dead-code report (advisory, not gating)
pnpm verify           # typecheck + lint + test — THE gate for every PR
```

**`pnpm verify` must pass before you open a PR. No exceptions.**

---

## 3. Test conventions — this is load-bearing

`vitest.config.ts` defines three projects. **The file extension picks the environment:**

| File pattern | Project | Environment | Use for |
|---|---|---|---|
| `packages/ng-react/src/**/*.test.ts` | `kernel` | **node** — no DOM, no React renderer | kernel, container, lifecycle, `createTestKernel` |
| `packages/ng-react/src/**/*.test.tsx` | `kernel-dom` | jsdom + React plugin | React bindings only |
| `apps/react/src/**/*.test.{ts,tsx}` | `demo` | jsdom | demo app / acceptance |

This is not cosmetic. **Acceptance criterion 7** requires that the kernel and
`createTestKernel` work "in a plain Jest/Vitest environment with no React renderer". The
`node` project is the machine-checked proof of that: if a kernel test needs jsdom, the
kernel has a dependency it must not have.

Tests live **next to** the code they test (`src/container/resolver.ts` →
`src/container/resolver.test.ts`).

Spec-traceability: name tests after the requirement they pin, e.g.

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [V3RON/ng-react](https://github.com/V3RON/ng-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
