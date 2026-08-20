---
trigger: always_on
description: - 유저가 모호하게 요청하면 바로 작업하지 말고, 용어집(docs/terminology.md)의 용어로 확인 질문을 한 뒤 진행할 것
---

# CLAUDE.md

## Communication Rules
- 유저가 모호하게 요청하면 바로 작업하지 말고, 용어집(docs/terminology.md)의 용어로 확인 질문을 한 뒤 진행할 것
- 용어집에 없는 새 UI 요소나 개념이 등장하면 용어집에 추가 제안할 것
- 작업 중 발견한 프로젝트 구조, 배포 방식, 기술 스택 등 중요한 정보는 이 CLAUDE.md에 자동으로 추가/갱신할 것

## Code Quality Rules
- **한 탭에 요청된 기능/변경은 나머지 탭(제작/요리/보스)에도 자동 적용할 것** — 요리솥 탭은 구조가 다르므로 예외이나, 적용 가능하다고 판단되면 함께 적용
- **중복 코드는 자동으로 공통화할 것** — 2곳 이상에서 동일/유사한 패턴이 발견되면 공유 컴포넌트, 훅, 유틸리티로 추출. 유저 요청을 기다리지 않고 선제적으로 수행
- 공통화 후 반드시 `docs/ui.md`의 공유 컴포넌트/훅 목록을 업데이트할 것
- 새 UI 작업 시 `docs/ui.md`의 화면 구조도와 공유 컴포넌트 목록을 먼저 확인하여 기존 것을 재사용
- 화면 구조가 변경되면(탭 추가, 새 패턴 도입 등) `docs/ui.md`의 화면 구조도를 자동 업데이트할 것

## UI Design Rules
- **새 화면/컴포넌트 작성 전 반드시 `docs/ui.md` 참고** — 기존 UI 패턴과 통일성 유지
- 게임 아이템 나열(재료, 전리품 등)은 반드시 `ItemSlot` 사용, 메타 정보(카테고리, 속성)는 `TagChip` 사용
- 기존 컴포넌트로 표현 가능한 경우 새 UI를 만들지 말 것 — 통일성 우선
- 아이콘/이미지 선택 시 **게임 내 이미지를 항상 우선** 사용할 것 (lucide/SVG 아이콘은 게임 이미지가 없을 때만 fallback)
- 개발/디버그 전용 페이지나 도구를 만들면 **DevMenu에 자동으로 항목 추가**할 것 (`src/components/AppShell.tsx`의 `DevMenu` → `items` 배열)

## Project
- Don't Starve Together 크래프팅 레시피 가이드 웹앱
- Next.js 16 (App Router, Static Export) + TypeScript + Tailwind CSS v4 + shadcn/ui
- Mac Mini 셀프호스팅 (Nginx + Cloudflare Tunnel), PWA 지원

## Session Start
- 세션 시작 시 글로벌 SessionStart hook이 `git fetch --all --prune` + `git pull --ff-only`를 자동 실행. Claude는 hook 출력을 확인하여 충돌/divergence가 있으면 사용자에게 알릴 것.
- ff-only 실패(브랜치 발산) 시 임의로 merge/rebase하지 말고 사용자에게 상태 보고.

## Branch & Deploy Strategy

**핵심 원칙**: feat 단위로 격리해서 작업하고, beta는 staging-only(검증용 여러 feat 합집합), main은 통과한 feat만 골라 머지. `/release`가 "beta 전체 → main"이 아니라 **"특정 feat 브랜치 → main"** 으로 동작.

### 배포 매핑
- `beta` push → `beta.dstcraft.com` (Mac mini 셀프호스팅, Cloudflare Tunnel, GitHub Actions self-hosted runner)
- `main` push → `www.dstcraft.com` (Production, 동일 runner)

### 작업 진입점: `/task`
**모든 코드 변경 작업은 `/task <한 줄 설명>`으로 시작**한다. 이 스킬이 다음을 자동 처리:
- GitHub 이슈 오픈 (제목/AC 초안 → 사용자 확인)
- main에서 분기한 `feat/<issue-num>-<slug>` 브랜치 생성
- `../dst-craft-<issue-num>` 워크트리 생성
- 현재 세션이 그 워크트리로 `cd` 이동 → 같은 세션에서 그대로 작업 진행

**왜**: feat 단위 격리로 main 머지 시 의도치 않은 변경 혼입 방지. 멀티 세션 충돌이 우려되면 다른 세션을 정리한 뒤 진행.

코드 변경이 없는 질문/탐색/설명만 `/task` 없이 메인 세션에서 답변. **메타/문서 변경(CLAUDE.md, `.claude/skills/`, `docs/`, `memory/`)도 같은 워크트리 패턴**을 따른다 — 일관성을 위해 예외 없음.

### feat 워크트리 워크플로우 (`/task` 이후)
1. **feat 분기 base는 항상 `main`** — `/task`가 자동으로 `git worktree add ../dst-craft-<num> -b feat/<num>-<slug> origin/main` 수행
   - `beta`에서 분기하지 말 것 — beta의 in-flight 커밋이 딸려 들어와 main 머지 시 의도치 않은 변경 포함 가능
   - feat끼리 독립 → 한 feat이 다른 feat의 미완성 변경을 끌어들이지 않음
2. **워크트리 안에서 작업 + 커밋** — `feat/<num>-<slug>` 브랜치에 누적 (`/task`가 현재 세션을 워크트리로 cd 이동시킨 뒤 그대로 진행)
3. **beta 배포 (staging)** — `/beta` 호출. 미커밋 변경이 있으면 자동으로 commit + origin push 한 뒤, 타겟 브랜치를 `beta`에 머지·푸시 → `beta.dstcraft.com` 자동 배포
   - 인자 없으면 현재 워크트리 브랜치. 다른 feat을 빠르게 beta로 올리려면 `/beta <브랜치명|이슈번호|이슈URL|자연어>`
   - 릴리즈노트/버전은 **건드리지 않음** — deploy-only
   - `/beta clear` 서브커맨드로 누적된 staging 머지를 청산 (origin/beta를 origin/main 기준으로 리셋, 파괴적 — 사용자 확인 필수)
4. **테스트 통과** — beta.dstcraft.com에서 의도대로 동작 확인
5. **main 머지 (production 배포)** — `/release` 호출. 인자 없으면 현재 워크트리 브랜치를 자동 인식.
   - `/release`가 그 feat 브랜치 하나만 main에 `--no-ff` merge
   - 머지 커밋에 `Closes #<num>` 자동 포함 → GitHub가 이슈 자동 close
   - main 머지 직전에 릴리즈노트/버전 bump를 한 번에 작성 (그 feat 분량만)
6. **워크트리 정리** — `/release`가 자동으로 `git worktree remove ../dst-craft-<num>` + `git branch -d` 처리

### 메인 워킹 디렉터리 규칙
- 메인 워킹 디렉터리(`/Users/jihwan-kim3/private-works/dst-craft`)는 항상 `main` 브랜치 유지
- 새 작업 시작 시 워크트리 생성 제안 (다른 feat과의 충돌 방지)
- SessionStart hook은 워크트리 디렉터리에서는 그 브랜치를 그대로 유지함 (main 강제 X)

### Beta 워크트리 (영속)
- 별도 워크트리 `../dst-craft-beta`에 `beta` 브랜치를 영속 유지 (없으면 `/beta` 스킬이 자동 생성: `git worktree add ../dst-craft-beta beta`)
- `/beta` 스킬은 이 워크트리에서 fetch/pull/머지/push 수행 — 메인 워크트리(=main)를 건드리지 않음
- 이 워크트리에서 직접 작업 금지 — 오직 `/beta`가 배포 용도로만 사용

### 직접 작업 / 머지 방향 규칙
- **`main` 직접 작업 금지** — 사용자가 *명시적으로* main 작업/푸시를 요청한 경우에만 허용
- **`beta` 직접 작업 금지** — 메타든 docs든 모두 워크트리 패턴(`/task` → `/beta` → `/release`)을 따른다
- **`main ← beta` 방향 머지 절대 금지** — beta는 in-flight feat의 합집합 검증용일 뿐 main의 입구가 아니다. `git merge --ff-only beta`(main에서) / `git merge beta` 등 beta를 main으로 흘리는 모든 명령 금지. 어기면 검증 안 끝난 다른 feat의 in-flight 커밋이 production에 따라 들어간다 (2026-05-08 사고, `docs/mistakes.md` 참조)
- **올바른 머지 방향**: `feat → beta` (=`/beta`), `feat → main` (=`/release`) 두 가지뿐. 두 쪽으로 각각 직접 머지하는 구조

### beta 브랜치 정리
- beta는 main + 검증중 feat들의 합집합. 머지된 feat가 main에 들어가도 beta에 남아있음 (no-op)
- **언제든 청소 가능** — beta는 staging 배포 전용이라 오염됐다 싶으면 `/beta clear`로 origin/main 기준으로 다시 만들면 됨. 파괴적 작업이 아닌 일상 청소.

### 슬래시 명령어 의미 정리
- **`/beta [타겟]`**: 미커밋 변경이 있으면 자동 commit + origin push, 그다음 타겟 브랜치를 `beta`에 머지·푸시 → `beta.dstcraft.com` 자동 배포. 인자 없으면 현재 워크트리 브랜치. 인자는 브랜치명 / 이슈번호 / 이슈URL / 자연어
- **`/beta clear`**: origin/beta를 origin/main 기준으로 리셋 (staging 청소). 일상 작업 — 오염됐다 싶으면 호출
- **`/release [타겟]`**: 타겟 브랜치를 `main`에 머지·푸시 → `www.dstcraft.com` Production 배포. 릴리즈노트/버전 bump 작성. 워크트리 정리

Vercel은 watchdog failover 용도로만 유지 (Phase 6 자동 DNS 전환).

## Architecture
- **프론트엔드**: `src/` — Next.js Static Export → Nginx serving (Cloudflare Tunnel 뒤)
  - 빌드: `npm run build` → `out/` (정적 파일)
  - 배포: `scripts/deploy-frontend.sh main|beta` (timestamped release + atomic symlink swap)
  - 캐시: prod HTML 1분, beta HTML no-store, 정적 자산 1년 immutable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fankimm/dst-craft](https://github.com/fankimm/dst-craft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
