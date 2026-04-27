---
trigger: always_on
description: `ado-backup` is a CLI tool that backs up Azure DevOps organisations to disk using
---

# CLAUDE.md - Azure DevOps Backup Utility

## Project overview

`ado-backup` is a CLI tool that backs up Azure DevOps organisations to disk using
only the Azure CLI (`az`) as the API transport. It requires no additional Python
dependencies beyond the standard library (PyYAML is optional). The entry point is
`src/cli.py`; run with `python src/cli.py --help`.

## Repository layout

```
src/
  cli.py          # Argument parsing, logging setup, config validation, entry point
  config.py       # BackupConfig dataclass; merges YAML → env vars → CLI args
  orchestrator.py # run_backup(): drives scope calls, handles fail-fast, compression
  azcli.py        # Thin subprocess wrapper around az CLI; git_clone(); retry logic
  backoff.py      # retry() with exponential backoff and jitter
  paginator.py    # paginate() helper for continuation-token APIs (rarely used directly)
  paths.py        # BackupPaths class; safe_name(); parse_org_url()
  writers.py      # Atomic JSON/binary file writes; SHA-256 hashing; restricted perms
  redact.py       # redact_dict(); strips sensitive keys + isSecret pattern before persisting
  compress.py     # tar.gz compression: repos / project / all modes
  inventory.py    # Inventory class: tracks entities, errors, writes manifest + JSONL errors
  verify.py       # verify_backup(): samples backed-up items and checks against live ADO
  scopes/
    org.py            # Organisation metadata
    projects.py       # Project list + metadata
    git.py            # Repo clone (--mirror), branches, tags, policies, repo ACLs
    boards.py         # Work items (WIQL), queries, tags, board config, team settings
    pipelines.py      # Pipeline definitions + build history
    pull_requests.py  # PRs across all repos
    artifacts.py      # Artifact feeds + packages
    dashboards.py     # Dashboard JSON
    permissions.py    # Organisation-level ACLs
    wikis.py          # Wiki metadata + pages
    testplans.py      # Test plans + suites

tests/              # pytest unit tests (no integration tests; mocking az CLI)
examples/           # config.yaml, GitHub Actions, Azure Pipelines workflow examples
dashboard/          # Azure Function + vanilla JS frontend for backup observability
  api/              # Python Azure Functions v2 endpoints (reads _indexes/ from Blob Storage)
  frontend/         # Static HTML/CSS/JS served by the function app
```

## How to run

```bash
# Install (editable)
pip install -e .

# Or run directly (no install needed)
PYTHONPATH=src python src/cli.py --org-url https://dev.azure.com/your-org

# With a config file
python src/cli.py --config examples/config.yaml

# Run tests
pytest
```

## Configuration precedence

CLI args override env vars, which override YAML file values, which override defaults.

| Setting | CLI flag | Environment variable | Default |
|---------|----------|----------------------|---------|
| Org URL | `--org-url` | `AZURE_DEVOPS_ORG_URL` | - (required) |
| PAT | - | `AZURE_DEVOPS_EXT_PAT` or `SYSTEM_ACCESSTOKEN` | - (uses az login) |
| Output dir | `--output-dir` | `ADO_BACKUP_OUTPUT_DIR` | `ado-backup` |
| Timeout (s) | - | `ADO_BACKUP_TIMEOUT` | `120` |

## Output directory structure

```
ado-backup/
  dev.azure.com/
    {org}/
      {YYYYMMDDTHHMMSSZ}/
        _indexes/
          manifest.json          # start/end time, entity counts, limits applied
          inventory.json         # per-file SHA-256 checksums
          errors.jsonl           # one JSON record per error
          verification_report.json  # written when --verify is used
        org/                     # organisation-level data
        projects/
          {project}/
            metadata/
            git/
              repos.json
              {repo}/            # bare git mirror clone
              {repo}_branches.json
              {repo}_tags.json
              {repo}_policies.json
              {repo}_permissions.json
            boards/
              work_items/
                index.json       # list of IDs
                {id}.json        # full work item
                {id}_revisions.json
                {id}/attachments/
            pipelines/
            pull_requests/
            artifacts/
            dashboards/
            wikis/
            test_plans/
```

## Key design decisions

- **No pip dependencies at runtime.** `az devops invoke` is the only API transport.
  PyYAML is imported opportunistically; the code ships its own minimal YAML parser
  as a fallback.
- **Atomic writes.** `writers.write_json` / `writers.write_binary` write to a temp
  file in the same directory and `replace()` atomically. Partially-written files are
  cleaned up on error.
- **Directory permissions.** On Unix, `writers._secure_mkdir` chmod 0o700 newly
  created directories to prevent credential leakage from group-readable mounts.
- **PAT never in argv.** `azcli.git_clone` passes the PAT via `GIT_CONFIG_*`
  environment variables (not on the command line). `_mask_pat` redacts `--pat`
  arguments in debug log output.
- **Sensitive data redaction.** Every scope writes data through `redact.redact()`.
  `redact_dict` catches keys by name (`SENSITIVE_KEYS`), by dot-path
  (`SENSITIVE_PATHS`), and by the ADO `{"isSecret": true, "value": ...}` pattern.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christopher-talke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
