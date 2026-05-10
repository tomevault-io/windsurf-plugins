---
trigger: always_on
description: Self-hosted Railway-style deploy platform for Kubernetes. One operator, one
---

# Mortise

## What this is

Self-hosted Railway-style deploy platform for Kubernetes. One operator, one
Helm chart. Users connect a git repo or pick an image → Mortise handles
builds, deploys, domains, TLS, env vars, volumes, preview envs, and bindings.
Kubernetes is fully abstracted away from the user.

Read SPEC.md for the full product spec. Read ARCHITECTURE.md for system
diagrams. This file is the operating manual for working in this codebase.

## Releases

See `RELEASING.md` for the full convention. Short version: one semver
git tag (`v0.1.1`) triggers the `release.yml` workflow, which builds a
multi-arch image (`ghcr.io/mortise-org/mortise:0.1.1`), stamps and
packages both charts, publishes to `gh-pages`, and creates a GitHub
Release. Chart version, `appVersion`, and image tag are always the same
number. Never edit `Chart.yaml` `version:` or `appVersion:` by hand.
CI owns them.

## Tech stack

- **Operator + API:** Go, kubebuilder, controller-runtime
- **UI:** SvelteKit + TypeScript (embedded in operator binary via embed.FS)
- **CLI:** Go (cobra)
- **Helm charts:** charts/mortise/ (batteries-included umbrella), charts/mortise-core/ (operator only)
- **CRDs:** Project, App, PlatformConfig, GitProvider, PreviewEnvironment, ProjectMember
- **Bundled tools (in umbrella chart):** BuildKit (image builds), OCI registry, Traefik
  (ingress), cert-manager (TLS)

## Architecture rules

These are non-negotiable. Violating any of them is a bug.

### Standards, not implementations

Mortise couples to standards, not specific tools:
- k8s Ingress API: not Traefik-specific annotations
- OCI Distribution Spec: not implementation-specific APIs
- OIDC: not Authentik-specific APIs
- ACME (via cert-manager): not Let's Encrypt-specific

If you're about to write code that only works with one specific tool (Traefik,
GitHub), it must be behind an interface in internal/<name>/.

### Controllers never import third-party SDKs

All external calls go through interfaces defined in internal/<name>/.
Controllers import only Mortise's own types. Never import go-github,
moby/buildkit/client, or any other third-party SDK in a controller file.

### Interfaces (internal seams, not plug-in APIs)

These exist for testability and version-bump isolation. They are NOT extension
points for outside implementers.

```
controller  →  AuthProvider     →  native DB | generic OIDC
controller  →  PolicyEngine     →  native (admin/member)
controller  →  GitAPI           →  GitHub | GitLab | Gitea
controller  →  GitClient        →  go-git (single impl)
controller  →  BuildClient      →  BuildKit (single impl)
controller  →  RegistryBackend  →  generic OCI (config-driven)
controller  →  IngressProvider  →  generic annotation-driven
```

DNS is annotation-driven via ExternalDNS (no Go interface: see SPEC §11.1).

No interface without a real v1 implementation behind it.

### Everything is an App, grouped under Projects

One CRD for workloads, one concept. Source type (git | image) determines
how it deploys. No separate ServiceInstance, DatabaseInstance, or similar
CRDs. Backing services (Postgres, Redis) are Apps with
`network.public: false` and a `credentials` block. Other Apps bind to them.

**Apps live inside Projects, not free-floating.** Every App belongs to
exactly one Project; its *control* namespace is `pj-{project-name}` and
its *workload* namespaces are `pj-{project-name}-{env-name}` (one per
environment). Apps, PreviewEnvironments, and other project-owned CRDs
live in the control namespace; Deployments, Services, Ingresses, Pods,
PVCs, env-scoped Secrets and ConfigMaps live in the per-env namespace.
Users, URLs, and CLI commands scope to a current project context. When
in doubt about where something goes: does it exist PER PROJECT (apps,
secrets, domains, preview envs) or PER PLATFORM (users, git providers,
DNS config, platform domain)? Per-project → lives under the control
namespace. Per-platform → lives in `mortise-system` or a cluster-scoped
CRD.

### Mortise owns only what it creates

Never touch resources Mortise didn't create. Coexist with Argo CD, Flux,
manually-deployed resources, other operators. Check for ownership labels
before modifying or deleting any resource.

### No addon/plug-in architecture

Mortise is one operator, one Helm chart. No subcharts, no plug-in SDK, no
extension registry. Third-party integration happens through Kubernetes
primitives (ESO writes k8s Secrets, OPA gates admission, Prometheus scrapes
standard metrics). If you're tempted to add a plug-in system, stop.

## Behavioral guidelines

### Think before coding

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them: don't pick silently.
- If a simpler approach exists, say so.

### Simplicity first

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

### Surgical changes

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style.
- Every changed line should trace directly to the task.
- Remove imports/variables/functions YOUR changes made unused.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mortise-org/mortise](https://github.com/mortise-org/mortise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
