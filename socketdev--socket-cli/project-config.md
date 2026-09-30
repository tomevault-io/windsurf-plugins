---
trigger: always_on
description: - Identify users by git credentials; use "you/your" directly; shorthand phrases have fixed meanings. [`vocabulary`](docs/fleet/agents.md/vocabulary.md)
---

# AGENTS.md


<!-- <fleet> -->

## 📚 Fleet

- Identify users by git credentials; use "you/your" directly; shorthand phrases have fixed meanings. [`vocabulary`](docs/fleet/agents.md/vocabulary.md)
- Multiple Claude sessions may target one checkout: never run a git command that mutates state outside the file you just edited. [`parallel-claude-sessions`](docs/fleet/agents.md/parallel-claude-sessions.md)
- Follow explicit user instructions over peer changes; do not ask again. [`parallel-claude-sessions`](docs/fleet/agents.md/parallel-claude-sessions.md)
- Local main is canonical: origin ahead by own/bot squash commits ≠ newer truth. [`parallel-claude-sessions`](docs/fleet/agents.md/parallel-claude-sessions.md)
- Active-edits ledger coordinates concurrent actors: a path another live actor wrote within 5 min is blocked, as are open-ended wait promises. [`parallel-claude-sessions`](docs/fleet/agents.md/parallel-claude-sessions.md)
- Keep repo paths local. Only validated Wheelhouse commit-cascade may cross repos. [`parallel-claude-sessions`](docs/fleet/agents.md/parallel-claude-sessions.md)
- Use `pnpm run worktree:create`. [`parallel-claude-sessions`](docs/fleet/agents.md/parallel-claude-sessions.md)
- Check `who_owns`/`list_claims` before non-trivial work; `claim_paths` what you take, `release_paths` when done. [`claim-before-you-work`](docs/fleet/agents.md/claim-before-you-work.md)
- Never hard-code `main` in scripts: resolve the default branch via `git symbolic-ref`, fall back `main` → `master`. [`default-branch-resolution`](docs/fleet/agents.md/default-branch-resolution.md)
- Write no real customer name, private repo, Linear ref, or Slack thread on a public surface. [`public-surface-hygiene`](docs/fleet/agents.md/public-surface-hygiene.md)
- Root `README.md` follows the fleet skeleton - 5 level-2 sections in order, every member. [`public-surface-hygiene`](docs/fleet/agents.md/public-surface-hygiene.md)
- Fleet repos use Conventional Commits `<type>(<scope>): <description>`, lowercase, with NO AI attribution. [`commit-cadence-format`](docs/fleet/agents.md/commit-cadence-format.md)
- No fleet commit trailer or branch name carries an AI tool's mark. (`scripts/fleet/check/commits-have-no-ai-attribution.mts`) [`agent-detection-surfaces`](docs/fleet/agents.md/agent-detection-surfaces.md)
- Run human-facing prose through the `prose` skill before it lands. (`.claude/hooks/fleet/anti-prose-guard/`) [`prose-style-and-doctrine`](docs/fleet/agents.md/prose-style-and-doctrine.md)
- Report to the operator in ASD-STE100: one topic per sentence (max 20/25 words), active voice, no synonym variation, warnings first. [`reporting-in-ste100`](docs/fleet/agents.md/reporting-in-ste100.md)
- PR review comments use the fleet format: severity-sorted `<details>` `<abbr>` circles, `Suggestion 💡:` labels, junior-dev sentences, dup-PR scan. [`pr-review-comments`](docs/fleet/agents.md/pr-review-comments.md)
- Some fleet repos squash the default branch on a cadence: land fast and don't fuss. [`history-rewrites`](docs/fleet/agents.md/history-rewrites.md)
- The `squash-history` opt-in tracks the release boundary: the first release FREEZES history through that commit, and only the unreleased tail squashes. [`squash-until-release`](docs/fleet/agents.md/squash-until-release.md)
- `fleet-main-protection` blocks force-push, `fleet-tag-protection` blocks `v*` tag deletes. [`history-rewrites`](docs/fleet/agents.md/history-rewrites.md)
- npm stages burn versions: minor default, odai patch/minor, major needs `X.Y.Z-prerelease`. [`version-bumps`](docs/fleet/agents.md/version-bumps.md)
- NEVER open a pull request to land a version bump: the bump commit goes DIRECTLY on the default branch via the release App. (`.claude/hooks/fleet/no-version-bump-pr-guard/`) [`version-bumps`](docs/fleet/agents.md/version-bumps.md)
- Dot-naming `@owner/<name>[.<lang>].<target>[-<platform>]`: the `.target` token carries the domain. [`binary-vs-napi-naming`](docs/fleet/agents.md/binary-vs-napi-naming.md)
- A private package is unscoped `local-<directory>` at version `0.0.0`. [`private-package-identity`](docs/fleet/agents.md/private-package-identity.md)
- Every `release.publishedPackages` entry is non-private and the set carries ONE version. (`scripts/fleet/check/published-packages-are-release-ready.mts`) [`private-package-identity`](docs/fleet/agents.md/private-package-identity.md)
- External refs pin the SHA and comment the label (`<sha> # v3.2.1`). (`scripts/fleet/check/external-refs-carry-sha-and-label.mts`) [`immutable-references`](docs/fleet/agents.md/immutable-references.md)
- Anything invoking the `claude` CLI or Agent SDK sets all four lockdown flags. [`locking-down-claude`](docs/fleet/agents.md/locking-down-claude.md)
- **`pnpm`, from the repo root**: no `npx`/`dlx`, `tsx`/`ts-node`, `cd <subpkg> && pnpm`, or `corepack`. [`tooling`](docs/fleet/agents.md/tooling.md) [`database`](docs/fleet/agents.md/database.md) (`.claude/hooks/fleet/corepack-guard/`)
- Test and coverage entrypoints reject incomplete workspace installations. (`scripts/fleet/check/workspace-installation.mts`) [`workspace-installation`](docs/fleet/agents.md/workspace-installation.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SocketDev/socket-cli](https://github.com/SocketDev/socket-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
