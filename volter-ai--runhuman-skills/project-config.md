---
trigger: always_on
description: Create and manage human-powered QA tests using Runhuman CLI. Use this skill when you need to test web applications with real human testers, get UX feedback, validate user flows, check mobile responsiveness, or find bugs that automated tests miss.
---


# Runhuman CLI

Runhuman connects AI coding tools to on-demand professional human testers. You describe what to test in natural language, a real human performs the test, and you get structured feedback back.

## When to Use Runhuman

**Good for:** User flows (signup, checkout), visual/layout testing, mobile responsiveness, UX feedback, exploratory testing, cross-browser checks.

**Not for:** Unit tests, performance benchmarks, security audits, or high-frequency automated test suites.

## Quick Start

```bash
# 1. Install (or use npx runhuman for any command without installing)
npm install -g runhuman

# 2. Authenticate (opens browser for GitHub OAuth)
runhuman login

# 3. Set up a project
#    List existing projects:
runhuman projects list
#    Or create a new one:
runhuman projects create "My App" --organization <orgId> --default-url https://staging.myapp.com --set-default
#    Switch to an existing project:
runhuman projects switch <projectId>

# 4. Create a test
runhuman job create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

**Deciding between new vs existing project:** Ask the user whether they want to create a new project or use an existing one. Use `runhuman projects list` to show what exists, then either `projects create` with `--set-default` or `projects switch` to set the active project. `projects switch` sets the default globally (in `~/.config/runhuman/config.json`) by default; pass `--global false` to set it locally for the current directory only.

**Setting defaults:** Use `runhuman config set <key> <value>` to set defaults like `project`, `color`, or `apiUrl`. Use `runhuman config list` to see the full configuration hierarchy.

**Local repo config:** Run `runhuman init` to create a `.runhumanrc` in the current directory with project name, default duration/device class, and one-or-more linked GitHub repos. Pass `--yes` to skip prompts, or `--name`, `--github-repo <owner/repo>`, or `--github-repos <a/b,c/d>` to fill values non-interactively.

## Job Command Namespace

All job commands live under `runhuman job`:

```bash
runhuman job create   # create a new test
runhuman job status   # check a job's status
runhuman job wait     # block until a job completes
runhuman job results  # view detailed results
runhuman job list     # list jobs (with filters)
runhuman job watch    # tail a live job
runhuman job artifact # download a raw session artifact
runhuman job share    # toggle public sharing on
runhuman job unshare  # toggle public sharing off
runhuman job create-issue  # file an extracted finding as a GitHub issue
```

> **Deprecated flat shims.** The older flat forms (`runhuman create`, `runhuman status`, `runhuman wait`, `runhuman results`, `runhuman list`, `runhuman watch`) still resolve but emit a one-line deprecation warning and are scheduled for removal. **Always emit `runhuman job <subcommand>` in new scripts** — don't introduce flat-shim invocations.

### Truncated IDs

All commands accept truncated ID prefixes, similar to git short hashes. The CLI resolves the best match automatically:

```bash
runhuman job status 712e          # resolves to full job ID
runhuman projects switch proj     # resolves to matching project
```

If multiple IDs match, the CLI asks for more characters. Destructive operations (`projects delete`, `keys delete`, `projects transfer --to-org`) require full IDs.

### Wait for Results

```bash
# Block until the test completes (useful in CI/CD)
runhuman job create https://staging.myapp.com \
  -d "Test checkout flow end-to-end" \
  --sync

# Check status of an existing job
runhuman job status <jobId>

# Wait for a previously-created job to finish
runhuman job wait <jobId>

# View detailed results
runhuman job results <jobId>
```

## Creating Tests

The `job create` command is the primary way to submit tests. At minimum, provide a URL and either a description or a template.

### With a Description

```bash
runhuman job create https://staging.myapp.com \
  -d "Test the signup flow: click Sign Up, fill the form, verify confirmation page"
```

### With an Output Schema

Use `--schema` or `--schema-inline` to get structured JSON results back:

```bash
# Schema from a file
runhuman job create https://staging.myapp.com \
  -d "Test the search feature" \
  --schema ./search-schema.json \
  --sync --json

# Inline schema
runhuman job create https://staging.myapp.com \
  -d "Test signup and login" \
  --schema-inline '{"signupWorks":{"type":"boolean"},"loginWorks":{"type":"boolean"},"issues":{"type":"array","items":{"type":"string"}}}' \
  --sync --json
```

Example schema file (`search-schema.json`):
```json
{
  "searchWorks": {
    "type": "boolean",
    "description": "Does the search return results?"
  },
  "resultCount": {
    "type": "number",
    "description": "Number of results shown"
  },
  "issues": {
    "type": "array",
    "description": "List of any bugs or issues found"
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [volter-ai/runhuman-skills](https://github.com/volter-ai/runhuman-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
