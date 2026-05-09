---
trigger: always_on
description: - AgentLimits is a macOS Sonoma+ menu bar app with WidgetKit widgets that display usage limits (Codex/Claude Code) and ccusage token usage.
---

# AgentLimits Contributor Notes

## Purpose
- AgentLimits is a macOS Sonoma+ menu bar app with WidgetKit widgets that display usage limits (Codex/Claude Code) and ccusage token usage.
- The app logs in via an embedded WKWebView and fetches:
  - Codex: `https://chatgpt.com/backend-api/wham/usage`
  - Claude Code: `https://claude.ai/api/organizations/{orgId}/usage`
- ccusage token usage is fetched via CLI:
  - Codex: `npx -y @ccusage/codex@latest daily`
  - Claude Code: `npx -y ccusage@latest daily`
- Each widget reads a provider-specific snapshot from the App Group storage and only renders UI.
- Menu bar displays real-time usage percentages for enabled providers.

## Key Features

### Menu Bar Status Display
- Real-time usage percentage display in menu bar (5h/weekly)
- Two-line layout (line 1: provider name, line 2: `X% / Y%`)
- Color-coded status (used vs pacemaker comparison when available; otherwise secondary)
- Per-provider toggle (Codex/Claude Code separately)
- Responds to display mode changes (used/remaining)
- Pacemaker value shows `<used>% (<pacemaker>)%` with toggleable pacemaker value display (from Pacemaker settings)
- Status colors are customizable from Notification settings
- Menu bar menu includes Display Mode, Language selection, Wake Up → Run Now, and Start app at login

### Pacemaker Mode
- Time-based usage benchmark: calculates elapsed percentage of usage window
- Formula: `(elapsed time / window duration) × 100`
- Compares actual usage against elapsed time to determine if user is on track
- Status levels based on difference (usedPercent - pacemakerPercent):
  - Green: at or below pacemaker (on track)
  - Orange: exceeds pacemaker by warning delta (default: 0%)
  - Red: exceeds pacemaker by danger delta (default: 10%)
- Widget shows dual rings when pacemaker data is available: outer = actual usage, inner = pacemaker percentage
  - When usage exceeds pacemaker in **used mode only**, the outer ring is segmented and color-coded (green → orange → red) to show warning/danger zones (toggleable via `pacemaker_ring_warning_enabled`, enabled by default)
- Menu bar pacemaker indicator display is toggleable (Pacemaker settings), used by widgets as well
- Pacemaker ring/text colors are configurable in Pacemaker settings
- Warning/danger delta thresholds are configurable in Pacemaker settings

### Usage Monitoring
- Auto refresh: configurable 1-10 minutes while the app is running (usage limits)
- Display mode: used% or remaining% (set from menu bar, shared across app + widgets)
- Language preference: stored in App Group under `app_language`
- Usage tab keeps login WKWebView in a bottom collapsible panel (`chevron up/down`), collapsed by default
- Expanded login panel opens upward and can be collapsed by tapping the handle or dimmed background
- Color-coded percentage display in widgets based on usage level and display mode
- Usage screen includes **Clear Data** to remove embedded browser login data and website storage

### Token Usage (ccusage)
- Periodic CLI fetch (Codex/Claude Code) and snapshot persistence
- Separate widgets for Codex/Claude Code token usage (today/this week/this month)
- Per-provider enable/disable with additional CLI arguments support
- **Small widget**: Usage summary only
- **Medium widget**: Usage summary + GitHub-style heatmap
  - 7 rows (Sun-Sat) × 4-6 columns (weeks)
  - 5-level color intensity based on quartile distribution
  - Weekday labels (Mon, Wed, Fri) displayed
  - Desktop pinned mode support (opacity-based white colors)
- Auto refresh: configurable 1-10 minutes (ccusage settings screen)
- Widget tap action is configurable (default opens `https://ccusage.com/` via app deep link)

### Wake Up (CLI Scheduler)
- Schedules CLI commands at user-defined hours via LaunchAgent
- Commands: `codex exec --skip-git-repo-check "hello"` / `claude -p "hello"`
- LaunchAgent plist files: `~/Library/LaunchAgents/com.dmng.agentlimit.wakeup-*.plist`
- Logs: `/tmp/agentlimit-wakeup-*.log`
- Per-provider schedule with additional CLI arguments support

### Threshold Notification
- Sends system notifications (UNUserNotificationCenter) when usage exceeds threshold
- Per-provider settings (Codex / Claude Code separately)
- Per-window settings (5h / weekly separately)
- Default threshold: 90%
- Duplicate prevention: notifies only once per reset cycle (tracked by `lastNotifiedResetAt`)
- Usage color settings (donut + status colors) are configured in Notification settings

### Advanced Settings (CLI Paths / Scripts / Widget Tap)
- Full path overrides for `codex`, `claude`, `npx`
- Resolved PATH results shown in UI
- Bundled status line script path shown with copy action
- Widget tap action configuration (open website / refresh data)

### Claude Code Status Line Script
- Bundled script for Claude Code status line integration
- Reads Claude Code usage snapshot + App Group settings (display mode, language, thresholds, colors)
- Outputs a single line with 5h/weekly usage, reset times, and update time
- Supports overrides: `-ja`, `-en`, `-r` (remaining), `-u` (used), `-p` (pacemaker), `-i` (usage + pacemaker inline), `-d` (debug)
- Requires `jq`

## Key Decisions
- App Group ID: `group.com.dmng.agentlimit`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nihondo/AgentLimits](https://github.com/Nihondo/AgentLimits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
