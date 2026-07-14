---
trigger: always_on
description: >
---


# Plannode 배지 매핑 지침 (가져오기 파이프라인)

**근원:** Cursor 플랜 `가져오기_배지_파이프라인_구현결과_bbe7c690.plan.md`를 규칙용으로 재구성·코드 동기화.  
**제품 의미의 “학습”:** 머신러닝 학습이 아니라 **규칙 테이블 + 브라우저 누적 저장**이다.

---

## 0. 표준 배지 풀 기반 배지 매핑 — 기술 지침

### 0.1 목표 (제품 정렬)

- **표준 배지 풀**이 허용하는 토큼만 최종 칩·저장물에 남긴다: `filterBadgeSetToCanonicalPool`가 최종 게이트.
- 풀 안에서 **동의어·메타 추론·가져오기 누적 규칙**으로 들어갈 수 있는 것은 최대한 매핑한다(외부 임의 문자열 원문 보존은 제품 범위 밖 — PRD·아키텍처와 동일).

### 0.2 단일 진실·호출 순서 (구현 불변)

| 역할 | 코드 정본 |
|------|-----------|
| 런타임 유효 풀(기본 21 + 사용자 확장) | `getEffectiveBadgePool` — `badgePoolConfig.ts` |
| 원문 토큰 → 트랙·표준 대문자 토큰 | `resolveImportedBadgeToken` — `badgeImportAliases.ts` |
| 명시 배지 + 메타 힌트 병합 | `getBadgeSetFromNodeInput` → `inferBadgeHintStringsFromMetadata` 병합 순서 **1→4** — `badgePromptInjector.ts`, `badgeMetadataInference.ts` |
| 저장·가져오기 sanitize | `sanitizeNodeBadgesForTreeV1` / `applySanitizeImportedPlannodeNodeV1` |
| 협업 배지 송수신·projectId | §6.9 · [`plannode-architecture.mdc`](./plannode-architecture.mdc) §10.10.1 |
| 노드 카드 표시 | **동일** `getBadgeSetFromNodeInput` 계열 — 단, 파일럿 `render()`는 **표시 전용 게이트**(§6.2)로 `inferHints`를 제한할 수 있음 |

가져오기·아젠다 생성·클라우드 머지 등 **진입점이 달라도** 위 단일 파이프를 깨거나 **둘째 배지 저장소**를 두지 않는다.

**파일럿 표시 게이트**는 `metadata.badges`·`badges[]`·`sanitize` 결과와 **충돌하지 않게** 동작해야 한다(§6).

### 0.3 표준 풀 확대·증식 시 동기화 (필수)

표준 풀은 **지속적으로 확대할 계획**이므로, 토큰을 추가·변경할 때 아래를 **한 세트**로 갱신한다. 누락 시 가져오기 해석·칩·LLM 프롬프트가 어긋난다.

| 단계 | 할 일 |
|------|--------|
| 1 | `badgePoolConfig.ts` — 기본 풀 정의·검증·스토리지 키 일관성 |
| 2 | `badgeImportAliases.ts` — `ALIAS_GROUPS`·신규 토큰으로 들어올 외부 표기 동의어 |
| 3 | (해당 시) `badgeMetadataInference.ts` — 키워드·extras가 새 도메인 의미를 다루면 정규식·힌트 보강 |
| 4 | Vitest — `badgeImportAliases.test.ts`, CRAZYSHOT·파이프라인 회귀 샘플 갱신 또는 케이스 추가 |
| 5 | 아젠다 프롬프트 등 **인라인 풀 문구** — `.cursor/plans/plannode_dev_spec_v1.0.md` §3-1 `BADGE_SPEC`, `agendaPromptAgent.ts` 동기 래퍼 |
| 6 | UI 칩 라벨·색 — `plannode-ui-identity.mdc`·컴포넌트에서 신규 트랙·토큰 표기 필요 여부 |

제품 공표가 필요하면 **`plannode-prd.mdc`** M1 F1-3 등과 한 줄 교차한다.

### 0.4 지능적 학습 자동화 (향후 고려 — 구현은 TASK·GATE·GP-12)

현재 구현층은 **규칙 기반**: 동의어표·키워드·`mergeLearnedBadgeRulesFromImportedNodes`로 **브라우저에 규칙 누적**. 이것만으로도 풀이 커질수록 **별칭·규칙 테이블 유지 비용**이 늘어난다.

**확대 증식에 맞춘 자동화·지능화 방향(선택·단계적):**

1. **데이터 기반 동의어 후보:** 가져오기 시 `resolveImportedBadgeToken`에 걸리지 않은 원문(또는 빈도)을 개발·디버그 리포트로 모아, `badgeImportAliases` 보강 후보를 만들 수 있다.
2. **사용자 규칙 UI:** `setUserBadgeInferenceRules`를 콘솔 없이 편집 — 비개발자 튜닝·GP-12 범위 내에서 TASK 승인 후.
3. **학습 저장소 정리:** `AI_LEARNED_RULES_MAX`·충돌 시 우선순위를 제품 정책으로 문서화.
4. **진짜 ML·임베딩 매핑**은 PRD·`plan-output`·GATE에 명시되기 전까지 **도입하지 않는다**(오버엔지니어링 견제). 도입 시에도 **구조 골격은 트리·풀 고정**, 모델은 보조 배지 제안 정도로 한정하는 방향이 PRD F2-4·§10.4와 정합하다.

에이전트는 위 **0.3 동기화 표**를 풀 변경 시 1차 체크리스트로 삼고, **0.4**는 설계 메모로만 참고하고 임의 신규 모듈을 추가하지 않는다.

### 0.5 BADGE-ALIGN (2026-05 · DEV 16 · IA 구조 우선)

**목표:** 노드 카드 배지는 **화면 형태(UX)·도메인·구현 조건(DEV)·기획 산출(PRJ)** 만 남기고, 범용 **CRUD·배포 공정**·과광 alias는 기본 풀·추론·프롬프트에서 제거·보수화한다.

| 항목 | 정책 |
|------|------|
| **기본 DEV 풀** | **16종** — `badgePoolConfig.ts` `DEFAULT_DEV_KEYS` 정본 |
| **기본 풀에서 제거** | `CRUD`, `LOCAL`, `STAGING`, `PROD`, `DEPLOY`, `HOTFIX`, `PR`, `JSON`, `RENDER` |
| **가져오기 `crud`** | `badgeImportAliases`에서 `CRUD`로 해석 가능하나 **풀에 없음** → `filterBadgeSetToCanonicalPool` 후 **칩·저장물에서 제거** |
| **추론** | `keywordHints`에 범용 CRUD·배포 토큰 없음 · **§6.2** 설명 없으면 추론 off 유지 |
| **구조 우선** | `inferBadgeHintStringsFromMetadata` — `treeImportExtras` → **`iaGrid.screenType`·`path`** → `keywordHints` → 사용자·AI 규칙(§4.4) |
| **LLM** | `agendaPromptAgent.ts` `BADGE_SPEC` — 16 DEV + UX 26 + PRJ 9, 제거 토큰 예시 없음 |

프로젝트 **커스텀 `badge_pool`**에 레거시 `CRUD`가 남아 있으면 해당 프로젝트만 예외 허용(문서·BACKLOG). 기본 풀·외부 AI JSON은 **≈51 토큰** 기준.

---

## 1. 학습·매핑 저장 계층 (상위 학습기록)

추론 파이프라인은 아래 **브라우저 `localStorage` 키**와 연동된다. 에이전트·구현 시 **동일 키명**을 유지한다.

| 계층 | `localStorage` 키 | 역할 | 누적 |
|------|-------------------|------|------|
| **표준 배지 풀** | `plannode.standardBadgePool.v1` | 기본 **DEV 16 · UX 26 · PRJ 9**(합계 ≈51, BADGE-ALIGN 2026-05) 외 커스텀 토큰·트랙 — `getEffectiveBadgePool` | 사용자가 표준 배지 설정에서 저장 시 갱신 |
| **사용자 추론 규칙** | `plannode.badgeInferenceUserRules.v1` | `UserBadgeInferenceRule[]` — `setUserBadgeInferenceRules` / UI 미구현 시 API·콘솔 | 사용자가 덮어쓰기·초기화 가능 |
| **AI·외부 트리 누적 학습** | `plannode.badgeInferenceAiLearnedRules.v1` | 가져온 노드의 `metadata.badges`(표준 풀로 해석)로 규칙 생성·병합 — **`name` 전체**, 조건부 **`description` 첫 줄 발췌**, **`metadataHaystack` 한 줄 시그니처** | **누적** — 동일 `(field, contains)`면 `suggestBadges`만 합침; 최대 `AI_LEARNED_RULES_MAX`(400) 초과 시 배열 앞쪽 규칙 드롭 |

**AI 학습기록 갱신 트리거 (코드 정본):**

- `src/lib/stores/projects.ts` — `upsertImportedPlannodeTreeV1` 성공 후 **`mergeLearnedBadgeRulesFromImportedNodes(nodeList)`** (원본 가져오기 노드 기준).
- 프로그램적 로드: `mergeLearnedBadgeRulesFromPlannodeExportUnknown(obj)` — `{ nodes: [...] }` 형태 JSON 객체.

**해석:** 외부 AI(예: Crazyshot `BADGE_FULL` 등)가 채운 `metadata.badges`는 **동의어 해석 후 표준 토큰만** 규칙에 들어간다. 풀에 없는 문자열은 `resolveImportedBadgeToken`에서 탈락 — **표준 배지 풀 확장** 시 이후 가져오기부터 학습에 반영 가능.

### 1.1 재검증 — 「매핑율이 낮다」와 실제 동작

| 확인 항목 | 결과(샘플 파일·코드 기준) |
|-----------|---------------------------|
| **CRAZYSHOT `crazyshot_v5_plannode_BADGE_FULL.json`** | 노드 **약 119개** 중 **118개**가 `metadata.badges`에 배지 1개 이상. 파일 내 **고유 토큰 18종**은 모두 기본 풀 표기이거나 `badgeImportAliases` 동의어로 **표준 토큰으로 해석**된다(예: `ANALYSIS`→`API`, `COMPETITIVE`→`USP`). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
