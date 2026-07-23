---
trigger: always_on
description: Self-hostable control plane for ephemeral cloud agents. A user points
---

# Warren

Self-hostable control plane for ephemeral cloud agents. A user points
warren at a GitHub repo, picks an agent, writes a prompt; warren spawns
the agent inside a sandbox, streams events back to the UI, lets the user
steer mid-run, then pushes the workspace branch. One container, one
volume, one HTTP API, one UI.

The fresh-install path is standalone: the built-in `claude-code` agent
ships inline (`src/registry/builtins/`), so a user with a GitHub URL and
an Anthropic key can dispatch a run end-to-end with no other tooling.

Warren also bundles five [os-eco](https://github.com/jayminwest/os-eco)
data-plane tools as **opt-in built-in features**, not required
infrastructure:

- **canopy** — versioned prompt libraries for custom agents. Activated by
  setting `CANOPY_REPO_URL`; library agents override built-ins by name.
- **mulch** — persistent agent memory across runs. Activated by the
  project having a `.mulch/` directory.
- **seeds** — integrated issue queue agents read from and write to.
  Activated by the project having a `.seeds/` directory.
- **sapling** — alternative steerable coding harness. Ships inline as a
  built-in agent alongside claude-code.
- **plot** — shared coordination substrate where humans and agents are
  peer nodes on a per-Plot event log. Activated by the project having a
  `.plot/` directory **and** the dispatch carrying a `plot_id`; warren
  injects `PLOT_ID` + `PLOT_ACTOR` into the sandbox, appends
  `run_dispatched`, and mirrors agent-emitted events at reap. See
  SPEC §11.O.
- **plan-run** — serial sd plan execution; activated by the project
  having a `.seeds/` directory. A dispatch mode on top of the existing
  single-run primitive (not a sixth bundled feature): `POST /plan-runs`
  walks a seeds plan's children one at a time, gating each on the
  previous PR merging before the next dispatches. Re-dispatching the
  same plan resumes from the next open child. When the project also
  ships `.plot/` and the dispatch carries `plot_id`, plan-runs compose
  onto Plot: one `plan_run_dispatched` event at start, per-child
  `PLOT_ID` injection + `run_dispatched` for free, and an auto-`done`
  Plot transition when the final child merges. See SPEC §11.P and
  §11.P.Plot.

Same code, same depth — only the user-facing framing surfaces them as
opt-in. When you change cross-cutting docs (README, SPEC §1/§2, package
description), keep the standalone path primary and the integrations as
features that light up when used.

Warren runs against a swappable **runtime provider**, resolved once at
boot from `WARREN_RUNTIME` (`src/runtime/registry.ts`) behind the
`RuntimeProvider` contract (`src/runtime/contract.ts`). Two backends:
`LocalProvider` (`src/runtime/local/`, the default) wraps the
co-tenanted [burrow](https://github.com/jayminwest/burrow) sandbox
daemon; `K8sProvider` (`src/runtime/k8s/`, `WARREN_RUNTIME=k8s`) runs
each agent as a Kubernetes pod with no burrow at all. Burrow is the
LocalProvider's substrate, **not a required warren dependency** — see
"Relationship to burrow" below and [docs/RUNBOOK-K8S.md](docs/RUNBOOK-K8S.md)
for the K8s topology.

[SPEC.md](SPEC.md) is the V1 design record. The manual-run path
(`warren run <agent> <project> -p "..."`) and the cron half of the
scheduler (`.warren/triggers.yaml` + past-due `scheduledFor` seed
extensions, SPEC §11.I) are what V1 ships; GitHub webhook triggers
and library API exports are deferred to V2.

### Per-project config (`.warren/config.yaml`)

The canonical home for per-project defaults is `.warren/config.yaml`
(legacy `.warren/defaults.json` still loads with a deprecation warning).
Schema lives in `src/warren-config/schema.ts` (`DefaultsConfigSchema`)
and is surfaced by `loadWarrenConfig()`. Notable knobs:

- `defaultRole`, `defaultPrompt`, `defaultProvider`, `defaultModel`,
  `defaultBranch`, `runBranchPrefix` — dispatch-time defaults; see
  SPEC §11.H.
- `preview` — per-run preview environments; canonical home is
  `.warren/preview.yaml`, see SPEC §11.L.
- `agent.pauseTimeoutMs` (default `1800000` = 30 min, bounds 1s..24h)
  — wall-clock budget for paused interactive turns and batch runs that
  emit `question_posed`. Consumers fall back to
  `DEFAULT_AGENT_PAUSE_TIMEOUT_MS` when the block is absent. SPEC §11.O
  (warren-cd37 / pl-0344 step 2).
- `agent.skipGitHooks` (default `false`) — set to `true` to skip arming
  the project's git pre-commit gate on the host clone before each run.
  By default warren detects a `git config core.hooksPath` call in the
  project's `package.json` prepare script and applies it to the clone's
  `.git/config` so every worktree (agent sandbox) inherits the hook.
  Flip this when a project's hooks are too slow, require tools not
  available on the warren host, or you explicitly want agent commits
  unfiltered (warren-8f4c).
- `plotSync` — per-project Plot sync to GitHub configuration.
  `mergeStrategy` (`immediate` | `auto` | `manual`, default `manual`)
  controls whether sync PRs are auto-merged; `targetBranch` overrides
  the project's `defaultBranch` for the PR base. `POST /plots/:id/sync`
  triggers manually; formalize and status-change fire background syncs.
- `admission.maxConcurrentRuns` — per-project cap on simultaneous

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jayminwest/warren](https://github.com/jayminwest/warren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
