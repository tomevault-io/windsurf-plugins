---
trigger: always_on
description: - Telemetry attributes: follow rules in https://github.com/getlantern/semconv/blob/main/AGENTS.md
---

- Telemetry attributes: follow rules in https://github.com/getlantern/semconv/blob/main/AGENTS.md

## Code Comments

**Language doc conventions take precedence.** When writing a doc comment that the language's tooling formats or renders (Go's `// Foo ...`, Python docstrings, JSDoc, rustdoc, etc.), follow that convention even if it conflicts with the "lead with the *why*" guidance — for Go that means start with the identifier's name. The *why* still belongs in the comment, just in the body after the conventional opening.

- **Default: no comment.** Only comment when necessary to explain a non-obvious contract, invariant, rationale, or surprising behavior.
- Comments must answer *why* something is done a particular way, not *what* is being done (which should be clear from the code and naming).
- Before adding a comment, ask:
  - Is this information not obvious from the code or naming?
  - Does it document a constraint, invariant, concurrency guarantee, or error condition that would surprise a reader?
  - Is it essential for future maintainers to understand the reasoning or risk behind this code?
- **Do not add comments that:**
  - Restate the identifier name (in-line only).
  - Narrate the next line of code.
  - Reference tickets, coworkers, or code locations (these belong in commit messages).
  - Describe the mechanism instead of the contract.
  - Are aesthetic or redundant ("well-documented").
- Prefer documenting contracts at the declaration site. Use inline comments only for truly non-obvious lines.
- Remove or update obsolete comments promptly.
- **TODOs:** Must state both what needs to be done and why it isn’t done now. Remove or resolve unclear TODOs.

**Examples:**

```go
// BAD: Restates what the code does
// Cancel any in-flight requests.
cancelRequests()

// GOOD: Explains why this is necessary
// Must cancel in-flight requests to avoid leaking goroutines on shutdown.
cancelRequests()
```

---

```go
// BAD — restates name, generic preamble, narrates the code
// updateSelectionHistoryListener manages the lifecycle of the selection history listener
// across VPN status changes. Connected always re-attaches (canceling any
// existing listener) so a stale event still leaves the listener bound to
// the live storage.

// GOOD — leads with the trap, no narration
// Status events are dispatched in unordered goroutines, so reacting to
// intermediate statuses risks a stale handler tearing down a listener
// a concurrent Connected handler just attached. Only Connected (which
// re-attaches unconditionally) and terminal-down statuses are acted on.
```

```go
// BAD — narrates the next line
// Cancel any in-flight offline tests and wait for them to finish.
c.offlineTestCancel()
<-done

// GOOD — no comment; the names already say it
c.offlineTestCancel()
<-done
```

```go
// BAD — references ticket and coworker
// Per Freshdesk #172640 (reported by Alice), saveServers held the lock
// for 1+ minute. We now release access before disk I/O.

// GOOD — states the invariant; the ticket lives in git history
// access is released before disk I/O so a slow write can't starve readers.
```

```go
// BAD — doc block enumerates every branch; only one branch has hidden why,
// the rest restate cases the code already shows
// mapStatusEvent maps a radiance VPN status event to the wire value sent
// to Dart. Three cases deviate from a direct pass-through:
//   - vpn.Restarting collapses into vpn.Connecting so the UI shows a
//     transitional state during a tunnel restart.
//   - A non-empty evt.Error always maps to vpn.ErrorStatus.
//   - An unrecognized status falls back to Disconnected.
func mapStatusEvent(evt vpn.StatusUpdateEvent) (vpn.VPNStatus, string) { ... }

// GOOD — no doc block; inline comment on the only branch with hidden context
func mapStatusEvent(evt vpn.StatusUpdateEvent) (vpn.VPNStatus, string) {
	if evt.Error != "" {
		return vpn.ErrorStatus, evt.Error
	}
	switch evt.Status {
	case vpn.Connected, vpn.Connecting, vpn.Disconnecting, vpn.Disconnected, vpn.ErrorStatus:
		return evt.Status, ""
	case vpn.Restarting:
		// Map to Connecting; Dart's parser falls back to Disconnected otherwise.
		return vpn.Connecting, ""
	default:
		return vpn.Disconnected, ""
	}
}
```

Before writing an inline comment, consider whether a doc comment on the enclosing function or type would make it unnecessary. Prefer documenting contracts at the declaration over explaining implementation details inline.

Conversely, before writing a multi-bullet doc block that enumerates branches or cases, check each bullet against the line that implements it. If only one bullet carries hidden *why* and the rest restate visible branches, drop the doc block and put a single inline comment on the surprising branch. Doc blocks belong on contracts that surprise as a whole, not on functions where one corner of the implementation is non-obvious. The bar is higher for unexported helpers: the Go doc convention targets exported API, and unexported functions should default to no comment unless the contract genuinely surprises.

TODO comments must state *what* needs to happen and *why* it isn't done now. `TODO: ???` is not actionable — either resolve it or remove it.

## Go Doc Comments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getlantern/radiance](https://github.com/getlantern/radiance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
