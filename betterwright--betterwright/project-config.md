---
trigger: always_on
description: For automatic hardware detection, model/quant selection, and runtime installation,
---

# The built-in agent harness (`betterwright exec`)

For automatic hardware detection, model/quant selection, and runtime installation,
run `betterwright --local`. See [one-command local AI](local-ai.md). Once setup
passes its image/tool-call check, the harness uses `local` by default unless you
explicitly select another model or endpoint.

This page covers the **standalone** shape: BetterWright supplies a
browser-tuned agent loop, you plug a *model* into it, and you hand it a
natural-language task. For how it compares to the integrated shape, see
[Pick your shape first](getting-started.md#pick-your-shape-first).

The two nest: a coding agent can shell out `betterwright exec "<task>"` as a
browser **sub-agent** — one command in, one JSON answer out, with the entire
browsing transcript (snapshots, retries, verification) kept out of the caller's
context.

This shape exists because the browser runtime is rarely the slow part.
The end-to-end gap is usually the *agent scaffold* — a browser-specialized
loop takes fewer, tighter steps than a general coding agent.
`betterwright exec` gives BetterWright that scaffold.

## CLI

```bash
betterwright exec "find the top Hacker News story and give me its title and points" --model gpt-5.6-sol
```

Shells expand dollar signs inside double quotes, so use single quotes for tasks
that contain prices or other literal `$` text:

```bash
betterwright exec 'find options under $4000' --model gpt-5.6-sol
printf '%s\n' 'find options under $4000' | betterwright exec --stdin --model gpt-5.6-sol
```

The `--stdin` form is also useful for generated or multiline tasks because the
task does not go through another round of shell parsing.

Progress notes stream to stderr as the loop runs — the last one summarizes the
run's cost (`done in 6 steps, 7 tool calls, 11.4s, 6,880 in / 1,330 out · 40,000
cache read · context 20,000`) — and the final result is one JSON object on
stdout:

```json
{
  "ok": true,
  "answer": "…",
  "steps": 6,
  "reason": "done",
  "toolCalls": 7,
  "usage": {
    "inputTokens": 6880,
    "outputTokens": 1330,
    "cacheReadTokens": 40000,
    "cacheWriteTokens": 0,
    "context": 20000
  },
  "durationMs": 11400,
  "timing": { "modelMs": 9100, "toolMs": 1900 },
  "proof": "/…/proof-….png",
  "recordings": []
}
```

`toolCalls` counts all calls the model issued: `browser`, `login`, `ask`,
`live_view`, `handoff`, and `done` (it can exceed `steps` when a turn batches
several). `usage` sums the token counts the model adapter reported across turns
(a field is `0` when the provider returned no
usage block); `inputTokens` is fresh input only: each turn's provider input total
minus the portion served from cache.
`cacheReadTokens` and `cacheWriteTokens` come straight from the provider's usage
block — the Responses API's `input_tokens_details.cached_tokens` /
`cache_write_tokens`, the Chat Completions `prompt_tokens_details` equivalents, or
Anthropic's `cache_read_input_tokens` / `cache_creation_input_tokens`. The CLI
always shows cache reads; it shows cache writes only when the run has a positive,
provider-reported count. It never derives writes from fresh input. `context` is
the full prompt size at the **end** of the task — the last turn's provider input
total, i.e. how much context the model was holding when it finished. `durationMs`
is the task wall-clock (it excludes tearing down a browser the loop created for
itself), and `timing` splits it into time spent waiting on model turns
(`modelMs`) and inside browser calls (`toolMs`); the CLI prints the same split
after the total. The remainder is loop overhead and human waits. `recordings`
lists saved page-recording paths from this task, in the order they finished.
The loop has no fixed step cap, but it does have a 30-minute wall-clock
budget and a 1,000,000-character transcript bound so a stalled or repetitive
provider cannot run forever or grow context without limit. Expiry aborts model
requests, and BetterWright's worker timeout terminates in-flight browser work.

A third bound catches the loop that is running but not progressing: when a
browser step fails **the same way three times in a row**, the observation carries
a warning telling the model to change approach; at five, the run ends with
`reason: "no_progress"` rather than spending the rest of the budget on a step
that cannot succeed. Any successful browser call — or new human steering through
the live view — clears the streak.

Tasks matching the `checkout-verification` skill also get a compact independent
completion check. Before accepting a final answer, the host reads bounded fresh
UI evidence, product context, and form values, then asks the same configured
model to check the answer without tools or the full browsing transcript. A
bounded earlier UI observation and its executed code provide context for
distinguishing a new result from an old receipt. Empty compact regions get a
small full accessibility snapshot automatically. When the supplied evidence
still omits the receipt, the checker can request a bounded full or scoped
snapshot, including iframe content. The host executes only these read-only
snapshot requests, never checker-authored browser code. Duplicate requests and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BetterWright/betterwright](https://github.com/BetterWright/betterwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
