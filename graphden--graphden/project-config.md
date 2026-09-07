---
trigger: always_on
description: The single rule for WHERE a function's property lives — in the graph (default), a DB column (only when the graph can't serve it), and NEVER a name/prefix. Apply when adding/using any per-function property, classification, or metadata (visibility, kind, secret, anonymity, org, etc.), or when touching name-based dispatch/classification.
---


# Function metadata & identity — where a property lives

Graphden's premise is **code = graph**, and PHILOSOPHY §3 requires that
**behavior be visible in graph structure** (no magic, no context-dependent
semantics). That fixes where a function's properties must live. This rule is the
single conceptual key every metadata/classification decision follows.

## The rule (graph → field → never a name)

1. **Graph by default.** A function's SEMANTIC property is expressed in the
   graph itself — as **inheritance from a marker/base fn** (`parent-ids`),
   a **binding on a slot**, or the fn's **type**. It is then visible in
   structure, inherited, and needs no new DB column. The canonical precedent is
   **`secret`**: a fn is a secret because it inherits `:secret-leaf`
   (`parent-ids: [secret-leaf]`) — there is no `secret?` column. Classify by the
   graph relation (is `secret-leaf` in the parent closure, by **id**), never by
   name.
2. **A DB column only when the graph cannot serve it.** Add/keep a base-schema
   column ONLY for:
   - **Org-isolation enforced by RLS** — `org_id`. RLS operates on columns; the
     graph cannot enforce tenant isolation.
   - **Content-addressed identity / dedup the executor needs at O(1)** —
     `anonymous-hash` (are these two inline shapes identical?), `name = nil`
     (a local/unnamed fn). A parent-closure walk cannot answer these cheaply,
     and they are the fn's OWN identity, not domain metadata.
   - **VCS / lifecycle plumbing** — `branch-id`, `deleted-at`, `created-at`, the
     version-plane columns.
   A column that merely CACHES a structural fact (e.g. a type-kind) is allowed
   as a denormalized cache, but must be documented as derived-from-structure,
   never as the source of truth, and never a prefix.
3. **Never a name or a name-part.** Names are per-namespace LABELS
   (`docs/adr/ADR-identity-model.md`), not carriers of meaning. No code branches
   on a literal fn/package name or a name prefix (`_`, `_anon-`, …) to determine
   a property. Base-fn / seed *identity* may be resolved from its globally-unique
   name to its **id** ONCE at boot and then used by id (this is the ADR's
   name-keyed base-fn identity, e.g. resolving `secret-leaf` / `vault-get` → id);
   that is identity resolution, not per-fn classification.

## Verdict per current property (graph / column / fixed)

| Property | Where it lives | Rationale |
|---|---|---|
| `secret` | **Graph** — inherit `:secret-leaf` | Already graph; the model to copy. Classify by id. |
| package **visibility / public interface** | **Graph** — a marker/export construct (TBD shape), NOT a prefix or column | Semantic; enforceable + structure-visible. Supersedes the earlier "namespace structure (visual)" idea in the packages spec. |
| anonymity — **composite-TYPE anon** (inline record shape) | **Column** — `name = nil` + `anonymous-hash` set (UNIQUE) | Content-dedup identity, O(1), hot path. Classify by the field. |
| anonymity — **composed anon fn-def** (inline `{:parent …}` lift) | **Identity in the synthetic `_anon-<hash>` NAME**; `anonymous-hash = NULL` | The name IS the content-addressed use-site identity (the hash embeds shape+host+ns; two use-sites get distinct names — `records-test/anon-use-site-identity-includes-namespace`). ADR name→identity, not a domain property in a prefix. Recognize an anon by `(or anonymous-hash (starts-with? name "_anon-"))` — the name half is IDENTITY. |
| local/unnamed fn | **Column** — `name = nil` | Identity representation; not a prefix (`_`-authoring syntax lowers to name=nil). |
| `org_id` | **Column** (+ RLS) | Isolation; cannot be graph. |
| type-kind (`role`) | **Column as derived-cache** of a structural fact (type-row = no impl + slots/refine) | Kept for classification speed; document as derived, never a prefix. |
| service / app-route | **Separate rows keyed by fn-id** (kept) | Carry runtime desired-state / a routing table; not fn-intrinsic metadata (see the domain-router decision — a routing table above the graph). |

## Name/prefix-hardcode audit + remediation status (2026-08-13)

Audit of the whole tree for "property encoded in a name/prefix." The dangerous
class (dispatch/identity/cache/classify by name) is otherwise clean + guarded
(ADR-identity-model + `id_resolution_guard_test`).

**Correction (verified against the live DB, not theory).** An earlier pass tried
to replace EVERY `_anon-` name check with an `:anonymous-hash`-field check on the
premise that the field is the single anonymity marker. That premise is FALSE.
`select` over `fn` shows two anonymity classes: composite-TYPE anons (name NULL,
`anonymous_hash` set) AND composed anon fn-defs (`_anon-<hash>` name,
`anonymous_hash` NULL — the majority). For the composed class the field is null,
so a field-only check silently stops recognizing them. Those "fixes" were
therefore REGRESSIONS (sync leftover-scan would flood with synthetic rows; the
type-picker would leak `_anon-` candidates) and were **reverted**. See the two

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Graphden/graphden](https://github.com/Graphden/graphden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
