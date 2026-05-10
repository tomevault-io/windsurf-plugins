---
trigger: always_on
description: `calrs` is an open-source scheduling platform written in Rust. It is a self-hostable alternative to Cal.com, starting as a CLI tool before adding a web interface. The project is named **calrs** (potential domain: `cal.rs`).
---

# calrs — Claude Code Context

## Project overview

`calrs` is an open-source scheduling platform written in Rust. It is a self-hostable alternative to Cal.com, starting as a CLI tool before adding a web interface. The project is named **calrs** (potential domain: `cal.rs`).

**Core concept:** Connect your CalDAV calendar(s), define bookable meeting types with availability rules, and eventually share a booking link. No Node.js, no PostgreSQL, no SaaS subscription.

**License:** AGPL-3.0

---

## Tech stack

| Concern | Choice | Notes |
|---|---|---|
| Language | Rust (2021 edition) | Targeting stable |
| CLI | `clap` v4 (derive API) | Subcommand tree pattern |
| Async runtime | `tokio` (full features) | Used throughout |
| Database | SQLite via `sqlx` 0.7 | WAL mode, foreign keys enabled, migrations inlined |
| HTTP client | `reqwest` (rustls, no openssl) | CalDAV PROPFIND/REPORT requests |
| XML parsing | `quick-xml` 0.31 | CalDAV responses are XML over WebDAV |
| iCal | `icalendar` crate | Parsing/generating VEVENT data |
| Time | `chrono` + `chrono-tz` | Timezone handling is a known complexity area |
| IDs | `uuid` v1 | UUID v4 for all primary keys |
| Terminal output | `colored` + `tabled` | Colored text and ASCII tables in CLI output |
| Web server | `axum` 0.8 | HTTP booking page, served from CLI |
| Templates | `minijinja` 2 | Jinja2-compatible, loaded from `templates/` dir |
| Encryption | `aes-gcm` | AES-256-GCM encryption for stored credentials |
| Auth | `argon2` + `password-hash` | Argon2 password hashing for local accounts |
| Auth (OIDC) | `openidconnect` 4.x | OpenID Connect SSO (Keycloak, etc.) with PKCE |
| Sessions | `axum-extra` (cookies) | Server-side sessions in SQLite, HttpOnly cookies |
| Email | `lettre` 0.11 | SMTP with STARTTLS, async tokio transport |
| Logging | `tracing` + `tracing-subscriber` | Structured logging with env-filter |
| HTTP tracing | `tower-http` 0.6 | TraceLayer for request-level observability |
| Error handling | `anyhow` (app-level) + `thiserror` (lib-level) | Standard Rust pattern |
| Config/paths | `directories` crate | XDG-compliant data dir: `$XDG_DATA_HOME/calrs` |

---

## Project structure

```
calrs/
├── Cargo.toml
├── CLAUDE.md                     ← you are here
├── README.md
├── .gitignore
├── migrations/
│   ├── 001_initial.sql           ← full SQLite schema
│   ├── 002_auth.sql              ← users, sessions, auth_config, groups
│   ├── 003_username.sql          ← username column on users
│   ├── 004_oidc.sql              ← OIDC columns on auth_config
│   ├── 005_requires_confirmation.sql ← requires_confirmation on event_types
│   ├── 006_group_event_types.sql ← slug on groups, group_id on event_types, assigned_user_id on bookings
│   ├── 007_caldav_write.sql      ← write_calendar_href on caldav_sources, caldav_calendar_href on bookings
│   ├── 008_recurrence_id.sql     ← recurrence_id column on events
│   ├── 009_uid_recurrence_unique.sql ← composite unique index (uid, recurrence_id) on events
│   ├── 010_confirm_token.sql     ← confirm_token on bookings for email approve/decline
│   ├── 011_event_type_calendars.sql ← junction table for per-event-type calendar selection
│   ├── 012_reminders.sql         ← reminder_minutes on event_types, reminder_sent_at on bookings
│   ├── 013_booking_email.sql     ← booking_email on users
│   ├── 014_team_links.sql        ← team_links, team_link_members, team_link_bookings tables
│   ├── 015_user_profile.sql      ← title, bio, avatar_path on users
│   ├── 016_booking_unique.sql    ← partial unique index for double-booking prevention
│   ├── 017_events_per_calendar.sql ← per-calendar event uniqueness (uid, calendar_id)
│   ├── 018_private_invites.sql   ← is_private on event_types, booking_invites table
│   ├── 019_team_link_reusable.sql ← one_time_use column on team_links
│   ├── 020_booking_attendees.sql ← max_additional_guests on event_types, booking_attendees table
│   ├── 021_accent_color.sql      ← accent_color on auth_config
│   ├── 022_theme.sql             ← theme preset + custom color columns on auth_config
│   ├── 023_team_link_windows.sql ← availability_windows on team_links
│   ├── 024_team_link_features.sql ← location, description, reminder on team_links
│   ├── 025_reschedule_by_host.sql ← reschedule_by_host flag on bookings
│   ├── 026_visibility.sql        ← visibility column on event_types (public/internal/private)
│   ├── 027_calendar_sync_token.sql ← sync_token on calendars
│   ├── 028_company_link.sql      ← company_link URL on auth_config
│   ├── 029_scheduling_mode.sql   ← scheduling_mode on event_types (round_robin/collective)
│   ├── 030_member_weight.sql     ← weight on user_groups for round-robin priority
│   ├── 031_fix_legacy_timezones.sql ← fix bare timezone names to IANA identifiers
│   ├── 032_event_type_member_weights.sql ← per-event-type member weights table
│   ├── 033_group_profile.sql     ← description and avatar_path on groups
│   ├── 034_teams.sql             ← unified teams: teams, team_members, team_groups tables; migrates groups + team_links

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olivierlambert/calrs](https://github.com/olivierlambert/calrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
