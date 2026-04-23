---
trigger: always_on
description: This document tells you how to work inside the SolidType repo.
---

# SolidType – AGENTS GUIDE

Welcome, agent 👋  
This document tells you how to work inside the SolidType repo.

Before you write _any_ code, read:

- [`docs/OVERVIEW.md`](docs/OVERVIEW.md) – **What** SolidType is and why it exists.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) – **How** it is structured (packages, layers, data flow).
- [`docs/STATUS.md`](docs/STATUS.md) – **Current implementation status** and what's next.
- [`/plan/*`](plan/*) – The **phase-by-phase implementation plan** you must follow.

For specific subsystems, also read:

- [`docs/DOCUMENT-MODEL.md`](docs/DOCUMENT-MODEL.md) – **Yjs document schema** specification.
- [`docs/TOPOLOGICAL-NAMING.md`](docs/TOPOLOGICAL-NAMING.md) – **Persistent naming** design (FreeCAD-style algorithm).
- [`docs/AI-INTEGRATION.md`](docs/AI-INTEGRATION.md) – **AI system architecture** (Durable Streams, tools, SharedWorker).
- [`docs/CAD-UX-SPEC.md`](docs/CAD-UX-SPEC.md) – **UX specification** for sketch and feature tools.

Treat those documents as the source of truth. If they conflict with existing code, the docs win and the code should be brought back into line.

---

## Reference Implementations

The [`/refs/`](refs/) directory contains downloadable source code from production CAD kernels for reference:

- **OCCT** (Open CASCADE) – Production B-Rep kernel with boolean operations
- **CGAL** – Robust geometry algorithms, especially planar arrangements (DCEL)
- **FreeCAD toponaming** – Persistent naming implementation (realthunder's branch)
- **Fornjot** – Modern Rust B-Rep kernel with similar goals to SolidType

To download references:

```bash
cd refs && ./download-refs.sh
```

See [`refs/README.md`](refs/README.md) for detailed guidance on what to study in each reference.

---

## 1. Your Responsibilities

As an agent, your job is to:

1. Implement the plan in `PLAN.md` **incrementally**, phase by phase.
2. Keep the architecture described in `ARCHITECTURE.md` intact.
3. Preserve the overall goals and constraints described in `OVERVIEW.md`.

If you are unsure how to implement something:

- Prefer a **small, clean, testable** implementation that matches the intent,
- Add a `// TODO(agent): ...` comment referencing the relevant doc section,
- Do **not** invent new architecture without clear justification in comments.

---

## 2. Package & Layer Rules

You are working in a pnpm monorepo with:

- `@solidtype/core` (CAD kernel wrapper with OO API, powered by OpenCascade.js),
- `@solidtype/app` (full CAD application).

### OCCT Integration Guidelines

The `@solidtype/core` package now uses OpenCascade.js for all B-Rep operations:

**DO:**

- All modeling operations go through `SolidSession` (the public API)
- Use the kernel/ module for OCCT wrappers (internal only)
- Keep the sketch/ module in pure TypeScript (no OCCT dependency)
- Dispose OCCT objects properly to prevent memory leaks

**DON'T:**

- Never import from `kernel/` in the app - use `SolidSession` instead
- Never expose OCCT types (TopoDS_Shape, etc.) in the public API
- Don't add OCCT dependencies to the sketch constraint solver

### Core rules

- `@solidtype/core`:
  - **No DOM / browser APIs.**
  - **No three.js** or other rendering code.
  - Only minimal dependencies (testing, build tooling).
  - Exposes an **object-oriented API** (`SolidSession`, `Body`, `Face`, `Edge`, `Sketch`) as the primary interface.
  - Internal modules use data-oriented style (struct-of-arrays, handles) for performance.

- `@solidtype/app`:
  - May use `three.js`, Vite, React, etc.
  - Uses the OO API from `@solidtype/core` for modeling operations.

If you find yourself wanting to put core geometry or topology logic into `app`, stop and move it into `@solidtype/core`.

### Local Development: HTTP/2 Proxy

Vite's dev server only supports HTTP/1.1, which limits browsers to 6 simultaneous connections. This causes issues with Electric SQL sync and long-polling.

**Always use Caddy as an HTTP/2 reverse proxy during development:**

```bash
caddy reverse-proxy --from localhost:3010 --to localhost:3000 --internal-certs
```

Access the app at `https://localhost:3010`. See [Electric SQL Troubleshooting](https://electric-sql.com/docs/guides/troubleshooting#solution-mdash-run-caddy) for details.

---

## 3. Preferred Libraries & Tooling

When building UI and application logic in the viewer/app package, use these approved libraries:

### 3.1 UI Components – Base UI

Use **[Base UI](https://base-ui.com/react/components)** for all UI components wherever possible.

- Base UI is a headless component library (similar to Radix UI) that provides unstyled, accessible primitives.
- Available components include: Dialog, Menu, Context Menu, Popover, Tooltip, Select, Tabs, Accordion, Checkbox, Radio, Switch, Slider, Progress, Toast, and many more.
- Prefer Base UI over building custom components from scratch.
- Style components using CSS (the project already uses `.css` files alongside components).

### 3.2 Schemas & Validation – Zod

Use **[Zod](https://zod.dev)** for runtime schema validation and type inference.

- Define schemas for data structures that need validation (e.g., user input, API responses, file formats).
- Use `z.infer<typeof schema>` to derive TypeScript types from Zod schemas.
- Prefer Zod over manual validation logic.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samwillis/solidtype](https://github.com/samwillis/solidtype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
