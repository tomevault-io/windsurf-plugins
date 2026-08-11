---
trigger: always_on
description: Page presenters, routes, parent/child wiring, toast patterns (A/B), area barrels, mutation flows
---


# Page presenter & route conventions (`$lib/area-*`)

Use this rule for **route-level pages**, **page presenters** under `web/src/lib/area-*/`, and how **`+page.svelte`** / **`$lib/ui`** children wire to them.

**Related:** repository layering, DTO/PM/`Get*` mapping, and naming for **repository outputs** live in **web-repository-presenter-architecture** — do not duplicate repository rules here.

---

## Page presenter — role and placement

- **File:** `$lib/area-protected/*Page.presenter.svelte.ts`, `$lib/area-admin/*Page.presenter.svelte.ts`, `$lib/area-public/*Page.presenter.svelte.ts`, etc.
- **Class name:** e.g. `ProtectedSettingsPagePresenter`, `AdminFeedbackManagerPagePresenter`.
- **Wiring:** Instantiate the page presenter in the area **`index.ts`**, export the **singleton** (e.g. `protectedSettingsPagePresenter`) and **status enums** the route needs (e.g. `UpdateProfileStatus`). See **Area `index.ts` exports** below for what not to re-export.
- **Shapes:**
  - **Thin coordinator:** constructor takes feature presenters (and deps); exposes getters or delegates; route uses one import from the area index.
  - **Stateful page:** owns `$state`, calls repositories and/or `Get*` presenters for loads and mutations.
  - **Cross-presenter composition (reference: analytics):** a page presenter may inject **another area page presenter** when that screen reuses its read model and behavior

---

## Feature presenters (injected into page presenters)

A **feature presenter** is a stateful **`*.presenter.svelte.ts`** under a **feature** package (e.g. `$lib/posts/Scheduler.presenter.svelte`, `$lib/posts/CreateSocialPost.presenter.svelte`) that owns reusable UI orchestration for that domain: repository calls, derived or **`$state`** view models (e.g. **`ScheduledPostsCalendarVm`** on **`SchedulerPresenter.scheduledPostsCalendarVm`**), and methods the screen needs. It is **not** an area route file; it lives next to the feature repository / exports.

- **Injection:** The **page presenter** receives feature presenters **via constructor** as **`readonly`** fields (e.g. **`schedulerPresenter`**, **`createSocialPostPresenter`**). The route should use **`protectedCalendarPagePresenter.schedulerPresenter`** / **`.createSocialPostPresenter`** (or the home page singleton’s **`createSocialPostPresenter`**) for **`prepareOpen`**, **`bind:presenter`**, and passing into **`$lib/ui`** — **not** a parallel import of the same singleton from **`$lib/posts`** when the area barrel already wires it onto the page presenter.
- **Composition root:** Construct repositories and shared helpers first, then **feature presenters**, then **`Protected*PagePresenter`** inside **`$lib/area-protected/index.ts`** (or the relevant **`area-*`** barrel), in **dependency order**. Example: **`GenerateMediaModalPresenter`** → **`CreateSocialPostPresenter(postsRepository, composerMediaModalPresenter)`** → **`ProtectedHomePagePresenter(..., createSocialPostPresenter)`** → **`ProtectedCalendarPagePresenter(..., schedulerPresenter, createSocialPostPresenter)`**. Keep feature **`index.ts`** (e.g. **`$lib/posts/index.ts`**) limited to **repository + class/type exports** so it does not import **`area-protected`** — cross-cutting wiring stays in the **area** barrel to avoid circular dependencies.
- **Shared instances:** The **same** feature presenter instance may be injected into **more than one** page presenter when the product shares one composer or scheduler across screens; document that in the area **`index.ts`** next to the constructors.
- **Boundary:** **Page presenter** = screen-specific orchestration and presenter-owned screen state; **feature presenter** = vertical slice reused across account routes. **`Get*Presenter`** remains stateless PM→VM only — see **web-repository-presenter-architecture**.

### Repository call naming (`resultPm`)

In **page presenters**, **feature presenters**, and stateless **`Get*Presenter`** classes, bind the value returned from **`await …Repository.someMethod(...)`** to **`resultPm`** (repository / programmer-model layer: discriminated `{ ok: … }` unions, PM payloads, etc.). Avoid opaque names like **`r`**, **`res`**, or **`data`** for that binding. For list reads use **`listPm`** (e.g. **`ApprovedAppsRepository.list()`** → **`ListApprovedAppsProgrammerModel`**); for a single mutation outcome **`revokePm`** / **`createPm`** is fine when **`resultPm`** would be ambiguous.

**Type vs field naming (required):** exported row / screen types use the **`ViewModel`** suffix only (e.g. **`ApprovedAppRowViewModel`**, **`AccountListingCollectionItemViewModel`**). Presenter properties that hold those values use the **`*Vm`** suffix (e.g. **`itemsVm`**, **`exploreExtensionCardsVm`**, **`bookmarkedExtensionsVm`**). Do not name exported types **`…Vm`**; do not name presenter VM state without the **`Vm`** suffix (e.g. avoid **`exploreExtensionCards`**).

After mapping to UI-facing shapes, use **`resultVm`** (or domain-specific names like **`groupVm`**) for the mapped view model. Example:

```ts
const resultPm = await this.postsRepository.getPostGroup(postGroup);
if (!resultPm.ok) return null;
return toPostGroupDetailsVm(resultPm.group);
```

### Mutation return types (`*ViewModel`, not PM)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
