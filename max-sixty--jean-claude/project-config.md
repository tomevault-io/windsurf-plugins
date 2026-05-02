---
trigger: always_on
description: A Claude Code plugin for Gmail, Google Calendar, Google Drive, and iMessage.
---

# jean-claude Development

A Claude Code plugin for Gmail, Google Calendar, Google Drive, and iMessage.

Two parts: a **skill** that agents read, and a **library** that implements it.
The skill (`skills/jean-claude/SKILL.md`) documents commands and outputs. The
library (`jean_claude/`) handles the complexity — OAuth, APIs, AppleScript.

## Design Philosophy

The library exists to make the skill simple. Every design decision follows from
this: complexity belongs in the Python code, not in agent workflows.

When an agent reads email, it runs `jean-claude gmail list` and gets JSON. The
agent doesn't know about OAuth tokens, API pagination, rate limits, or retry
logic. The library handles all of that.

**When adding features, ask:** "Can the agent's job be simpler?" Push complexity
into the library. Handle edge cases in Python. Parse and validate in code.

## Two Audiences

- **Agents** read SKILL.md, call CLI commands, never touch Python code. If a
  feature isn't documented there, it doesn't exist for agents.
- **Developers** read this file, work on code in `jean_claude/`.

SKILL.md documents what the CLI does. CLAUDE.md documents how it works.

The skill is the only API. Breaking changes to the library are fine — just keep
the skill in sync.

## Writing Skill Content

**Skill files are instructions for agents, not documentation for humans.**

**Anonymize real contacts.** When improving skills based on real interactions
(email threads, message conversations), replace actual names and companies with
fictitious ones. The skill is public; real contacts are not.

Examples should remain **specific and concrete** — use "Google Research" not
"a major tech company", use "SF" not "a city". Vague placeholders make poor
examples. The goal is privacy, not genericness.

**Agents only receive SKILL.md initially.** When the plugin loads, Claude Code
reads `skills/jean-claude/SKILL.md` and nothing else. Any other files in the
skill directory (like `ONBOARDING.md`) must be explicitly referenced with a path
so the agent knows to read them:

```markdown
# In SKILL.md
If setup is needed, read the onboarding guide:
cat ${CLAUDE_PLUGIN_ROOT}/skills/jean-claude/ONBOARDING.md
```

When writing or editing files in `skills/jean-claude/`, remember:

- The **agent** reads the skill and runs commands
- The **user** sees natural conversation, not command output
- Commands in skill files are what the agent runs, not what users type

**Wrong framing:**
```markdown
After setup, run these commands to check your inbox:
- `jean-claude gmail inbox -n 10` - Recent emails
- `jean-claude gmail search "is:unread"` - Unread emails
```

This reads like user documentation. Users don't run commands.

**Correct framing:**
```markdown
After setup, offer to help with common tasks. Run the appropriate command
based on what the user wants:

- Check inbox: `jean-claude gmail inbox -n 10`
- Find unread: `jean-claude gmail search "is:unread"`

Present results conversationally, not as command output.
```

This tells the agent what to do and how to communicate with the user.

**Key principles:**

1. **Agent runs commands** — Never tell the user to run a command. The agent
   runs it and explains the result.
2. **User does manual steps** — Some things require user action (granting macOS
   permissions, scanning QR codes). Guide them through these conversationally.
3. **Present results, not output** — Don't show raw JSON. Summarize: "You have
   12 unread emails, 3 from today."
4. **Conversational tone** — The skill describes how the agent should talk to
   the user, not command syntax.

### Command Format in Skills

Use the short form `jean-claude ...` in skill files. The skill defines the
command prefix once at the top — agents prepend it automatically.

## Testing Is Real

**Every command you run sends real messages to real people.**

When developing messaging features (iMessage, WhatsApp, Gmail), "testing" still
means sending actual messages. There's no sandbox. Getting into a "just testing
the code" mindset is dangerous — test messages go to real recipients.

**Before running any send command, ask for permission:**

> **Test message I'd like to send:**
> - To: [recipient]
> - Body: "[message content]"
>
> Can I send this?

This applies even when:
- You're "just checking if the code works"
- The message seems trivial ("test", "hello")
- You're iterating on a bug fix
- The recipient is a group chat

Read-only commands (`list`, `search`, `messages`, `chats`) are safe to run
freely. Anything that sends, creates, or modifies requires explicit approval.

## Updating the Skill

When modifying CLI commands or adding features:

1. Update code in `jean_claude/`
2. Update `skills/jean-claude/SKILL.md` to match
3. Regenerate command reference: `uv run python scripts/generate-command-reference.py`

The skill contains hand-written context (safety rules, workflows, setup) plus
auto-generated command reference. The script creates files in
`skills/jean-claude/commands/` with hyphenated names mirroring the command
structure (`gmail-draft.txt` for `gmail draft`, etc.).

**Plugin cache vs repo:** For quick testing, you can edit files directly in
`~/.claude/plugins/cache/jean-claude/...` — changes take effect immediately.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [max-sixty/jean-claude](https://github.com/max-sixty/jean-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
