---
trigger: always_on
description: 이 문서는 agent용 허브입니다. 상세 규칙을 이 파일에 복붙하지 말고, 작업 종류에 맞는 source of truth 문서를 먼저 읽습니다.
---

# DONGCHIMI-CLIENT Agent Guide

이 문서는 agent용 허브입니다. 상세 규칙을 이 파일에 복붙하지 말고, 작업 종류에 맞는 source of truth 문서를 먼저 읽습니다.

## 저장소 정체

- Product: 동치미 클라이언트
- Jira root key: `DCMFE-*`
- Client web key: `DCMCL-*`
- Market owner web key: `DCMSM-*`
- Design system key: `DCMDS-*`
- Design system web key: `DCMDSW-*`
- 현재 app: `apps/client`, `apps/market-owner`
- 예정 app: `apps/design-system-web`, 선택적 `apps/admin`, `apps/mobile`
- 현재 package: `packages/design-system`, `packages/eslint-config`, `packages/typescript-config`
- 예정 package: `packages/shared`, `packages/tailwind-config`
- Package manager, app layout, build system은 실제 `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `docs/architecture/*`를 source of truth로 봅니다.

## 기본 순서

1. 요구사항 분석과 설계 -> Jira 이슈 작성 -> Jira 이슈 티켓 발급
2. Jira 진행중 상태 변경 -> turbo gen으로 파일 생성 -> Jira 이슈 내용 기반 page, components, hooks 등 spec 문서 작성
3. 실제 컴포넌트, 페이지, 기능 등 구현 시작
4. 구현 내용 확인 및 검토
5. 추가 작업
   - 컴포넌트: story 추가 및 동작 검증
   - hook: 필요하면 단위 테스트 및 동작 검증
   - page: 로컬 route 확인 및 검증
6. 구조 개선
7. 커밋 계획 수립 -> 기능/작업 단위 커밋 -> 푸시 및 PR 작성
8. Jira 이슈 리뷰중 상태 변경 -> 코드 리뷰
9. 머지 및 Jira 이슈 진행상황 완료 변경

Jira/Figma/사진 기반 작업은 항상 `jira-design-implementation-workflow`를 먼저 적용합니다. Jira 이슈 작성만 요청받은 경우에는 구현을 시작하지 않습니다.

## 먼저 볼 문서

비자명한 작업은 아래 문서를 먼저 확인합니다. 단순 오탈자나 한 줄 변경은 필요한 문서만 비례해서 봅니다.

- `README.md`, `docs/index.md`, `docs/code-quality/index.md`
- `docs/architecture/repo-structure.md`
- `docs/conventions/git.md`, `docs/conventions/package-management.md`
- `docs/workflows/jira-issue-authoring.md`, `docs/workflows/spec-writing.md`, `docs/workflows/testing.md`, `docs/workflows/turbo-generators.md`, `docs/workflows/sentry.md`, `docs/workflows/pr-checklist.md`, `docs/workflows/pull-request-writing.md`

## 작업 유형별 라우팅

| 작업 유형                                  | 먼저 볼 문서                                                                                                                                                                                              |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Jira 이슈 생성, 보강, parent/sub-task 분해 | `docs/workflows/jira-issue-authoring.md`, `.agents/skills/jira-design-implementation-workflow/SKILL.md`, `templates/jira-issue-template.md`                                                               |
| Jira/Figma/사진 기반 작성 또는 착수        | `.agents/skills/jira-design-implementation-workflow/SKILL.md`, `recipes/jira-design-to-implementation.md`                                                                                                 |
| 프론트엔드 작업 오케스트레이션             | `.agents/skills/frontend-task-orchestrator/SKILL.md`, 가까운 recipe/spec                                                                                                                                  |
| 페이지 또는 라우트                         | `.agents/skills/page-feature-workflow/SKILL.md`, `recipes/add-page.md`, `templates/page.spec.md`                                                                                                          |
| 앱 shared 컴포넌트                         | `.agents/skills/app-shared-component-workflow/SKILL.md`, `recipes/add-app-shared-component.md`, `templates/component.spec.md`                                                                             |
| 앱 로컬 컴포넌트                           | `recipes/add-component.md`, 필요 시 `.agents/skills/refactor-evaluator/SKILL.md`                                                                                                                          |
| 디자인시스템 컴포넌트                      | `.agents/skills/design-system-component-workflow/SKILL.md`, `recipes/add-design-system-component.md`                                                                                                      |
| hook 또는 상태 로직                        | `recipes/add-hook.md`, 필요 시 `.agents/skills/refactor-evaluator/SKILL.md`                                                                                                                               |
| API query 또는 mutation                    | `.agents/skills/api-integration-workflow/SKILL.md`, `recipes/add-api-query.md`                                                                                                                            |
| form 또는 validation 흐름                  | `.agents/skills/form-flow-workflow/SKILL.md`, `recipes/add-form.md`                                                                                                                                       |
| refactor 판단                              | `.agents/skills/refactor-evaluator/SKILL.md`, `recipes/refactor-component.md`                                                                                                                             |
| 성능 이슈                                  | `.agents/skills/performance-diagnosis/SKILL.md`                                                                                                                                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TEAM-DONGCHIMI/DONGCHIMI-CLIENT](https://github.com/TEAM-DONGCHIMI/DONGCHIMI-CLIENT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
