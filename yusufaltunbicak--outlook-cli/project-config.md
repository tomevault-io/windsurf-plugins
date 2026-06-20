---
trigger: always_on
description: CLI skill for Outlook 365 to read, send, search, and manage emails, calendar events, categories, and contacts from the terminal without API keys or admin consent
---


# outlook-cli Skill

Use this skill when the user wants to read, send, search, or manage Outlook 365 emails and calendar events from the terminal. Also supports file attachments, follow-up flags, message pinning, opening items in browser, recurring events, shared calendars, free/busy scheduling, categories, contacts, and signatures.

## Prerequisites

```bash
# Install (requires Python 3.10+)
cd ~/outlook-cli && pip install -e .
playwright install chromium
```

## Authentication

- First run: `outlook login` opens Chromium, user logs in, bearer token is auto-captured from OWA requests.
- Named profiles are supported via `outlook account add|list|current|switch|remove`.
- Tokens, browser state, ID maps, scheduled-send tracking, signatures, and per-profile config are isolated per account profile.
- Profile-scoped cache lives under `~/.cache/outlook-cli/accounts/<profile>/`; config lives under `~/.config/outlook-cli/accounts/<profile>/`.
- Existing single-account installs continue to work through the implicit `default` profile and legacy cache paths.
- Auto re-login on 401 is profile-aware.
- `OUTLOOK_TOKEN` is still supported, but if a profile is bound it must match that profile's mailbox.
- Bearer tokens are stored in the OS keychain/keyring; `token.json` remains on disk only as non-secret metadata and is migrated automatically on first use.

```bash
outlook login                              # Interactive browser login
outlook login --force                      # Force re-login, ignore saved session
echo $TOKEN | outlook login --with-token   # Skip browser, read token from stdin
outlook login --with-token < token.txt     # Read token from file
outlook whoami                             # Verify current user
outlook account add work   # Create and bind a named profile
outlook account list
outlook account current
outlook account switch work
outlook whoami --account work
```

### Account Selection

- Every non-account command accepts `--account NAME`.
- Selection precedence is: `--account NAME` → `OUTLOOK_ACCOUNT` → persisted current account → implicit `default`.
- `outlook whoami` shows the active profile in both human and JSON output.
- `--no-input` disables prompts and makes mutating commands fail safely unless `-y` is also provided.
- `--dry-run` previews mutating commands without making API calls.
- `--enable-commands` restricts which top-level commands are allowed for an agent/session.

```bash
outlook inbox --account work
outlook calendar --account personal --days 3
outlook schedule-list --account work
outlook --enable-commands whoami,inbox whoami --account work
```

### Automation / CI Safety

```bash
outlook whoami --json --no-input
outlook delete 3 --no-input              # Fails safely unless -y is provided
outlook send "to@email.com" "Subject" "Body" --dry-run --json
outlook schedule "to@email.com" "Subject" "Body" "+1h" --dry-run
```

## Command Reference

### Inbox

```bash
outlook inbox                                    # List inbox (shows unread/total count)
outlook inbox --max 50                           # Limit count
outlook inbox --unread                           # Unread only
outlook inbox --from "alice.smith"                # Filter by sender
outlook inbox --subject "Q4 Report"              # Filter by subject
outlook inbox --from "acme" --subject "Project"  # Combined filters
outlook inbox --after 2026-03-01                 # After date
outlook inbox --before 2026-03-08               # Before date
outlook inbox --has-attachments                  # Only with attachments
outlook inbox --unread --after 2026-03-09        # Combine any filters
outlook inbox --json                             # JSON output
outlook inbox --json -o emails.json              # Save to file
```

### Read Email

```bash
outlook read 3             # Read message by display number
outlook read 3 --raw       # Show raw HTML body
outlook read 3 --json      # JSON output
```

### Conversation Thread

```bash
outlook thread 3           # Show full conversation for message #3
outlook thread 3 --json    # JSON output
```

### Send / Reply / Forward

```bash
outlook send "to@email.com" "Subject" "Body"                     # Shows confirmation prompt
outlook send "to@email.com" "Subject" --body-file message.txt
printf 'Body from stdin' | outlook send "to@email.com" "Subject" --body-file -
outlook send "to@email.com" "Subject" "Body" -y                  # Skip confirmation
outlook send "a@b.com,c@d.com" "Subject" "Body" --cc e@f.com
outlook send "to@email.com" "Subject" "<h1>Hi</h1>" --html
outlook send "to@email.com" "Subject" "Body" --signature default  # Append saved signature
outlook send "to@email.com" "Report" "See attached" -a report.pdf         # With attachment
outlook send "to@email.com" "Files" "Here" -a file1.pdf -a file2.xlsx     # Multiple attachments

outlook reply 3 "Thanks!"                       # Shows confirmation prompt
printf 'Thanks from stdin' | outlook reply 3 --body-file -
outlook reply 3 "Thanks!" -y                    # Skip confirmation
outlook reply 3 "Noted, will fix." --all         # Reply all
outlook reply 3 "Here it is" -a requested.pdf    # Reply with attachment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusufaltunbicak/outlook-cli](https://github.com/yusufaltunbicak/outlook-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
