---
trigger: always_on
description: cd kof-operator && make build|run|docker-build|test|lint
---

# Copilot Coding Agent Instructions for k0rdent/kof

### Common Commands

```bash
# Operator validation commands
cd kof-operator && make build|run|docker-build|test|lint

# Web UI validation commands
cd kof-operator/webapp/collector
npm install && npm run build
npm run lint   # max-warnings=0 enforced
npm test       # vitest + jsdom

# Helm validation command
make helm-package
```

### CI/CD Requirements

All PRs must pass:
- ✓ Conventional commits (`feat`, `fix`, `docs`, `test`, `ci`, `refactor`, `perf`, `chore`, `revert`)
- ✓ Go tests (`make test`), React lint + tests, `npm audit` (no moderate+)
- ✓ Helm docs generated and current
- ✓ PRs touching charts: deploy to kind, both `dev` and `dev-istio` scenarios

**PR Title Format:** `<type>(<scope>): <description>`

---

## PR Review: Context-Awareness Rules

> These rules are derived from patterns where reviewers pushed back on Copilot comments. Read context before flagging.

1. **`helm upgrade -i --reset-values` is used in this project.** Don't flag missing default values as "will break upgrades" — reset-values means the chart always starts fresh.
2. **Check `values.yaml` for existing docs before flagging missing documentation.** Reviewers have rejected "this is undocumented" comments when the docs were already present.
3. **Global Helm values applying to multiple resources is intentional by design.** Don't suggest splitting `global.helmRepo.*` or similar shared config blocks into per-resource blocks unless there is a concrete conflict.
4. **For HTTP POST endpoints, consuming all parameters from the request body (and not re-adding them to the URL) is intentional** — URLs have a ~4 KB limit; this is not a bug.
5. **CI `paths:` trigger restrictions are often intentional.** Don't flag a workflow that only triggers on `charts/**` as "missing coverage" — that may be a deliberate design to limit CI scope.
6. **Infrastructure-specific values (e.g., `gatewayClassName: cloud-provider-kind`) map to actually installed components** — don't flag them as "unlikely to resolve" without verifying the deployment environment.
7. **The mothership cluster has no collectors** (`DISABLE_KOF_COLLECTORS=true`). Don't assume a multi-cluster step should run against mothership context; check which clusters actually have the target resource.
8. **Helm `mergeOverwrite` order is intentional.** When you see `mergeOverwrite $a $b`, the author wants `$b` to win. Don't suggest swapping order unless you can prove correctness requires it.
9. **Avoid making the same comment multiple times in one PR.** If duplicate files contain the same pattern, note it once and reference the other locations.
10. **Prefer concise over verbose.** The team rejects suggestions that replace a working one-liner with a multi-line refactor for style reasons alone.

---

## PR Review Focus Areas

### Go

**Real issues to catch**

- `sync.WaitGroup` has no `.Go()` method is a ligit error only if go version below 1.25
- Don't mutate cache objects — always `DeepCopy()`
- Set `OwnerReferences` for garbage collection
- Use proper requeueing (exponential backoff, not immediate requeue on error). This is [enabled by default](https://github.com/kubernetes-sigs/controller-runtime/blob/6210f847b2c1df3f28e5be34a4b1458f03896c73/pkg/controller/controller.go#L252-L254).
- Use typed clients, predicates, finalizers correctly
- Go naming conventions: `tenantID` not `tenantId`, `userID` not `userId`
- `nil` pointer dereference risk on optional config fields — guard before dereference

**Consistency with existing codebase:**

- Use `res.Logger` and `res.Fail(...)` for error handling in HTTP handlers — not `logrus` + `http.Error`
- Avoid naming packages `utils` — use domain-specific names (`labels`, `k8s`, `handlers`, etc.)

### React / TypeScript

**General:**
- No `any` without justification (TypeScript strict mode)
- Memoize expensive computations (`useMemo`, `useCallback`)
- Handle loading and error states
- Clean up effects (return cleanup from `useEffect`)

### Kubernetes Manifests

- Include resource requests/limits; don't use `latest` tags
- Run as non-root, read-only filesystem; least-privilege RBAC
- `EnvVar.value` must be a **string** — `value: true` (boolean) will fail schema validation
- Placeholder substitutions in YAML templates (e.g., `{clusterName}`) must be quoted/escaped to stay YAML-safe
- Use `.Release.Namespace`, not hardcoded namespace values

### Helm Charts

**Version/Metadata:**
- Keep `Chart.lock` in sync (`helm dependency update`) after changing `Chart.yaml` dependencies

**Templates:**
- Dynamic key access for names with hyphens: use `index .Values "kof-mothership"` or `["kof-mothership"]` — dot notation breaks on hyphens in `yq` and Helm
- Handle nil/missing values: `dig "annotations" "key" "default"` instead of `index .Cluster.metadata.annotations "key"` — the map may not exist
- `now | unixEpoch` in templates makes renders **non-deterministic** — every upgrade triggers a diff; avoid unless intentional
- Add an `else` branch with `fail` for `if/else if` provider selectors — an unsupported value should error, not silently render a broken resource
- Sveltos template expressions mixed with Helm `{{ }}` should be explicit about which layer evaluates each expression by using the unified opening/closing tags:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k0rdent/kof](https://github.com/k0rdent/kof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
