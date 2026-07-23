---
trigger: always_on
description: Use first-person singular ("I", "my") in all prose written on behalf of the user: issue descriptions, PR bodies, feature requests, comments, documentation. Never use first-person plural ("we", "our") unless the text genuinely refers to a group.
---

@~/.claude/tropes.md

## Voice and punctuation

Use first-person singular ("I", "my") in all prose written on behalf of the user: issue descriptions, PR bodies, feature requests, comments, documentation. Never use first-person plural ("we", "our") unless the text genuinely refers to a group.

Use ":" instead of em dashes for inline elaboration or appositive clauses.

## Code organization

Do not make autonomous decisions about module boundaries, file placement, or architectural structure. When intent is ambiguous, ask before reorganizing. The user has strong opinions about where code lives and how modules are scoped.

## Commits and PRs

Never run `git commit`, `git push`, `gh pr create`, or any other command that creates a commit, pushes to a remote, or opens a pull request unless I have explicitly authorized that specific action in the current conversation. Staging changes, drafting commit messages, and showing diffs are fine; the actual commit, push, or PR creation requires my explicit go-ahead each time. A prior authorization does not carry over to later actions.

Never include AI attribution in commits or PRs. No `Co-Authored-By` lines, no "Generated with Claude Code", no mention of being an AI or which model produced the code. Do not reference model names, versions, or codenames in commit messages, PR titles, or PR bodies.

Write commit messages as a human developer would — describe what the code change does and why, not how it was produced. Keep internal tooling references (specific tools, Slack channels, internal links) out of public-facing text.

Never write `#N` (a literal `#` followed by a number) in commit messages, PR titles, or PR bodies unless N is the actual number of a GitHub issue or pull request in the target repository. GitHub auto-links every `#N` token to issue/PR N, so positional references like `test #1` or `tests #14 and #15` render as misleading cross-references to unrelated tickets. Use plain numbers (`test 1`, `tests 14 and 15`), backtick-quote the identifier when it names a slot in a test plan or list (`` test `1` ``, `` item `14` ``), or rephrase (`the first test`, `the fourteenth case`).

## Shell commands

Never use `$()` command substitutions inside `gh` (or any other) Bash calls. The sandbox flags `$()` as a separate security check that fires regardless of permission allow rules — it can't statically verify what executes inside a substitution. Instead, run compound commands as separate sequential Bash calls: capture the inner result first, then use it in the next call. Both commands then match the allow rules individually and auto-approve.

## Code generation preferences

For any non-trivial workflow, data processing, or multi-step logic: write Python, not Bash. The user is an advanced Python developer and can quickly read, inspect, and validate Python code. Short one-liners and simple Bash scripts are fine for convenience and performance, but anything with branching logic, string manipulation, data transformation, or error handling should be Python.

## Data visualization

When producing matplotlib figures, follow the design system at https://github.com/temataro/better-graphs for readable, presentation-ready plots: it codifies Tufte-style design rules, a chart-selection guide, and a `house_style.py` styling module (`apply_theme()`, `polish()`, `takeaway_title()`) that replaces matplotlib defaults with accent-led palettes, trimmed spines, unit-aware ticks, and takeaway-focused titles.

## Terminology and spelling

Use correct capitalization for proper nouns and trademarked names:

<!-- typos:off -->

- **PyPI** (not ~~PyPi~~): the Python Package Index. The "I" is capitalized because it stands for "Index". See [PyPI trademark guidelines](https://pypi.org/trademarks/).
- **GitHub** (not ~~Github~~)
- **GitHub Actions** (not ~~Github Actions~~ or ~~GitHub actions~~)
- **JavaScript** (not ~~Javascript~~)
- **TypeScript** (not ~~Typescript~~)
- **macOS** (not ~~MacOS~~ or ~~macos~~)
- **iOS** (not ~~IOS~~ or ~~ios~~)

<!-- typos:on -->

## Version formatting

The version string is always bare (like `1.2.3`). The `v` prefix is a **tag namespace**: it only appears when the reference is to a git tag or something derived from a tag (action ref, comparison URL, commit message). This aligns with PEP 440, PyPI, and semver conventions.

Rules:

1. **No `v` prefix on package versions.** Anywhere the version identifies the *package* (PyPI, changelog heading, CLI output, `pyproject.toml`), use the bare version: `1.2.3`.
2. **`v` prefix on tag references.** Anywhere the version identifies a *git tag* (comparison URLs, action refs, commit messages, PR titles), use `v1.2.3`.
3. **Always backtick-escape versions in prose.** Both `v1.2.3` (tag) and `1.2.3` (package) are identifiers, not natural language. Wrap them in single backticks: `` `v1.2.3` ``, `` `1.2.3` ``.
4. **Development versions** follow PEP 440: `1.2.3.dev0` with optional `+{short_sha}` local identifier.

## File naming conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdeldycke/dotfiles](https://github.com/kdeldycke/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
