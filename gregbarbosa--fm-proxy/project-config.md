---
trigger: always_on
description: The **`fm-proxy.js`** path is the primary, supported way to do tool calling + PCC with
---

## Project direction

The **`fm-proxy.js`** path is the primary, supported way to do tool calling + PCC with
Pi/Opencode (see runbook below). It sits in front of Apple's entitled `fm serve` and
speaks the OpenAI Chat Completions dialect. (An earlier in-process Swift `fms` app was
explored but removed: it could not do PCC because of the Apple-private entitlement
`com.apple.modelmanager.inference`, grantable only to Apple-signed binaries. As these
are early betas, Apple may lift the entitlement gate — worth rechecking periodically.)

## `fm` CLI reference

The full `fm` command tree (every subcommand, option, default, and discussion) is
generated from Apple's binary and committed for offline/agent use. Pull from these
instead of re-deriving help text:

| Resource | Path | Notes |
|---|---|---|
| Generator | `tools/gen-fm-docs.py` | Runs `fm --experimental-dump-help` (one call, no recursive `--help` scraping) and emits the markdown reference. Re-run after any `fm` update: `python3 tools/gen-fm-docs.py`. |
| Markdown reference | `docs/fm-reference.md` | Per-command option tables; best for grepping / LLM context. |

Source of truth is the installed binary (`/usr/bin/fm`): the docs reflect whatever
version is on disk, so regenerate rather than hand-editing.

## Tool calling with Pi (PCC + rich schemas) — runbook

An in-process app cannot do PCC inference (it's gated on the Apple-private entitlement
`com.apple.modelmanager.inference`, which fails with `ModelManagerError 1046` for any
binary not signed by Apple). To get tool calling + PCC's 32k context working
with Pi/Opencode, proxy through Apple's own entitled `fm serve`:

```
Pi  ──▶  fm-proxy.js (:1977)  ──▶  Apple `fm serve` (:1976)
         flattens tool schemas      entitled engine: system + pcc, runs tools
```

Apple's `fm serve` has very limited JSON-Schema support for tool parameters (flat only:
no nested `type:"object"`, no array-of-objects, no `anyOf/allOf/oneOf/$ref/$defs/
patternProperties`; root `required` must be present). `fm-proxy.js` rewrites incoming tool
schemas so Pi's rich definitions are accepted.

**Nested params use a lossless JSON-string round-trip.** A param fm can't represent
(nested object, or array whose items are objects — e.g. Pi's `edit` tool with
`edits: [{oldText, newText}]`) is declared to fm as a `type:"string"` whose description
says "… JSON string matching: {schema}". The model returns JSON in that string, and the
proxy re-parses it back into the real object/array in the tool_call's `arguments` before
forwarding to Pi — so the client sees the exact nested shape its tool validates against.
Flat tools (e.g. `write`) pass through untouched.

The embedded schema and every property are stripped of decorative keys fm ignores
(`description` on nested fields, `title`, `examples`, `default`, `$id`, …) before
serialization — pure token savings (~31% on a sample nested tool, 177→122 tokens via
`fm token-count`) with no loss of shape. See `EMBED_STRIP_KEYS` / `STRIP_KEYS` in
`fm-proxy.js`.

### Start it (from a Terminal signed into Apple Intelligence — PCC needs the attribution)

**One command (recommended):** `fm-launch.sh` starts the proxy (backgrounded), then runs
`fm serve` in the **foreground** (it blocks the terminal). It prints `stack up …` once
fm serve is healthy:

```bash
./fm-launch.sh            # quiet: startup + proxy errors/warnings only
./fm-launch.sh --verbose  # also shows the proxy's per-request [assembled] telemetry
```

> **fm serve must run in the foreground.** macOS only grants PCC attribution to a
> **foreground, TTY-attached** `fm serve`. Backgrounding it — the old node launcher
> (`zsh → node → fm`), or a shell `&` — makes every `pcc` request fail with
> `ModelManagerError 1013` / `"not available in this context"` (HTTP 503) while `system`
> keeps working. A bash intermediary is fine; **foreground vs backgrounded is the
> decisive line** (probed exhaustively — see memory `launcher-breaks-pcc-attribution`).
> So the launcher foregrounds `fm serve` and backgrounds the proxy (which only forwards,
> no PCC needed). `fm available` is a one-shot that keeps PCC in every context, so it's a
> poor predictor of `fm serve`'s attribution — don't use it to validate a launcher.

Use **Ctrl-C to stop** — the trap on INT/TERM/HUP/EXIT reaps the proxy. Do **not**
Ctrl-Z: a suspended foreground `fm serve` isn't reaped and will strand the port
(`kill -9` the `fm serve` PID to recover). fm serve's own output is untagged (piping it
is untested for attribution safety); only the proxy's output is tagged.
Errors/retries/overflows are **always** shown even without `--verbose`; only the routine
per-request telemetry is hidden. Ports/binary are overridable: `--fm-port`,
`--proxy-port`, `--fm-bin`, `--health-timeout` (or the `FM_PORT`/`PROXY_PORT` env vars).

**Manual (two tabs)**, if you want the processes separated:

```bash
# tab 1 — Apple's entitled engine (does the inference, incl. PCC):
/usr/bin/fm serve --port 1976

# tab 2 — the schema-flattening proxy (where Pi points):
node fm-proxy.js          # listens on :1977, forwards to :1976
```

Point Pi's " FM" provider `baseUrl` at `http://127.0.0.1:1977/v1`. Both `system` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gregbarbosa/fm-proxy](https://github.com/gregbarbosa/fm-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
