---
trigger: always_on
description: Generated: 2026-05-02
---

# TEST KNOWLEDGE BASE

Generated: 2026-05-02
Commit: b0ef65d

## OVERVIEW
Vitest suites for OAuth flow, request transforms, response handling, rotation logic, storage, CLI management, repo hygiene, and more.
**4909 tests** across **317 test files** with 80%+ coverage threshold.

## STRUCTURE
```
test/
├── accounts.test.ts                # multi-account storage/rotation
├── ansi.test.ts                    # ANSI escape helpers
├── audit.test.ts                   # rotating file audit log
├── auth-menu-builder.test.ts       # auth dashboard view-model formatters
├── auth-menu-hotkeys.test.ts       # auth menu hotkey behavior
├── auth-rate-limit.test.ts         # token bucket for auth
├── auth.test.ts                    # OAuth PKCE + JWT decoding
├── update-notice.test.ts           # npm version notice
├── browser.test.ts                 # platform-specific browser open
├── capability-policy.test.ts       # model capability enforcement
├── chaos/
│   └── fault-injection.test.ts     # chaos/fault injection tests
├── circuit-breaker.test.ts         # failure isolation
├── cli-auth-menu.test.ts           # CLI auth menu integration
├── cli.test.ts                     # CLI helpers
├── codex-bin-wrapper.test.ts       # bin wrapper lazy-load, missing dist handling
├── codex-cli-state.test.ts         # CLI state management
├── codex-cli-sync.test.ts          # CLI sync coordination
├── codex-host-resolver.test.ts     # host resolver
├── codex-manager-cli.test.ts       # CLI settings Q cancel, all 5 panels
├── codex-prompts.test.ts           # Codex prompt generation
├── codex-routing.test.ts           # Codex routing decisions
├── codex.test.ts                   # Codex instructions/caching
├── config-files.test.ts            # config file handling
├── config.test.ts                  # configuration parsing/merging
├── context-overflow.test.ts        # context length handling
├── copy-oauth-success.test.ts      # build script tests
├── dashboard-settings.test.ts      # dashboard settings
├── documentation.test.ts           # docs parity, CLI command flags, config precedence, governance
├── entitlement-cache.test.ts       # entitlement cache
├── errors.test.ts                  # custom error types
├── eslint-config.test.ts           # ESLint config validation
├── failure-policy.test.ts          # retry/failover policy
├── fetch-helpers.test.ts           # fetch flow helpers
├── fixtures/
│   └── v3-storage.json             # V3 storage fixture
├── forecast.test.ts                # account forecast
├── hashline-tools.test.ts          # hashline tool helpers
├── health.test.ts                  # account health status
├── host-codex-prompt.test.ts       # host-specific prompts
├── index-retry.test.ts             # plugin retry logic
├── index.test.ts                   # main plugin integration, email dedup
├── input-utils.test.ts             # input filtering
├── install-codex-auth.test.ts      # installer tests
├── live-account-sync.test.ts       # live account sync
├── health-check.test.ts            # runHealthCheck quick + live probe paths
├── logger.test.ts                  # logging functionality
├── login-flow.test.ts              # runAuthLogin transports and cap handling
├── login-menu-accounts.test.ts     # account-row assembly for the login menu
├── login-menu-actions.test.ts      # manage actions, identity re-resolution
├── login-menu-data.test.ts         # dashboard row view model, drift sync
├── login-oauth-selection.test.ts   # login-oauth account selection
├── model-map.test.ts               # model name normalization
├── oauth-server.integration.test.ts # OAuth server (port 1455)
├── package-bin.test.ts             # package.json bin field
├── parallel-probe.test.ts          # concurrent health checks
├── paths.test.ts                   # project root detection, worktree identity, UNC paths
├── plugin-config.test.ts           # plugin config defaults
├── preemptive-quota-scheduler.test.ts # quota deferral
├── proactive-refresh.test.ts       # token refresh before expiry
├── property/
│   ├── account-identity.property.test.ts # identity matching/dedup invariants
│   ├── helpers.ts                  # property test utilities
│   ├── model-fallback.property.test.ts # unsupported-model fallback invariants
│   ├── rotation.property.test.ts   # rotation property tests
│   ├── setup.test.ts               # property test setup
│   ├── setup.ts                    # property test config
│   ├── settings-write-queue.property.test.ts # write-queue ordering/clamp invariants
│   └── transformer.property.test.ts # transformer property tests
├── quota-cache.test.ts             # quota cache
├── quota-probe.test.ts             # quota probe
├── rate-limit-backoff.test.ts      # exponential backoff
├── rate-limit-decision.test.ts     # rate-limit/invalidation decision tree
├── recovery-constants.test.ts      # recovery constants
├── recovery-storage.test.ts        # recovery storage
├── recovery.test.ts                # session recovery
├── refresh-guardian.test.ts        # refresh guardian
├── refresh-lease.test.ts           # refresh lease
├── refresh-queue.test.ts           # queued token refresh
├── repo-hygiene.test.ts            # repo cleanup/check, Windows removeWithRetry
├── request-transformer.test.ts     # request body transforms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndycode/codex-multi-auth](https://github.com/ndycode/codex-multi-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
