---
trigger: always_on
description: <!-- AUTOPUS:BEGIN -->
---

<!-- AUTOPUS:BEGIN -->
# Autopus-ADK Harness

> 이 섹션은 Autopus-ADK에 의해 자동 생성됩니다. 수동으로 편집하지 마세요.

- **프로젝트**: autopus-adk
- **모드**: full
- **플랫폼**: claude-code, codex, antigravity-cli, opencode, omp

## Installed Components

- Claude Skills: .claude/skills/<name>/SKILL.md
- Claude Agents: .claude/agents/autopus/
- Claude Rules: .claude/rules/autopus/
- Claude Hooks: .claude/hooks/autopus/
- Claude Settings: .claude/settings.json
- Claude Statusline: .claude/statusline.sh
- Claude Root Doc: CLAUDE.md
- Codex Native Skills: .codex/skills/codex-<name>/SKILL.md
- Codex Agents: .codex/agents/
- Codex Hook Scripts: .codex/hooks/
- Codex Hook Registry: .codex/hooks.json
- Codex Config: .codex/config.toml
- Codex Plugin Router: @auto ... / $codex-auto-<route>
- Gemini Skills: .gemini/skills/autopus/<name>/SKILL.md
- Gemini Agents: .gemini/agents/autopus/
- Gemini Rules: .gemini/rules/autopus/
- Gemini Commands: .gemini/commands/
- Gemini Hooks: .gemini/hooks/
- Gemini Settings: .gemini/settings.json
- Gemini Statusline: .gemini/statusline.sh
- Gemini Hook Registry: .agents/hooks.json
- Gemini Root Doc: GEMINI.md
- Gemini Plugin Bundle: .agents/plugins/autopus/
- Shared Commands: .agents/commands/
- OpenCode Rules: .opencode/rules/autopus/
- OpenCode Commands: .opencode/commands/
- OpenCode Agents: .opencode/agents/
- OpenCode Plugins: .opencode/plugins/
- OpenCode-owned Shared Skills: .agents/skills/
- OMP Skills: .omp/skills/
- OMP Agents: .omp/agents/
- OMP Rules: .omp/rules/
- OMP Commands: .omp/commands/
- Plugin Marketplace: .agents/plugins/marketplace.json


## Language Policy

These settings are prompt instructions for every agent in this project. They are
not mechanically enforced: no hook, linter, or CI gate inspects the language of
comments, commit messages, or responses. The pre-commit Lore check validates the
commit type prefix and sign-off trailers only, never the language. Treat a
violation as a review finding, not as something a gate will catch.

- **Code comments**: en
- **Commit messages**: ko
- **AI responses**: ko

## Autopus Branding

The canonical banner header is the first line below. Start a `/auto` or `@auto`
response with it and end the completed response with `🐙`.

```text
🐙 Autopus ─────────────────────────
  프로젝트: {project-name} | 모드: {mode}
  SPEC: {draft}개 draft · {approved}개 approved · {implemented}개 구현중 · {completed}개 완료
  다음: {next-step recommendation}
```

Subagent completion summaries use the A3 Agent Result shape:

```text
🐙 {agent-name} ─────────────────────
  {key metric 1} | {key metric 2} | {key metric 3}
  다음: {next step guidance}
```

General responses that applied no harness rule carry no banner, footer, or emoji.

## Document Storage

IMPORTANT: A document stored in the wrong place causes sync failures and version
control gaps.

| Document Type | Location | Git Repo |
|---------------|----------|----------|
| Project context | Root | meta repo |
| Harness bootstrap config | Root | meta repo |
| Generated harness/runtime surface | Local working copy only | Do not commit |
| Cross-module SPEC | Root `.autopus/specs/` | meta repo |
| Module-specific SPEC | `{module}/.autopus/specs/` | module repo |
| Brainstorm/runtime output | Local working copy only | Do not commit |
| Module CHANGELOG | `{module}/CHANGELOG.md` | module repo |

Module detection: match the referenced `pkg/`, `cmd/`, `internal/`, `src/`, or
`app/` paths to their owning submodule. Paths spanning 2+ modules are
cross-module and belong at the root.

SPEC and BS IDs MUST be globally unique across the workspace. Scan both
`.autopus/specs/SPEC-*` and `*/.autopus/specs/SPEC-*` (and the matching
`brainstorms/BS-*` pair) before allocating an ID; a collision is a hard error.

Run `auto sync verify` before committing. It is read-only and prints the
topology it detected. In a single repository holding `autopus.yaml` it
partitions every dirty path into a commit candidate, a blocked
generated/runtime path, or an unclassified path. In a multi-repo workspace the
same partition splits candidates into Phase A (module) and Phase B (meta)
commits. Where neither layout applies it stops with a dedicated
`unsupported topology:` diagnostic instead of a classification result.
`auto check --hygiene --staged` is not an equivalent substitute: it checks
generated/runtime hygiene only and never partitions the full dirty path set.

## Execution Model

- **Codex V2**: multi_agent_v2 uses spawn_agent, send_message, followup_task, wait_agent, interrupt_agent, list_agents.
- **Codex Concurrency**: spawn 되는 worker 상한은 `autopus.yaml`의 `codex.agents.max_concurrent_threads`이며(기본 4), coordinator thread는 세지 않습니다. 요청한 값은 요청일 뿐입니다. provider/account/host 한도가 우선하고, 변경은 새 세션부터 적용되며, config 파일 쓰기 성공은 확보된 capacity의 근거가 아닙니다. `auto doctor`의 `doctor.codex.agents.concurrency`에서 requested/loaded/effective를 확인하세요. 무거운 로컬 test/build 병렬도는 이 값이 관리하지 않습니다.
- **Codex Workspace**: 모든 agent는 shared cwd/filesystem을 사용합니다. fork_turns는 대화 context만 분기하며 filesystem isolation이나 merge를 제공하지 않습니다.
- **Codex Router**: @auto <route> ...는 auto plugin을 호출하고 상세 workflow는 $codex-auto-<route> skill로 라우팅합니다.
- **Codex /goal**: Codex goals feature를 사용합니다. @auto goal은 이 기능의 thin wrapper이며, active goal이 있으면 get_goal로 목표를 반영하고 create_goal/update_goal은 Codex goal tool contract를 만족할 때만 사용하세요.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autopus-ai/autopus-adk](https://github.com/autopus-ai/autopus-adk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
