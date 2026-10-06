---
trigger: always_on
description: - Use ASCII characters only in all written output, code comments, commit messages, and documentation. Forbidden characters include em/en dashes (— –), curly quotes (“ ” ‘ ’), ellipses (…), or other non-ASCII Unicode. Prefer plain ASCII equivalents at all times. The only exceptions to this rule are box-drawing characters (├, └, │, ─) in tree diagrams.
---

# AGENTS.md

## Working agreements

### Output

- Use ASCII characters only in all written output, code comments, commit messages, and documentation. Forbidden characters include em/en dashes (— –), curly quotes (“ ” ‘ ’), ellipses (…), or other non-ASCII Unicode. Prefer plain ASCII equivalents at all times. The only exceptions to this rule are box-drawing characters (├, └, │, ─) in tree diagrams.
- Minimize use of semicolons.
- Default to pragmatic, concise communication.
- Keep edits minimal and well-scoped; explain changes.
- Prefer local context; use the web only when needed.
- Make safe assumptions; ask only when necessary.

### Filesystem

- When deleting files or directories, prefer `trash` as a safer default instead of `rm -rf`; use permanent deletion only with explicit user approval.
- When working with temporary files, do not use `/private`. Use `/tmp` or the directory given by `$TMPDIR` instead.

### Shell commands

- Run commands directly when possible so approval rules can match the executable and its arguments.
- Do not add explicit shell wrappers such as `/bin/zsh -lc`, `zsh -lc`, or equivalents unless the command specifically requires login-shell initialization.
- When shell syntax is necessary, use the execution tool's existing shell instead of starting a nested shell.

### Git worktrees

- Codex-managed worktrees must use the default location `~/.codex/worktrees` and must not be created in `.worktrees`.
- ChatGPT and Codex must use the ChatGPT desktop app Worktree or Handoff controls for task isolation.

### Git commits

- Do not create Git commits on `main`; always work on a feature branch.
- Do not Git push to `main`; always push to a feature branch.
- Format Git commit messages in the following style:

  ```text
  Imperative commit title limited to 50 characters

  Format commit messages as if they were Markdown.

  Begin by describing how the code works now and why a change is needed.
  The commit message body can be detailed. Full paragraphs are acceptable.
  Lines in commit message paragraphs should be limited to 72 characters.

  Summarize changes by saying "This commit will" and using the imperative.

  - The end of the commit message should have a list of references.
  - Add an unordered list item for each URL.
  - Do not hard wrap URLs. URLs can exceed 72 characters if needed.
  ```

### GitHub pull requests

- Always open GitHub pull requests in draft mode.
- Each time a new commit is pushed to a pull request branch, check the pull request title and description and update them if needed to match the current state of the pull request.
- Format GitHub pull request titles and descriptions in the following style:
  - Limit the PR title to around 50 characters so it fits into a squash commit title.
  - Write for a reviewer who has not read the conversation. Describe the final proposal and keep detail proportional to the change. Omit the implementation diary and abandoned approaches unless they explain a relevant tradeoff.
  - Use plain, direct sentences. Minimize semicolons. Split sentences that join separate ideas or stack qualifications.
  - Include a concise PR description with these sections:
    - `## Description`: explain the current behavior, problem, and reason for the change. Default to one short paragraph. Keep the proposed solution, implementation details, and resulting behavior in `## Changes`.
    - `## Changes`: explain the proposed approach in a short opening paragraph when useful, then summarize changes using imperative bullets prefaced with "This PR will:" (e.g. "Fix incorrect styling"). Group related changes by purpose and explain each change once. Add `Validation:` and `Notes:` within this section only when useful.
    - `## Related`: unordered list of links to related resources. Do not link the PR to itself.
  - Summarize validation with the relevant checks, outcomes, and material gaps. Retain meaningful failures, skipped checks, and remaining verification. Summarize adversarial review outcomes without recounting each resolved finding. Include unresolved risks and waiver rationale when they affect the review decision. Place terminal output/log snippets in fenced code blocks inside HTML `<details><summary>...</summary> ... </details>` sections.
  - State qualifications as concrete facts, limitations, or required actions that affect review or rollout. Preserve material uncertainty and migration requirements. Omit vague disclaimers and lists of things the PR does not do. For example, write "Configure the production domain in Vercel before sharing the demo" when that action is required, rather than "These are supported address formats, not a claim that we have added these domains to our projects."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [br3ndonland/dotfiles](https://github.com/br3ndonland/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
