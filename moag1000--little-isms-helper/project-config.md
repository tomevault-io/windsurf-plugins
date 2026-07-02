---
trigger: always_on
description: Guidelines for Claude Code in this repository.
---

# CLAUDE.md

Guidelines for Claude Code in this repository.

## Pre-Commit/Push Checklist

**Before EVERY commit:**
- Update relevant documentation (prefer editing existing files over creating new ones)
- Run `php bin/console lint:twig templates/` for template validation
- Optional: Run `python3 scripts/quality/check_translation_issues.py` for translation quality check

**Before EVERY push, verify:**
1. No syntax errors: `find src -name "*.php" -print0 | xargs -0 -n1 php -l`
2. No database errors: `php bin/console lint:container`
3. All templates valid: `php bin/console lint:twig templates/`
4. No runtime errors: `php bin/phpunit` (all tests pass)

## Quick Reference

**Stack:** Symfony 7.4 LTS, PHP 8.4+ (8.5 tested), Doctrine ORM 3.6, Doctrine-Migrations-Bundle 4.0, Twig 3.24, Stimulus 3.2 / Turbo 8, Bootstrap 5.3, FairyAurora v4 Design System, PHPUnit 13.1, Chart.js 4, API Platform 4.3

**Multi-tenancy:** All entities use `tenant_id` field. `TenantContext` service manages context.

**RBAC:** USER → AUDITOR → MANAGER → ADMIN → SUPER_ADMIN, plus holding-level
ROLE_GROUP_CISO + ROLE_KONZERN_AUDITOR, plus persona-roles
ROLE_CISO / ROLE_RISK_MANAGER / ROLE_DPO / ROLE_COMPLIANCE_MANAGER (each
gates own dashboard at `/dashboards/<persona>`). 50+ permissions.

## Essential Commands

```bash
# Development
composer install && php bin/console importmap:install
symfony serve

# Database
php bin/console doctrine:migrations:migrate --no-interaction
php bin/console doctrine:migrations:diff  # after entity changes

# Testing
php bin/phpunit
php bin/phpunit tests/Service/RiskServiceTest.php  # specific test

# Workflows (Regulatory Compliance)
php bin/console app:generate-regulatory-workflows  # Generate all workflows
php bin/console app:generate-regulatory-workflows --workflow=data-breach  # Specific workflow
php bin/console app:process-timed-workflows  # Process time-based auto-progression
php bin/console app:process-timed-workflows --dry-run  # Test without changes

# Quality
php -l src/Service/MyService.php  # syntax check single file
php bin/console cache:clear
php bin/console lint:container    # validate service wiring
php bin/console lint:twig templates/  # validate all templates
python3 scripts/quality/check_twig_macro_scope.py  # embed-macro-import scope (CI-gate since v3.5)
python3 scripts/quality/check_translation_issues.py  # i18n quality
```

## Operator-UI (CLI-Vermeidung)

Self-hosted-Operator hat `/quick-fix` als Web-UI fuer:
- Pending-Migrations anwenden (mit Auto-Chain Reconcile bei non-destructive Drift)
- Schema-Drift reconcilen (mit destructive-Confirm-Checkbox bei DROP/TRUNCATE)
- DataRepair: Orphans assignen, Tenant-Mismatches fixen, Duplikate bereinigen
- "Alles-Sicher-Reparieren" Convenience-Button

CLI nur fuer destructive-edge-cases wenn UI-Auto-Recovery scheitert. Doku:
`docs/user-guide/QUICK_FIX.md`.

## Async Admin Jobs

Long-running admin tasks (> PHP-FPM 30 s limit) run asynchronously and report
status via `var/jobs/<uuid>.json`. The frontend polls
`GET /admin/jobs/{uuid}/status` every 3 s (Stimulus `async-job` controller).

**Two execution strategies, chosen by the `app.async_job.runner` parameter:**

| Strategy            | Use when                              | Worker needed? |
|---------------------|---------------------------------------|----------------|
| `in_request` (default) | Shared hosting / no shell access     | No — uses `fastcgi_finish_request()` |
| `messenger` (opt-in) | Multi-server / dedicated worker box  | Yes — `messenger:consume async` |

Switch via env: `APP_ASYNC_JOB_RUNNER=messenger`.

### In-request runner (default — shared-hosting friendly)

The runner flushes the response to the browser, calls
`fastcgi_finish_request()` (or `litespeed_finish_request()`), then keeps the
PHP-FPM worker executing the job until completion. Falls back to synchronous
execution under CLI / non-FPM SAPIs so phpunit + console commands work
unchanged.

`InRequestJobRunner` lives at `src/Service/Job/InRequestJobRunner.php`.
The detach helper is shared with the setup-wizard via
`App\Controller\Trait\DetachableResponseTrait`.

### Adding a new async admin job

1. Create `src/Job/MyJob.php` implementing `App\Job\AsyncJobInterface`
2. Inject services via constructor (autowired automatically)
3. Implement `run(JobContext $ctx)` — call `$ctx->progress()` and `$ctx->message()`
4. In the controller — use the **`AsyncJobDispatcher` facade** (P-16, since
   2026-05-23) for the canonical PRG-redirect pattern:
   ```php
   return $this->asyncJobDispatcher->dispatchWithProgress(
       request: $request,
       jobClass: MyJob::class,
       jobArgs: ['myArg' => $val],
       jobName: 'admin.my_job',
       payload: ['_label' => '…', '_subtitle' => '…'],
       returnUrl: $this->generateUrl('admin_my_index'),
   );
   ```
   For the rare case where you need direct template rendering (non-Turbo /
   XHR JSON envelope / payload-patching with the freshly-minted UUID), drop
   to the lower-level primitives:
   ```php
   $jobId = $this->jobStatusService->create('admin.my_job', $payload);
   $response = $this->render('...progress.html.twig', [
       'jobId' => $jobId, 'cancelUrl' => '...',
   ]);
   return $this->jobDispatcher->dispatch(
       MyJob::class,
       ['myArg' => $val],
       $jobId,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moag1000/Little-ISMS-Helper](https://github.com/moag1000/Little-ISMS-Helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
