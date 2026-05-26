---
trigger: always_on
description: Cross-platform Kubernetes desktop client. Multi-context cluster management with live resource updates, log streaming, exec, port-forwarding, full RBAC, **Custom Resource Definitions**, **Helm**, **Argo CD** and **Gateway API** support. **Nothing is installed in the cluster** — Klustr is a pure client that drives the standard Kubernetes API using the user's `~/.kube/config`.
---

# Klustr

Cross-platform Kubernetes desktop client. Multi-context cluster management with live resource updates, log streaming, exec, port-forwarding, full RBAC, **Custom Resource Definitions**, **Helm**, **Argo CD** and **Gateway API** support. **Nothing is installed in the cluster** — Klustr is a pure client that drives the standard Kubernetes API using the user's `~/.kube/config`.

## Tech Stack

| Layer | Choice |
|---|---|
| Desktop framework | Wails v2 (Go backend + native webview) |
| Backend | Go 1.26 + `client-go` (typed + dynamic + discovery) |
| Custom resources | `client-go/dynamic` + the apiextensions CRD list, watched live |
| Gateway API | `sigs.k8s.io/gateway-api` typed informer factory (not the dynamic client) |
| Helm | upstream `helm.sh/helm/v3` library, no shelling out |
| Frontend | React 19 + TypeScript + Vite |
| UI | Tailwind CSS + shadcn/ui + lucide-react + sonner (toasts) |
| Real-time state | Zustand |
| Mutations | TanStack Query (mutations only — no query cache) |
| Tables | TanStack Table |
| Terminal (logs + exec) | xterm.js |
| Code editor (YAML) | Monaco |
| Toolchain | mise (pins Go, Node, Wails CLI versions) |
| Lint / format | golangci-lint (Go) + ESLint (frontend) |
| Tests | `go test` (backend) + Vitest + jsdom (frontend) |
| CI release builds | GitHub Actions matrix on hosted runners (macOS, Windows, Linux) |
| Release publishing | `softprops/action-gh-release` (macOS .tar.gz + Linux .tar.gz + .deb assets today), Homebrew cask auto-bump for macOS + AUR `klustr-bin` auto-bump for Arch |

## Project Structure

```
klustr/
├── .mise.toml                    tool versions (Go, Node, Wails CLI)
├── .golangci.yml                 golangci-lint v2 config
├── wails.json                    Wails project config
├── main.go                       application entry point
├── internal/                     pure Go business logic (no Wails imports)
│   └── kube/
│       ├── config.go                kubeconfig parsing, context discovery
│       ├── path.go                  GUI-launch PATH augmentation for exec credential helpers
│       ├── manager.go               ClientManager lifecycle (Clientset / Ping / Watch /
│       │                            StopWatch) + Logs / Exec / PortForward / CRD forwarders
│       │                            + watcher() helper
│       ├── manager_<group>.go       per-sidebar-group Wails-facing forwarders
│       │                            (workloads / networking / config / storage / cluster /
│       │                             autoscaling / admission / rbac / helm / gateway / pods)
│       ├── mutate.go                generic apply / delete / scale via dynamic client +
│       │                            kindToGVR map
│       ├── informers.go             contextWatcher lifecycle + the single start() that wires
│       │                            every kind's event handler + shared helpers
│       │                            (sortByNamespaceName, formatLabelSelector, OwnerRef, …)
│       ├── informers_<group>.go     per-sidebar-group XxxInfo types and lister methods
│       ├── details.go               shared types (ContainerSummary) + helpers
│       │                            (matchLabels, deploymentConditions, quantitiesToStrings,
│       │                             policyRules, rbacSubjects, …)
│       ├── details_<group>.go       per-sidebar-group XxxDetail structs and Get() builders
│       ├── crd.go                   apiextensions CRD discovery + per-CR dynamic informers
│       ├── helm.go                  helm v3 list / install / upgrade / rollback / uninstall +
│       │                            repo / chart search
│       ├── helm_cache.go            chart cache helpers for repo browsing
│       ├── argocd.go                Application list + Sync / Refresh through the K8s API
│       │                            (no argocd CLI, no argocd-server dependency)
│       ├── gateway.go               typed Gateway API informers + status / route helpers
│       ├── rollout.go               Deployment / StatefulSet / DaemonSet rollout history
│       │                            and one-click revert (kubectl rollout undo path)
│       ├── install.go               one-click metrics-server install / uninstall from
│       │                            upstream components.yaml
│       ├── events.go                core/v1 Events list filtered by involvedObject
│       ├── metrics.go               metrics.k8s.io pod CPU/memory usage (polled, not watched)
│       ├── overview.go              cluster-wide CPU / memory / pod aggregation
│       ├── logs.go                  streaming log sessions
│       ├── exec.go                  SPDY exec sessions
│       └── portforward.go           port-forward registry & lifecycle
├── app/                          Wails binding adapter (thin layer over ClientManager)
├── frontend/
│   ├── eslint.config.js          ESLint flat config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SametKUM/klustr](https://github.com/SametKUM/klustr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
