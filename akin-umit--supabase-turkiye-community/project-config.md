---
trigger: always_on
description: This file applies to every human contributor, AI coding agent, and AI-assisted developer working in this repository. Read it before inspecting, editing, generating, committing, reviewing, or publishing changes.
---

# Supabase Turkiye Community - AI Contributor Instructions

This file applies to every human contributor, AI coding agent, and AI-assisted developer working in this repository. Read it before inspecting, editing, generating, committing, reviewing, or publishing changes.

## Project Identity

- This is an independent community project for Turkish-first Supabase documentation and reusable self-hosting improvements.
- It is not Supabase Cloud, is not maintained by Supabase, and must not claim official partnership, endorsement, or regional representation.
- Preserve attribution to `supabase/supabase`, Apache-2.0 notices, third-party licenses, and trademark boundaries.
- Public examples must use neutral values such as `supabase.example.com`, `app.example.com`, `YOUR_SECRET`, and `YOUR_SERVER_IP`.

## Turkish-First Documentation Rule

- The primary audience of this repository is Turkish-speaking self-host users.
- New documentation must be written in Turkish first. English may be added as a secondary translation, summary, or `*.en.md` mirror.
- Do not make the English version richer than the Turkish version. If a concept, warning, setup step, limitation, or troubleshooting note exists in English, the Turkish document must contain the same practical information.
- Main repository entry points must prefer Turkish labels and Turkish reading order: `README.md`, `docs/TURKCE-KURULUM.md`, `docs/TROUBLESHOOTING.md`, `PLATFORM-CAPABILITIES.md`, `DASHBOARD-ROADMAP.md`, and `CHANGELOG.md`.
- English files are kept for international contributors and upstream context, but they do not replace the Turkish guide.
- When only one language can be updated in a small emergency fix, update Turkish first and open a follow-up item for the English mirror.
- AI agents must not leave Turkish users with only an English explanation for installation, security, backup, migration, dashboard limitations, or error recovery.

## Mandatory Start Check

1. Read `AGENTS.md`, `README.md` or `README.en.md`, `CONTRIBUTING.md`, `SECURITY.md`, and the task-specific document.
2. Run `git status --short`, `git branch -vv`, and `git remote -v`.
3. Identify whether the change is documentation-only, deployment-only, security-sensitive, or an upstream candidate.
4. Search existing Issues, Discussions, and pull requests before duplicating work.
5. Preserve unrelated changes and keep the patch narrowly scoped.
6. If the task depends on earlier private validation, public plans, or forum
   updates, read `COMMUNITY-PUBLICATION-FLOW.md` before editing.

## AI and AI-Assisted Contributions

- AI-generated code and documentation are allowed, but the human contributor remains responsible for every submitted line.
- Pull requests must disclose meaningful AI assistance and name the verified areas, not private prompts or credentials.
- Do not claim a command, deployment, migration, restore, or test was executed when it was only suggested or simulated.
- Do not use chat history as the durable project memory. Important decisions,
  phase status, forum-publication plans, and private-to-public promotion rules
  must be written into repository documents.
- Review generated commands for destructive behavior, platform assumptions, outdated versions, and secret exposure.
- Verify changeable technical claims against current official Supabase or component documentation.
- Never paste private chat history, credentials, customer data, production logs, internal URLs, or proprietary code into this repository.
- AI review is advisory. It does not replace CI, human ownership, clean-environment testing, or maintainer approval.

## Security and Privacy

Never commit or publish:

- `.env` files, API/JWT/service-role/secret keys, passwords, private keys, SMTP/database credentials, cookies, tokens, or webhook secrets.
- Real customer, tenant, user, billing, database, backup, Storage, incident, or production log data.
- Private domains, IP addresses, repository links, server/Coolify resource IDs, deployment IDs, container IDs, or screenshots containing them.
- Runtime volumes, database dumps, generated credentials, or private deployment configuration.

Stop immediately if a diff contains an unknown token-like value, real hostname, personal data, production evidence, or an unclear generated file. Remove it, rotate credentials if exposure occurred, and repeat the complete scan.

## Implementation Rules

- Prefer upstream Supabase patterns and current official Compose behavior over community-only abstractions.
- Do not weaken authentication, RLS, TLS, firewall guidance, secret scans, health checks, backups, or branch protection to simplify setup.
- A reverse proxy does not automatically close directly published host ports. Document firewall or bind-address requirements explicitly.
- One self-hosted Compose stack represents one isolated Supabase project unless a separately reviewed architecture says otherwise.
- Keep private deployment changes out of public commits. Reproduce reusable fixes with placeholders in a clean public branch.
- Do not automatically sync private repository history into this repository.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akin-umit/supabase-turkiye-community](https://github.com/akin-umit/supabase-turkiye-community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
