---
trigger: always_on
description: Web repository / presenter layering — DTO/PM, Get* mappers, gateways; points to page-presenter rule for routes
---


# Web repository / presenter architecture (short)

## Layering (non‑negotiable)

- **Route / component (`.svelte`)**: UI only — **no** `httpGateway` or repository calls from routes. See **web-page-presenter-conventions** for how routes import **page presenters** and wire toasts.
- **Page presenter (`$lib/area-*/...Page.presenter.svelte.ts`)**: orchestrates a screen; owns `$state` where needed; accepts **feature presenters** (and repos) via constructor injection. **Route/toast/parent-child / feature injection** → **web-page-presenter-conventions**.
- **Feature presenter (`$lib/<feature>/*.presenter.svelte.ts`)**: stateful presenter for a vertical slice (scheduler, composer, …); calls **repositories**; may expose **`$state`** view models; **no** `$app/*` / `goto`. Instantiated in **`area-*`** **`index.ts`** and injected into **page presenters**. Imports resolve as **`$lib/<feature>/<Name>.presenter.svelte`** (on-disk **`*.presenter.svelte.ts`**). Not the same as **`Get*`** (those are stateless PM→VM mappers).
- **Repository (`*.repository.svelte.ts`)**: I/O via `HttpGateway`, **DTO → PM**, returns **PM** only.
- **Get presenter (`Get*.presenter.svelte.ts`)**: stateless **PM → VM** mapping for reads/lists.

## Type placement (default)

- **DTOs / `*ResponseDto`**: next to the **repository** that consumes them.
- **PM (`*ProgrammerModel`)**: same file as the **repository** class (unless shared across repos).
- **VM for `Get*` outputs** (reads/lists): same file as **`Get*.presenter.svelte.ts`** (e.g. **`BlogPostCommentViewModel`** next to **`loadPublishedBlogPostComments`**).
- **Mutation result `*ViewModel`** (discriminated `{ ok: … }` exposed to routes/UI): **not** in the repository file. Prefer **page presenter** when screen-specific (**`PublicBlogMutationResultViewModel`**), or **feature presenter** when shared orchestration wraps the repo (**`PlugMutationResultViewModel`** on **`UpsertGlobalPlugPresenter`**). Map from **`BlogUpsertProgrammerModel`**, **`PlugUpsertProgrammerModel`**, etc. inside that presenter.
- **Feature `*.types.ts`**: Zod / form types and other **non-repository-only** shared UI types.

## Naming (quick rules)

- Repository results: **`*Pm`** (`resultPm`, `listPm`).
- **Exported type / interface names** (rows, mutation results, screen VMs): **`*ViewModel`** only — e.g. **`AccountListingCollectionItemViewModel`**, **`ExtensionCardViewModel`**. **Do not** use **`*Vm`** as a type suffix (avoid **`FooRowVm`**, **`AccountListingCollectionItemVm`**).
- **Presenter fields, locals, and getters** holding those types: **`*Vm`** suffix — e.g. **`exploreExtensionCardsVm`**, **`bookmarkedExtensionsVm`**, **`resultVm`**. Avoid bare names like **`exploreExtensionCards`** when the value is presenter-owned VM state.
- DTOs: **`*Dto`**, **`*ResponseDto`**.

### `Get*Presenter` method naming (project convention)

- **PM → VM mappers**: prefer **`toXxxVm(pm)`** (or `toXxxListVm(listPm)` when helpful). Pure **`map*` / `merge*` / `format*`** helpers may live in the same file as **`Get*`** when they support one read path and tests
- **Read/load methods that return UI shapes**: prefer **`loadXxxVm(...)`** / **`loadXxx*Vm(...)`**. For **aggregate reads** (parallel repository calls, merged result), use a **`loadXxxVmStateless`** name when the method **does not** mutate any presenter **`$state`**; a discriminated **`{ ok: true; … } | { ok: false; error: string }`** return is acceptable when the caller must treat **partial HTTP failure** as one screen-level error
- **Avoid** `mapXxxPmToVm` / `getXxxVm` / `listXxx` on `Get*Presenter` **method names** unless there is a very specific reason (standalone **`map*`** **functions** next to **`Get*`** are fine).

## Rule of thumb (enforced)

- **Repositories return PM** (including discriminated **`{ ok: … }`** unions), not arbitrary screen **`…ViewModel`** types — except documented **DTO == PM** aliases.
- **`Get*Presenter`** is the **read/list VM boundary** (map PM → VM); **`load…Vm`** methods return **`Vm`**, **`Vm[]`**, or **`Vm | null`** — not PM arrays to **`$lib/ui`** or routes.
- **Page / feature presenters** are the usual **mutation VM boundary**: convert **`resultPm`** from **`await repo…`** into a **`*MutationResultViewModel`** (or delegate to a feature presenter that already returns one). Do not thread **`BlogUpsertProgrammerModel`** / **`PlugUpsertProgrammerModel`** through callback props typed for UI consumers.
- **DTO == PM** is allowed only when the wire shape is identical; still treat the return as PM at the boundary.
- **Prefer clean returns in presenters:** for public **read** flows, prefer returning `null` / `[]` over `{ ok: true } | { ok: false }` unions. Keep unions for **repository I/O**, **aggregate multi-fetch** **`Get*`** loads, or when the caller truly needs multiple failure modes.

---

# Repository / presenter architecture (web)

We use a **concept-first, layered architecture** so logic lives outside the framework and we can test **view models (presenter state) instead of the DOM**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
