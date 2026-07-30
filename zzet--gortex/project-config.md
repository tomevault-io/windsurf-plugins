---
trigger: always_on
description: `gortex install` (once per machine) and `gortex init` (once per repo)
---

# Agent Integrations

`gortex install` (once per machine) and `gortex init` (once per repo)
auto-configure Gortex for every AI coding assistant detected on your
machine. Eighteen adapters ship today.

- `gortex install` writes user-level machinery: `~/.claude.json` MCP,
  `~/.claude/skills/gortex-*`, `~/.claude/commands/gortex-*.md`,
  `~/.gemini/antigravity/` Knowledge Items, and user-level hooks.
- `gortex init` writes per-repo machinery: `.mcp.json`, per-agent
  MCP configs (`.cursor/mcp.json`, `.vscode/mcp.json`, …), repo-local
  hooks where supported, per-agent marker-guarded community-routing
  blocks, and `.claude/skills/generated/` per-community SKILL.md.

Run `gortex init doctor` to see what's currently configured. Both
commands accept `--agents=<csv>` to constrain setup and
`--agents-skip=<csv>` to exclude an adapter.

## Adapter matrix

| Name            | What gets written                                                                               | Mode       | Docs link                                                           |
| --------------- | ----------------------------------------------------------------------------------------------- | ---------- | ------------------------------------------------------------------- |
| `claude-code`   | `.mcp.json`, `.claude/*`, `CLAUDE.md`, `.claude/skills/generated/*`, `~/.claude/skills/gortex-*`, `~/.claude/commands/gortex-*.md`, `~/.claude.json` | both       | https://docs.claude.com/en/docs/claude-code/overview                |
| `aider`         | `.aiderignore` block, `CONVENTIONS.md` communities block                                        | project    | https://aider.chat/docs/config/aider_conf.html                      |
| `antigravity`   | `~/.gemini/antigravity/mcp_config.json` + Knowledge Item                                        | user       | https://antigravity.google/docs/mcp                                 |
| `cline`         | `cline_mcp_settings.json` (per VS Code / Cursor globalStorage), `.clinerules/gortex-communities.md` | both     | https://docs.cline.bot/mcp/mcp-overview                             |
| `codex`         | `~/.codex/config.toml` (`[mcp_servers.gortex]` + `SessionStart`, `UserPromptSubmit`, Bash/Gortex-read `PreToolUse`, and Bash/`apply_patch` `PostToolUse` hooks), `AGENTS.md` communities block | both       | https://developers.openai.com/codex/mcp                             |
| `continue`      | `.continue/mcpServers/gortex.json`, `.continue/rules/gortex-communities.md`                     | project    | https://docs.continue.dev/customize/deep-dives/mcp                  |
| `cursor`        | `.cursor/mcp.json` (project) or `~/.cursor/mcp.json`, `.cursor/rules/gortex-communities.mdc`    | both       | https://docs.cursor.com/en/context/mcp                              |
| `gemini`        | `.gemini/settings.json` or `~/.gemini/settings.json`, `GEMINI.md` communities block             | both       | https://geminicli.com/docs/tools/mcp-server/                        |
| `hermes`        | `~/.hermes/config.yaml` + `profiles/*/config.yaml` (`mcp_servers`), hooks, `~/.hermes/skills/gortex/SKILL.md` | user       | https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp  |
| `kilocode`      | `mcp_settings.json` + `.kilocode/mcp.json`, `.kilocoderules` communities block                  | both       | https://kilo.ai/docs/features/mcp/using-mcp-in-kilo-code            |
| `kimi`          | `.kimi-code/mcp.json` (project) or `~/.kimi-code/mcp.json` + `~/.kimi-code/config.toml` (`UserPromptSubmit` / `PreToolUse` / `Stop` / `SubagentStart` hooks) | both       | https://www.kimi.com/code/docs/en/kimi-code-cli/customization/hooks.html |
| `kiro`          | `.kiro/settings/mcp.json` + steering/hooks or user-level                                        | both       | https://kiro.dev/docs/mcp/configuration                             |
| `oh-my-pi`      | `.omp/mcp.json`                                                                                 | project    | https://github.com/can1357/oh-my-pi/blob/main/docs/mcp-config.md   |
| `opencode`      | `opencode.json` (or existing `opencode.jsonc`), `AGENTS.md` communities block                   | project    | https://opencode.ai/docs/mcp                                        |
| `openclaw`      | `~/.openclaw/openclaw.json` (`mcp.servers.gortex`)                                              | user       | https://docs.openclaw.ai/cli/mcp                                    |
| `pi`            | `.pi/extensions/gortex/index.ts` (project) or `~/.pi/agent/extensions/gortex/index.ts`; `AGENTS.md` communities block only when `--skills` | both | https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md |
| `vscode`        | `.vscode/mcp.json` (`servers` key, 1.102+), `.github/copilot-instructions.md` communities block | project    | https://code.visualstudio.com/docs/copilot/chat/mcp-servers         |
| `windsurf`      | `~/.codeium/mcp_config.json`, `.windsurfrules` communities block                                | both       | https://docs.windsurf.com/plugins/cascade/mcp                       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zzet/gortex](https://github.com/zzet/gortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
