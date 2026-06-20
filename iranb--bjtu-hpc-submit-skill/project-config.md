---
trigger: always_on
description: Use when an agent needs to refresh/save BJTU HPC portal auth, add or switch BJTU portal accounts/tokens, run a local transfer dashboard, upload/download files, reuse shared datasets across accounts, copy account-local runtime environments, schedule two execution-slot experiments plus two queued follow-up experiments per saved account, submit CPU/GPU jobs, inspect job status including GPU counts, monitor resumable dataset uploads, or probe the runtime environment from a BJTU HPC helper workspace.
---


# BJTU HPC Submit

Tool-first workflow for BJTU HPC portal work from a local helper workspace. This public version is sanitized: replace placeholder paths, account names, and project directories with your own local values before use.

## Runtime Defaults

- Work from the helper workspace unless the project says otherwise:

```bash
PY=/path/to/python3
SLURM_DIR="/path/to/bjtu-hpc-helper"
PROJECT_DIR="/path/to/your/project"
```

- Prefer the same Python environment used to install the helper dependencies. If dependencies are missing, install the helper requirements and Playwright Chromium:

```bash
cd "$SLURM_DIR"
"$PY" -m pip install -r requirements.txt
"$PY" -m playwright install chromium
```

- When working from a project, save portal and Slurm evidence under a project-local log directory such as `$PROJECT_DIR/hpc_stdout/`.
- Use broad keywords for general queue checks. Use narrow keywords only for targeted follow-ups.

## Entry Points

- Start with `hpc_doctor.py --json`; it checks dependencies, account state, browser profile, and token validity without printing secrets.
- For a local GUI, run `hpc_transfer_web.py` from the helper workspace and open the reported localhost URL.
- Upload and download through helper wrappers such as `hpc_upload.py` and `hpc_download.py`; include `--auth-account <name>` when scripts support it.
- Use portal-app submit wrappers only for resource-shape-compatible jobs or lightweight probes. Prefer verified wrappers over raw submit scripts when a portal app is used.
- For CPU/GRES-sensitive jobs, use uploaded native `sbatch` scripts through the portal SSH path instead of portal-generated PyTorch app scripts, then verify native Slurm allocation.
- For MCP clients, prefer tools that expose auth status, submit-and-verify, pending reason, allocation verification, stdout tailing, and SFTP info.

Useful status commands:

```bash
cd "$PROJECT_DIR"
"$PY" "$SLURM_DIR/hpc_jobs.py" list --keyword <keyword> --size 30 --paths
"$PY" "$SLURM_DIR/hpc_jobs.py" list --keyword <keyword> --size 30 --paths --json > hpc_stdout/bjtu_jobs_YYYYMMDD_HHMM.json
"$PY" "$SLURM_DIR/hpc_pending_reason.py" <slurm_job_id> --no-sinfo
```

## Auth

- Saved accounts usually live in `~/.bjtu_hpc_accounts.json`; a legacy single-token cache may live in `~/.bjtu_hpc_token`.
- Treat the saved account store as the source of truth; the legacy file is only a compatibility cache for older scripts.
- Select accounts with `--auth-account <name>` or `HPC_AUTH_ACCOUNT=<name>`.
- Never print portal tokens, cookies, temporary certificates, passwords, or captured browser storage.
- Treat portal codes `11009`, `11011`, and `11012` as expired or invalid auth.
- Treat portal HTTP `401`, token-validation transport errors, and missing profile tokens as auth-blocked for user-requested live status until a fresh validation succeeds. Stale snapshots may be reported only as `last trusted`.
- If the user explicitly asks for a captcha/verification-code-only login flow, save CAS login credentials only through the local helper, with user-specific values supplied at runtime:

```bash
cd "$SLURM_DIR"
"$PY" hpc_credentials.py set <account_name> --login-name <portal_user>
"$PY" hpc_credentials.py list
```

  The helper should store credentials only on the controller machine with restrictive file permissions. Never commit credentials, passwords, browser storage, portal tokens, or temporary certificates to Git.
- Auth refresh is not an experiment launch. If an expired token blocks a user-requested BJTU status, progress, upload, download, pending-reason, or submit check, run the integrated visible refresh flow immediately unless the user explicitly forbids token refresh or browser use.

### Multi-Account Tokens

Use `hpc_accounts.py` for account-local tokens instead of copying a legacy token file between users:

```bash
cd "$SLURM_DIR"
"$PY" hpc_accounts.py list
"$PY" hpc_accounts.py add <account_name> --refresh --browser playwright --fresh-page --timeout 600
"$PY" hpc_accounts.py refresh <account_name> --browser playwright --headless --fresh-page
"$PY" hpc_accounts.py validate <account_name>
"$PY" hpc_accounts.py use <account_name>
```

- Adding or refreshing an account should discover the portal user, cluster, and cluster OS account from the portal token when the helper supports it. Do not copy metadata from another saved account unless the user explicitly provides it.
- Do not sync a secondary account into `~/.bjtu_hpc_token` unless the user intentionally wants to change the legacy default.
- Use `--auth-account <account_name>` on every submit, job-list, upload, download, or proxy-info command in a multi-account workflow.

### Auth Recovery State Machine


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Iranb/bjtu-hpc-submit-skill](https://github.com/Iranb/bjtu-hpc-submit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
