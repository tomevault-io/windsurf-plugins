---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Collaboration workflow (user-defined — two co-equal collaborators)

`Milkbuttercheese2/WhatIhavetodo` (owner's GitHub account, formerly `wooseongkyun`) is the **single shared repo** — both collaborators have Write access directly on it. There is no fork-based workflow anymore: `origin` points straight at `Milkbuttercheese2/WhatIhavetodo`. (A personal fork at `rfastball/WhatIhavetodo` still exists as a backup/history artifact from before the collaborator invite, but is not part of the active workflow — don't push there.)

**Never commit directly to `main`.** For any change:
1. Branch off `main`: `git checkout -b <type>/<short-name>` — prefix matches this repo's commit-message convention (`feat/`, `fix/`, `docs/`, `refactor/`, `test/`), e.g. `feat/recurring-tasks`.
2. Push the branch to `origin` and open a PR (branch → `main`) **within this one repo** — not cross-repo.
3. Merge via the PR once reviewed; delete the branch after.

Branch-protection rules on `main` (require PR, block direct push) have **not** been configured yet on GitHub as of 2026-07-11 — that requires the repo owner's admin access, which the assistant does not have. Until it's set, direct pushes to `main` are *technically* possible but against this project's convention; follow the branch+PR flow regardless. There is no always-on CI on this repo — `npm test` / `cargo test --lib` are run locally before opening/merging a PR. The only GitHub Actions workflow is `build-windows-exe.yml` (manual `workflow_dispatch` only): it runs both test suites, does the official MSVC release build on windows-latest, and commits the exe to the branch it was run on.

## Versioning convention (user-defined)

Current release = **v2.5.7**.

> ⚠️ **버전 이력 주의 (반드시 읽을 것 — 과거에 여기서 혼선이 있었다):** 저장소 git 히스토리에는
> `v3.0.0`·`v3.1.0` 커밋과 CHANGELOG 항목이 남아 있지만, **v3.x는 실제로 배포된 적이 없다.**
> 그 작업은 *다른 계정에서 진행되던 미배포 개편*이었고, 당시 파일 링크·이름 변경 정도의 변화를
> 두고 **대규모 개편(major)도 아닌데 버전을 3.0.0으로 임의 상향**해 버린 것이다(잘못된 범프).
> 저장소 소유자 결정으로 이를 바로잡아, 실제 정식 배포는 **구 v2.31 라인을 잇는 `v2.4.0`**
> 으로 되돌렸다. 따라서:
> - **현재/실제 배포 버전 = v2.5.7.** 세 매니페스트(package.json·Cargo.toml·tauri.conf.json) 동일.
> - v3.0.0·v3.1.0 CHANGELOG 항목은 *내부 이력*으로만 읽고, 실 배포 번호로 착각하지 말 것.
> - **다음 버전은 v2.4.x(버그·사소한 조정) / v2.5.0(기능 추가)로 이어간다.** 진짜 호환성 깨지는
>   대규모 개편이 아닌 한 다시 3.x로 올리지 말 것. (자동 업데이터·버전 비교 로직이 없어 역전은 무해.)
> - **버전업 정책(소유자 지정): 사소한 수정이라도 새로 배포(빌드/머지)할 때마다 패치(Z)를 올린다.**
>   매 배포마다 세 매니페스트 버전을 함께 올리고 CHANGELOG에 해당 버전 항목을 추가하며 PR 제목에도
>   `[vX.Y.Z]`를 붙인다.

**The project uses standard X.Y.Z semver** (X = 큰 개편/호환성 변화, Y = 기능 추가, Z = 버그 수정·소규모 조정), and the same `"X.Y.Z"` string goes into all THREE manifests together (`src-tauri/Cargo.toml`, `src-tauri/tauri.conf.json`, `package.json`). The UI header shows `getVersion()` verbatim (v3.0.0에서 구 십진수 규칙의 trailing-`.0` 절삭을 폐지 — `main.js`). The app was also renamed at v3.0.0: **'뭐하려 했더라'** (formerly '뭐해야 했더라'); the internal `productName`/crate name stay `wmhh-desktop` and the bundle identifier stays `com.wooseongkyun.wmhh` on purpose — changing the identifier would move `app_local_data_dir()` and orphan existing users' config/data. Legacy versions (≤ v2.31) used a decimal-magnitude scheme (v2.2 → v2.21 → … → v2.3(=2.30) → v2.31): read old CHANGELOG entries with that rule, and note those old manifest versions sort non-monotonically as semver (harmless — no auto-updater or version-comparison logic exists; don't add tooling that assumes semver ordering across the v2 era). A structural analysis lives in `구조 분석 보고서.md`; the built exe ships in `최종 프로그램 산출물/뭐하려 했더라v{X.Y.Z}.exe` (v2.5.3부터 파일명에 버전 표기 — 소유자 지정; 빌드 워크플로가 이전 버전 exe를 지우고 최신 하나만 남긴다) and IS committed to git per user request.

**Changelog rule (user-defined, since v2.3):** every update pushed to GitHub must add an entry to `CHANGELOG.md` describing what changed (Korean, grouped 변경/추가/수정). Started at the v2.2→v2.3 transition — write the entry as part of the same PR/commit that makes the change.

## Migration in progress

This repo is being converted from the legacy single-file HTML app (still in `legacy/`) into a **Tauri (Rust) + SQLite** desktop app for offline/air-gapped 공무원 내부망 deployment as a small portable `.exe`. The full rationale and architecture (DB schema, data-safety measures, migration/versioning approach, why Tauri, why EAV over a JSON column, etc.) live in the plan doc at `C:\Users\rhama\.claude\plans\rustling-seeking-flurry.md` — read it before making architectural changes to `src-tauri/`.

Status: **Phase 1 and Phase 2 both done.** The frontend now lives in `src/` as browser-native ES modules (see "Frontend layout" below) with `STORE` delegating to Rust via `invoke()`; SQLite is the single source of truth (IndexedDB/localStorage are no longer used). The legacy HTML file in `legacy/` remains the behavioral reference if a business-logic question arises, but `src/` is the live implementation.

## Legacy app (`legacy/뭐해야 했더라v1.41.html`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Milkbuttercheese2/What-I-have-to-do](https://github.com/Milkbuttercheese2/What-I-have-to-do) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
