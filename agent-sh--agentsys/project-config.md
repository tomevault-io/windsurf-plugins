---
trigger: always_on
description: > **Cross-tool compatible**: Works across all 5 platforms - Claude Code, OpenCode, Codex CLI, Cursor, and Kiro.
---

# Project Memory: AgentSys

> **Cross-tool compatible**: Works across all 5 platforms - Claude Code, OpenCode, Codex CLI, Cursor, and Kiro.

<project-memory>

<critical-rules>
## Critical Rules (Priority Order)

1. **Production project** - Real users depend on this. Test thoroughly and verify all edge cases before committing.
   *WHY: Breaking changes affect all plugin users immediately.*

2. **Plugin for OTHER projects** - Optimize for plugin users, not internal dev convenience.
   *WHY: Every decision MUST improve the experience for developers using this in their repos.*

3. **Use CHANGELOG.md for completion tracking** - MUST use CHANGELOG.md for release notes. NEVER create `*_AUDIT.md`, `*_SUMMARY.md`, `*_COMPLETION.md` files.
   *WHY: Summary files clutter repos and add no value. Report completion verbally.*

4. **Unless** it is a very small change of max few lines, or an urgent hotfix, **MUST create PRs for all changes** - No direct pushes to main.
   *WHY: PRs enable reviews, CI checks, and rollback if needed. Direct pushes are risky.*

5. **PR reviews** - Wait 3 min for auto-reviewers, address ALL comments (Copilot, Claude, Gemini, Codex).
   *WHY: Skipping comments leads to merged issues. Every comment must be addressed or explained.*

6. **Read checklists BEFORE multi-file changes** - **MUST** read the relevant checklist before starting:
   - **Cross-platform work → `checklists/cross-platform-compatibility.md`** (MASTER REF)
   - Release → `checklists/release.md`
   - New command → `checklists/new-command.md`
   - New agent → `checklists/new-agent.md`
   - New skill → `checklists/new-skill.md`
   - New lib module → `checklists/new-lib-module.md`
   - OpenCode plugin update → `checklists/update-opencode-plugin.md`
   - Repo intel changes → `checklists/repo-intel.md`
   *WHY: Multi-file changes have hidden dependencies. Checklists prevent missed updates.*

7. **Especially Before release, and when delivering ANY work** - Check the relevant checklist for completion requirements:
   - Identify which checklist applies to your work (see rule #6)
   - Go through EVERY item in that checklist and apply it (e.g. run commands, which files to bump versions in, etc)
   - Run the `/enhance` command on new or modified commands, agents, skills, hooks or prompts
   - Verify cross-platform compatibility (OpenCode + Codex)
   - Update `bin/cli.js` mappings if new command/agent added
   - Only mark complete after ALL checklist items are done
   *WHY: Checklists exist because we kept missing things. They are the definition of "done".*

8. **Use plain text markers** - MUST use `[OK]`, `[ERROR]`, `[WARN]`, `[CRITICAL]` for status. NEVER use emojis or ASCII art boxes.
   - Save tokens - conciseness and clarity over decorations
   - Use markdown formatting instead of decorative borders
   *WHY: Emojis and ASCII art waste tokens. AI models parse plain text better.*

9. **gh/git on Windows** - Escape `$` as `\$` in GraphQL queries, avoid `!=` in jq (use `== "A" or == "B"`).
   - `gh pr checks` uses `state` not `conclusion` - returns UPPERCASE: `SUCCESS`, `FAILURE`, `PENDING`
   - Single quotes unreliable - use double quotes with escaped inner quotes
   *WHY: Windows shell interprets $ and ! differently. These cause silent failures.*

10. **Always run git hooks** - Run all pre-commit and pre-push hooks. If a hook blocks, fix the reported issue.
   - Hooks catch problems before they reach the repo
   - Fix the root cause, then retry
   *WHY: Hooks are safety nets. Bypassing them defeats their purpose.*

11. **Fix all test failures** - NEVER skip or ignore a failing test because it's "out of scope" or "pre-existing". Always fix it.
   - If a test fails during your work, fix it before proceeding
   - No test is someone else's problem
   *WHY: Skipping failures erodes test trust. Every green run must mean everything works.*

12. **Report script failures before manual fallback** - When any project script fails (npm test/run/build, scripts/*, agentsys-dev, node bin/dev-cli.js), you MUST:
   - Report the failure with exact error output to the user
   - Diagnose the root cause of the failure
   - Fix the script/tooling issue, not work around it manually
   - NEVER silently fall back to doing the work by hand
   *WHY: Silent fallbacks mask broken tooling. A failed script needs fixing, not bypassing.*
</critical-rules>

<architecture>
## Architecture

<!-- GEN:START:claude-architecture -->
```
lib/          → Shared library (vendored to plugins)
plugins/      → 0 plugins, 10 agents (0 file-based + 10 role-based), 0 skills
adapters/     → Platform adapters (opencode-plugin/, opencode/, codex/)
checklists/   → Action checklists (9 files)
bin/cli.js    → npm CLI installer
```

| Plugin | Agents | Skills | Purpose |
|--------|--------|--------|---------|
<!-- GEN:END:claude-architecture -->

**Pattern**: `Command → Agent → Skill` (orchestration → invocation → implementation)
</architecture>

<commands>
## Commands

### Core Workflow
- `/next-task` - Master workflow: task → implementation → PR → merge
- `/prepare-delivery` - Pre-ship quality gates: deslop, review, validation, docs sync
- `/gate-and-ship` - Quality gates then ship (/prepare-delivery + /ship)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-sh/agentsys](https://github.com/agent-sh/agentsys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
