---
trigger: always_on
description: Coding standards for the Databasus verification agent — a Go CLI worker that runs on a cloud-managed Linux VM, reports its capacity to the backend over HTTP, and (once the restore phase lands) claims and verifies backups. It has no Gin HTTP server and owns no database schema. The long-running goroutine in this phase is the **capacity heartbeat loop**; the claim/report runner arrives with the restore phase. There is no Windows daemon — the agent runs in the foreground under systemd or as a contai
---

# Verification agent guidelines (Go CLI)

Coding standards for the Databasus verification agent — a Go CLI worker that runs on a cloud-managed Linux VM, reports its capacity to the backend over HTTP, and (once the restore phase lands) claims and verifies backups. It has no Gin HTTP server and owns no database schema. The long-running goroutine in this phase is the **capacity heartbeat loop**; the claim/report runner arrives with the restore phase. There is no Windows daemon — the agent runs in the foreground under systemd or as a container.

For project-wide engineering philosophy, naming, and lint/format commands, see the root `CLAUDE.md`. For the backend (Gin/GORM/Swagger) ruleset, see `backend/CLAUDE.md`.

---

## Table of Contents

- [Spacing between logical statements](#spacing-between-logical-statements)
- [Comments](#comments)
- [File organization](#file-organization)
- [Background services](#background-services)
- [Testing](#testing)
- [Time handling](#time-handling)
- [Logging](#logging)
- [Modern Go](#modern-go)

---

## Spacing between logical statements

Add blank lines between logical blocks so the flow is visible at a glance:

- before the final `return`
- after variable declarations, before they're used
- between error handling and subsequent logic
- between distinct logical operations

Bad:

```go
func encodeMessages(messages []Message) (string, error) {
	if len(messages) > 0 {
		messagesBytes, err := json.Marshal(messages)
		if err != nil {
			return "", err
		}
		return string(messagesBytes), nil
	}
	return "", nil
}
```

Good:

```go
func encodeMessages(messages []Message) (string, error) {
	if len(messages) > 0 {
		messagesBytes, err := json.Marshal(messages)
		if err != nil {
			return "", err
		}

		return string(messagesBytes), nil
	}

	return "", nil
}
```

---

## Comments

- **No obvious comments** — don't restate what the code already shows.
- **Explain *why*, not *what*** — code shows what happens; comments explain business rules, hidden constraints, or non-obvious optimizations.
- **Prefer refactoring over commenting** — better names or smaller functions usually beat a comment.
- **Complex algorithms deserve comments** — formulas, business rules, non-obvious optimizations.
- **No "Summary" / "Conclusion" sections in `.md` files** unless explicitly requested.

Bad (each comment restates the function name):

```go
// Send heartbeat
sendHeartbeat(request)

// CreateValidLogItems creates valid log items for testing
func CreateValidLogItems(count int, uniqueID string) []LogItemRequestDTO {
```

---

## File organization

One responsibility per file. Don't dump a whole package into one file — split
by role so a reader can find a type by its filename. Conventional names within
a feature package:

- `doc.go` — package doc comment, once the package spans more than one file
- `<feature>.go` — the core type and its methods (the orchestrator/executor)
- `dto.go` — request/response and cross-package data + interface seams
- `errors.go` — sentinel errors (`var Err... = errors.New(...)`)
- `enums.go` — typed-constant groups (`type Status string` + its values)
- `constants.go` — package-level constants that aren't an enum
- background loops, reapers, and pools get their own file (`reaper.go`, `pool.go`)

Only create a file when there is real content for it — an empty `enums.go` or
`constants.go` is noise, not structure. Test files mirror the source split:
`restorer.go` → `restorer_test.go`, `diskexhaustion.go` →
`diskexhaustion_test.go`.

---

## Background services

The agent ships at least one long-running goroutine (the capacity `Heartbeater`; also `BackgroundUpgrader`). Calling `Run()` twice on the same instance is always a bug — duplicate goroutines leak resources and corrupt state. **Always panic; never just log a warning.**

```go
type Heartbeater struct {
    // ...
    hasRun atomic.Bool
}

func (h *Heartbeater) Run(ctx context.Context) {
    if h.hasRun.Swap(true) {
        panic(fmt.Sprintf("%T.Run() called multiple times", h))
    }

    ticker := time.NewTicker(heartbeatInterval)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            h.beat(ctx, logger)
        }
    }
}
```

`atomic.Bool.Swap(true)` does the check-and-set atomically — no `sync.Once` needed.

---

## Testing

**Always run tests after writing them and verify they pass.**

### Naming

- `Test_WhatWeDo_WhatWeExpect`
- `Test_WhatWeDo_WhichConditions_WhatWeExpect`

Examples: `Test_DeriveCapacity_WhenConcurrentJobsExceedCPU_ReturnsError`, `Test_ValidateTransport_WhenHttpAndNotTTYWithoutFlag_FailsFast`, `Test_Heartbeat_WhenCalled_SendsFlatEnvelopeWithBearerAndAgentPath`, `Test_BackgroundUpgrader_WhenRunCalledTwice_Panics`.

### Where tests live


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databasus/databasus](https://github.com/databasus/databasus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
