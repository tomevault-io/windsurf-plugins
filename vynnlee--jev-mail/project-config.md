---
trigger: always_on
description: Jev-Mail is a CLI-installed and CLI-managed Gmail classifier. The CLI owns local configuration, Google authorization, Apps Script project upload, deployment, preview, status, and pause/resume operations. Google Apps Script owns the scheduled worker so classification continues when the user's computer is off.
---

# AGENTS.md: Jev-Mail agent operations

Jev-Mail is a CLI-installed and CLI-managed Gmail classifier. The CLI owns local configuration, Google authorization, Apps Script project upload, deployment, preview, status, and pause/resume operations. Google Apps Script owns the scheduled worker so classification continues when the user's computer is off.

The product boundary is deliberately small:

```text
jev-mail CLI -> Google OAuth and Apps Script project -> GAS time trigger -> Gmail + TypeSafe Jev
```

Do not turn this project into a web dashboard, a local daemon, a reply generator, or an agent that mutates Gmail directly. Keep the Gmail worker deterministic around the Jev judgment: validate the typed response, build an explicit decision, check current Gmail state again, then apply the smallest permitted change.

## Security and credentials

Never ask the user to paste a TypeSafe key into chat, a commit, a source file, or a command-line argument.

Use one of these paths:

- Run `jev-mail init` interactively and let its masked prompt collect the key.
- Set `TYPESAFE_API_KEY` in the process environment before `init` in a controlled shell.
- If the user chooses manual setup, have them enter the key directly in Apps Script Project Settings as the `TYPESAFE_API_KEY` Script Property.

Do not print, log, fixture, or snapshot credentials. Redact the key from errors. OAuth client JSON and refresh token files are local secrets and must not be committed.

## Required Google onboarding

For agent-assisted installation, follow [the packaged setup skill](skills/jev-mail-setup/SKILL.md). The user owns their Google Cloud project and account. An agent may prepare local files, validate YAML, run the CLI, read structured diagnostics, and resume the same installation. The user performs Google sign-in and consent, Apps Script editor approval, and masked key entry. An agent may help configure the user-owned Cloud project through an authorized session. Do not borrow a shared OAuth client or ask for credential contents in chat.

The onboarding flow cannot be reduced to a local command because Google requires a browser OAuth approval, a Cloud project link, and one Apps Script editor action.

Before asking the user to run `init`, ensure they understand this sequence:

1. Create or select a standard Google Cloud project.
2. Enable the Apps Script API and Gmail API.
3. Enable Apps Script API in [Apps Script user settings](https://script.google.com/home/usersettings).
4. Configure OAuth consent and add the operating account as a test user when the consent screen is in testing mode.
5. Create and download a Desktop OAuth client JSON from that same Cloud project.
6. Run `jev-mail init --credentials PATH --project-number NUMBER`.
7. Link the created Apps Script project to that exact numeric Cloud project number.
8. Run `installTrigger` once in the Apps Script editor and approve the script scopes.
9. Run the same `init` command again. The CLI verifies the remote setup and stores the TypeSafe key through the authenticated execution API.
10. Run `preview`, inspect the result, then run `enable`.

The numeric project number must belong to the project that owns the Desktop OAuth client. Do not substitute a project ID. The CLI cannot create the installable time trigger through the Apps Script API, so an agent must not claim that setup is complete until the user has run `installTrigger` and `status` reports the expected trigger.

If setup pauses, show the exact browser links and the next required action. Do not retry blindly or create a second project. `--home DIR` creates an intentional separate installation.

Use `doctor --json` to resume: inspect `schemaVersion`, `checks`, and `nextActions`, act on agent-owned steps, and present user-owned steps with their URLs. The default doctor does not change remote state or call the model; OAuth refresh may update the local token file. `--verify-model` is an explicit, potentially billable synthetic check. Do not infer trigger installation or mailbox behavior from a successful upload. If Google returns `org_internal`, inspect the OAuth audience and signed-in account; do not change the audience of an existing shared project without the owner's instruction.

## Configuration protocol

The public configuration is YAML, validated before upload. Use `jev-mail.example.yaml` as the template. The schema contains:

- `version: 1` and `model`.
- `labels.action` and `labels.review`.
- `thresholds.actionRequired`, `actionNotRequired`, `important`, and `categoryConfidence`.
- `categories[]` with `key`, `label`, `description`, `examples`, and `archive`.
- `runtime.batchSize`, `maxScan`, `maxRuntimeSeconds`, `mode`, and `intervalMinutes`.

The default onboarding mode is `label-only`. Keep it enabled for the first preview and enable archive mode only after the user has reviewed representative decisions. Never silently replace a user's YAML with the default taxonomy. Run `config validate` before `config apply` or `init`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vynnlee/jev-mail](https://github.com/vynnlee/jev-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
