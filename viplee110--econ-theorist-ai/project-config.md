---
trigger: always_on
description: This folder contains an economic theory paper or research workflow targeting frontier-level development, originally calibrated against Econometrica-level standards.
---

# Project Instructions

This folder contains an economic theory paper or research workflow targeting frontier-level development, originally calibrated against Econometrica-level standards.

The system is target-calibrated rather than Econometrica-only. Econometrica-level means a high standard for theoretical clarity, novelty, rigor, and contribution discipline; it is not the only possible submission target. A confirmed `target_journal_profile.md` may calibrate the reader path, referee mix, exposition style, and fit standard for RAND, JET, Theoretical Economics, GEB, ReStud, AER, or another venue without lowering the quality floor.

## Language and Artifact Policy

- This is an English research system with multilingual command understanding.
- In chat, respond in the user's language when convenient.
- Keep research-facing outputs English by default: workflow artifacts, field profiles, target journal profiles, human decision logs, style calibration files, referee reports, theorem/proof/literature notes, revision logs, and manuscripts.
- Chinese is allowed only in chat interaction and a small routing trigger list. Do not write Chinese into research artifacts or manuscripts unless the user explicitly asks for a separate Chinese explanatory note outside the manuscript workflow.
- Ordinary Q&A remains ordinary Q&A: explanatory, translation, GitHub, software, or conceptual questions should not be over-routed into the research workflow.
- Research execution is serious by default. Casual wording must not downgrade a research idea; explicit quick requests may receive a compact screen, but the quality floor remains.
- The paper project is folder-portable. If the whole paper project folder is copied or synced to another computer, continue from the artifacts in that folder after running the local toolchain check on the new computer.

## Path and PDF Output Hygiene

- Never write a local filesystem path as raw prose. Wrap every local path, compiled PDF path, and command path in backticks or a fenced code block so Markdown cannot consume backslashes.
- Prefer display-safe paths such as `C:/Dropbox/Shufe/Research/Project/.../paper.pdf`, or exact Windows paths inside code spans such as `C:\Dropbox\Shufe\Research\Project\...\paper.pdf`. Do not write `C:\...` outside code.
- When reporting a generated or compiled PDF, first verify the real file with `Test-Path -LiteralPath` or `Resolve-Path -LiteralPath`, then report a separate `PDF path:` line in code formatting.
- When creating a PDF file card or Markdown file link, the visible title must be the filename only, such as `model_setup_basic_results.pdf`. Never use the absolute path as the visible card title or link label.
- Put the full absolute path on a separate `Full path:` line in code formatting. If using a Markdown link, use filename-only label plus resolved path target, for example `[model_setup_basic_results.pdf](<C:/Dropbox/Shufe/Research/Project/My Paper/model_setup_basic_results.pdf>)`.
- Construct paths with `Join-Path`, `Resolve-Path -LiteralPath`, `pathlib`, or equivalent path APIs. Do not concatenate directory strings in a way that can drop `\` or `/` separators.
- If a path contains spaces, brackets, or non-ASCII characters, quote it in shell commands and keep the path in code formatting. Use a Markdown link only after the path has been resolved, and put the link target in angle brackets when needed.

Default entry point: first read `ECONOMETRICA_ORCHESTRATOR.md` when it exists. Use it to route broad user requests to the correct workflow module and stage, so the user does not need to remember stage numbers or file names.

For pre-manuscript topic discovery, model generation, early derivation, and idea kill tests, first read `ECONOMETRICA_DISCOVERY_WORKFLOW.md` when it exists in this project.

For long-horizon human-AI manuscript development, first read `ECONOMETRICA_AI_HUMAN_WORKFLOW.md` when it exists in this project. Treat it as the stage-gated collaboration protocol for idea testing, contribution locking, manuscript revision, simulated review, and human decision checkpoints.

For mathematical derivations, theorem checks, comparative statics, equilibrium calculations, counterexample searches, numerical examples, symbolic algebra, or formal proof attempts, first read `ECONOMETRICA_VERIFICATION_WORKFLOW.md` when it exists in this project.

For version control, checkpoints, git diffs, branches, worktrees, commits, tags, rollback, and protection of human edits, first read `ECONOMETRICA_VERSION_CONTROL.md` when it exists in this project.

For high-stakes idea, model, theorem, manuscript, review, or revision decisions, first read `ECONOMETRICA_PANEL_PROTOCOL.md` when it exists. Use independent specialist reports, AE synthesis, Co-Editor decision, and parent-agent summary to avoid single-agent self-confirmation.

## Local Verification Toolchain

- Prefer the shared verification tool root outside paper folders. On Windows the default is `C:\Tools\CodexVerification`; users may override it with `CODEX_VERIFICATION_HOME`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [viplee110/econ-theorist-ai](https://github.com/viplee110/econ-theorist-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
