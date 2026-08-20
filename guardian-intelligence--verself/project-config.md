---
trigger: always_on
description: This is a polyglot monorepo structured as a modular monolith. It contains all code for infrastructure, service, and client applications for a multi-tenant cloud computing company. It is the only repo for the entire company.
---

# verself.sh (Verself)

This is a polyglot monorepo structured as a modular monolith. It contains all code for infrastructure, service, and client applications for a multi-tenant cloud computing company. It is the only repo for the entire company.

Console: verself.sh
Auth portal: verself.sh
Services: <service>.api.verself.sh
Company website: guardianintelligence.org
Letters - Blog posts from the founder: guardianintelligence.org/letters
Newsroom - Business updates: guardianintelligence.org/newsroom

<available_tooling>
Integrations: `aspect integrations`
</available_toling>


<coding_contract>
* Always lean on open standards where possible. Avoid re-inventing the wheel.

* Expect to build with the level of rigor that would make FedRAMP HIGH certification seem realistic.
* Keep OpenTofu provisioning lean -- It does a narrow job. Let Ansible keep the boxes in order, and Bazel for build graph and Nomad for deployment orchestration. Every layer does what it's good at.
* Use nftables for perimeter, host, and guest-boundary policy. Do not encode service-to-service reachability or dependency ports in nftables.
* Always think of the governance, IAM, quotas, and metering story behind service changes. Customers must know who did what, what they're allowed to do, and how much they used.
* Think in terms of providing users a "Digital Habitat" -- their sessions should be synced across devices as much as possible.
* Never use useEffect. Very rarely, if ever, use `useState` -- prefer TanStack Query primitives for all state. Sync snowflake client-side state with the URL.
* No shell scripts. The only exceptions are the platform bootstrap entrypoints under `src/tools/dev/bootstrap/`. Choose the appropriate language and check the result into a Bazel target. Treat scripts as core load-bearing architecture + sharp knives. They are extremely dangerous and should be carefully reviewed.
* Never construct OCSF events outside a single typed builder. Hand-rolled map[string]any events drift and break SIEM rules silently.
* Treat errors as data. Use tagged and structured errors to aid control flow.
* Avoid fallbacks and defaults. Runtime behavior should fail fast with useful logging.
* Avoid verbosity. When solving a specific problem, the patch should solve the general case. E.g. if solving a TOCTOU vuln, don't write a function named `fix_toctou_bug`, make the simple patch to use the toctou-safe call and optionally leave a comment (no more than a few words).
* Don't resolve failures through silent no-ops and imperative checks. Failures should be loud; signals should be followed to address root causes. Failures are useful data!
* When you run into a footgun, leave a comment around the code (no more than a sentence) explaining the footgun and how the code works around it.
* Browser coverage belongs in ongoing live canaries with ClickHouse evidence. Browser canaries using Playwright are preferred.

* ClickHouse inserts must use `batch.AppendStruct` with `ch:"column_name"` struct tags. `batch.Append` silently corrupts data when columns are added or reordered.
* ClickHouse schema design: ORDER BY columns are sorted on disk and control compression — order keys by ascending cardinality (low-cardinality columns first). Avoid `Nullable` (it adds a hidden `UInt8` column per row); use empty-value defaults instead. Use `LowCardinality(String)` for columns with fewer than ~10k distinct values. Use the smallest sufficient integer type (`UInt8` over `Int32` when the range fits).
* Browser canaries should use short operation deadlines and diagnose behavior from traces, logs, and ClickHouse evidence instead of extending waits. Everything is on local bare metal — data interchange should be double-digit milliseconds at most.
* Our customers will use our services via API and browser. Fix issues at the service level; don't paper over them in any one domain. E2E test the browser primarily since it exercises the same API that API consumers call directly.
* No global, hand-managed /usr/local/bin. Let Bazel call out to package-specific toolchains for dev tools and deployment requirements.
* For local development, packages should offer to install onto the caller's $HOME/.local/bin, requiring an explicit --bin-dir. These shims should point back to Bazel-resolved outputs or package-manager-resolved binaries and not duplicate version state.
* When adding a binary dependency, classify it as controller/dev tooling under `src/tools/dev/binaries` or runtime/deployed tooling under the owning component's Bazel targets before exposing it through Aspect, Ansible, or deployment code.

* Avoid drift between what runs in CI and what you run for local development. CI is basically a warm dev box. Local development should give high confidence on correctness.

* The only shell scripts allowed are the platform bootstrap entrypoints under `src/tools/dev/bootstrap/`. Scripts are load-bearing tooling and infrastructure so choose the right tool for the job (it's never a shell script).
* Binaries are versioned, built, packaged, and installed by Bazel declarations owned by the component or tool that uses them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guardian-intelligence/verself](https://github.com/guardian-intelligence/verself) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
