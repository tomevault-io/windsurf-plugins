---
trigger: always_on
description: Instructions for an LLM to discover and add new BOFs to the catalog.
---

# BOF Catalog Maintenance Instructions

Instructions for an LLM to discover and add new BOFs to the catalog.

## Task: Weekly Issue Triage (primary workflow)

A GitHub Actions workflow runs weekly and creates an issue titled "Weekly BOF discovery report" with candidate repos. This is the main way new BOFs get added.

### Step 1: Check the Latest Open Issues

```bash
gh issue list --limit 2 --state open --json number,title,author,body,url,createdAt
```

Look for issues titled "Weekly BOF discovery report" created by `app/github-actions`.

### Step 2: Evaluate Candidates

For each candidate in the issue, check whether it's a real BOF worth adding:

```bash
gh api repos/OWNER/REPO --jq '{description,stargazers_count,language,topics,created_at,pushed_at}'
gh api repos/OWNER/REPO/readme --jq '.content' | base64 -d | head -80
```

**Include** repos that are:
- Actual Beacon Object Files (C source with beacon.h, .cna aggressor scripts, etc.)
- For any C2 framework (Cobalt Strike, Sliver, Havoc, Adaptix, etc.)
- Have real source code and a reasonable README
- Low-star repos are fine if the BOF is functional and focused

**Exclude** repos that are:
- Not BOFs (false positives like "Bank of America", game mods, buffer overflow labs)
- Download-focused repos pushing pre-built binaries without proper build-from-source workflow
- Forks/copies of existing BOFs with no meaningful changes
- Empty, no description, or clearly abandoned learning repos
- Generic tool collections, cheatsheets, or pentest dumps that aren't BOF-specific
- Already in the catalog (grep BOF-CATALOG.md first)

**Suspicious/Copycat Detection** — apply extra scrutiny to repos that:
- Share the same name as an existing catalog entry but have significantly fewer stars
- Have pre-compiled binaries (.o, .exe, .dll) without corresponding source code or build instructions
- Were created by accounts less than 90 days old with fewer than 3 public repositories
- Have descriptions identical or near-identical to an existing catalog entry
- Are forks of existing entries with no meaningful commits beyond the fork point

**Malware (zip-dropper) pattern** — seen on GitHub targeting security tooling.
An attacker copies a legitimate repo's source tree and adds a payload. Reject
any candidate matching two or more of:
- `.zip` / `.7z` / `.rar` archive committed into the repo tree, especially
  inside an oddly-named subdirectory (Aquarius, Taurus, Orion and other
  zodiac/constellation names are a recurring campaign signature)
- README contains a download button or badge linking to
  `raw.githubusercontent.com/<owner>/<repo>/.../*.zip`
- README instructs users to disable antivirus/firewall/Defender, add
  exclusions, or whitelist the download
- README uses "Getting Started / Download the Application / System
  Requirements / Choose Your Version" marketing boilerplate uncharacteristic
  of BOF repos
- Repo description uses emoji + vague phrasing ("🔧 Simplify X with essential
  BOFs, providing convenient helper scripts...") without naming concrete
  techniques

When the weekly discovery issue includes **Warnings** annotations, investigate each warning before adding. If a repo has a POSSIBLE_COPYCAT warning, verify it is not a legitimate independent implementation by checking commit history and code differences.

Run the scanners periodically to audit the full catalog:
```bash
python3 scripts/audit_catalog.py          # broad quality/copycat audit
python3 scripts/malware_scan.py           # deep malware/dupe scan (zip-dropper focus)
```

### Step 3: Add to BOF-CATALOG.md

Place each BOF in the correct section:

| Section | Criteria |
|---------|----------|
| `## 🧰 BOF Collections` | Multi-BOF suites/toolkits (5+ BOFs) |
| `## 🤏 Smaller BOF Packs` | 2-5 related BOFs bundled together |
| `## C2 specific BOFs` | BOFs written for a non-Cobalt Strike C2 (Adaptix, Havoc, Sliver, etc.) |
| `## 🧩 Other BOFs` | Individual/single-purpose BOFs (default category) |

Use this table row format:
```
| [RepoName](https://github.com/owner/repo) | Short description of what it does | ![](https://img.shields.io/github/stars/owner/repo?label=&style=flat) | ![](https://img.shields.io/github/last-commit/owner/repo?label=&style=flat) |
```

Or use the helper: `python3 scripts/generate_md.py https://github.com/owner/repo`

### Step 4: Rebuild Search Index and Sync

```bash
python3 scripts/bof_indexer.py
bash scripts/update-site-data.sh
```

### Step 5: Commit and Push

Stage all three files, commit, and push:
```bash
git add BOF-CATALOG.md bof-index.json site/data/bof-index.json
git commit -m "feat(catalog): add N BOFs from weekly discovery issues #X and #Y"
git push
```

If push is rejected (remote has new commits), pull with rebase first:
```bash
git pull --rebase && git push
```

### Step 6: Close the Processed Issues

After pushing, close each triaged issue with a structured summary comment containing two tables — one for added repos and one for skipped repos:

```bash
gh issue close NUMBER --comment "$(cat <<'EOF'
## Triage Summary

### Added (N)
| Repo | Description | Category |
|------|-------------|----------|
| [repo-name](https://github.com/owner/repo) | Short description | Other BOFs |

### Skipped (M)
| Repo | Reason |
|------|--------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chryzsh/awesome-bof](https://github.com/chryzsh/awesome-bof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
