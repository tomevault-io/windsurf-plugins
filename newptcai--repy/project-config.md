---
trigger: always_on
description: Always request user approval before running any `julia` command so the harness can escalate out of the restrictive sandbox.
---

# Julia Invocation Approval

Always request user approval before running any `julia` command so the harness can escalate out of the restrictive sandbox.

# Git Repository Guidelines

## Commit Message Guidelines (Codex CLI)
- Use GitHub emoji style subjects in imperative voice, ≤72 chars.
  - Format: `EMOJI (scope): Short imperative subject`.
  - Scope: usually a script name (kebab-case) or area like `img`, `audio`, `net`.
  - Examples: `✨ (mdview.sh): Add GUI preview via xdg-open`, `🐛 (img-trim.sh): Fix threshold for white borders`.
  - Common emojis:
    - ✨: feature/new option
    - 🐛: bug fix
    - 📝: docs
    - ♻️: refactor
    - 🎨: style/formatting
    - ⚡: performance
    - ✅: tests
    - 🔧: config/chore
    - 🚚: move/rename
    - 🔥: remove code
- Keep subjects focused and group related script updates in one commit to avoid unrelated churn.
- Bodies: never embed literal "\n"; use multiple `-m` flags (each becomes a paragraph) or a here-doc to build multi-line messages. Prefer bullets or short paragraphs instead of inlined `\n` escape sequences.
- Safe patterns:
  - `git commit -m "✨ (tool): Add feature" -m "- First bullet" -m "- Second bullet"`
  - `git commit -F - <<'MSG'
    ✨ (tool): Add feature

    - First bullet
    - Second bullet
    MSG`
- Amending safely: `git commit --amend -m "SUBJECT" -m "Bullet 1" -m "Bullet 2"`.

## Pull Request Guidelines
- Include purpose, sample commands, expected/actual behavior, and any external tool requirements (e.g., `ffmpeg`, `ImageMagick`, `pdftk`). Add before/after snippets or file counts when relevant.

Example
```
✨ (vim): Add lexima rules for TeX

- Add $, \(\), and \[\ \\] pairing rules
- Guard Markdown vimtex init behind exists() check
```

## Gemini Added Memories
- For true or false problems, consult `templates/sample-true-false-problem.md` for the correct coding pattern (specifically using `statement_pool` and printing prompts directly).
- Prefer using `\bfq`, `\bfv`, etc., over `\mathbf{q}`, `\mathbf{v}` for bold vectors in LaTeX/Quarto files.
- In each subproblem of a QMD file, there should be exactly one `long_answer` or `short_answer` call.
- A problem's title in a QMD file should match the name of the section it belongs to.
- When using `short_answer` and "no prompt" is desired, provide only the solution LaTeXString. This string will then serve as the label for the answer box.
- When a string does not contain math expressions (in $...$) use "..." instead of L"..."
- New problem files should include "-v1" in their filenames (e.g., "ex-19-v1.qmd").
- When turning a string `s` into a `LaTeXString`, use `LaTeXString(s)` instead of `latexstring(s)`.
- Always write \mathcal{B} in subscript as _{\mathcal{B}}.

## Development Guidelines
- AGENTS.md, GEMINI.md, and CLAUDE.md are the same file; changing one updates the other two automatically.
- `ROADMAP.md` is the single progress document: feature status, remaining work, and deferred hard problems live there. Small, insulated tasks are recorded as `improvements/improvement-XX.md` files (see "Codex Improvement Tasks" below). Do not create additional to-do/status files.
- When changing shortcuts or keybindings, always update the in-app help window and README.md in the same change.
- Commit frequently with small, focused changes.
- Test-driven development for complex components.
- Cross-platform mindset (Linux/macOS/Windows).
- Preserve epy behavior while improving performance.
- Always update `Cargo.lock` with `cargo update` before tagging a release to avoid CI failures.

## Codex Improvement Tasks

Small, insulated improvement tasks live in `improvements/improvement-XX.md`.
When asked to implement one of these tasks (typically as Codex running in
full-auto mode), follow these rules:

- Implement exactly one task per run. Read the whole task file first; stay
  inside its stated scope and "Out of scope" limits.
- You own the full slice: implementation, tests, docs. Do not leave TODOs or
  half-wired code paths.
- Before committing, all of these must pass:
  - `cargo fmt --all` (then check nothing is left dirty by formatting)
  - `cargo clippy --all-targets` with no new warnings
  - `cargo test`
- Snapshot tests: never run `cargo insta review` (interactive). For
  intentional snapshot changes or new snapshots, run tests then
  `cargo insta accept`, and eyeball the accepted `.snap` diffs yourself.
- Keybinding or UI changes must update the in-app help window and README.md in
  the same commit (project rule above).
- When the task is complete and green, change the task file's `Status:` line
  from `todo` to `done` and commit everything per the commit message
  guidelines in this file (one focused commit is fine; two if code + docs
  separation is clearly cleaner).
- Do not bump the version or tag releases as part of an improvement task.

## Release Process
1. Bump version in `Cargo.toml` (minor bump: 0.8.x → 0.9.0; patch bump: 0.8.x → 0.8.(x+1)).
2. Run `cargo update` to refresh `Cargo.lock`.
3. Commit: `🔧 (repy): Bump version to X.Y.Z`
4. Tag: `git tag vX.Y.Z && git push --tags`


## Success Metrics
- Feature parity with epy
- Performance improvement (per-chapter caching implemented)
- Memory efficiency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [newptcai/repy](https://github.com/newptcai/repy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
