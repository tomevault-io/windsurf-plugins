---
trigger: always_on
description: Manage GitHub Projects (V2), Issues, and Boards. View project boards, create/move issues, comment, and track status via CLI. Use when you need to interact with GitHub Projects.
---


# GitHub Project Skill

This skill provides a comprehensive CLI wrapper for the GitHub GraphQL API, enabling efficient management of GitHub Projects V2 and Issues directly from the terminal.

## Usage

The main entry point is `dist/cli.js` (relative to the project root). You can verify it works with:
```bash
node dist/cli.js --help
```

**Global Options:**
- `-j, --json`: Output results in JSON format. **ALWAYS use this flag** when parsing output programmatically.

Note: In `--json` mode, the CLI prints **only JSON** to stdout.

### 0. 🔍 Discovery
**Goal:** Explore owner resources and project structure.

#### List Projects
- **Command:** `node dist/cli.js project-list <owner>`
- **Usage:**
  ```bash
  node dist/cli.js project-list "my-org" --json
  ```

#### List Repositories
- **Command:** `node dist/cli.js repo-list <owner>`
- **Usage:**
  ```bash
  node dist/cli.js repo-list "my-org" --json
  ```

#### Get Project Board Details
- **Command:** `node dist/cli.js board <owner> <project>`
- **Arguments:** `<project>` can be the Project Number or the Project Title (case-insensitive).
- **Options:** `--status-field <name>` (Default: "Status")
- **Usage:**
  ```bash
  node dist/cli.js board "my-org" "My Project Name" --json
  ```
  _Returns project ID, global node ID, and column definitions (including Option IDs needed for moving items)._

### 1. 📋 Board Items
**Goal:** List all items (Issues/PRs/Drafts) on the board with their current status.
- **Command:** `node dist/cli.js items <owner> <project>`
- **Usage:**
  ```bash
  node dist/cli.js items "my-org" 1 --json
  ```
  _Returns list of items with their Item IDs (for moving/archiving) and Content Node IDs._

#### List Issues Linked to a Project (Project-Centric)
**Goal:** One consistent way for AI agents: start from Project, get issues + both IDs (Issue node ID and Project item ID).
- **Command:** `node dist/cli.js project-issues <owner> <project>`
- **Options:**
  - `--status-field <name>` (Default: "Status")
  - `--state <open|closed|all>` (Default: `open`)
  - `--limit <n>` (Default: `50`)
  - `--repo <owner/repo>` (Optional: narrow search to a single repo)
- **Usage:**
  ```bash
  node dist/cli.js project-issues "my-org" "Trionix Lab" --state open --limit 50 --json
  # optionally scope to one repo for speed/limits
  node dist/cli.js project-issues "my-org" 1 --repo "my-org/my-repo" --json
  ```
  _Implementation note: uses GitHub GraphQL `search` + `Issue.projectItems` to avoid cases where `ProjectV2.items` listing is incomplete for some integrations._

#### Show Project Item by Item ID
**Goal:** Inspect a specific project item when listing is incomplete.
- **Command:** `node dist/cli.js item-show <itemId>`
- **Options:** `--status-field <name>` (Default: "Status")
- **Usage:**
  ```bash
  node dist/cli.js item-show "PVTI_..." --json
  ```
  _Useful workaround if `items` does not show Issues/PRs due to access restrictions._

### 2. 🆕 Issue & Draft Management

#### List Issues
**Goal:** List issues in a repository (with pagination).
- **Command:** `node dist/cli.js issue-list <owner/repo>`
- **Options:** `--state <open|closed|all>` (Default: `open`), `--limit <n>` (Default: `50`)
- **Usage:**
  ```bash
  node dist/cli.js issue-list "my-org/my-repo" --state open --limit 50 --json
  ```

#### Create Issue
- **Command:** `node dist/cli.js issue-create <owner/repo> "<title>"`
- **Options:** `--body "<body>"`
- **Usage:**
  ```bash
  node dist/cli.js issue-create "my-org/my-repo" "Fix bug" --body "Details" --json
  ```

#### Create Draft
- **Command:** `node dist/cli.js draft-create <owner> <project> "<title>"`
- **Options:** `--body "<body>"`
- **Usage:**
  ```bash
  node dist/cli.js draft-create "my-org" "Product Roadmap" "Task title" --body "Details" --json
  ```

#### Show Issue
- **Command:** `node dist/cli.js issue-show <owner/repo> <issue_number>`
- **Usage:**
  ```bash
  node dist/cli.js issue-show "my-org/my-repo" 123 --json
  ```

#### Get Issue by Node ID (Agent-Friendly)
**Goal:** Fetch issue content when you already have `issue.id` (node ID).
- **Command:** `node dist/cli.js issue-get <issueNodeId>`
- **Usage:**
  ```bash
  node dist/cli.js issue-get "I_kwDO..." --json
  ```

#### Update Issue
- **Command:** `node dist/cli.js issue-update <issueNodeId>`
- **Options:** `--title <text>`, `--body <text>`
- **Usage:**
  ```bash
  node dist/cli.js issue-update "I_kwDO..." --title "New Title" --json
  ```

#### Close Issue
- **Command:** `node dist/cli.js issue-close <issueNodeId>`
- **Usage:**
  ```bash
  node dist/cli.js issue-close "I_kwDO..." --json
  ```

#### Delete Issue (Behavior)
**Note:** GitHub APIs do not support hard-deleting Issues. This command closes the issue.
- **Command:** `node dist/cli.js issue-delete <issueNodeId>`
- **Usage:**
  ```bash
  node dist/cli.js issue-delete "I_kwDO..." --json
  ```

### 3. 💬 Collaboration
**Goal:** Add comments to issues.
- **Command:** `node dist/cli.js issue-comment <issueNodeId> "Comment text"`
- **Usage:**
  ```bash
  node dist/cli.js issue-comment "I_kwDO..." "Done" --json
  ```

#### List Comments
**Goal:** Get existing comments for an issue (for agents that need context).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slonikmak/gh-skill](https://github.com/slonikmak/gh-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
