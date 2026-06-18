---
trigger: always_on
description: This is the platform for a grant/hackathon program for students called Fallout, where they can log and submit hours they spend on projects & their engineering process for prizes and and invite to our summit event.
---


# General

This is the platform for a grant/hackathon program for students called Fallout, where they can log and submit hours they spend on projects & their engineering process for prizes and and invite to our summit event.

Keep changes low impact, responses concise. No summaries, no testing. Reference the existing codebase for style consistency. Read all context carefully before making changes — code may be manually modified between messages; do not suggest code that has been deleted or is no longer relevant. If asked to change feature requirements, update all previous implementations to match. Always ask questions when needed.

# Architecture Documentation

Detailed architecture docs live in [agents-docs/](agents-docs/INDEX.md). Before working on an unfamiliar area, scan the index and read the relevant doc — they contain gotchas, patterns, and implementation details that prevent common mistakes. The docs are point-in-time snapshots and may be out of date; treat them as a starting overview and verify against the current code before relying on specifics.

!!! IMPORTANT: When you make changes that affect documented architecture (new controllers, models, policies, services, access control changes, new shared props, etc.), YOU MUST update the corresponding doc in `agents-docs/` as part of the change.

# Stack

Ruby 3.4.4, Rails 8.1.3, React 19, Tailwind 4.1.18 via inertia-rails. Only suggest changes applicable to these versions. Prefer CLI-generated boilerplate over manual file creation — you can always modify generated output.

In-house services: HCA is our unified authentication system. Hackatime is the time tracking system, where Lapse is the timelapse tool for Hackatime. HCB is our "bank" (real US dollars).

Inertia bridges Rails and React and allows for SPA behavior with <Link> and Inertia Modals All attributes passed to the frontend — even unused ones — are visible in developer tools; for security & access reasons, be careful what you expose. Inertia docs: https://inertia-rails.dev/llms-full.txt

# Security & Access Control

HCB controls money for the program. **DO NOT EDIT ANY CODE RELATED TO HCB WITHOUT EXPLICIT WRITTEN APPROVAL.** Alert in chat before making any HCB changes. Do not run any tests or console code related to HCB without **EXPLICIT WRITTEN APPROVAL**.

## Pundit

Pundit policies enforce authorization at a low level and should always be used. This pertains to security — if unsure how to modify a policy, ask for clarification. Follow the principle of least privilege: only grant access necessary for the feature to function. Docs: https://www.rubydoc.info/gems/pundit

## User Types

Two user types exist: **full users** (authenticated through HCA, cross-device access, can access non-public data) and **trial users** (email-based login, device-cookie-scoped, limited access). For privacy and security, multiple trial accounts with the same email cannot access each other's data. Trial users become full users upon completing HCA authentication. Consider both user types when making changes and enforce access controls via Pundit when making changes.

## Staff Roles & PII

Staff users have one or more roles stored in a PostgreSQL array column: `time_auditor`, `requirements_checker`, `pass2_reviewer`, `admin`, or `hcb`. Each reviewer role grants access only to its specific review queue(s) — use `user.can_review?(queue)` to check. Admins have access to everything except real money movement, which is reserved for the `hcb` role: only users with `user.hcb?` true can issue or top up HCB project funding card grants. Regular admins can read grant orders, edit `HcbGrantSetting`, adjust admin notes, and move orders to pending/on_hold/rejected — but cannot transition an order to `fulfilled` (which triggers an HCB topup) or mark a pending topup as completed during reconciliation.

**PII (email, full name, etc.) must only be exposed to admins.** Non-admin reviewers see display names and avatars but never email addresses or other identifying information. When serializing user data for the admin frontend, always check `current_user.admin?` before including PII fields. The `/admin/users` pages are already admin-only via `require_admin!`.

## Fail-Closed Defaults

By default, every action requires full HCA authentication, completed onboarding, and Pundit authorization. Only relax defaults when explicitly necessary, for specific actions only. When in doubt, deny access. Assume developers will forget to configure access on new actions — the system must fail closed.

## `only:` vs `except:` Rule

If a developer forgets to list a new action, the result must always be _less_ access, never _more_:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackclub/fallout](https://github.com/hackclub/fallout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
