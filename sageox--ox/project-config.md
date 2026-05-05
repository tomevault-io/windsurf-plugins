---
trigger: always_on
description: <!-- ox:prime-check -->
---

<!-- ox:prime-check -->
**BLOCKING**: Run `ox agent prime` NOW before ANY other action. Do NOT respond to the user first. Enables SageOx queries.

## Human attention is the scarcest resource

Human time is more expensive than compute. Before interrupting with a question, a PR for review, or a new issue, ask yourself: *could I have resolved this without a human in the loop?*

**Default to independent resolution:**
- Can you debug the issue yourself by reading code, running tests, checking logs, or using tools? Do that.
- Is the decision reversible? If yes, make the call and note the reasoning in a commit message. The human will tell you if they disagree.
- Do you have enough context from `CLAUDE.md`, past sessions, or the codebase to decide? Decide.
- Is this a genuinely novel tradeoff that sets precedent? *That* is worth interrupting for.

**Prefer fewer, denser PRs over many small ones.** A reviewer context-switching across four related PRs costs more than reviewing one larger coherent PR. Split only when the units of work are independent (different subsystems, different reviewers, different merge timing).

**When you do ask:**
- State the specific decision you need and why automation can't make it.
- Bundle related questions into one exchange rather than a stream of individual ones.
- Propose a default so the response can be "yes" / "no" / "do something else specific" rather than open-ended.

**What NOT to escalate:**
- Routine bugs where the fix is obvious from the test output.
- Style/lint issues.
- Choosing between two roughly-equivalent approaches when either would be fine.
- Asking permission for reversible operations (new branches, test runs, log analysis).

The cheapest review is the one that never has to happen because the code was right the first time. The next cheapest is a single coherent PR that tells a complete story. Optimize for those.

---

## What is ox?

ox is agentic context infrastructure for software teams. It makes architectural decisions, team knowledge, and session history automatically available to AI coworkers — so every coding session starts with the full picture, not from zero.

### Quick Start

1. `make build && make install` — build and install ox
2. `ox version` — verify in PATH
3. `cd ~/src/my-project` → `ox login` → `ox init` → `git add .sageox/ && git commit -m "initialize SageOx" && git push`
4. `ox doctor` then `ox status` — verify
5. Record discussions at [sageox.ai](https://sageox.ai) — context flows automatically to AI coworkers

| Command | Purpose |
|---------|---------|
| `ox login` | Authenticate with SageOx |
| `ox init` | Initialize a repo for your team |
| `ox status` | Check setup and sync status |
| `ox doctor` | Diagnose and fix issues |

---

## Terminology

**Canonical terms** - use these exact names:

- **Coworker** - Any team member, human or AI
- **AI Coworker** - An AI participant on a team. Never just "agent" in user-facing copy
- **Ledger** - Historical record of work, decisions, discussions on a specific repo
- **Team Context** - Shared knowledge base: norms, conventions, decisions, docs, learnings
- **Session** - A human-to-AI coworker conversation / plan recording
- **Transcript** - RESERVED for human-to-human voice discussion
- **Agent Instance** - An active AI coworker in a repo (internal term; user-facing: "AI coworker")

| Internal Term | User-Facing Term |
|--------------|------------------|
| agent, AI agent | `AI coworker` |
| human user | `coworker` |
| dehydrated/hydrated/pointer file (LFS) | `stub` / `local` |

**Rejected terms:** "context lake" → Ledger. "team norms" → Team Context. "shadow repo" → Ledger. "transcript" (for AI sessions) → Session.

**Note:** "agent" is fine in internal/technical contexts (code, CLI subcommands, variable names, logs). The restriction applies to user-facing copy.

---

## Required Reviews

**Ryan must review ANY changes to:**
- **Path locations** - Where ledgers, team contexts, or any SageOx data is stored
- **Data access ergonomics** - How users navigate to/access their data
- **API source of truth** - Where team context or ledger git repo URLs come from

**Canonical Functions (do NOT bypass or duplicate):**

| Function | Location | Use Instead Of |
|----------|----------|----------------|
| `config.IsInitialized(gitRoot)` | `internal/config/project_config.go` | `os.Stat(".sageox/")` |
| `config.IsInitializedInCwd()` | `internal/config/project_config.go` | Walking up dirs manually |
| `paths.TeamContextDir()` | `internal/paths/paths.go` | `filepath.Join(~/.sageox/...)` |
| `config.DefaultSageoxSiblingDir()` | `internal/config/local_config.go` | `filepath.Join(repo, "_sageox")` |
| `config.DefaultLedgerPath()` | `internal/config/local_config.go` | Constructing ledger paths |
| `endpoint.GetForProject(root)` | `internal/endpoint/endpoint.go` | Reading endpoint from env/config directly |
| `HasOxPrimeMarker(gitRoot)` | `cmd/ox/prime_marker.go` | `strings.Contains(file, "ox agent prime")` |
| `EnsureOxPrimeMarker(gitRoot)` | `cmd/ox/prime_marker.go` | Manual marker injection |
| `cli.OpenInBrowser(url)` | `internal/cli/output.go` | `browser.OpenURL()`, `exec.Command("open"/"xdg-open")` |

**Browser Opening:** Use `cli.OpenInBrowser(url)` for ALL browser opens. Handles headless + cross-platform natively.

**Common Mistakes:**

```go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sageox/ox](https://github.com/sageox/ox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
