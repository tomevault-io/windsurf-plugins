---
trigger: always_on
description: >-
---


# Vibe Retrospect

A dual-purpose skill for vibe coding knowledge transfer.

- **Retrospect mode**: Analyze a completed project → generate an HTML article (for humans) + a knowledge doc (for agents)
- **Learn mode**: Import someone else's knowledge doc → configure CLAUDE.md with teaching rules + project references → agent guides the user through building

## Mode Detection

At the start, determine which mode the user needs:

**Retrospect mode** triggers:
- "复盘", "retrospect", "总结开发经验", "打包我的开发经验", "review my project"
- User is in a project directory with code/git history
- User wants to generate output FROM their project

**Learn mode** triggers:
- "我想学做这个项目", "帮我学习这个", "参考这个来做", "coach me", "learn from this"
- User provides a `-knowledge.md` file or mentions someone else's project
- User wants to BUILD something based on a reference

If ambiguous, ask: "你是想复盘自己的项目，还是想参考别人的项目来学习开发？"

---

## Mode 1: Retrospect (复盘)

### Goal

Analyze a completed vibe coding project and generate THREE outputs:
1. **`{project-name}-story.html`** — A self-contained HTML article for humans to read
2. **`{project-name}-knowledge.md`** — A structured knowledge document for agents to reference
3. **`outputs/cards/card-*.png`** — Social media image cards (1080x1350 @2x, 4:5 ratio) for sharing on Xiaohongshu/Instagram

### Step 1: Locate All Data Sources

Silently gather everything before talking to the user.

**1A. Codebase**
```bash
# Project identity
basename $(pwd)
git remote get-url origin 2>/dev/null
```

Read (if they exist):
- `CLAUDE.md` — project rules, collaboration norms, workflow
- `README.md` — project description
- `docs/` — specs, decisions, status, tasks
- Main source files, config files (`package.json`, `wrangler.toml`, etc.)
- Deployment scripts

**1B. Git history**
```bash
git log --format='%ai' --all | tail -1    # first commit date
git log --format='%ai' --all | head -1    # last commit date
git log --all --oneline | wc -l           # total commits
git log --all --format='%ai | %s' --reverse  # full timeline
git log --all --format='%ad' --date=short | sort | uniq -c | sort -rn | head -5  # peak days
git log --all --name-only --format= | sort | uniq -c | sort -rn | head -10  # most changed files
```

**1C. Session logs (the real gold)**

Claude Code stores conversation history at `~/.claude/projects/{encoded-path}/`.
The path encoding replaces `/` with `-` and prepends `-`.

```bash
CWD=$(pwd)
ENCODED=$(echo "$CWD" | sed 's|/|-|g')
CLAUDE_DIR="$HOME/.claude/projects/$ENCODED"
ls "$CLAUDE_DIR"/*.jsonl 2>/dev/null
```

Each `.jsonl` file = one conversation session. Extract from them:
- **User messages**: every decision, feedback, question the human made
- **Decision points**: where user said "不要"/"改成"/"可以"/"停"/"no"/"change"/"ok"
- **Collaboration feedback**: "以后"/"下次"/"记住"/"always"/"never" — rules born from experience
- **First message**: often reveals the origin story
- **Debugging sessions**: long back-and-forth exchanges = pitfalls

Use the parse script: `bash ~/.claude/skills/vibe-retrospect/scripts/parse-sessions.sh "$CLAUDE_DIR" ./tmp/session-analysis`

**1D. Memory files**
```bash
ls "$CLAUDE_DIR/memory/" 2>/dev/null   # MEMORY.md, daily notes
```

**1E. GitHub (if remote exists)**
```bash
gh pr list --state all --limit 50 2>/dev/null
gh issue list --state all 2>/dev/null
```

### Step 2: Present Summary, Ask for Gaps

Show the user what you found. Ask them to supplement what can't be extracted automatically:

1. **Origin story** — "I see your first message was '{first_msg}'. What triggered this idea?"
2. **Key turning points** — "On {date} you made {N} commits. What happened that day?"
3. **Advice for others** — "If someone wants to build something similar, top 3 things to tell them?"
4. **Cost** — "What's the monthly cost to run this?"
5. **Anything sensitive to redact?**

User can skip any question. Fill gaps from session logs and git history.

### Step 3: Generate Three Outputs

Place all in `./outputs/` directory.

#### Output 1: `{project-name}-story.html`

A self-contained HTML article with embedded CSS. Warm, editorial style. For humans to read and share.

Structure:
- Project origin story (with direct quotes from session logs)
- Key decisions and why they were made
- Architecture overview
- Development timeline with milestones
- Pitfalls encountered and how they were solved
- Collaboration methodology (how human + AI worked together)
- Lessons learned

Design guidelines:
- Self-contained (all CSS inline/embedded, no external dependencies)
- Readable typography (serif for body, sans for headings)
- Warm color palette (cream/off-white backgrounds)
- Responsive (works on mobile and desktop)
- Include direct quotes from session logs throughout
- Must include in `<head>`: SVG emoji favicon (`<link rel="icon" href="data:image/svg+xml,...">`) and Open Graph meta tags (`og:title`, `og:description`, `og:type`, `og:url`) so browser tabs show an icon and social sharing shows preview

#### Output 2: `{project-name}-knowledge.md`

A structured knowledge document following the **Knowledge Document Schema** below. This is NOT a skill — it is reference material that another user's agent can read.

---

## Knowledge Document Schema


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kennyzheng-builds/vibe-retrospect](https://github.com/kennyzheng-builds/vibe-retrospect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
