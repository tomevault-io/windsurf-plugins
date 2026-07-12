---
trigger: always_on
description: This repository is a structured archive of security research Proof-of-Concept entries.
---

# Copilot Instructions — POC Archive

This repository is a structured archive of security research Proof-of-Concept entries.
Each POC lives in `pocs/<category>/<YYYY-MM-DD>_<vuln-name>/` and follows a strict template.

---

## Repository layout

```
pocs/
  web / network / binary / crypto / cloud / hardware / social-engineering / misc
    <YYYY-MM-DD>_<vuln-name>/
      README.md          ← filled-in copy of templates/POC_TEMPLATE.md
      exploit.*          ← copied exploit files from the source repo
      screenshots/
      references/
archive/
  YYYY.md                ← auto-generated, one file per CVE year (do not edit manually)
templates/POC_TEMPLATE.md
scripts/index.sh         ← regenerates INDEX.md and archive/YYYY.md
INDEX.md                 ← current CVE year only, auto-generated, do not edit manually
```

---

## How to ingest a POC from a GitHub URL

When an issue contains a GitHub repository URL (e.g. `https://github.com/owner/repo`),
do the following:

### 1 — Fetch repository information

Use `curl` or Python to call the GitHub API and collect:
- Repository name, description, topics/tags
- README content
- Primary language(s)
- Stars, license

```bash
curl -s https://api.github.com/repos/OWNER/REPO
curl -s https://api.github.com/repos/OWNER/REPO/topics \
  -H "Accept: application/vnd.github.mercy-preview+json"
```

Decode the README:
```bash
curl -s https://api.github.com/repos/OWNER/REPO/readme \
  | python3 -c "import sys,json,base64; d=json.load(sys.stdin); print(base64.b64decode(d['content']).decode())"
```

### 2 — Shallow-clone the repo

```bash
git clone --depth=1 https://github.com/OWNER/REPO /tmp/poc-source
```

List the exploit-relevant files (`.py`, `.sh`, `.go`, `.rb`, `.c`, `.cpp`, `.js`, `.ts`, `.rs`).

### 3 — Determine metadata

From the README, description, topics, and file content, determine:

| Field | How to decide |
|---|---|
| **category** | `web` for HTTP/browser vulns; `network` for protocol/packet; `binary` for BOF/heap/ROP; `crypto` for cipher/oracle flaws; `cloud` for AWS/GCP/Azure; `hardware` for firmware/side-channel; `social-engineering` for phishing; `misc` for anything else |
| **severity** | Critical (CVSS 9-10), High (7-8.9), Medium (4-6.9), Low (1-3.9), Informational |
| **cvss_score** | Extract from README/advisory or estimate. Use "N/A" if unknown |
| **cve** | Extract CVE-YYYY-XXXXX from README/topics. Use "N/A" if none |
| **status** | Weaponized if working exploit code exists; Researched if write-up only; Patched/Unpatched based on advisory; Unknown otherwise |
| **tags** | Comma-separated: exploit technique + affected tech + auth level (e.g. `RCE, unauthenticated, nginx, path-traversal`) |
| **related** | If this is a resurface of a known CVE, set to the path of the original entry (e.g. `pocs/web/2021-12-10_log4shell/`). Otherwise `N/A`. |
| **last_updated** | Date of the most recent commit to the upstream repo that changed exploit code (not just README edits). Use `git log --format="%ad" --date=short -- <exploit-files> \| head -1` on the cloned repo. Use `N/A` if indeterminate. |

### 4 — Create the folder

```
pocs/<category>/YYYY-MM-DD_<vuln-name>/
```

- Date = today's date in `YYYY-MM-DD` format
- `vuln-name` = short kebab-case name derived from the repo name or CVE
  (e.g. `nginx-rift-path-traversal`, `log4shell-bypass`, `openssl-heartbleed`)

### 5 — Write README.md

Copy `templates/POC_TEMPLATE.md` into the folder as `README.md`.
Fill in **every field** in the metadata and affected-target tables.
Write the summary, root cause, attack vector, and impact sections based on what you learned from the source repo.
In the References section, always include the source repo URL.
In the Notes section, add: `Auto-ingested from https://github.com/OWNER/REPO on YYYY-MM-DD.`

### 6 — Copy exploit files

Copy the exploit-relevant files from the cloned repo into the POC folder.
Keep original filenames. Do not copy binaries, lock files, or `.git/` contents.

### 7 — Update INDEX.md and archives

Run the index script:

```bash
bash scripts/index.sh
```

This regenerates `INDEX.md` and `archive/YYYY.md` files, grouped by **CVE year** (not Date Added).
- Entries with a CVE ID go into the file matching that CVE's year (e.g. `CVE-2024-21338` → `archive/2024.md`)
- Entries with no CVE fall back to the Date Added year
- `INDEX.md` contains only the current calendar year's CVEs; all older CVE years live in `archive/`

### 8 — Commit

Stage and commit:

```bash
git add pocs/ INDEX.md archive/ docs/data.json
git commit -m "feat: ingest <vuln-name> from <owner>/<repo>"
```

Then open a PR targeting `main` with:
- **Title:** `[POC] <Display Name> — <CVE or N/A>`
- **Body:** summary table (category, severity, CVE, tags, source URL) + note that human review is needed before merging

---

## How to ingest a resurface PoC (same CVE, new researcher)

A **resurface** is when a new GitHub repository implements a PoC for a CVE that already exists in the archive — written by a different researcher or using a different technique. Treat it as a **new ingest**, not an update.

### 1 — Detect the overlap

Before ingesting, search the archive for the CVE:

```bash
grep -rl "CVE-YYYY-XXXXX" pocs/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intelseclab/poc-archive](https://github.com/intelseclab/poc-archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
