---
trigger: always_on
description: Use when troubleshooting, implementing, reviewing, or auditing platform infrastructure as a system — where Kubernetes, GitOps, CI/CD, and security concerns intersect. Provides structured diagnosis with blast radius, validation steps, and rollback plan for: Kubernetes, Flux CD, Argo CD, Terraform, GitHub Actions (composite actions, OIDC, SHA pinning), AWS, Azure, GKE, Linkerd, KEDA, Karpenter, supply chain security (Cosign, SBOM, SLSA), Falco, Chaos Engineering, DORA metrics, Datadog/Dynatrace/LL
---


# Platform Skills

Use this skill for hands-on help with Kubernetes, GitOps, cloud infrastructure, CI/CD, secrets management, service mesh, Linux administration, networking, and platform product thinking — whether you are a solo developer or part of a large platform team.

## Pick the right tool for the job

| Layer | When to use |
|---|---|
| `Terraform` | Cloud primitives, cluster bootstrap, IAM, networking, secrets backends |
| `Kubernetes` | Workload, RBAC, network policy, platform baseline across distributions |
| `OpenShift` | Kubernetes patterns adapted to OpenShift routing, SCC, and OLM |
| `Flux` / `Argo CD` | In-cluster reconciliation, Helm releases, workload promotion |
| `GitHub Actions` | Validate, package, gate, and promote. Keep workflows declarative. |
| `AWS` / `Azure` / `GKE` | Provider-specific account, identity, and governance patterns |
| `Linkerd` | Automatic mTLS, golden-signal observability, traffic management |
| `Linux & Networking` | DNS, load balancer routing, VPC/VNet, kernel tuning, connectivity |
| `Compliance` | SOC 2 controls in Terraform — IAM, encryption, audit logging, Checkov |
| `Helm (Helmcheck)` | Chart scaffolding, lint/validate pipeline, values design, security hardening |
| `MCP` | Build/debug MCP servers — tools, resources, transports, auth |
| `AWS MCP Profiles` | Discover/switch AWS profiles across VS Code + Claude Code MCP configs — multi-account, SSO, Granted, credential_process |
| `Observability` | Prometheus, OpenTelemetry, Grafana, alerting, k6 load tests, capacity |
| `Documentation` | Docstrings (Google/NumPy/JSDoc), OpenAPI 3.1, MkDocs, guides |
| `Datadog` | Agent on Kubernetes, APM, monitors, dashboards, SLOs, LLMObs |
| `Dynatrace` | OneAgent Operator, auto-instrumentation, anomaly detection, SLOs |
| `Conventional Commits` | Generate WHY-driven commit messages, atomic staging, validate |
| `OPA / Conftest` | Rego policies, unit tests, fmt/regal/verify pipeline, debug |
| `Kyverno` | CEL-based ValidatingPolicy, MutatingPolicy, ImageValidatingPolicy |
| `PR Review` | Cost, drift, ownership, SOC 2, deprecated APIs, rollback feasibility |
| `PR Triage` | Classify comments ACTIONABLE_FIX/INFORMATIONAL/NOT_APPLICABLE, fix, reply |
| `KEDA` | ScaledObject/ScaledJob, all scalers, TriggerAuthentication, scale-to-zero |
| `Karpenter` | NodePool/EC2NodeClass design, Spot diversity, disruption strategy, capacity planning, audit, CA migration, v0→v1 upgrade |
| `Agent Self-Improvement` | `.learnings/` workspace, LRN/ERR lifecycle, WAL, VFM, ADL |
| `Supply Chain Security` | Cosign signing, Syft SBOM, Trivy/Grype CVE gates, SLSA Level 2 |
| `Runtime Security` | Falco eBPF, custom rules, Falcosidekick routing, Kyverno enforcement |
| `Awesome Docs` | Animated SVG Markdown — README, runbook, RFC, architecture, post-mortem |
| `Composite Actions` | Full action repo scaffold, SHA pinning, secrets-as-inputs, actionlint |
| `GitOps debug` | 5-workflow structured debug → 5-section report with root cause |
| `GitOps audit` | 6-phase repo audit → prioritized Critical/Warning/Info report |
| `Platform Mindset` | DevEx, friction audits, RFC/ADR, incident communication, post-mortems |
| `Renovate` | Dependency update automation — generate renovate.json from repo scan, emit GHA validation workflow |
| `Setup Agents` | Scaffold multi-agent AI configs for any repo — interview-driven, specific to this codebase |

If a task spans multiple areas, decide which layer owns the source of truth and keep the other layers consumers of that state.

## Apply These Platform Rules

- Separate reusable platform building blocks from live environment configuration.
- Prefer GitOps pull-based reconciliation for cluster state and CI push-based automation for validation and packaging.
- Choose either Flux or Argo CD for a given ownership boundary unless the task is explicitly about migration between them.
- Keep Terraform responsible for bootstrapping clusters, cloud resources, secrets backends, and access primitives. Do not let Flux or Argo CD recreate those foundations unless there is a deliberate controller-based design.
- Use Flux or Argo CD for in-cluster add-ons, workloads, Helm releases, and app-level environment promotion after bootstrap.
- Use GitHub Actions for checks, plans, policy gates, artifact publishing, and promotion orchestration. Do not store long-lived environment truth in workflow YAML.
- Prefer OIDC or workload identity over static cloud credentials.
- Model environments explicitly. Promotion should be visible in Git history and reversible by commit rollback.
- For Linux and networking changes, validate at each layer before escalating: confirm the process is listening (`ss -tulnp`), then L3 reachability (`ping`), L4 connectivity (`nc -zv`), L7 response (`curl -v`), and security group / NACL rules last. Do not skip layers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nitinjain999/platform-skills](https://github.com/nitinjain999/platform-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
