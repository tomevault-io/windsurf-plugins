---
trigger: always_on
description: Capy MCP tools protect your context window by keeping raw tool output in sandboxed subprocesses and indexing it for on-demand search.
---

# capy — context-window routing

Capy MCP tools protect your context window by keeping raw tool output in sandboxed subprocesses and indexing it for on-demand search.

## Decision principle

Choose the tool based on what you need from the output:

- **Comprehension** (understand the full content) → direct tools (Bash, Read)
- **Extraction** (specific facts from large output) → capy tools

## When to use direct tools (Bash / Read)

- **Git commands:** diffs, logs, status, branch — always Bash. Diffs are comprehension content; BM25 fragments destroy review quality.
- **Small-output commands (<~50 lines):** ls, wc, file, git status — Bash directly. Sandbox overhead exceeds savings.
- **Files to comprehend or edit:** Read tool. Required before Edit.
- **Sequential/ordered content:** test output, build logs where order matters — Bash or Read.
- **Instruction files, checklists, configs:** Read and internalize whole. BM25 returns ranked fragments, destroying structural relationships.
- **Small authoritative web pages:** GitHub issues, PR descriptions, short specs, doc pages — use runtime-native tools when available (e.g., `gh issue view` for GitHub, `WebSearch` for general queries). These are comprehension content; BM25 fragments destroy context just as they do for diffs.

## When to use capy tools

- **`capy_batch_execute`:** Broad exploration — multiple commands + queries in one call. Example: initial repo scan with `rg --files` + symbol searches.
- **`capy_execute` / `capy_execute_file`:** Single command or file producing hundreds+ lines where you only need extracted facts. API calls, log analysis, data processing.
- **`capy_fetch_and_index`:** Fetch and index large web content for extraction. Default ephemeral (24h TTL, excluded from default search). Best for large web artifacts (transcripts, logs, long docs) and content needing follow-up `source:` queries. Pass `kind: "durable"` for reference docs to persist across sessions. NOT for small authoritative pages you need to comprehend — use runtime web tools for those.
- **`capy_index`:** Persist curated knowledge durably. Content you explicitly want searchable across sessions.
- **`capy_search`:** Query indexed content. Batch all questions as array. Default excludes ephemeral — use `include_kinds` or `source:` to include.

## Blocked commands — enforced by hooks

### curl / wget — BLOCKED
Any Bash command containing `curl` or `wget` is intercepted and replaced with an error message. Do NOT retry.
Instead use:
- `capy_fetch_and_index(url, source)` to fetch and index web pages
- `capy_execute(language: "javascript", code: "const r = await fetch(...)")` to run HTTP calls in sandbox

### Inline HTTP — BLOCKED
Any Bash command containing `fetch('http`, `requests.get(`, `requests.post(`, `http.get(`, or `http.request(` is intercepted and replaced with an error message. Do NOT retry with Bash.
Instead use:
- `capy_execute(language, code)` to run HTTP calls in sandbox — only stdout enters context

### WebFetch — BLOCKED
WebFetch calls are denied entirely. Instead use:
- For git platform issues/PRs/MRs: platform CLI (e.g., `gh issue view N`) or `WebSearch` for full comprehension
- For other small pages needing comprehension: `WebSearch` or other runtime web tools
- For large web content or extraction: `capy_fetch_and_index(url, source)` then `capy_search(queries)`

## Source kinds

Every indexed entry has a **kind** that controls its lifecycle and search visibility:

| Kind | What produces it | Retention | Included by default in search? |
|------|-----------------|-----------|-------------------------------|
| `durable` | `capy_index`, `capy_fetch_and_index(kind: "durable")` | Retention-score tiers (hot → warm → cold → evictable) | Yes |
| `ephemeral` | `capy_execute`, `capy_execute_file`, `capy_batch_execute`, `capy_fetch_and_index` (default) | Strict TTL — swept after expiry | No |
| `session` | `capy sweep` (indexes past conversation transcripts) | Strict TTL — swept after expiry | Yes |

**Querying non-default kinds:** pass `include_kinds` to `capy_search`:
- `include_kinds: ["durable", "ephemeral"]` — recover output from earlier commands in this session
- `include_kinds: ["durable", "ephemeral", "session"]` — search everything
- Or use `source: "<label>"` to bypass kind filtering entirely (matches any kind)

## Read vs capy_execute_file

**Default to `Read`.** It's cheap for normal-sized files, shows you actual content (not just patterns you knew to grep for), and is required if an Edit follows. Use `offset`/`limit` to scope large files.

**Reach for `capy_execute_file` only when ALL of these hold:**
1. The file is genuinely large (10k+ lines, or measured >100 KB), AND
2. You want a *derived answer* (count, stats, extracted pattern, structural summary) — not the content itself, AND
3. You can write the exact grep/awk/script upfront. If you'd struggle to, you don't know enough yet — just `Read`.

**Anti-patterns — do NOT do this:**
- `capy_execute_file` to grep section headings, then `Read` the file anyway to Edit it. The Read makes the capy call pure overhead.
- `capy_execute_file` on a code file to "explore structure." Use `grep`, `find`, or language-specific tools instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serpro69/kotlin-faker](https://github.com/serpro69/kotlin-faker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
