---
trigger: always_on
description: This repository implements a browser-friendly simulation of key Kubernetes
---

This repository implements a browser-friendly simulation of key Kubernetes
components, with an emphasis on:

- an in-process apiserver/storage model
- etcd-like primitives
- a fake client library under `src/client/` that is meant to be type-compatible
  enough with `@kubernetes/client-node` for shared tests to run against both
  the real client and the fake client

The fake client is not a wrapper around the real client and should not depend on
`@kubernetes/client-node` at runtime or in its exported source types. That
package is available as a dev dependency for reference, comparison, and tests
only.

General rules for this repository:

- Prefer preserving the broad structure of the real
  `kubernetes-client/javascript` repository where that helps compatibility, but
  keep the fake implementation human-readable and editable.
- The simulator is not currently trying to support init containers, ephemeral
  containers, or Kubernetes' volumes / CSI subsystem. Keep new simulator work
  scoped to regular containers unless a task explicitly changes that scope.
- The simulator is not currently trying to support image pull secrets,
  credential providers, or private registry authentication. Preserve upstream
  call shapes around image pull credentials where useful for parity, but keep
  credential resolution as a no-op unless a task explicitly expands image
  authentication support.
- The simulator is not currently modeling pod/container resource requests,
  limits, resize policy, in-place pod vertical scaling, or kubelet
  `allocationManager` behavior. Do not add allocation-manager parity code in
  pod workers unless the task explicitly expands simulator resource handling.
- The simulator has partial static pod bookkeeping, but does not currently
  support static pods end to end. Do not add static-pod-specific behavior unless
  the task explicitly expands static pod support.
- The simulator does not currently model Kubernetes `RuntimeClass` objects or
  kubelet's `runtimeClassManager`. Preserve upstream call shapes around runtime
  handlers where useful for parity, but resolve runtime handlers as the default
  empty string unless a task explicitly expands RuntimeClass support.
- In cluster simulation code, do not call global timer/time APIs such as
  `setTimeout`, `setInterval`, or `Date.now` directly. Route timeout, interval,
  and current-time behavior through the cluster `Clock` instance so the
  simulator can be paused and controlled deterministically.
- When a function or constructor accepts a `context.Context`, make that argument
  the first parameter and name it `ctx`, matching Kubernetes Go conventions.
- For intentionally unused parameters, prefix the parameter name with an
  underscore, such as `_ctx`; do not add `void parameter;` statements just to
  silence unused-variable checks.
- When transliterating Go functions into TypeScript, use TypeScript naming
  conventions for function names. In particular, do not preserve Go's exported
  initial capital; `GetPartialReference` should become `getPartialReference`.
- When mirroring Go code that returns `(value, error)`, prefer explicit tuple
  returns such as `[value, err]` over throwing exceptions. If you find
  try/catch or thrown errors in mirrored code where upstream returns an error,
  treat that as parity debt and convert it unless there is a clear local
  boundary that requires exceptions.
- Do not copy upstream comments when transliterating code. Keep local comments
  only when they explain simulator-specific behavior or non-obvious local
  constraints; readers can refer to the upstream source for upstream commentary.
- Files that mirror upstream Kubernetes code should keep local source-mapping
  comments in the form `// Models <upstream path> <name>` before mirrored
  declarations. These comments are not copied upstream commentary; they are
  required breadcrumbs for reviewing parity.
- For Kubernetes `pkg/` paths, this repository's `src/` directory acts as the
  package root. For example, upstream `pkg/util/parsers` should be mirrored
  under `src/util/parsers`, not `src/pkg/util/parsers`.
- When changing code that mimics real Kubernetes behavior, protect parity even
  in conversation. If a requested change would move the implementation away
  from the upstream structure or behavior being modeled, challenge the request,
  explain the parity issue, and lay out the viable options before editing.
- When mirroring Go `iota` enums in TypeScript, use string unions for the named
  enum values. If code needs to model the Go zero value, use the first `iota`
  enum value; do not add an empty string to the union to emulate a zero value.
- When adding zero-value helper constructors in TypeScript, name them
  `new<TypeName>`, such as `newContainerStatus`, accept a `DeepPartial` of the
  type they create for overrides, and define them in the same file as the type
  they construct.
- Simulator behavior is currently targeting Kubernetes 1.36. A local checkout
  of `kubernetes/kubernetes` at commit
  `ecf6decece6a6de25a57aad9ba90b6ce580f6f78` is available at
  `~/Developer/github.com/kubernetes/kubernetes` for upstream source reference.
- When asked to copy, mirror, port, transliterate, audit, or compare against

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngrok/webernetes](https://github.com/ngrok/webernetes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
