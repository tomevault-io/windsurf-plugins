---
trigger: always_on
description: This is a "learn Claude Code by doing" repository. Users clone it and pick one of 5 options to master every major Claude Code feature through 10 progressive modules.
---

# cc-self-train

This is a "learn Claude Code by doing" repository. Users clone it and pick one of 5 options to master every major Claude Code feature through 10 progressive modules.

## Repository Structure

```
cc-self-train/
├── README.md                    # Top-level overview, quick start, feature matrix
├── CLAUDE.md                    # This file — project conventions for Claude
├── .claude/
│   ├── skills/start/SKILL.md    # /start onboarding skill — the entry point
│   ├── scripts/welcome.sh       # SessionStart hook — prints welcome banner
│   ├── scripts/check-updates.js # SessionStart hook — checks for newer CC versions
│   └── settings.json            # Hook configuration (welcome + update checker)
├── context/                     # Reference docs — `ls context/` to discover all files
│   ├── claudemd.txt             # CLAUDE.md hierarchy, @imports, rules
│   ├── skillsmd.txt             # Skills system (SKILL.md, frontmatter)
│   ├── hooks.txt                # Hook lifecycle, events, scripting
│   ├── interactive-mode.txt     # Keyboard shortcuts, vim mode
│   └── ...                      # 15 more files (mcp, subagents, tasks, plugins, etc.)
├── projects/
│   ├── canvas/
│   │   ├── README.md            # Project overview, setup, module list
│   │   └── modules/             # Individual module guides (01 through 10)
│   ├── forge/
│   │   ├── README.md
│   │   └── modules/
│   ├── nexus/
│   │   ├── README.md
│   │   └── modules/
│   ├── sentinel/
│   │   ├── README.md
│   │   └── modules/
│   └── byop/
│       ├── README.md
│       └── modules/
└── workspace/                   # User project directories (gitignored)
    └── <project-name>/          # Scaffolded by /start, has its own git repo
```

## Onboarding Flow

When a user runs `claude` in this repo:
1. Claude Code detects two project hooks in `.claude/settings.json` and prompts the user to trust them. They should approve both — they are read-only and safe. Users can review them with `/hooks` if they want to.
2. **Hook 1 — welcome.sh**: Prints a welcome banner telling them to type `/start` and explains the hooks.
3. **Hook 2 — check-updates.js**: Pings GitHub to check if a newer version of Claude Code is available and checks if the repo is behind origin. Shows a banner with update instructions if either is true. Fails silently if offline.
4. The `/start` skill walks them through: pick a project (Canvas, Forge, Nexus, Sentinel, or Bring Your Own Project), pick a language (skipped for Canvas and BYOP), optionally choose an environment (skipped for Canvas and BYOP; venv/conda/Docker for others), verify environment, scaffold project in `workspace/<name>/` (or reference an external path for BYOP), then deliver Module 1 inline.
5. From there, users say "next module" to continue through Modules 2-10. Claude reads the current module file from `projects/<name>/modules/` and walks them through it — all within the same cc-self-train session. No terminal switching needed.

## Conventions

- **Language-agnostic:** Never assume a specific programming language. Describe what to build, not how. When giving examples, show multiple languages or use pseudocode.
- **Local-only:** No cloud services required. All projects work with local files, local git, and local tools.
- **Same curriculum, different domains:** All 5 options cover the same 10 modules and the same CC features. The user picks based on what they want to build, not difficulty.
- **Hands-on:** Every module ends with verification. Users should be doing, not just reading.
- **Subdirectory projects:** Tutorial projects live in `workspace/<name>/` with their own nested git repos. The `workspace/` directory is gitignored by cc-self-train. BYOP projects stay at their external path — referenced via `@import` in CLAUDE.local.md. Users stay in the cc-self-train session for all modules.

## First Message Behavior

When a user starts a session in this repo, ALWAYS greet them warmly and direct them to get started.

**If `CLAUDE.local.md` exists with an active project**, greet the user and offer to continue (e.g., "Welcome back! You're on Module 3 of Forge. Say 'next module' when you're ready to continue."). Read `CLAUDE.local.md` for the project name, language, directory, and current module. Then silently detect the default remote branch and check for updates:

```bash
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
if [ -n "$DEFAULT_BRANCH" ]; then
  git fetch origin --quiet
  git rev-list HEAD..origin/"$DEFAULT_BRANCH" --count 2>/dev/null
fi
```

If the repo is behind, mention: "I noticed there are curriculum updates available (N commits behind). Want me to pull them before we continue?" If the default branch can't be resolved, the fetch fails, or the repo is up to date, say nothing.

**If `CLAUDE.local.md` does not exist** (new user), and they send a vague first message (like "hi", "hello", "help", "what is this", or anything that suggests they're new), respond with:

1. A brief welcome explaining this is a hands-on Claude Code learning repo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zainnab-sparq/cc-self-train](https://github.com/zainnab-sparq/cc-self-train) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
