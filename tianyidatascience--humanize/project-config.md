---
trigger: always_on
description: Use this skill when the user wants to generate or optimize Chinese communication copy so it sounds more human, more natural, less templated, and less like polished AI writing. 中文文案去 AI 味和人味优化 skill，适合自媒体文案、客户邮件、微信回复、售后沟通、面试跟进、上级汇报、产品宣传和社群通知。The user normally only needs to provide the task and constraints. If they also provide an original draft, the skill switches to rewrite mode automatically.
---


> Important: all `scripts/` paths are relative to this skill directory.
> Preferred entrypoint: `cd {this_skill_dir} && python3 humanize.py --text "{entire_user_request}" --output-root ./runs`
> Lower-level scripts still work via: `cd {this_skill_dir} && python3 scripts/...`
> The runtime and model cache live under `${COPAW_WORKING_DIR:-~/.copaw}/models/humanize/`.
> In agent shell calls, prefer one-line commands. Do not paste backslash-continued multi-line commands. Always set the shell tool timeout to at least `120` seconds for `python3 humanize.py`.
> 用户可见输出硬规则：如果 `python3 humanize.py ...` 打印 `=== HUMANIZE_FINAL_RESPONSE_BEGIN ===` / `=== HUMANIZE_FINAL_RESPONSE_END ===`，这两个标记中间的 markdown 就是最终答案。必须原样粘贴给用户，不能总结、不能改写、不能只给最终文案、不能写“已经帮你优化完成了”。用户要看的就是完整过程。
> Final relay rule: if `python3 humanize.py ...` prints `=== HUMANIZE_FINAL_RESPONSE_BEGIN ===` / `=== HUMANIZE_FINAL_RESPONSE_END ===`, return exactly the markdown between them as the final answer. Do not summarize it, do not paraphrase it, and do not add any explanation.
> Fallback relay rule: if the shell output is truncated or the final response block is not visible, open the latest `user-visible.md` in the run directory and return that markdown exactly.
> Invocation rule: do not build helper JSON or temporary Python snippets to call this skill. Invoke `python3 humanize.py` directly.
> Forbidden invocation: do not call `copaw skills run humanize`, `python -m skills.humanize...`, or any package-style wrapper. Do not pass `--mode`; rewrite mode is inferred automatically from the full `--text` request or `--original`. They are not the canonical entrypoint for this skill.
> Preservation rule: pass the user's full request verbatim via `--text` by default. If the request contains `原文`, `原稿`, `正文`, `draft`, or a long draft body, never reinterpret it into separate `--task` / `--constraints` arguments and never drop the original draft.
> Compatibility rule: this skill is not CoPaw-only. CoPaw / OpenClaw / Hermes style agents should use the same `SKILL.md + python3 humanize.py --text ...` flow. The CoPaw installer is only a convenience sync script for the currently known CoPaw workspace path, not a separate skill protocol. Claude Code and other local coding agents can invoke the same CLI after reading this `SKILL.md`.

# Humanize

## What This Skill Does

This skill is a practical AutoResearch-style loop for one narrow job:
optimize Chinese communication copy until it reads more like something a real
person would send.

The user normally only needs to define:

- `task`: what situation this message is for
- `constraints`: hard limits such as length, phrases to keep, or phrases to avoid

Optional:

- `original draft`: only when the user wants rewrite mode instead of generate mode

This skill then:

1. Bootstraps a local runtime and downloads the default local scorer model
2. Normalizes the user's input into a spec and session mode
3. Creates a run folder with the spec and drafts
4. Generates a baseline when the user did not provide one
5. In rewrite mode, adds `direct-rewrite` from the current main model to the candidate pool
6. Generates heuristic challenger drafts such as `heuristic-natural` and `heuristic-balanced`
7. Scores every candidate in one unified pool with the official local scorer
8. If the best candidate improves but fails the quality gate, repairs best-so-far with `direct-repair` plus heuristic repair candidates
9. Persists a small strategy state so the next run starts from the better policy bias
10. Records each round in JSON and renders a visible report so the process is inspectable

V2 product rule:

```text
humanize is not competing against the main model's direct rewrite.
It includes the main model's direct rewrite in the candidate pool,
then uses the local scorer + repair loop to choose the steadier version.
```

## First Run

Before the first evaluation, bootstrap the local runtime:

```bash
cd {this_skill_dir} && python3 scripts/bootstrap_runtime.py
```

This installs a dedicated venv under CoPaw's working directory and downloads
the default scorer model:

- `BAAI/bge-reranker-v2-m3`

## Inputs You Need From The User

Always collect or infer these before you start iterating:

- `task`
- `hard_constraints.min_chars` / `hard_constraints.max_chars` when length matters
- `hard_constraints.must_include` when facts must be preserved
- `hard_constraints.banned_phrases` for phrases the user dislikes

Optional:

- `original draft`

Default assumptions for V1:

- `goal`: built in, unless the user explicitly overrides it
- `max_rounds`: defaults to `3`, and stops early when the quality gate passes
- `style_notes`: infer from the task and constraints unless the user adds special tone requirements
- `session_mode`: `generate` unless the user provides an original draft

If the user does not explicitly give a spec file, create one in the run folder.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TianyiDataScience/humanize](https://github.com/TianyiDataScience/humanize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
