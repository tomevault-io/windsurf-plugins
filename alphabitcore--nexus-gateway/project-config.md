---
trigger: always_on
description: Canonical = OpenAI shape; each non-OpenAI adapter owns its bidirectional translation (8 binding rules from provider-adapter-architecture.md §3a)
---


# Provider adapter: canonical = OpenAI shape (binding)

You are editing a provider adapter or its codec. The 8 binding rules below come from `docs/developers/architecture/services/ai-gateway/provider-adapter-architecture.md` §3a. **Apply to every adapter PR.**

**Read `docs/developers/architecture/services/ai-gateway/provider-adapter-architecture.md` §3a BEFORE editing.**

## Rule 1 — Canonical format is OpenAI chat-completions shape

All internal flow (router input, cache key, hook input, audit envelope, request lineage) sees the **canonical form**, which is OpenAI's shape:

```
model · messages[] · max_tokens / max_completion_tokens · temperature · top_p · top_k ·
stream · stop · response_format · tools[] · tool_choice · parallel_tool_calls ·
metadata · stream_options
```

**New canonical fields require an architecture-doc PR** — adapters do not add canonical fields unilaterally.

## Rule 2 — Each non-OpenAI adapter owns its full bidirectional translation

When you add an Anthropic / Gemini / Bedrock / Cohere / Replicate codec:

- `SchemaCodec.EncodeRequest` does **canonical → wire**.
- `SchemaCodec.DecodeResponse` does **wire → canonical**.

The **OpenAI side stays pure** (identity codec) — it never carries case-statements for "this came from Anthropic so do X". OpenAI shape is the bus; every other shape adapter wires itself into the bus.

## Rule 3 — Per-model wire quirks belong in their own adapter

HTTP-400-deprecations, parameter renames, mandatory clamping — they live in the adapter that talks to that wire. **Not** in cross-adapter case-statements in `spec_adapter.go`'s shared helpers.

| Quirk | Lives in |
|---|---|
| claude-opus-4-7 deprecates temperature/top_p/top_k | `specs/anthropic/codec/codec.go::anthropicModelRejectsSamplingParams` |
| claude-4.x rejects temperature + top_p together | `specs/anthropic/codec/codec.go::anthropicModelRejectsTempTopPTogether` |
| gpt-5.x / o-series rename max_tokens → max_completion_tokens + strip temp/top_p | `specs/openai/rewrites` (`ApplyReasoningRewrites`, wired as the OpenAI `PassthroughRewrite`) |
| kimi-k2.5/k2.6 require temperature=1 | `specs/compat/moonshot/rewrites.go` (`ApplyRewrites`, wired as the Moonshot `PassthroughRewrite`) |

When a new family ships an HTTP-400-deprecation, find the adapter that owns its wire and add the prefix-rule there. Cross-adapter shared helpers create the wrong dependency direction.

## Rule 4 — `nexus.ext.<provider>.<key>` is the canonical extension namespace

Fields with no clean OpenAI mapping (Anthropic's `thinking`, Gemini's `thinkingConfig`, Anthropic's `cache_creation_input_tokens`, Bedrock's `anthropic_version`) ride along inside `nexus.ext.<provider>.<key>` on the canonical body.

Package: `providers/canonicalext/`. Use:

- `canonicalext.Get`
- `canonicalext.Set`
- `canonicalext.ScanUnsupported`
- `canonicalext.WarnOnce` — adapters that observe an unsupported canonical field emit a one-shot WARN so operators see drift between the canonical surface and the codec.

## Rule 5 — `SchemaCodec.EncodeRequest` contract: input is canonical, output is target wire

Callers that have an **ingress-format body** (Anthropic `/v1/messages`, Gemini `:generateContent`) MUST canonicalize first via:

```go
canonical, err := canonicalbridge.IngressChatToCanonical(ingress, body, target)
```

…before invoking `adapter.PrepareBody` / `SchemaCodec.EncodeRequest`. Skipping canonicalization makes the OpenAI identity codec forward the ingress body verbatim, and the upstream returns 400 (or worse, parses partially and produces gibberish).

`EncodeRequest` accepts canonical-or-codec-empty (passthrough); it does NOT accept "any old shape and we'll figure it out".

## Rule 6 — Both streaming and non-streaming are in scope

A codec rule that strips `temperature` from a non-streaming request **must also strip it from the streaming variant** — the upstream rejects both. The streaming session's pre-dispatch body construction goes through the same `PrepareBody` path, so this typically falls out for free.

The gap usually appears on **error-frame construction** (response side) when the gateway hand-builds an SSE error and forgets the ingress format. Rule 8 covers this.

## Rule 7 — Add empirical evidence to every prefix-list

Every "model X rejects param Y" rule MUST be backed by an **observed 400** (logged trace_id or direct test call). Speculative rules cause silent flattening of caller intent — strip a param that the model actually accepts, and you've degraded behaviour without surfacing why.

The comment above each prefix-list switch documents the observation (date + error message). Canonical example: `anthropicModelRejectsSamplingParams` in `specs/anthropic/codec/codec.go` carries the observed-400 evidence in its comment block.

```go
// Date: 2026-05-09. Observed via trace_id=abc... on claude-opus-4-7:
//   { "type":"invalid_request_error", "message":"temperature is not allowed for this model" }
// Verified again 2026-05-12 via direct curl. Confirmed: deprecation, not a transient.
var anthropicModelRejectsSamplingParams = []string{
    "claude-opus-4-7",
    ...
}
```

Without evidence:
- Reviewers can't tell if the rule is correct or speculative.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlphaBitCore/nexus-gateway](https://github.com/AlphaBitCore/nexus-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
