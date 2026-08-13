---
trigger: always_on
description: deslop is a [Vale](https://vale.sh) style package that flags AI-slop in prose: the stock
---

# deslop — Agent Guide

deslop is a [Vale](https://vale.sh) style package that flags AI-slop in prose: the stock
phrases, hedges, and structural tics that mark machine-written text. It ships as a release
asset (`Deslop.zip`) that other repos consume through Vale's `Packages` mechanism.

## Layout

| Path | What it is |
| --- | --- |
| `styles/Deslop/*.yml` | The rules. One file per tell family (e.g. `SlopVocab.yml`, `HollowCloser.yml`). Each is a Vale rule extending `existence`, `substitution`, or `occurrence`. |
| `styles/Deslop/meta.json` | Package metadata Vale reads on `vale sync`. |
| `.vale.ini` | Lints this repo's own prose. `Deslop` only, no `Vale` base style. Excludes `tests/`, which holds deliberate slop. Consumers use a different `.vale.ini` (see below). |
| `tests/run.sh` | The rule suite. Run it after every rule change. |
| `tests/should-flag.md` | Deliberate slop. Every rule must fire here at least once. |
| `tests/should-pass.md` | Ordinary technical prose. Any alert here is a false positive and fails the build. |
| `tests/expected.tsv` | Pinned `<rule>`/`<phrase>` pairs, mostly inflections and narrowed tokens. |
| `tests/expected-branches.tsv` | Pinned branch lists for `occurrence` rules, which nothing else protects. |
| `tests/check.py` | The assertions behind `run.sh`. |
| `docs/ADOPTING.md` | Copy-paste integration guide: `.vale.ini`, CI, customizing, composing private styles. |
| `.github/workflows/test.yml` | Runs the rule suite on every push and PR. |
| `.github/workflows/vale.yml` | Lints this repo's `*.md` on PRs. |
| `.github/workflows/release.yml` | On a `v*` tag, runs the suite, builds `Deslop.zip`, publishes a GitHub release. |
| `CHANGELOG.md` | What changed between released versions. |
| `README.md` | Human-facing overview and the full rule table. |

## How to use it (consuming the package)

A downstream repo adds a `.vale.ini`:

```ini
StylesPath = styles
MinAlertLevel = suggestion
Packages = https://github.com/JMill/deslop/releases/latest/download/Deslop.zip

[*.{md,mdx}]
BasedOnStyles = Deslop
```

Then `mkdir -p styles && vale sync && vale "**/*.md"`. The `mkdir -p styles` must come before
`vale sync` or Vale stages to a temp path and leaves `StylesPath` empty. Full guide:
[docs/ADOPTING.md](docs/ADOPTING.md).

## Writing a rule

The common change is adding a tell. Create one file under `styles/Deslop/`. Most rules extend
`existence` (flag any match) and take a list of regex `tokens`:

```yaml
# styles/Deslop/SlopVocab.yml (excerpt)
extends: existence
message: "'%s' is an AI-slop tell. Cut it or say the thing plainly."
level: warning
ignorecase: true
tokens:
  - 'delv(?:e|es|ed|ing)'
  - 'plethora'
  - 'when\s+it\s+comes\s+to'
```

Use `occurrence` for density caps (cap how often a `token` appears in a `scope`, as
`HollowIntensifier.yml` does) and `substitution` for "prefer X over Y" swaps. A new file is
picked up automatically once it is placed under `styles/Deslop/`, because `.vale.ini` lists
the `Deslop` style directory in `BasedOnStyles` (you list style directories there, not
individual rule stems). The rule name is the file stem (`Deslop.SlopVocab`). See the
[Vale styles docs](https://vale.sh/docs/topics/styles/).

**Always run `tests/run.sh` after touching a rule.** Every gotcha below produces a rule that
loads without error and silently matches nothing or too much. Vale reports no diagnostic for
any of them; the suite is the only thing that will tell you.

### Gotchas that cost real debugging time

- **`tokens` entries are wrapped in `\b...\b`.** So `delve` does not match `delved`, and
  `seamless` does not match `seamlessly`. Spell inflections out: `delv(?:e|es|ed|ing)`.
  This is the single most common way to ship a rule that looks right and under-fires.
  It applies to verb-headed idioms too, where the base form is the *least* common one in
  practice: `move the goalposts` misses `moved the goalposts`, and `circle back` misses
  `circled back`. Write `mov(?:e|es|ed|ing)\s+the\s+goalposts`.
- **Cover the whole paradigm, including irregulars and the gerund.** `navigate(?:s|d)?`
  silently drops `navigating`, because the gerund changes the spelling of the stem.
  `(?:driv(?:e|es|ing)|drove)` drops `driven`. And an alternation like `remain(?:s|ed)`
  drops the *base* form, since the suffix group is mandatory without a trailing `?`.
- **A `tokens` entry ending in punctuation never matches.** The trailing `\b` cannot be
  satisfied after `!` or `,`. Use `raw` for those.
- **Multiple `raw` entries are concatenated, not alternated.** A list of five `raw`
  patterns compiles to one impossible regex and the rule flags nothing, forever. Write a
  single `raw` entry with top-level alternation, as `AssistantOpener.yml` does.
- **Do not mix `tokens` and `raw` in one rule.** The rule stops matching entirely.
- **`raw` is not wrapped in `\b`.** Add the boundaries by hand or the pattern matches
  inside words.
- **Lookarounds are accepted and then ignored.** `(?<!test\s)harness\s+the` compiles
  without complaint and still matches "test harness the". You cannot carve out exceptions
  this way; narrow the pattern with real context instead, or let consumers use a Vale
  vocabulary (`accept.txt` entries are filtered out of every rule's matches).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JMill/deslop](https://github.com/JMill/deslop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
