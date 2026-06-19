---
trigger: always_on
description: Respond to all code review comments on a pull request, implement fixes, run tests, commit, reply on GitHub (in Chinese), and resolve conversations. Supports multi-round review loops with AI review bots. Use when the user says \"处理审查意见\", \"回复 review\", \"resolve PR comments\", \"修复审查意见\", \"处理PR评论\", \"review feedback\", \"address comments\", \"reply to review\", or similar phrases.
---


# /pr-review-respond

Handle all code review comments on a GitHub PR: categorize into fixed / needs-fix / wontfix, implement fixes, test, commit, push, reply in Chinese, and resolve all threads. Supports multi-round review loops to handle AI review bot feedback on pushed changes.

## When to use

- A PR has pending review comments that need to be addressed
- The user says "处理审查意见", "回复 review", "resolve PR comments", etc.
- Review feedback from any review tool (codereviewbot, human reviewers, etc.)

## When NOT to use

- The PR has no review comments yet — nothing to respond to
- You don't have write access to the repository
- The review comments require discussion with the original author before acting
- The skill is being invoked on a fork without upstream write permissions
- The PR is merged, closed, or in draft state
- The current branch is the repository's default branch
- The `gh` CLI is not installed or not authenticated

## Usage

```
/pr-review-respond <owner/repo> <PR number>
```

If no arguments given, detect from `git remote get-url origin` and the current branch's associated PR:

```bash
# Auto-detect PR number from current branch
gh pr view --json number --jq '.number'
```

## Workflow

### Pre-Step — Validate environment and initialize session state

Before starting, run these validation checks:

1. **gh CLI check**: Run `gh --version`. If it fails, abort with "错误：未安装 GitHub CLI（gh），请先安装。"
2. **Auth check**: Run `gh auth status`. If it fails, abort with "错误：GitHub CLI 未登录，请先运行 gh auth login。"
3. **PR state check**: Run:
   ```bash
   gh pr view <PR> --json state,isDraft,merged,headRefName,baseRefName,headRefOid
   ```
   - If `state == "CLOSED"` or `merged == true`: abort with "PR #{PR} 已关闭/合并，无需处理审查意见。"
   - If `isDraft == true`: abort with "PR #{PR} 为草稿状态，请先将其标记为 Ready for Review。"
   - Save `headRefName`, `baseRefName`, `headRefOid` as `pr_metadata`.
4. **Default branch check**: Run:
   ```bash
   gh repo view <owner/repo> --json defaultBranch --jq .defaultBranch
   ```
   If current branch equals default branch: abort with "错误：当前分支为仓库的默认分支（{default_branch}），禁止直接在该分支上处理审查意见。"
5. **Fetch latest base**: `git fetch origin <baseRefName>`

Then initialize session state:
- `round = 1`
- `max_rounds = 3` (configurable via environment variable `PR_REVIEW_MAX_ROUNDS`)
- `processed_thread_ids = []` — set of thread IDs already replied to and resolved in prior rounds
- `round_summary = []` — structured log of what each round accomplished

### Loop body: Steps 1–4

The four steps below form the body of the multi-round loop. After completing Step 4, a decision is made whether to continue to the next round.

### Step 1 — Fetch all review comments

Use `gh api` with retry logic for all calls below. For each API call:
1. Execute the command
2. If it fails with `rate limit` / `403` / `429`: wait 60 seconds and retry
3. For other errors: retry up to `PR_REVIEW_RETRY_COUNT` times (default 3) with 1-second backoff
4. If all retries fail: report the error and abort

Fetch all pull request review comments via REST:

```bash
comments_json=$(gh api repos/<owner>/<repo>/pulls/<PR>/comments --paginate \
  --jq '.[] | {id, body: .body[0:100], path, commit_id, user: .user.login, created_at, in_reply_to_id, node_id}')
```

Validate output: if `comments_json` is empty or `null`, report "PR #{PR} 没有待处理的审查评论。" and exit.

Also fetch review threads via GraphQL with cursor-based pagination:

```bash
# Initialize
all_threads="[]"
cursor="null"
has_next_page="true"

while [ "$has_next_page" = "true" ]; do
  page=$(gh api graphql -f query='
  query($owner:String!,$repo:String!,$pr:Int!,$after:String) {
    repository(owner:$owner,name:$repo) {
      pullRequest(number:$pr) {
        reviewThreads(first:100, after:$after) {
          pageInfo { hasNextPage endCursor }
          nodes { id isResolved comments(first:1) { nodes { id } } }
        }
      }
    }
  }' -F owner=<owner> -F repo=<repo> -F pr=<PR> ${cursor:+-F after="$cursor"})

  page_nodes=$(echo "$page" | jq '.data.repository.pullRequest.reviewThreads.nodes')
  all_threads=$(echo "$all_threads" | jq --argjson nodes "$page_nodes" '. + $nodes')
  has_next_page=$(echo "$page" | jq -r '.data.repository.pullRequest.reviewThreads.pageInfo.hasNextPage')
  cursor=$(echo "$page" | jq -r '.data.repository.pullRequest.reviewThreads.pageInfo.endCursor')
done
```

After each successful API call, check remaining rate limit:
```bash
remaining=$(gh api rate_limit --jq '.rate.remaining' 2>/dev/null)
if [ "$remaining" -lt 100 ]; then
  echo "警告：GitHub API 速率限制剩余 ${remaining} 次。"
fi
```

**Thread-to-comment ID mapping**: The REST API returns comments with a numeric `id` (e.g., `12345`) and a globally unique `node_id` (e.g., `MDI0OlB1bGxSZXF1ZXN0UmV2aWV3Q29tbWVudDEyMzQ1`). The GraphQL API returns threads with an `id` field that corresponds to the REST comment's `node_id`. Mapping: `rest_comment.node_id === graphql_thread.id`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kingside33/claude-code-pr-review-respond](https://github.com/kingside33/claude-code-pr-review-respond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
