---
trigger: always_on
description: Systematic release workflow for cloud projects. Auto-detects tech stacks and deployment targets, runs security audits, build verification, and generates deployment plans. Works with Claude Code and Codex.
---


# Cloud Release — Systematic Release Workflow

A zero-config, auto-detecting release workflow for cloud projects. Detects your tech stack, deployment target, and project structure, then guides you through a structured release process with security checks, build verification, and deployment planning.

<HARD-GATE>
Do NOT execute any deployment commands or modify production systems until the user has reviewed and approved the deployment plan at Phase 6. This applies to EVERY release regardless of perceived simplicity.
</HARD-GATE>

## Step 0: Version Check & Upgrade

> **Skip this step if the user invoked the skill with `--upgrade` flag** — jump directly to the upgrade flow below.

### 0.1: Check for updates

Detect the current local version and compare against the latest GitHub release.

**Get local version:**
```bash
# Read from CHANGELOG.md — first version header (e.g. ## [2.0.0])
grep -m1 '## \[' {skill_root}/CHANGELOG.md 2>/dev/null | grep -oE '[0-9]+\.[0-9]+\.[0-9]+'
```

**Get remote version** (GitHub API, no auth required):
```bash
curl -sf "https://api.github.com/repos/rrred0324/cloud-release/releases/latest" \
  2>/dev/null | grep '"tag_name"' | grep -oE '[0-9]+\.[0-9]+\.[0-9]+'
```

Override the default repo with `skill_repo` from `.cloudrelease.yml` if set:
```bash
# If .cloudrelease.yml has: skill_repo: owner/repo
# Use: https://api.github.com/repos/{skill_repo}/releases/latest
```

If the curl fails (no network, rate-limited, private repo), skip silently — do not block the release workflow.

**If remote version > local version**, notify the user once per session:

```
💡 cloud-release {remote_version} is available (you're on {local_version})
   Run: /cloud-release --upgrade   to update
   Or continue with the current version — this won't affect your release.
```

Then continue with the normal release workflow. Do not block or prompt for confirmation.

### 0.2: Upgrade flow (`--upgrade` flag or user confirms upgrade)

**Triggered by:** `/cloud-release --upgrade` (Claude Code) or `codex exec "/cloud-release --upgrade"` (Codex)

**Steps:**

1. Detect `skill_root` — the directory containing this SKILL.md file:
   ```bash
   # Claude Code: skill is in ~/.claude/skills/cloud-release/ or .claude/skills/cloud-release/
   # Codex: skill is in .agents/skills/cloud-release/ or agents/skills/cloud-release/
   # Check both locations; use whichever exists
   ```

2. Confirm the upgrade with the user:

   **[PLATFORM:INTERACT]**
   question: Upgrade cloud-release from {local_version} to {remote_version}?
   options:
     A) Yes, upgrade now (recommended)
     B) No, skip
   recommendation: A
   **[/PLATFORM:INTERACT]**

3. If confirmed, download and apply the update:
   ```bash
   # Download latest release archive
   curl -sfL "https://github.com/rrred0324/cloud-release/archive/refs/heads/main.tar.gz" \
     -o /tmp/cloud-release-latest.tar.gz

   # Backup current version
   cp -r "{skill_root}" "{skill_root}.bak.$(date +%Y%m%d%H%M%S)"

   # Extract and overwrite (preserve .cloudrelease.yml in project — it's not part of the skill)
   tar -xzf /tmp/cloud-release-latest.tar.gz -C /tmp/
   cp -r /tmp/cloud-release-main/. "{skill_root}/"

   # Cleanup
   rm -f /tmp/cloud-release-latest.tar.gz
   rm -rf /tmp/cloud-release-main
   ```

4. Verify the upgrade:
   ```bash
   grep -m1 '## \[' "{skill_root}/CHANGELOG.md" | grep -oE '[0-9]+\.[0-9]+\.[0-9]+'
   ```

5. Report result:
   - Success: `✅ cloud-release upgraded to {new_version}. Run /cloud-release to start a release.`
   - Failure (download error, extraction error): restore from backup and report:
     `⛔ Upgrade failed — restored previous version {local_version}. Error: {error_message}`

6. **Exit** — do not continue into the release workflow after an upgrade.

---

## Step 0b: Platform Detection

Determine which AI platform is running this skill and load the appropriate adapter.

**Detection logic:**
- If `AskUserQuestion` tool is available → Claude Code → load `platforms/claude-code.md`
- If running in Codex environment → load `platforms/codex.md`
- Otherwise → default to text-based interaction (similar to Codex adapter)

**Action:** Read the detected platform adapter file and follow its interaction patterns for all subsequent steps.

## Step 1: Project Detection

Auto-detect the project structure, tech stacks, and deployment targets.

### 1.1: Load configuration (if exists)

Search for `.cloudrelease.yml` or `.cloudrelease.yaml` in:
1. Current directory
2. Git project root

If found, read it. Configuration values override auto-detection results.

### 1.2: Detect project type

```bash
# Is this a git repo?
git rev-parse --show-toplevel 2>/dev/null || echo "NOT_A_GIT_REPO"

# Current branch
git branch --show-current 2>/dev/null

# Uncommitted changes
git status --porcelain | wc -l
```

### 1.3: Detect tech stacks

For each stack module in `stacks/`, check its detection signals against the project:

```bash
# Check each stack's detection signals
# Python: requirements.txt / pyproject.toml / Pipfile
# Node.js: package.json / tsconfig.json
# Java: pom.xml / build.gradle
# Go: go.mod
# Rust: Cargo.toml
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rrred0324/cloud-release](https://github.com/rrred0324/cloud-release) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
