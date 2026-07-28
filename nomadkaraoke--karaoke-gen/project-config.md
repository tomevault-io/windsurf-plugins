---
trigger: always_on
description: **Karaoke video generation platform** - CLI tool and web service that creates professional karaoke videos with synchronized lyrics.
---

# Karaoke-Gen AI Assistant Guidelines

## Project Overview

**Karaoke video generation platform** - CLI tool and web service that creates professional karaoke videos with synchronized lyrics.

- **Production**: <https://gen.nomadkaraoke.com> (frontend), <https://api.nomadkaraoke.com> (backend)
- **CLI**: `karaoke-gen` (local), `karaoke-gen-remote` (cloud)
- **Repo**: <https://github.com/nomadkaraoke/karaoke-gen>

## Quick Reference

| What | Where |
|------|-------|
| Product vision & goals | `docs/PRODUCT-VISION.md` |
| Current status | `docs/README.md` |
| Architecture | `docs/ARCHITECTURE.md` |
| Dev setup & testing | `docs/DEVELOPMENT.md` |
| **Testing & code quality** | `docs/TESTING.md` |
| API reference | `docs/API.md` |
| Past learnings | `docs/LESSONS-LEARNED.md` |
| **Operational runbooks** | `docs/TROUBLESHOOTING.md` |
| GDrive validator & gaps | `docs/GDRIVE-VALIDATOR.md` |
| **Product communication** | `docs/PRODUCT-COMMUNICATION-GUIDE.md` |
| Brand style guide | `docs/BRAND-STYLE-GUIDE.md` |

## Tech Stack

- **Backend**: FastAPI on Cloud Run, Firestore, GCS, Secret Manager
- **Frontend**: Next.js on Cloudflare Pages
- **Processing**: Cloud Run GPU (L4 audio separation), AudioShake/Whisper (transcription)
- **Infra**: Pulumi IaC, GitHub Actions CI/CD

## Essential Rules

### Git Workflow
- **Never commit directly to `main`** - use `/new-worktree <description>` to start
- **Follow global workflow** - see `~/.claude/CLAUDE.md` for command sequence
- Create PR with summary, changes, testing info
- Merge only after CI passes

### Testing (Required)
**Before planning any implementation work**, read `docs/TESTING.md` and ensure your plan includes a thorough testing strategy that follows the guidance there. This includes:
- Which test types are needed (unit, integration, E2E)
- Where tests should be placed
- What mocking approach to use
- Coverage expectations
- **Production E2E tests for user-facing features** - see below

**Critical:** If your plan includes "Manual Testing" steps, convert them to automated Playwright E2E tests. Production E2E tests (in `frontend/e2e/production/`) are valuable even if only run once after deployment - they codify expected behavior and verify the deployed code works. See `docs/TESTING.md` for details.

**Before committing**, run all tests:
```bash
make test 2>&1 | tail -n 500
```

This single command:
- Installs dependencies automatically (backend + frontend)
- Runs all backend tests (unit, integration, emulator)
- Runs all frontend tests (Jest unit + Playwright E2E)
- Takes ~5-10 minutes

**All tests must pass.** Don't dismiss failures as "pre-existing" - investigate and fix them.

### Testing in Production (for Agents)

When asked to "test it yourself in prod" or verify a fix in production:

```bash
# Get admin token
export KARAOKE_ADMIN_TOKEN=$(gcloud secrets versions access latest --secret=admin-tokens --project=nomadkaraoke | cut -d',' -f1)

# Option 1: Use the debug template directly
cd frontend && node e2e/helpers/debug-prod-template.mjs

# Option 2: Copy template for customization (gitignored)
cp frontend/e2e/helpers/debug-prod-template.mjs frontend/test-my-issue.local.mjs
# Edit the script, then run it
node test-my-issue.local.mjs

# Option 3: Run existing production E2E tests
KARAOKE_ADMIN_TOKEN=$KARAOKE_ADMIN_TOKEN npx playwright test e2e/production/admin-dashboard.spec.ts
```

Files matching `test-*.local.*` and `debug-*.local.*` are gitignored - you can hardcode tokens in them safely.

See `docs/TESTING.md` § "Ad-Hoc Production Debugging" for full details.

### Version Bumping
- Bump `tool.poetry.version` in `pyproject.toml` for code changes
- Skip for docs-only changes

### Infrastructure
- **All GCP changes via Pulumi PRs** - changes in `infrastructure/` deploy automatically on merge to main
- **Always run `pulumi up` locally BEFORE merging PRs** - CI deploys run on spot VMs that can be preempted mid-apply, causing partial state. Apply locally first, then merge (CI re-runs as no-op)
- Never modify GCP resources directly via console or `gcloud` CLI
- `gcloud` CLI for reading/debugging only (e.g., checking logs, SSH to VMs)
- Stop and notify user on auth issues

### Internationalization (i18n)

- All user-facing strings live in `frontend/messages/{locale}.json` (33 locales), NOT in components
- Backend strings live in `backend/translations/{locale}.json` (currently 3 locales: en, es, de)
- Components use `useTranslations('namespace')` from next-intl
- Pages are under `frontend/app/[locale]/` — locale-aware routing (customer-facing: `/`, `/app/*`, `/order/*`, `/auth/*`, `/payment/*`, `/r/*`)
- Non-locale routes at those same paths (e.g. `frontend/app/app/jobs/[[...slug]]/page.tsx`) are `<LocaleRedirect />` shims that redirect into `[locale]` — they are NOT English-only surfaces
- The `/app/jobs/#/{id}/review`, `/instrumental`, and `/audio-edit` review UIs are fully multilingual (components under `components/lyrics-review/`, `components/instrumental-review/`, `components/audio-editor/` use `useTranslations` heavily)
- Only `/admin/*` is English-only — it has no `[locale]/admin` counterpart and inherits `DefaultIntlProvider` from the root layout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nomadkaraoke/karaoke-gen](https://github.com/nomadkaraoke/karaoke-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
