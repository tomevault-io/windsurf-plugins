---
trigger: always_on
description: Create new GitHub Copilot custom instructions, prompt files, and agent skills. Helps with creating reusable customizations from scratch, optimizing existing files, testing effectiveness, and benchmarking performance.
---


# Copilot Prompt Creator

A skill for creating new GitHub Copilot customizations and iteratively improving them.

At a high level, the process of creating a custom instruction goes like this:

- Decide what you want the customization to do and roughly how it should do it
- Write a draft of the file (`.prompt.md`, `.instructions.md`, or `SKILL.md`)
- Create a few test scenarios and run them manually in Copilot Chat using `/promptName` (for prompt files and skills) or by working with matching files (for instructions)
- Help the user evaluate the results both qualitatively and quantitatively
  - While the user tests, draft some quantitative assertions if there aren't any. Then explain them to the user (or if they already existed, explain the ones that already exist)
  - Use the `eval-viewer/generate_review.py` script to show the user the results for them to look at, and also let them look at the quantitative metrics
- Rewrite the customization based on feedback from the user's evaluation of the results (and also if there are any glaring flaws apparent from the quantitative benchmarks)
- Repeat until you're satisfied
- Expand the test set and try again at larger scale

Your job when using this skill is to figure out where the user is in the process and then jump in and help them progress through these stages. For instance, maybe they say "I want to make a prompt for X". You can help narrow down what they mean, write a draft, write test cases, figure out how they want to evaluate, run the prompts, and repeat.

On the other hand, maybe they already have a draft of the prompt. In this case you can go straight to the eval/iterate part of the loop.

Of course, you should always be flexible -- if the user says "I don't need to run a bunch of evaluations, just vibe with me", you can do that instead.

## Communicating with the user

The prompt creator is liable to be used by people across a wide range of familiarity with coding jargon. There's a trend now where AI tools are inspiring plumbers to open up their terminals, parents and grandparents to learn new tools. On the other hand, the bulk of users are probably fairly computer-literate.

So please pay attention to context cues to understand how to phrase your communication! In the default case, just to give you some idea:

- "evaluation" and "benchmark" are borderline, but OK
- for "JSON" and "assertion" you want to see serious cues from the user that they know what those things are before using them without explaining them

It's OK to briefly explain terms if you're in doubt, and feel free to clarify terms with a short definition if you're unsure if the user will get it.

---

## GitHub Copilot Customization Overview

Before diving into the creation process, here's how AI customization works in GitHub Copilot. There are several types, each serving a different purpose:

### Types of Customizations

1. **Always-on instructions** (`.github/copilot-instructions.md`):
   - Automatically included in ALL Copilot interactions within the repository
   - Good for project-wide conventions (coding style, architecture patterns, tooling preferences)
   - Single file per repository. Also supports `AGENTS.md` (cross-agent) and `CLAUDE.md` (Claude-compatible)

2. **File-based instructions** (`.github/instructions/*.instructions.md`):
   - Applied dynamically based on file patterns (`applyTo` glob) or description matching
   - Good for language-specific conventions, framework patterns, or rules for certain parts of your codebase
   - Each file targets specific file types or paths

3. **Prompt files / Slash commands** (`.github/prompts/*.prompt.md`):
   - Invoked manually with `/promptName` in Copilot Chat
   - Good for repeatable tasks like scaffolding components, running tests, preparing PRs
   - Support different agents: `agent`, `ask`, `plan`, or custom agents
   - Can reference workspace files using Markdown links
   - This is the primary format we create in this workflow

4. **Agent Skills** (`.github/skills/<skill-name>/SKILL.md`):
   - Folders containing instructions, scripts, examples, and resources
   - Loaded on-demand based on task relevance, also available as `/skillName` slash commands
   - Open standard ([agentskills.io](https://agentskills.io/)) -- works across VS Code, Copilot CLI, and Copilot coding agent
   - Good for specialized workflows with supporting scripts and resources

5. **Custom agents** (agent definition files):
   - Specialized AI personas with specific tools and capabilities
   - Can orchestrate sub-agents for complex workflows

### Prompt File Format (.prompt.md)

```markdown
---
description: Brief description of what this prompt does
agent: agent
tools:
  - codebase
  - terminal
  - github
---

# Prompt Title

Instructions go here...

Reference workspace files with Markdown links:
See [coding standards](../../coding-standards.md) for conventions.
```

### Prompt File Frontmatter Fields

- **`description`** (optional): Human-readable description shown when browsing with `/` in chat
- **`name`** (optional): Display name for the prompt. Defaults to the filename.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sarjangi/skill-creator](https://github.com/sarjangi/skill-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
