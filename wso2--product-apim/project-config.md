---
trigger: always_on
description: Rules for authoring Cucumber product tests in this module. Read before writing any test.
---

# Writing integration-v2 tests

Rules for authoring Cucumber product tests in this module. Read before writing any test.

## 1. Search before you write
Duplicate tests are the #1 problem here. First check the coverage tree
(`../../docs/devs/v2-public-feature-coverage-map.md`) and the existing `.feature` files for the
capability you're about to test. Extend an existing feature/scenario if one fits; only add a new
file when nothing does.

## 2. Where it goes
Folders are shallow and by capability: `features/{publisher,devportal,gateway,admin,key-manager,analytics}/`
plus non-product `common/`, `framework-verification/`, `migration/`. The folder is just the physical
home and shared-fixture context — **`@cap` is the source of truth**, so a file may live in `publisher/`
yet be `@cap:gateway`.
- **A scenario lives in the folder of its `@cap`, UNLESS it is bound to another feature's shared fixture.**
  When you add a scenario whose `@cap` differs from the file's folder, apply this test:
  - **Move it to its `@cap` folder** if it is **self-contained** (creates its own resources inline; no
    dependence on a `Background`/`_setup_*`/block fixture in the host file) **and thematically orthogonal**
    to the host feature's subject — i.e. it was merely *parked* here because it happens to touch the host's
    resource. Example (do NOT repeat): a `@cap:devportal` "API reflects a custom environment's vhost"
    scenario does not belong in `admin/gateway_environments.feature` just because it creates an environment;
    it is self-contained, so it moves to `devportal/` and tags the env/publish prerequisites `@dep:admin` /
    `@dep:publisher`.
  - **Leave it co-located** (the legitimate case §2 permits) if it **depends on the host's shared fixture or
    block setup** (e.g. an org-provisioning harness, an AI-API setup) or is an **intrinsic facet of a
    cohesive cross-plane suite**. Example: the `@cap:devportal` application-visibility scenario in
    `admin/organization_visibility.feature` needs that block's org-claim SOAP + org-user provisioning, so it
    stays — correctly tagged `@cap:devportal @dep:admin`.
  - Either way, **`@cap` states the true subject and every cross-capability prerequisite is a `@dep:<cap>`**
    (see §3) — never let a scenario's `@cap` drift to the folder, and never leave an admin/publisher/etc.
    prerequisite untagged.

## 3. Tags
Every product scenario is tagged. Valid `@cap`/`@feat` values are the closed vocabulary defined in
`../../docs/devs/capability-map.yml` — a tag not in that file fails lint.

| Tag | Cardinality | Meaning |
|-----|-------------|---------|
| `@cap:<id>` | exactly one | the capability under test (the subject of the assertions) |
| `@feat:<id>` | exactly one | feature under that capability |
| `@rule:<slug>` | 0–1 | free-text sub-grouping within a feature |
| `@type:smoke\|negative\|regression` | 0+ | test nature (selection axis) |
| `@dep:<cap>` | 0+ | a cross-capability **prerequisite** (NOT coverage of it) |
| `@legacy:<Class>` | 0+ | the legacy class this replaces (parity tracking) |

- **One `@cap` = one thing under test.** If you can't pick a single `@cap`, the scenario is doing too
  much — split it.
- `@dep` is for non-obvious cross-capability needs (e.g. gateway throttling needs an admin policy →
  `@dep:admin`). Don't tag the universal baseline (everything needs an API + token).
- Non-product features use exactly one exclusion marker — `@infra`, `@framework`, `@migration`, or
  `@setup` (reusable prerequisite features, see §10) — and are skipped from the product tree.

## 4. Isolation (the core concurrency rule)
Tests run in parallel on shared containers. **Every test owns its resources and shares nothing
mutable.**
- **Never name a resource by hand.** Use the shared naming utilities (`utils/Utils`,
  `utils/TestContext`) so names are unique by construction. Hardcoded names = cross-test collisions.
- No reliance on global state or on artifacts created by another scenario/class.
- **Cross-tenancy:** if a test needs other tenants, create them within that test class (via
  `utils/TenantUserProvisioner`), isolated from other classes — never reuse another class's tenant.
- **Wait, never sleep.** Poll/await for readiness (`utils/ServerReadiness` and equivalents). No
  `Thread.sleep` — it is the main cause of flaky parallel tests.

## 5. Cleanup
- Every resource a test creates is removed.
- Cleanup is **idempotent** and **runs even on failure** — do it in hooks, not as inline teardown
  scenarios that get skipped when an earlier step fails. Register created ids via
  `ResourceCleanup.register(CREATED_API_IDS / CREATED_APPLICATION_IDS, …)` (NOT bare
  `TestContext.addToList` — `register` tags each id with the **actor that created it** so teardown can delete
  it as that principal); both teardown paths below sweep them through `utils/ResourceCleanup`.
- **Teardown deletes each resource as its creating actor.** `ResourceCleanup` resolves the owner recorded at
  registration and deletes with that actor's token (devportal token for applications, publisher token for
  APIs/policies/scopes) — so a resource created by, say, `admin@tenant1` is deleted as `admin@tenant1`, not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wso2/product-apim](https://github.com/wso2/product-apim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
