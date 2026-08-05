---
trigger: always_on
description: > **Never run git write commands** (`git commit`, `git push`, `git tag`, `git rebase`, `git merge`, `git reset`, `git stash`, etc.) unless the user **explicitly requests** it. Only read-only git commands (`git status`, `git diff`, `git log`) may be run freely.
---

# AI Agent Instructions

---

> [!CAUTION]
> **Never run git write commands** (`git commit`, `git push`, `git tag`, `git rebase`, `git merge`, `git reset`, `git stash`, etc.) unless the user **explicitly requests** it. Only read-only git commands (`git status`, `git diff`, `git log`) may be run freely.

---

## Project Overview

Go-based terminal UI application for Microsoft Teams. Authenticates via OAuth2 Device Code Flow and displays chats and messages using the Microsoft Graph API. Built with the Bubble Tea TUI framework (MVU architecture).

---

## Key Architecture

### Authentication (`auth.go`)
- OAuth2 Device Code Flow with Microsoft Graph API
- Tokens stored in `~/.cache/teams-tui-go/token.json`
- Auto-refreshes expired tokens using `GetValidTokenSilent(clientID)`
- Client ID loaded in order: `.env` → `config.json` → built-in default
- **All background API calls must use `GetValidTokenSilent()`**, never the cached `accessToken` from startup
- **Dynamic scopes**: `StartDeviceFlow(clientID, scopes string)` and `RefreshAccessToken(clientID, refreshToken, scopes string)` accept an explicit scope string. Both callers pass `BuildScopes()` so that any enabled feature flags are included in the token request. The old `scopes` constant has been removed.

### Configuration (`config.go`)
- App data: `~/.config/teams-tui-go/` (via `GetAppDir()`)
- Cache: `~/.cache/teams-tui-go/` (via `GetCacheDir()`)
- Config struct: `ClientID *string`, `NotificationMode *NotificationMode`, `NotificationShowPreview *bool`, `NotificationPreviewLen *int`, `MessageLimit *int`, `SearchContextLimit *int`, `ChatLimit *int`, `ChatIconTheme *string`, `CustomChatIcons map[string]string`, `ExternalEditor *string`, plus six optional feature flags: `FilePreviewEnabled`, `PresenceEnabled`, `UserProfileEnabled`, `UserProfileExtended`, `TeamsChannelsEnabled`, `ChannelMentionsEnabled`
- `ResolveClientID()`, `ResolveMessageLimit()`, `ResolveSearchContextLimit()`, `ResolveChatLimit()`, and `ResolveExternalEditor()` implement the full precedence chain
- `InitConfig()` is run at application startup to populate any missing configuration keys in `config.json` with their default values and persist them to disk. It defaults `ChatIconTheme` to `"unicode"` and all feature flags to `false`.
- `BuildScopes()` assembles the OAuth2 scope string dynamically: always includes the four basic scopes (`User.Read Chat.ReadWrite offline_access`) and appends optional scopes for each enabled feature flag.
- Six `ResolveFeatureXxx()` helpers (one per feature) read the config and return a bool, used by `BuildScopes()` and during startup to populate `App.Features`.

### API Layer (`api.go`)
- **User Detection**: Identifies the current user by counting name frequency across `oneOnOne` chats
- **Display Names**: Pre-computed in `GetChats()` and stored in `Chat.CachedDisplayName` — **never compute display names in the UI layer**
- **Name Abbreviation**: Group chat members shown as "FirstName LastInitial" (`abbreviateName()`)
- **Filtering**: Current user is automatically filtered from all member lists by name match (not by ID — IDs vary per chat)
- **HTMLToText**: Uses `golang.org/x/net/html` tokenizer for robust HTML-to-text conversion, handling `<img>`, `<attachment>`, `<emoji>`, block elements, HTML entities
- **Read State**: `Chat` includes `Viewpoint` containing `LastMessageReadDateTime` from the server
- **Silent errors**: `GetChatMembers()` returns empty slice on error; `MarkChatAsRead()` silently ignores all errors

### Application State (`app.go`)
- `App` struct holds all runtime state: chats, messages, selection, input mode, notification mode, etc.
- `NotificationMode` enum is JSON-serialised as a string ("None", "Console", "System", "Both")
- `CurrentUserName` is used for filtering and message alignment; it is **not displayed in the UI**
- `FeatureFlags` struct (populated once at startup in `main.go` from `ResolveFeatureXxx()`) exposes booleans for each optional feature. **Always read feature state from `app.Features`** — never call `ResolveFeatureXxx()` inside the Bubble Tea event loop.
- New optional-feature popup / state fields on `App`: `PresencePopupMode`, `PresenceData`, `PresenceLoading`, `PresenceUserName`; `UserProfilePopupMode`, `UserProfileData`, `UserProfileLoading`; `AttachmentCursorMode`, `AttachmentSelectedIndex`; `TeamsData []TeamWithChannels`, `TeamsDataLoading`, `SelectedChannelTeamID`, `SelectedChannelID`; `HelpPopupMode`; `MentionPopupMode`, `MentionSearch`, `MentionSelectedIndex`, `MentionSuggestions`, `MentionStartIndex`, `TeamMembersCache`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nospor/teams-tui-go](https://github.com/nospor/teams-tui-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
