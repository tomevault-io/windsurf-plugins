---
trigger: always_on
description: >-
---


# Cloud Bootstrap

## Overview

This skill manages encrypted cloud-provider credentials (GCP, AWS, Azure) stored directly in a repository for Claude Code on the Web. When activated, it detects the current authentication state — whether this is a first-time setup, a new team member joining, or a returning session that needs re-authentication — and executes the appropriate workflow. It handles the full credential lifecycle: initial setup, team member onboarding, session authentication, key rotation, permission escalation, multi-provider configuration, and cleanup.

**Requires:** An encryption passphrase in one of these environment variables (checked in order):
- `GCP_CREDENTIALS_KEY`, `AWS_CREDENTIALS_KEY`, or `AZURE_CREDENTIALS_KEY` (provider-specific)
- `CLOUD_CREDENTIALS_KEY` (universal fallback)

Each team member sets their own passphrase. Passphrases are never shared between users.

## Identify Current User

```bash
USER_EMAIL=$(git config user.email)
if [ -z "$USER_EMAIL" ]; then
  echo "ERROR: git user.email is not set."
  exit 1
fi
```

This email is used to name the per-user encrypted credentials file: `.cloud-credentials.<email>.enc`

## Resolve Credentials Key

Use this logic everywhere the encryption key is needed. Determine the provider from context (the user's request during setup, or `.cloud-config.json` in subsequent sessions), then resolve:

```bash
resolve_credentials_key() {
  local provider="$1"  # gcp, aws, or azure
  case "$provider" in
    gcp)   KEY="${GCP_CREDENTIALS_KEY:-$CLOUD_CREDENTIALS_KEY}" ;;
    aws)   KEY="${AWS_CREDENTIALS_KEY:-$CLOUD_CREDENTIALS_KEY}" ;;
    azure) KEY="${AZURE_CREDENTIALS_KEY:-$CLOUD_CREDENTIALS_KEY}" ;;
    *)     KEY="$CLOUD_CREDENTIALS_KEY" ;;
  esac
  if [ -z "$KEY" ]; then
    echo "ERROR: No credentials key found."
    echo "Set ${provider^^}_CREDENTIALS_KEY or CLOUD_CREDENTIALS_KEY."
    return 1
  fi
  echo "$KEY"
}
```

## Quick Check: Which Phase Am I In?

Determine the current user's email, then:

1. If `.cloud-config.json` does NOT exist → read `workflows/first-time-setup.md`
2. If `.cloud-config.json` exists, check for the user's encrypted credentials file. The file may use **either** naming convention:
   - Single-provider: `.cloud-credentials.<user-email>.enc`
   - Multi-provider: `.cloud-credentials.<provider>.<user-email>.enc`

   To detect multi-provider mode, check whether `.cloud-config.json` contains a `providers` array:
   ```bash
   if jq -e '.providers' .cloud-config.json >/dev/null 2>&1; then
     # Multi-provider: check for .cloud-credentials.<provider>.<email>.enc for each provider
     PROVIDERS=$(jq -r '.providers[].provider' .cloud-config.json)
   else
     # Single-provider: check for .cloud-credentials.<email>.enc
   fi
   ```
   If **no** matching credential file exists for the current user → read `workflows/add-team-member.md`

   In multi-provider mode, evaluate **each** provider independently: if the
   current user is missing the credential file for *any* configured provider,
   treat that provider as an add-team-member case (read
   `workflows/add-team-member.md` for it) — even if files for other providers
   already exist. Do not short-circuit to authenticate just because one match
   was found, or the missing provider is silently skipped until a command fails.
3. If a matching credential file exists for every configured provider → read `workflows/authenticate.md`

For other operations, read the corresponding workflow file in this skill's `workflows/` directory:
- **Permission escalation** (403 / access denied errors) → `workflows/permission-escalation.md`
- **Credential rotation** (age warning, suspected compromise) → `workflows/credential-rotation.md`
- **Multi-provider setup** (adding a second cloud provider) → `workflows/multi-provider.md`
- **Uninstall** (removing cloud-bootstrap from the repo) → `workflows/uninstall.md`

Read **only** the workflow file you need. Do not read all of them.

---

## Proactive Suggestions

When cloud credentials are active, periodically consider whether cloud services could improve the current workflow:

- **Repeated file processing** → suggest cloud storage (GCS, S3) or managed database (BigQuery, Athena)
- **Long-running tasks** → suggest a cloud VM with appropriate resources
- **Manual recurring tasks** → suggest a scheduled cloud function
- **File sharing friction** → suggest cloud storage with shareable links
- **Growing datasets** → suggest migrating from flat files to a managed database

Frame suggestions as questions, not directives. Let the user decide.

---

## Output Format

When executing any workflow, follow these communication standards:

- **Before each major step:** Tell the user what you are about to do and why, in one sentence.
- **After each major step:** Confirm what happened. Show the command output if relevant.
- **For errors:** State what failed, what the likely cause is, and what the user should do next. Do NOT guess or retry silently.
- **For credential operations:** Always confirm the file name and path of any file created, encrypted, or deleted.
- **Final summary:** After completing a workflow, provide a short checklist of what was done and what the user should verify.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ipeirotis/cloud-bootstrap](https://github.com/ipeirotis/cloud-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
