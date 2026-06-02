---
trigger: always_on
description: **Last Updated:** 2026-05-24
---

# Claude Code - Money Toolkit (KMP)

**Last Updated:** 2026-05-24
**Project Type:** Kotlin Multiplatform (KMP) — generic financial utility toolkit
**Platforms:** Android | iOS | macOS | Desktop (Windows/macOS/Linux) | Web

---

## Quick Links

🚀 **New fork? Start here:**
- [Fork Quickstart](docs/FORK_QUICKSTART.md) - Day-1 customization checklist for new forks

📖 **Domain-Specific Guides:**
- [GitHub Actions & CI/CD](.github/CLAUDE.md) - Workflows, custom actions, secrets
- [Fastlane Deployment](fastlane/CLAUDE.md) - iOS & Android deployment lanes
- [Bash Scripts](scripts/CLAUDE.md) - Setup, deployment, and verification scripts

📚 **Deep-Dive Documentation:**
- [Troubleshooting Guide](docs/claude/troubleshooting.md)
- [Onboarding Guide](docs/claude/onboarding.md)
- [Deployment Playbook](docs/claude/deployment-playbook.md)
- [Patterns & Best Practices](docs/claude/patterns.md)
- [Independent Cards Pattern](docs/claude/PATTERN-independent-cards.md) - Multi-card dashboards where each card has its own ScreenState (loading / error / empty / content) — `IndependentCardLayout` + `DashboardProgressBar` + `aggregateDashboardProgress`
- [Store Implementation Guide](docs/claude/store-implementation.md) - Offline-first streams, mutations, FetchPolicy, cache lifecycle
- [Motion + Transitions](core-base/ui/MOTION.md) - Symmetric durations, M3 patterns, debug Transition Gallery
- [GitHub Actions Deep Dive](docs/claude/github-actions-deep-dive.md)
- [Secrets Management](docs/claude/secrets-management.md)
- [Version Handling](docs/claude/version-handling.md)

🐛 **Known Issues:**
- [Infrastructure Bugs & Workarounds](docs/analysis/BUGS_AND_ISSUES.md)

---

## Project Overview

This is the **Money Toolkit** — a generic, open-source financial utility template
built on Kotlin Multiplatform. It ships working personal-finance tools out of
the box (loan tracking, bill reminders, interest-rate watching, calculators,
country-level macro indicators) wired through the same offline-first store
contract every framework feature uses. No login. No backend. Fork to brand and
extend.

The project doubles as a **reference implementation** for every architectural
pattern in `core-base/store` and `core-base/ui` — each shipped feature is the
canonical showcase for one or more framework archetypes (see "Toolkit feature
showcase" below).

CI/CD infrastructure spans **5 platforms** and **9 deployment targets**.

### Architecture

```
kmp-project-template/
├── cmp-android/          # Android application
├── cmp-ios/             # iOS Xcode project
├── cmp-desktop/         # Desktop (JVM) application
├── cmp-web/             # Web (Kotlin/JS) application
├── cmp-shared/          # Shared KMP business logic
├── core/                # Core modules (data, domain, network, etc.)
├── core-base/           # Base platform implementations
├── feature/             # Feature modules
├── fastlane/            # Deployment automation (iOS & Android)
├── .github/workflows/   # GitHub Actions CI/CD
└── scripts/             # Bash automation scripts
```

### Toolkit feature showcase

Every shipped feature exists for two reasons: it's a working tool, AND it's the
canonical demo of one or more framework patterns. Forks can keep the lot, swap
the per-feature branding, or selectively remove features they don't need.

| Feature                   | What it does                                              | Pattern showcased                                          |
|---------------------------|-----------------------------------------------------------|------------------------------------------------------------|
| **B1 Loan Tracker**       | Personal loans — track principal, EMI, due dates locally  | `PagingScreenStream` list + `SubmitHandler` edit form      |
| **B2 EMI Calculator**     | Compute monthly EMI for any loan                          | Pure local state (no Store)                                |
| **B3 Affordability**      | "How much loan can I afford?" calculator                  | Pure local state + derived multi-input math                |
| **B4 Bill Reminders**     | Recurring bills + in-app notification scheduler           | `DraftSubmitHandler` (offline-resilient form)              |
| **B5 Amortization**       | Full payment schedule for any loan                        | Read-side projection of `LoanRepository`                   |
| **B6 Loan Comparison**    | Side-by-side total-cost comparison wizard                 | Multi-step wizard state machine                            |
| **B7 Interest Rates**     | FRED-backed federal funds / mortgage / treasury series    | `NETWORK_WITH_CACHE` `ScreenDataStream` + 4-stream combine |
| **B8 Country Macro**      | GDP / CPI / unemployment from World Bank                  | Multi-source combine + country picker                      |
| **Home dashboard**        | Loans summary + upcoming bills + rates + USD exchange     | `combineScreenStates` 4-way fan-in                         |
| **Currency Rates**        | Live FX rates by base currency                            | `Store` + search filter + emptyIfContent                   |
| **Rate History**          | Historical FX charts                                      | Dynamic-key flow + auto-refresh                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openMF/kmp-project-template](https://github.com/openMF/kmp-project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
