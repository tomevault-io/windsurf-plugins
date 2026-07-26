---
trigger: always_on
description: Design rules that apply to all kubectl-status output and templates. Read this before writing or reviewing a template.
---

# kubectl-status Conventions

Design rules that apply to all kubectl-status output and templates. Read this before writing or reviewing a template.

For the step-by-step template authoring workflow (reading CRD schemas, sampling live instances, verifying output) see [`CONTRIBUTING.md`](CONTRIBUTING.md#your-first-code-contribution).

## Output philosophy

- **Human-only output.** Don't make output parser-friendly — no stable column widths, no machine-parseable structure.
- **Compact over complete.** Compact output is the main differentiator from `kubectl describe`. Omit fields with well-known defaults (e.g. `podIP`, `hostIP`, `containerID`).
- **Readable without color.** Users share output via copy-paste, losing ANSI codes. Never rely on color alone to convey state — use text that is unambiguous in plain output. E.g. prefer `Not Ready` over coloring the word `Ready` red.
- **Transform, don't transcribe.** Raw Kubernetes field values are often not human-friendly. Prefer `Not Ready` over `Ready: false`.
- **Be opinionated.** Express status clearly. Spell out impact where it matters: a Service with no endpoints likely means an outage — say so.
- **Surface what isn't in the resource.** When status fields alone are insufficient, make additional API calls. E.g. fetch NodeMetrics and Pods when showing a Node.
- **Spec fields only when contextually necessary.** Include spec only when it sets context for understanding current status (e.g. `.spec.replicas` for a ReplicaSet). Omit pure configuration (e.g. Ingress host values).
- **Promote generic patterns.** If a convention appears across multiple resource types (e.g. `observedGeneration`, `conditions`, `replicas`), implement it in `DefaultResource` or `common.tmpl` so all resources benefit.

## Color coding

Traffic-light convention, but restrained:

| Color | Use for |
|---|---|
| regular | Healthy / nominal state |
| `green` | Explicit healthy signal from a dedicated status field — `Ready: True`, `Running`, `Active`. Do **not** use just because counts match. |
| `yellow` | Known-transient issues or bad practices — ongoing rollout, `latest` image tag. |
| `red` | Faulty states requiring attention. Use for long messages (condition `.message`). |
| `bold red` | Single words, camelCase, or PascalCase in a faulty state (condition `.reason`, a resource kind). |

For short key/value pairs in a faulty state, colorize the whole expression — not just the key or just the value. E.g. paint `readyPodCount:0` as one `red` unit.

## Template conventions

### Section order

Every template follows this fixed structure. Do not add content that duplicates what `status_summary_line` already shows (Kind, Name, Namespace, creation time, owner reference, phase).

```
{{- template "status_summary_line" . }}
{{- template "kstatus_summary" . }}
{{- template "finalizer_details_on_termination" . }}
{{- template "observed_generation_summary" . }}
{{- template "application_details" . }}
... resource-specific content ...
{{- template "conditions_summary" . }}
{{- template "recent_updates" . }}
{{- template "events" . }}
{{- template "owners" . }}
```

The bookend sections (`kstatus_summary`, `conditions_summary`, `recent_updates`, `events`, `owners`) stay in this order. Resource-specific body sections go where they make most sense contextually — typically immediately after the content they annotate. Omit a bookend section when it adds no signal for the resource type (e.g. `kstatus_summary` always reports "Resource is always ready" for CronJob — omit it).

### Prose over key:value

When multiple related fields form a natural sentence, write prose rather than stacking `Label: value` pairs:

```
  Issued by ClusterIssuer/cluster-issuer for "foo" · stored in secret/foo-tls
    Org: ServiceNow
    Also valid for: foo.svc, foo.svc.cluster.local
```

Reserve `**Bold label**: cyan value` for fields that genuinely stand alone and don't connect to adjacent fields.

### Value highlighting

Apply `| cyan` to plain values so they are visually distinct from bold labels. Never stack `cyan` on top of a semantic color function — `redBoldIf`, `redIf`, `colorKeyword`, and `colorAgo` must not be overridden.

`cyan` expects a string; convert integers first: `{{ .count | toString | cyan }}`.

### Zero-value fields

`{{- with .field }}` skips when the value is `false`, `0`, or `""`. This hides operationally meaningful zeroes — `routes=0` means nothing is attached and is worth showing. Use `if hasKey` when zero is significant:

```
{{- if hasKey $status "attachedRoutes" }}, routes={{ $status.attachedRoutes | toString | cyan }}{{ end }}
```

Use `with` only when the zero/empty case genuinely means "omit this field entirely".

### Single-item list collapsing

An indented block for a single item wastes vertical space. When rendering a labeled list, check the length: if there is exactly one item, collapse it onto the title line. The block form only pays off with multiple items to scan.

Exception: when items themselves have rich sub-fields (conditions, nested refs) that always need indented lines, collapsing the header does not help.

### Merging parallel spec/status lists


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bergerx/kubectl-status](https://github.com/bergerx/kubectl-status) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
