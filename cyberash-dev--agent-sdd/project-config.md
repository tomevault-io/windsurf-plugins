---
trigger: always_on
description: This file is for any AI coding agent (Claude Code, Codex, Cursor,
---

# AGENTS.md

This file is for any AI coding agent (Claude Code, Codex, Cursor,
Aider, et al.) working in this repository. It describes the rules of
the road, the canonical sources of truth, and the boundaries you must
not cross.

`CLAUDE.md` (project-level, sibling of this file) is the
Claude-specific extension. The two are kept in sync — start here, and
read `CLAUDE.md` if you are Claude Code.

---

## Rule zero — spec is the source of truth

`spec/spec.md` is the authoritative specification. Every behavior in
the code traces to a normative ID (`BEH-*`, `CTR-*`, `INV-*`, `POL-*`,
`CST-*`, `EXT-*`, `GEN-*`, `IMP-*`, `DLT-*`).

If a code change does not match the spec, the spec change comes
**first**, in the same PR, and the `sdd lint` gate is run before
implementation. There is no exception. "Quick hack to fix the build"
is not an exception either — `sdd lint` will refuse it.

---

## Don't self-approve

You are an agent. The `--approver` argument of `sdd approve` MUST be a
human identity. The CLI rejects agent identities (Claude, Codex,
spec-author-bot, bot:*, sdd-cli itself) case-insensitively. This is
`SDD §7.5` and `INV-005`. Don't try to argue your way around it; don't
suggest a placeholder human name; just stop and ask the user.

If you are about to write `lifecycle.status: approved` directly into
spec.md without going through `sdd approve`, STOP. That is a forbidden
back-door. The only way to flip status is the CLI, with a real human
approver, which writes a typed `approval_record` block atomically
(`INV-007`).

---

## What you may change without ceremony

- New `proposed` IDs (Behaviors, Contracts, Invariants, etc.) with
  `lifecycle.status: proposed` and `approval_record:
  not_applicable_for_proposed`. These are sandbox until a human
  approves them.
- Tests under `tests/unit/` and `tests/integration/`. Always add a
  test for new behavior; bug fixes get a regression test that fails
  before the fix.
- Implementation files for the slice that owns a `proposed` ID.
- Documentation (`README.md`, `CLAUDE.md`, `AGENTS.md`, `CHANGELOG.md`).

## What you may NOT change

- The `lifecycle.status` field of any `approved`/`deprecated`/`removed`
  record (only `sdd approve`/`sdd deprecate`/`sdd remove` may flip it).
- The `approval_record` block of any approved record.
- Surface contracts (`SUR-*`, `CTR-*`) without a `Delta` and a major or
  minor bump on the owning Surface.
- The architecture invariant `INV-004` / `CST-003` (vertical slice +
  hexagonal, no global layer folders, no cross-feature imports).
- The built-in git subcommand allowlist `EXT-001` / `POL-002`, or the
  pluggable-VCS contract (`SUR-017`, `CTR-031` / `CTR-032`, `POL-004`),
  without a `Delta`.

---

## Architecture cheat sheet

```
src/
  cli.ts + cli*.ts                       # composition-root entry layer (parse, wire, route)
  features/{token,check,refresh,lint,approve,ready,record,install,…}/
    domain/                              # pure logic
    application/                         # use cases
    ports/{inbound,outbound}/            # interfaces
    adapters/{inbound,outbound}/         # only here you may import node:*
  shared/
    domain/                              # cross-feature primitives, incl. PartitionGrammar
                                         # (CST-007 marker + CTR-015 regex) and the Vcs
                                         # port + VcsConformance (pure types/validator)
  vcs/                                   # built-in GitVcs + resolveVcs loader (node:* allowed);
                                         # wired only from the composition root
```

- Inside a feature: `adapters → ports → application → domain`. No
  arrows pointing the other way.
- Cross-feature imports: forbidden. Reach via `src/shared/domain/`.
- `src/shared/domain/` imports no `node:*` module **except**
  `node:crypto` inside `src/shared/domain/Token.ts`.
- `src/vcs/` may import `node:*` and `src/shared/domain`, but never a
  feature; only the composition root (`cli*`) imports `src/vcs`.
- New feature? Mirror the same layout.

`tests/unit/layer-imports.test.ts` mechanically enforces all of the
above. Run it after every meaningful refactor.

---

## Commands you should know

```sh
npm install                  # bootstrap
npm run tsc                  # type-check (no emit)
npm run test:unit
npm run test:integration
npm test                     # both
npm run build                # produces dist/cli.js with shebang + +x

# the tool against itself (the repo's own spec.md)
node dist/cli.js --help
node dist/cli.js token
node dist/cli.js check
node dist/cli.js refresh
node dist/cli.js lint        # MUST exit 0 in CI
node dist/cli.js ready       # SHOULD exit 0 in CI (gate-3)

# navigate / edit the spec one record at a time (no whole-file read)
node dist/cli.js record list                # index: id · type · status · title
node dist/cli.js record get <id>            # one record, verbatim
node dist/cli.js record set <id> --from-file body.yaml      # draft/proposed only
node dist/cli.js record add --after <id> --content "$BODY"  # new draft/proposed record

# distribute the SDD methodology rules (+ Claude hooks) into the agent config
node dist/cli.js install all --dry-run      # preview, write nothing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyberash-dev/agent-sdd](https://github.com/cyberash-dev/agent-sdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
