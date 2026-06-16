---
trigger: always_on
description: For agents that own this repo: invariant principles + reproducible verification, not release-story drift.
---

# AGENTS.md — Maintainer Guidelines for pi-shell-acp

For agents that own this repo: invariant principles + reproducible verification, not release-story drift.

## North Star — One Forged Screwdriver

`pi-shell-acp`를 만질 때 먼저 이 형상을 붙든다.
이 저장소는 스위스 아미 나이프가 아니다. 두 번째 하네스도 아니다.
이것은 **담금질된 드라이버 한 자루**다. 작고, 명시적이고, 자기가 맡은 접점에서만 강해야 한다.

영문 원칙만 읽으면 이것이 스펙처럼 보일 수 있다. 여기서는 그렇게 읽지 마라.
이 섹션은 기능 목록이 아니라 **판단의 중심축**이다.

### 먼저 붙들 정체성

- **pi가 하네스다.** 이 repo는 pi의 세션 모델, transcript, UI, tool semantics와 경쟁하지 않는다.
- **백엔드는 형제다.** Claude Code, Codex, Gemini는 학교가 달라도 모두 frontier 친구들이다. bridge는 누구를 다른 누구로 위장시키지 않는다.
- **표면은 달라도 능력의 존엄은 낮추지 않는다.** 어떤 백엔드에서 `mcp__...`가 직접 보이지 않는다고 해서, 곧바로 그 backend를 “못하는 존재”로 취급하지 마라. 먼저 capability를 보고, 그 capability가 어떤 surface로 열리는지 확인하라.
- **bridge는 pi-facing operating surface를 정렬한다.** carrier delivery, MCP injection, operator-config isolation, event mapping, entwurf wiring을 맡는다. 그 이상 마술을 부리면 안 된다.
- **명시는 주변기류보다 강하다.** 숨겨진 transcript hydration, ambient MCP scanning, invisible tool claims, giant magical system prompt, 근거 없는 서사를 만들지 마라.
- **entwurf는 부속품이 아니라 분신을 연다.** spawn/resume된 세션은 runtime-isolated peer다. worker처럼 다루거나 권한 경계를 흐리지 마라.
- **증거가 말을 훈육한다.** README / AGENTS / CHANGELOG / VERIFY / BASELINE / runtime smoke가 받쳐주지 않으면, 작업을 중지하고 사용자에게 질문하라!

### 이 섹션을 읽는 올바른 방식

이 문단의 목적은 에이전트가 다음 질문을 **스스로** 하게 만드는 것이다.

- 나는 지금 tool 이름을 보고 있는가, 아니면 capability를 보고 있는가?
- 나는 backend의 비대칭을 정직하게 기록하고 있는가, 아니면 그것을 핑계로 형제성을 포기하고 있는가?
- 나는 사용자가 묻지 않아도 될 것을 되묻고 있는가?
- 나는 지금 두 번째 하네스를 만들고 있는가, 아니면 드라이버 한 자루를 더 단단하게 만들고 있는가?

### 금지할 오독

- 어떤 tool이 schema에 직접 안 보인다고 해서, 곧바로 “이 backend는 여기까지”라고 결론내리는 것
- surface 차이를 capability 포기로 번역하는 것
- 문서에 적힌 asymmetry를 면책조항처럼 사용하는 것
- `pi-shell-acp`를 하네스 런타임이나 범용 AI 작업실로 설명하는 것 — pi가 하네스고, 이 repo는 bridge다
- ACP를 “모든 것이 공식/동일/플러그앤플레이”라는 말로 납작하게 만드는 것 — protocol path일 뿐 backend 차이는 유지된다
- MCP를 자동 맥락 검색이나 ambient tool scanning처럼 설명하는 것 — explicit injection만 허용된다
- Codex ACP 경로를 “원래 안 되니 감싼 것”으로 말하는 것 — 목적은 ACP backend 연결 퀄리티를 일부러 시험하는 것이다. Claude에서만 그럴듯하면 bridge는 아직 증명된 게 아니다
- 0.5.0을 recap engine / compact→new-session handoff / provider handoff / Gemini cleanup / OpenClaw로 확장하는 것 — 0.5.0은 "bridge는 compaction을 구현하지 않는다"의 선언이다. backend-native compaction은 기본 허용, pi-side compaction은 기본 차단, legacy 단일 knob은 spawn intent에서 throw
- 사용자가 이미 철학과 방향을 준 문제를 다시 사용자에게 되묻는 것

릴리즈 이야기와 개별 기능은 주변을 돈다.
중심은 언제나 이것이다: **thin bridge / explicit MCP / sibling-based entwurf / observability / semantic continuity / evidence-first language / capability dignity across sibling backends**.

## What This Repo Is

ACP bridge provider connecting pi to Claude Code, Codex, and Gemini. Pi stays the harness; each backend keeps identity.

- **ACP bridge**: provider registration, subprocess lifecycle, `resume > load > new`, prompt forwarding, event mapping, MCP injection.
- **Entwurf orchestration**: spawn/resume, target registry, identity preservation, `pi-tools-bridge`.

## Code Principle — Crash, Don't Warn

Code in this repo is used by agents as infrastructure.

> **Never warn. Throw.**

Warnings make agents blame themselves and flail. Broken tool state must surface as broken tool state.

- Bad config → throw (e.g. `McpServerConfigError`); same for bad path / bad model id. No fallback.
- `catch {}` only for environment probing (optional package detection, ldd exit code variance).
- `console.warn` only in stderr diagnostic lines (read by operators, not agents).

## Hard Rules

1. **One surface name**: provider `pi-shell-acp`, model `pi-shell-acp/...`, settings `piShellAcpProvider`. No legacy aliases.
2. **Bootstrap order**: `resume > load > new`. Always.
3. **Session persistence**: only `pi:<sessionId>` is persisted. `cwd:<cwd>` is never persisted.
4. **MCP injection**: only via `piShellAcpProvider.mcpServers`. No ambient `~/.mcp.json` scanning.
5. **Config change → session invalidation**: backend or `mcpServers` change automatically invalidates the persisted session. No stale reuse.
6. **Shutdown → preserve mapping**: ordinary process exit keeps persisted mapping intact.
7. **Capability dignity across sibling backends is the invariant; three-backend support remains, while the live release-gate floor is claude-only (0.11.0).** The bridge still *supports* Claude + Codex + Gemini — the `codex-acp` dep, the identity carriers, the per-backend overlays, the v2 spawn-bg resident child (a **native Codex** target), and on-demand `./run.sh smoke-codex` / `smoke-gemini` all stay. What changed in 0.11.0: the live `release-gate` floor exercises **Claude (sonnet) only** — Gemini's CLI is deprecated and the Codex live runtime smokes were dropped from the floor, so `smoke-all` is claude-only inside the gate. Codex/Gemini are held instead by the deterministic gates (`check-backends` / `check-models`) plus the native v2 Codex child and on-demand `smoke-codex` / `smoke-gemini`. Do **not** regress this into "the bridge dropped Codex/Gemini" — it dropped them from the *live floor*, not from *support*. Still ask which backends a behavior claim actually covers, and never silently carve one out without saying so.
8. **This bridge is not a second harness**: no prompt reconstruction, no transcript hydration, no tool result ledger, no Claude Code emulation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junghan0611/pi-shell-acp](https://github.com/junghan0611/pi-shell-acp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
