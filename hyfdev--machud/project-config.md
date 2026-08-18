---
trigger: always_on
description: A beautiful, zero-config terminal system monitor for macOS — a TUI reimagining of
---

# machud

A beautiful, zero-config terminal system monitor for macOS — a TUI reimagining of
[Stats](https://github.com/exelban/stats) (mac-stats.com), built with
[vue-tui](https://github.com/vuejs-ai/vue-tui).

**machud is opinionated — that's the whole point.** Where btop wins on deep configurability,
machud wins on *beautiful + zero-config*: there are no settings, no config files, no layout
knobs. The single curated wide-screen dashboard — Apple-Silicon-aware, with one foreground palette
that works on light and dark terminal backgrounds — **is** the product. Want to tune everything yourself? Use btop. machud is the
considered default that looks right out of the box and never asks for your password.

> **Contributing:** machud is opinionated by design. Bug reports are welcome; **feature PRs
> opened without a prior, agreed-upon issue are closed directly**, and theming/feature/aesthetic
> changes are generally not accepted. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Run it

```bash
pnpm build          # tsdown → self-contained dist/main.mjs + third-party notices
pnpm start          # run the live dashboard (needs a real TTY)
pnpm dev            # Vite + @vue-tui/vite HMR
pnpm test           # Vitest unit + component render tests
pnpm run check:type # vue-tsc project type check
node dist/main.mjs --once   # render ONE real-data frame to stdout and exit
```

`--once` is the verification path: it needs no TTY, prints a single frame with live
readings, and doubles as a pipe-friendly snapshot.

## Design notes

- **Zero-sudo first.** All data comes from unprivileged commands (`sysctl`, `vm_stat`,
  `ioreg`, `pmset`, `netstat`, `df`, `ps`) + Node's `os`. Metrics that need `sudo`
  (powermetrics: precise per-cluster freq, GPU watts, fan RPM, die temps) are omitted
  (D2 — not shown as dead `—` rows), never block startup. See `.agents/docs/architecture.md`.
- **Never crash, just degrade.** `sh()` resolves "" on failure; each collector returns
  safe nulls; `collectAll()` swaps any throwing collector for its empty default.
- **Architecture.** `src/lib/collectors/*` (one file per module) → `useMetrics` poller →
  panel components. Full map in `.agents/docs/architecture.md` — read it before changing
  data sources or layout.
- **TDD by default.** For behavior changes, write or extend the failing verification
  first, watch it fail when practical, implement the smallest fix, then make the gate
  green. `pnpm verify` is the primary harness — it type-checks, builds, black-box-checks
  `--once`/`--json`, drives a PTY, AND runs the Vitest component/unit layer. Extend
  `scripts/verify.mjs` for modules / metrics / render / terminal guarantees; add
  per-panel component render tests under `tests/`.

> Component tests use `@vue-tui/testing@0.3.0` to mount real `.vue` panels through Runtime's
> production renderer. `vitest.config.ts` applies only the Vue transform; the terminal-owning
> `@vue-tui/vite` plugin remains exclusive to the development server. See D20.

<!-- PCR:START -->
## Project Context Records (PCR)

This project follows **Project Context Records (PCR)** — methodology: https://github.com/hyfdev/project-context-records. PCR keeps the project's durable judgment — the *why*, the decisions, the intent — so you inherit it instead of re-deriving or re-litigating what's already settled.

When working here:
- **Where records live.** Records are in `.agents/docs/`, one topic per file, cross-linked with relative Markdown links.
  - A `README.md` there is the **map**: it routes code areas or hotspots to the exact record or heading, one-line gist per route. Create it when retrieval stops being a glance or one record grows into a long ledger.
- **Read first.** Start from the map if present, else scan the folder. Open the records or headings that cover an area before changing or answering for it; if the area has a decision ledger, read it first.
- **Use the strongest durable form.** Machine-checkable constraints go in types, tests, lints, or CI; rules that must bind every session go in the agent-instructions file, outside the markers; single-spot rationale goes beside the code with a link; records carry the cross-cutting judgment, intent, and context that must stay prose.
- **Record as you go.** Capture context when a decision lands, a trap costs you, a human corrects you, or a human asks — anything true about this project, not durable in a stronger form, and useful beyond the moment.
  - Report what you record so a human can review or vouch it.
  - Records are as public as the repo: keep secrets out, and ask before recording rationale from private context.
- **Write to be acted on.** Lead with the current conclusion and where it applies; capture the why — trade-offs, alternatives rejected, known pitfalls. Keep each topic's current truth in one fresh place, updated in place: evolution belongs to git, never to supersede chains.
- **Keep it fresh.** Update affected records in the same change that touches their subject.
  - When code and a record disagree, decide which side went stale and fix that side.
  - Back facts with durable evidence — tests, reproducible commands, committed artifacts, stable URLs, commit hashes — not ephemeral paths or one session's output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyfdev/machud](https://github.com/hyfdev/machud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
