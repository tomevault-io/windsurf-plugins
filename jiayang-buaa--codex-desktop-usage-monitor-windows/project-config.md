---
trigger: always_on
description: These instructions apply to the entire repository. They are primarily for Codex or another coding agent helping a Windows user install, configure, update, or customize the monitor.
---

# Codex Instructions

These instructions apply to the entire repository. They are primarily for Codex or another coding agent helping a Windows user install, configure, update, or customize the monitor.

## Install For The Current User

When the user asks to install this project:

1. Confirm that the host is Windows and read `README.md` before acting.
2. Use the checked-out repository or an extracted official Release. Do not download executables or scripts from unrelated locations.
3. Run `pwsh -NoProfile -File .\install.ps1`. The installer checks Microsoft Store and common non-Store Codex paths first; if no executable is found, answer its path prompts instead of guessing. If PowerShell 7 is unavailable, use `powershell -NoProfile -ExecutionPolicy Bypass -File .\install.ps1`.
4. The installer must create a versioned copy under `%LOCALAPPDATA%\Programs\CodexUsageMonitor` and a desktop shortcut named `Codex Usage Monitor`.
5. Verify the installed version directory and shortcut. Its target must be `wscript.exe`, which launches PowerShell invisibly through `launch-codex-monitor-hidden.vbs`. Do not launch, restart, or terminate the user's current Codex session. Tell the user to exit Codex normally and then use the new shortcut.
6. Official subscription monitoring needs no API key or additional configuration.
   For Microsoft Store installations, run the official usage CLI from the versioned user-state mirror under `%LOCALAPPDATA%\CodexUsageMonitor\runtime\codex-cli`; never execute it directly from or modify `WindowsApps`.

## Codex-Assisted Configuration

When installation is being performed by Codex, guide the user through the available
data sources after the launcher has been verified:

1. Ask whether the user wants Official Subscription, API Account, API Key, or any
   combination of the three. Official Subscription requires no credential.
2. For API Account, explain that the current implementation expects a CCTQ-style
   account endpoint, a numeric user ID, and an account access token. Ask the user to
   look for the Base URL in the provider's API/developer documentation, the numeric
   user ID in User Profile, Account Information, or Personal Center, and the account
   access token in User Profile, Security Settings, Access Token, or User Token.
   Names vary by provider; make clear that the account access token may be different
   from an API key. If an item is not visible, ask for the provider name, public
   documentation, or a redacted screenshot instead of guessing. Ask the user to
   copy the complete token to the Windows clipboard and run
   `scripts/configure-api-account.ps1 -FromClipboard -UserId <real-id>`; never ask for
   the token in chat or print it. The script validates `/api/user/self` before saving
   the token with Windows DPAPI.
3. For API Key, ask for the provider's endpoint documentation and response example,
   not the key itself. Explain that API keys are usually created or copied from the
   provider's Developer Console, API Keys, Key Management, or Token Management page,
   while the usage endpoint is usually documented under Usage, Quota, Billing, or
   API documentation. Copy the key through the clipboard and use the appropriate
   `configure-*.ps1 -FromClipboard` command. Keep the key in DPAPI and keep the
   provider JSON limited to endpoint and field mappings. If the provider uses
   unfamiliar labels, ask for public documentation or a redacted screenshot and do
   not infer that an account access token and an API key are interchangeable.
4. Ask for the user's current real cumulative Token count when API Account is
   enabled. It must be a complete non-negative integer, without `万` or `亿`.
   Explain that this value is normally shown on the provider's Usage Statistics,
   Billing, Consumption Records, or Token Statistics page. If only a rounded value
   is visible, ask the user to open its details or export rather than inventing a
   precise number. Run
   `scripts/configure-token-baseline.ps1 -InitialTokens <value>` after the account is
   configured. If the user does not know the value, explicitly explain that `0` is
   used and new visible logs are accumulated from there. This baseline is needed
   because the upstream log endpoint exposes only a limited page of history.
   The same counter file stores a date-scoped, deduplicated daily Token total;
   preserve it when reinstalling or updating so a Codex/Windows restart does not
   reset the current day's value.
   When upgrading from version 1.7.0 or earlier, ask for the current real cumulative
   Token value and run the baseline command again because the older row-ID ledger
   cannot be repaired automatically.
5. If an API Account or API Key response does not match the example schema, inspect
   the documented/sanitized response shape without exposing credentials. Adapt the
   local account normalization/request mapping or the local Provider JSON selectors,
   add or update a fixture test for the new shape, and run the full test suite. Do not
   silently guess fields, weaken URL/authentication validation, or commit a user's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JiaYang-BUAA/Codex-Desktop-Usage-Monitor-Windows](https://github.com/JiaYang-BUAA/Codex-Desktop-Usage-Monitor-Windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
