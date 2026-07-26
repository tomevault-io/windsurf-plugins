---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

codebar planner is a Rails 8.1 application for managing [codebar.io](https://codebar.io) members and events. It handles workshop/event scheduling, member registration, invitations, RSVPs, and feedback collection for coding workshops organized by codebar chapters.

## Development Workflow

**CRITICAL**: Never work directly on the `master` branch. All changes must be made via pull requests from feature branches.

### Branching Strategy

1. **Create a feature branch** from `master`: `git checkout -b feature/descriptive-name`
2. **Make your changes** and commit them to the feature branch
3. **Push the branch** to the remote repository
4. **Create a pull request** for review
5. **Wait for approval** before merging to `master`

**Never:**
- Commit directly to `master`
- Push directly to `master`
- Merge without review

## Development Setup

**IMPORTANT**: Always use native installation with `bundle exec` commands. Never use Docker or `bin/d*` commands.

### Prerequisites

- **mise** — install via `brew install mise`, then enable `mise activate` in your shell profile (see [mise docs](https://mise.jdx.dev/getting-started.html)). Alternatively, use `mise exec` to run commands with project env vars without shell activation.
- PostgreSQL running locally (default port 5432, or configure via `DB_PORT`)

### Native Installation

```
cp mise.local.toml.example mise.local.toml
# Edit mise.local.toml with your GitHub OAuth credentials
# Optionally delete .env if it exists from a prior setup:
rm .env
bundle && rake db:create db:migrate db:seed
```

- **Server**: `bundle exec rails server`
- **Tests**: `make test` or `bundle exec parallel_rspec spec/ -n 3` - runs RSpec tests in parallel (3 processes is optimal)
- **Single test**: `bundle exec rspec spec/path/to/file_spec.rb:42`
- **Rails console**: `bundle exec rails console`
- **Run rake tasks**: `bundle exec rake [task]`
- **Linting**: `bundle exec rubocop`

### Environment Variables

Managed by `mise` via `mise.toml` (shared, committed) and `mise.local.toml` (secrets, gitignored). Automatically loaded when you enter the project directory (requires `mise activate`).

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_HOST` | `localhost` | Postgres host |
| `DB_PORT` | `5432` | Postgres port |
| `DB_USER` | (empty) | Postgres user |
| `POSTGRES_PASSWORD` | (empty) | Postgres password |
| `GITHUB_KEY` | — | GitHub OAuth client ID (set in `mise.local.toml`) |
| `GITHUB_SECRET` | — | GitHub OAuth client secret (set in `mise.local.toml`) |

Create GitHub OAuth app at https://github.com/settings/applications/new with callback URL `http://localhost:3000/auth/github`.

### Docker Setup (Not Used)

Docker setup exists in this repository (`bin/d*` commands) but is **not used** for development work with Claude Code, OpenCode, etc.

## Architecture & Domain Model

### Core Domain Concepts

**Members**: Users of the system. Can be students, coaches, both, or neither. Authenticated via GitHub OAuth (stored in `auth_services` table). Members have roles managed by Rolify (`admin`, `organiser` for chapters/workshops).

**Chapters**: Local codebar organizations (e.g., "London", "Berlin"). Chapters have organisers and host workshops/events.

**Workshops**: Regular coding workshops. Belong to one chapter. Send invitations to chapter subscribers. Attendance is first-come-first-served up to venue capacity, with automatic waiting list management.

**Events**: Multi-chapter events. Attendance requires admin verification/approval after RSVP.

**Sponsors**: Organizations providing venue space. Have addresses and member contacts. One sponsor acts as "host" (venue) for each workshop.

**Invitations**: Track member attendance status for workshops/events. Different classes:
- `WorkshopInvitation` - for workshops (auto-accepted up to capacity)
- `Invitation` - for events (require admin verification)

**Waiting Lists**: When workshops are full, members can join waiting list (`WaitingList` model with `auto_rsvp` flag). Automatically promoted when spaces become available.

### Key Model Relationships

```
Chapter
  has_many :workshops
  has_many :groups (for subscriptions)
  has_many :organisers (via permissions)

Workshop
  belongs_to :chapter
  has_many :workshop_sponsors
  has_many :invitations (WorkshopInvitation)
  has_one :host (sponsor where workshop_sponsors.host = true)

Member
  has_many :workshop_invitations
  has_many :invitations (for events)
  has_many :subscriptions
  has_many :groups, through: :subscriptions
  has_many :chapters, through: :groups
  has_many :auth_services

Sponsor
  has_one :address
  has_many :workshop_sponsors
  has_many member_contacts
```

See `app/models/README.md` for detailed data model documentation.

## Authorization & Authentication

- **Authentication**: GitHub OAuth via OmniAuth. Session stores `member_id`.
- **Authorization**: Pundit policies in `app/policies/`. Key roles:
  - `admin` - global admin access
  - `organiser` - per-chapter or per-workshop organiser role
- Access checks: `current_user.is_admin?`, `current_user.manager?` (admin or organiser)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codebar/planner](https://github.com/codebar/planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
