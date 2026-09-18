---
trigger: always_on
description: generates the shell/PowerShell installers and the Homebrew formula,
---

# AGENTS.md

Guidance for agents working in this repository. See [docs/DESIGN.md](docs/DESIGN.md)
for what trellis is and why.

## Product-name capitalization

Use **trellis** in running text and **Trellis** at the start of a sentence and
in document titles. Use sentence case for headings: "Installing trellis", not
"Installing Trellis". This keeps the existing convention without rewriting
CLI output or generated documentation.

- Keep the visual wordmark lowercase: **trellis**.
- Preserve literal spelling in commands (`trellis doctor`), binary and crate
  names, paths, URLs, config keys (`[tools.trellis]`), schema names
  (`trellis.doctor/1`), and environment variables (`TRELLIS_NO_UPDATE_CHECK`).
  Do not capitalize a command just because it starts a sentence or heading.
- Leave published release notes and quoted historical output unchanged.

## Naming: snake_case for everything we control

**Every identifier trellis defines is snake_case.** That covers:

- `[tools.trellis]` config keys — `tag_format`, `needs_deps`, `series_tag_format`
- `--json` object keys — `exit_code`, `duration_ms`, `auto_members`
- `--json` enum values — `member_glob`, `seed_changelog`, `up_to_date`
- `schema` payload names — `trellis.changelog_check/1`, `trellis.version_plan/1`
- `trellis ci outputs` GitHub Actions output names — `version_files`, `series_tags`

The config table lives inside `gleam.toml`, which spells its own settings
`internal_modules`, and snake_case is the Gleam convention generally. One rule
for everything we emit means nobody has to remember which surface uses which.

Two exclusions, both formats trellis does not own:

- **Gleam manifest keys.** `dev-dependencies` stays as Gleam spells it — see
  `src/gleam.rs`, `src/rewrite.rs`, `src/commands/new.rs`.
- **CLI long flags.** `--no-update-check`, not `--no_update_check`. Kebab is the
  universal convention there and clap's default.

Free-form table keys (`exclude`, `tasks`, `publish.package_tags_overrides`) are the
user's to name; a hyphen in one is not a violation, and `doctor` says nothing
about them.

Config keys released through v0.7.0 keep their kebab-case spelling as a
`#[serde(alias)]`, reported by `doctor` as a deprecation. These come out at 1.0;
new keys never get one.

## Changelog fragments

**Only user-facing product changes need a changelog fragment.** Documentation,
tests, CI, infrastructure, and internal maintenance do not need one unless they
also change user-facing product behavior.

For a product change, add a YAML file in `.changes/unreleased/` named
`<Kind>-<YYYYMMDD>-<slug>.yaml`, with `component`, `kind`, `body`, and `time`.
The audience is a stranger reading the release notes, not the reviewer of your PR.

**This section is about trellis's own changelog, which changie manages — not
about the fragments trellis writes.** The two formats are different and easy to
confuse: changie fragments are YAML keyed on `component`, while trellis's native
engine reads TOML keyed on `package`, `kind`, an optional `category`, and `body`
(`src/changelog.rs`). Everything below applies to this repository's
`.changes/unreleased/`; a Gleam workspace consuming trellis uses the TOML shape,
documented on the [changelog page](website/src/content/docs/docs/changelog.mdx).

Write the body as a `|-` block scalar with a **bolded lead-in sentence**, then
paragraphs:

```yaml
component: completions
kind: Added
body: |-
  **`trellis completions` generates tab-completion for five shells.** Candidates are computed by the binary as you type, so completion offers real package, task, and changelog-kind names from the surrounding workspace.

    Release archives also ship man pages under `man/`.
time: 2026-07-25T18:01:19.474540769-07:00
```

**One line per paragraph. Never hard-wrap a body**, however long the line gets —
the shipped entries run to 500+ characters. cargo-dist copies the version's
CHANGELOG.md section verbatim into the GitHub release body, and GitHub renders
release notes the way it renders comments: every single newline becomes a `<br>`,
unlike a `.md` file in the repo, where it collapses to a space. A body wrapped at
80 columns reads fine in CHANGELOG.md and arrives in the release notes as a
ragged column of short lines. Wrap only *between* paragraphs, with a blank line.

`component` is the subcommand the change belongs to, and renders as a `###`
heading above the `####` kind headings, so a reader can find what changed in
`doctor` without reading the whole release. The list and its render order live in
`.changie.yaml`; `changie new` prompts from it.

**`trellis` is the catch-all,** for a change that genuinely isn't scoped to one
subcommand: the exit-code contract, the global flags, a rule that holds for every
`--json` payload. It sorts first, ahead of the per-command sections.

Reach for it last. A change landing on two or three subcommands wants two or
three fragments, one per command, each saying what that command's payload or flag
actually does — not one `trellis` entry. Splitting usually sharpens the copy,
because the caveats differ per command: `run --json` documents `exit_code` naming
the failed command of a `needs_deps` task, which `exec --json` has no equivalent
of. Used as a bucket for "more than one command", `trellis` stops meaning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tylerbutler/trellis](https://github.com/tylerbutler/trellis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
