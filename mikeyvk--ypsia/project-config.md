---
trigger: always_on
description: **Auto-loaded by VS Code / Google Antigravity** via `chat.useAgentsMdFile: true` — single always-on instruction file.
---

# pgmcp — Agent Protocol

**Auto-loaded by VS Code / Google Antigravity** via `chat.useAgentsMdFile: true` — single always-on instruction file.
**Status:** Active | **Context:** AI-powered training, nutrition and weekly planning platform (ypsia)

---

## 🏛️ Architecture Contract (MANDATORY)

**Before writing any implementation code, read:**
**[docs/coding_standards/ARCHITECTURE_PRINCIPLES.md](docs/coding_standards/ARCHITECTURE_PRINCIPLES.md)**

This document is a **binding contract**. Code that violates these principles is **REJECTED**, regardless of whether tooling gates pass.

### Most common violations (quick reference):

| Violation | Correct pattern |
|---|---|
| Hardcoded phase/workflow names in Python | Read from config (WorkflowConfig / GitConfig) |
| `SomeManager()` inside `execute()` | Constructor injection via `__init__` |
| Write-capable interface for read-only consumer | Use narrow read-only interface (ISP) |
| `get_state()` calls `save()` | CQS violation — split the method |
| Module-level `Config.load()` | `ClassVar` + lazy init in `__init__` |
| if-chain on `phase == "implementation"` etc. | Registry or config-driven dispatch (OCP) |
| Value object without `frozen=True` | Add `@dataclass(frozen=True)` or `ConfigDict(frozen=True)` |
| Issue number extracted in state engine | Delegate to git conventions config class |

---

## 🔧 Tool Priority Matrix (MANDATORY)

**Never use `run_in_terminal` or default/built-in agent tools (e.g., `write_to_file`, `replace_file_content`, `multi_replace_file_content`) for these operations — use MCP tools instead:**

### Git Operations
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Create branch | `create_branch(name, base_branch, branch_type)` | `run_in_terminal("git branch")` |
| Commit | `git_add_or_commit(workflow_phase, message)` | `run_in_terminal("git commit")` |
| Checkout | `git_checkout(branch)` | `run_in_terminal("git checkout")` |
| Push | `git_push(set_upstream)` | `run_in_terminal("git push")` |
| Merge | `git_merge(branch)` | `run_in_terminal("git merge")` |
| Delete branch | `git_delete_branch(branch, force, mode)` | `run_in_terminal("git branch -d")` |
| Stash | `git_stash(action, message)` | `run_in_terminal("git stash")` |
| Status | `git_status()` | `run_in_terminal("git status")` |
| Restore | `git_restore(files, source)` | `run_in_terminal("git restore")` |
| Fetch | `git_fetch(remote, prune)` | `run_in_terminal("git fetch")` |
| Pull | `git_pull(remote, rebase)` | `run_in_terminal("git pull")` |
| List branches | `git_list_branches(verbose, remote)` | `run_in_terminal("git branch -a")` |
| Diff stats | `git_diff_stat(target_branch, source_branch)` | `run_in_terminal("git diff --stat")` |
| Verify merge reachability | `check_merge(merge_sha)` | `run_in_terminal("git merge-base --is-ancestor")` |

### GitHub Operations
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Create issue | `create_issue(title, body, labels, milestone, assignees)` | `run_in_terminal("gh issue create")` |
| Get issue | `get_issue(issue_number)` | `run_in_terminal("gh issue view")` |
| List issues | `list_issues(state, labels)` | `run_in_terminal("gh issue list")` |
| Update issue | `update_issue(issue_number, ...)` | `run_in_terminal("gh issue edit")` |
| Close issue | `close_issue(issue_number, comment)` | `run_in_terminal("gh issue close")` |
| Create PR (atomic) | `submit_pr(title, body, head, base, draft)` | `run_in_terminal("gh pr create")` |
| List PRs | `list_prs(state, base, head)` | `run_in_terminal("gh pr list")` |
| Merge PR | `merge_pr(pr_number, commit_message, merge_method)` | `run_in_terminal("gh pr merge")` |
| Get PR | `get_pr(pr_number)` | `run_in_terminal("gh pr view")` |
| Create label | `create_label(name, color, description)` | Manual GitHub UI |
| Add labels | `add_labels(issue_number, labels)` | `run_in_terminal("gh issue edit")` |
| Create milestone | `create_milestone(title, description, due_on)` | Manual GitHub UI |

### File Operations
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Edit file | `safe_edit_file(path, content/line_edits/insert_lines/search+replace, mode)` | `run_in_terminal("Set-Content")` |
| Scaffold code/docs | `scaffold_artifact(artifact_type, name, context)` | Manual creation |
| Inspect artifact context schema | `scaffold_schema(artifact_type)` | Guessing context fields or trial-and-error calls |

### Quality & Testing
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Run quality gates | `run_quality_gates(files)` | `run_in_terminal("pylint")` or `run_in_terminal("mypy")` |
| Run tests | `run_tests(path, markers, timeout, verbose)` | `run_in_terminal("pytest")` |
| Validate template | `validate_template(path, template_type)` | Manual review |

### Project & Phase Management
| Action | ✅ USE THIS | ❌ NEVER USE |
|--------|-------------|------------|
| Initialize project | `initialize_project(issue_number, issue_title, workflow_name)` | Manual .pgmcp/ file creation |
| Get project plan | `get_project_plan(issue_number)` | Manual .pgmcp/ file reading |
| Transition phase | `transition_phase(branch, to_phase)` | Manual .pgmcp/state.json edit |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MikeyVK/ypsia](https://github.com/MikeyVK/ypsia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
