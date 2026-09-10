---
trigger: always_on
description: <!-- Generated from mined PR-review precedents; additive to the repo's README "Development Guidelines". -->
---


<!-- Generated from mined PR-review precedents; additive to the repo's README "Development Guidelines". -->
# Music Assistant frontend — mined review precedents

Standards distilled from this repository's own past pull-request review discussions; each links the PR where it was set. They are ADDITIVE to the "Development Guidelines" in `README.md` — where they overlap, the README is authoritative and these add the precedent and specifics; do not restate what the README already covers. Treat a **MUST** or "won't support" deviation as a problem worth blocking and a *Prefer* deviation as a suggestion, and cite the linked PR when you raise one.

- **Won't support** editing non-English locale files in a PR — only `en.json` is hand-edited; Lokalise syncs the rest. ([frontend#680](https://github.com/music-assistant/frontend/pull/680#discussion_r1790641156): "dont edit translations from a PR, instead use lokalize. Only en.json may be edited") [mined · 2 PRs · 👍]
- **Won't support** adding new Vuetify UI — new and rewritten UI uses shadcn-vue. ([frontend#2180](https://github.com/music-assistant/frontend/pull/2180#issuecomment-5059719792): "we dont allow any more vuetify code being added") [mined · 2 PRs]
- **MUST** guard UI actions against disconnected/empty/missing state and never add a silent fallback default that masks a broken server wire-contract. ([frontend#769](https://github.com/music-assistant/frontend/pull/769#discussion_r1894318897): "the code should handle that instead of crashing"; [frontend#2083](https://github.com/music-assistant/frontend/pull/2083#discussion_r3565206399): "a fallback would mask a broken server contract") [mined · 4 PRs · 👍]
- **MUST** restore any stubbed/overridden global (localStorage, matchMedia, property descriptors) after each test to avoid cross-test pollution. ([frontend#2059](https://github.com/music-assistant/frontend/pull/2059#discussion_r3560611542): "Stubbed globals are now restored after each test") [mined · 2 PRs]
- **MUST** validate user/config-supplied URLs against an http/https allowlist before opening them (reject `javascript:`/`data:`/malformed). ([frontend#2204](https://github.com/music-assistant/frontend/pull/2204#discussion_r3650834374): "only opens http/https URLs (parse + protocol allowlist)") [mined · 1 PR]
- **Prefer** a player's `display_name` (which reflects any user-assigned custom name) wherever players are shown or sorted. ([frontend#374](https://github.com/music-assistant/frontend/pull/374#discussion_r1490753202): "always use display_name as that takes into account any custom name") [mined · 1 PR]
- **MUST** keep PRs small and single-purpose — one functional change per PR, with no unrelated lint/editor/formatting churn mixed in. ([frontend#20](https://github.com/music-assistant/frontend/pull/20#issuecomment-1540353368): "can you please try to create smaller PR's with only a single change"; [frontend#1338](https://github.com/music-assistant/frontend/pull/1338#issuecomment-3694602942): "It is way easier to give feedback and merge specific parts than one big PR.") [mined · 6 PRs · 👍]
- **MUST** not bump packages in a feature PR — dependency updates go in separate PRs (dependabot), unless the feature itself needs the new package. ([frontend#388](https://github.com/music-assistant/frontend/pull/388#discussion_r1501859925): "please try to not update packages in a feature PR"; [frontend#388](https://github.com/music-assistant/frontend/pull/388#issuecomment-1963001949): "remove the unrelated package bumps and leave that to dependabot") [mined · 1 PR]
- **MUST** implement behavior that all API clients need (volume logic, queue handling, item filtering) in the server, not the frontend — frontend-only workarounds for backend concerns are rejected. ([frontend#1569](https://github.com/music-assistant/frontend/pull/1569#issuecomment-4124730842): "We should not accept this to be implemented in the frontend at all"; [frontend#2140](https://github.com/music-assistant/frontend/pull/2140#discussion_r3582143735): "we should fix this serverside instead") [mined · 4 PRs]
- **MUST** include screenshots (or a short video) of the visual end result in any PR that changes UI. ([frontend#2028](https://github.com/music-assistant/frontend/pull/2028#issuecomment-4903645327): "Also include a screenshot of the change endresult."; [frontend#1669](https://github.com/music-assistant/frontend/pull/1669#issuecomment-4187307874): "Can you add some screenshots of how this looks like ?") [mined · 5 PRs]
- **Won't support** adding features or settings without a demonstrated use case and real user demand — every feature, big or small, adds maintenance burden. ([frontend#1335](https://github.com/music-assistant/frontend/pull/1335#issuecomment-3705122613): "with every feature we add (big or small) we add maintenance burden"; [frontend#506](https://github.com/music-assistant/frontend/pull/506#issuecomment-2111983019): "Can you tell me a bit about the usecase for this ?") [mined · 3 PRs]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [music-assistant/frontend](https://github.com/music-assistant/frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
