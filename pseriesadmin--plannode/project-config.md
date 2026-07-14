---
trigger: always_on
description: Plannode PRD — 상용 웹앱 개발계획 협업·M5·IA·와이어·LLM(§10)·DB·로드맵(§1.05~§1.06)
---


# Plannode PRD (Product Requirements Document)

**버전**: 1.4 (1.3 + **상용 웹앱 개발계획 협업 서비스** 포지셔닝 §1.05·§1.06, M5·로드맵 정합)  
**작성·갱신**: 2026-04-22 (§1.05·§1.06: 2026-06-04)  
**상태**: Development  
**제품 포지션(한 줄)**: **상용 웹앱 개발계획 협업 서비스** — 노드 트리 기반 구조 설계·팀 동기화·PRD/명세/IA/와이어 산출  
**상세 TypeScript/스키마 예시**: `.cursor/plans/plannode-ai-enhancement-v3.md` 참고

**구현 스택 정합성**: **배포 앱의 UI·라우팅·동기화 껍질**은 SvelteKit+TypeScript(`src/routes`, `src/lib/stores`, `src/lib/supabase`)이며, **트리 캔버스·줌·간선·문서 패널의 실행 단일 축**은 내장 파일럿 `src/lib/pilot/plannodePilot.js` + `pilotBridge.ts`가 담당한다(동작·포팅 기준은 여전히 `docs/PILOT_FUNCTIONAL_SPEC.md`, 루트 `index.html`+`plannode.js`와의 갭은 동 문서 §9~§10). **v2 LLM·DB 목표**(§10·§11)는 제품 방향의 진실로 두되, **클라이언트에 이미 존재하는 모듈**(`src/lib/ai/*` 일부: 직렬화·매트릭스·모델 선택·IA보내 보조 등)과 **Supabase에 실제로 올라간 스키마·RPC**를 우선해 코드·PRD 불일치 시 본문 또는 `plannode-architecture.mdc`에서 명시한다.

---

## 1. 서비스 개요

### 1.0 현재 구현된 시스템 (코드 기준 요약)

아래는 **요구사항이 아니라 저장소에 존재하는 구현**을 한 장으로 묶은 것이다. 세부 흐름·클라우드 동기·협업 계약은 `.cursor/rules/plannode-architecture.mdc`(특히 **§10**).

| 층 | 구현 내용 |
|----|-----------|
| **앱 셸** | `+layout.svelte`: Supabase 환경 가드·스플래시·`LoginGate`·세션(`authSession`) 후 슬롯. 클라우드 설정 시 로그인 뒤 `loadProjectsFromLocalStorage()`. |
| **메인 오케스트레이션** | `+page.svelte`: 툴바·뷰 전환·모달·클라우드 플러시·Presence 연동. 뷰는 `activeView`: `tree` \| `prd` \| `spec` \| `ia` \| `ai` — 파일럿 `pilotSetActiveView`와 쌍을 맞춘다. |
| **파일럿 런타임** | `plannodePilot.js` (`initPlannode`): 노드 DOM·SVG 간선·미니맵·PRD/기능명세/IA/AI 패널 갱신. 트리 편집 중 **단일 진실**은 파일럿 상태이며, 저장 시 브리지로 스토어에 반영된다. |
| **브리지** | `pilotBridge.ts`: `onPersist` → `pilotNodesToStore` → `persistNodesFromPilot`(localStorage + 더티 마킹), `currentProject` 변경 시 `hydrateFromStore`, 필요 시 액세스 토큰·`plan_project_id` 콜백. |
| **클라이언트 상태** | `projects.ts` 등: `Project`·`Node` 플랫 목록, `activeView`, `plannode_projects_v3` / `plannode_nodes_v3_<id>` / `plannode_current_project_v3`. 루트 노드 규칙은 아키텍처 문서 §4와 동일. |
| **Supabase(설정 시)** | `client.ts`·`env.ts`: 미설정 시 안전한 플레이스홀더. **워크스페이스**: 사용자별 `plannode_workspace` JSON 번들 업서트/풀, 업로드 전 `mergeRemoteWorkspaceBeforeUpload`(LWW 성격), `workspacePush`·`cloudBackgroundSync` 디바운스·주기·가시성 트리거. **협업**: ACL·공유 프로젝트 슬라이스·revision/lock RPC 경로(`sync.ts` 등). **Presence**: Realtime으로 **선택 노드 등 메타**만 — 노드 본문은 번들·RPC 경로(번들 Realtime 스트리밍 아님). |
| **데이터 교환** | `plannodeTreeV1.ts`: 트리 v1 스키마 파싱·가져오기/업서트(백업·이식). |
| **AI / v2 클라이언트(부분)** | `src/lib/ai/*`: `contextSerializer`, `promptMatrix`, `modelSelector`, IA보내·배지 파이프 등 **클라이언트 모듈**이 존재한다. **§11 전면**(예: `ai_generations` 영속, `plan_nodes.path` 트리거 일원화)은 로드맵·DB 마이그레이션과의 **갭**으로 본다 — 구현 시 PRD·SQL·아키텍처를 한 번에 맞출 것. |

**한 줄:** 로컬·파일럿에서 트리를 편집하고, Svelte 스토어·localStorage가 1차 저장이며, Supabase가 켜지면 **워크스페이스 번들 + ACL·슬라이스 + Presence**가 그 위에 얹힌다 — **팀이 같은 프로젝트 구조를 신뢰할 수 있는 협업 층**이 핵심이다.

### 1.05 제품 포지셔닝 (에이전트·기획 공통 — 정본)

| 항목 | 내용 |
|------|------|
| **포지션** | **상용 웹앱 개발계획 협업 서비스** — 웹·앱 **기능·화면·요구**를 노드 트리로 설계하고, PRD·기능명세·IA·와이어를 **팀이 동일 구조**로 편집·동기화·보내는 SaaS |
| **폐기 라벨** | “1인 내부 플래닝 도구”·“개인용 메모 수준” — **PRD·규칙·에이전트 기본 가정에서 사용 금지**. 저장소 **개발 운영**이 1인 에이전틱이어도 **제품·시스템 설계 완성도**는 **상용 협업** 기준을 따른다. |
| **핵심 가치 축** | (1) 트리 SSoT·구조적 합의 (2) **팀 공유·클라우드 동기화**(M5·§3) (3) 기획문서·IA/와이어 산출 (4) (선택) LLM·하네스 품질(§10) |
| **현행 코드 상한** | 프로젝트 접근 **최대 5계정**(소유자+멤버 4) — `plannodeCollabLimits.ts`. **베타·상용 확장**에서 인원·역할·충돌 UX는 단계 강화(§6 Phase 2+) |

### 1.06 에이전트 구현·설계 기준 (하네스 「경량화」와 구분)

`AGENTS.md` **GP-12**·하네스 **경량화**는 **PRD·TASK·plan-output 밖**의 불필요한 **신규 모듈·추상·미래용 뼈대** 억제이다. **아래는 “경량화”로 생략·축소하지 않는 상용 협업 정상 경로**다.

| 구분 | 에이전트가 따를 기준 |
|------|---------------------|
| **협업·동기화(M5)** | `plannode-architecture.mdc` **§10** — 번들·슬라이스·revision·structure_ops·Presence·모달 편집 중 pull 보류 등 **계약을 온전히** 유지·수정한다. “1인이면 pull/slice/ops 생략해도 됨” 설계 **금지**. |
| **동기화 결함** | meta drift false-skip·ACL 403·slice 누락 등은 **버그·불변식 위반** — BACKLOG·“나중에”로 미루지 않는다(§7·architecture §10.11~§10.12). |
| **알려진 한계** | OT/CRDT·필드 단위 병합 없음(LWW)은 **문서화된 제품 한계**이지, **번들/ops/revision 경로 자체를 단순화**할 명분이 아니다. |
| **오버엔지니어링** | PRD M#·F#·TASK에 **없는** 범용 프레임워크·중복 저장소·v2 LLM 전면 선구현은 여전히 **금지**(GP-12). **협업 경로의 견고함**과 **스코프 밖 확장**을 혼동하지 말 것. |

### 1.1 제품 정의

**Plannode**는 **상용 웹앱 개발계획 협업 서비스**(§1.05)로, **AI를 활용하는 기획 보조**(§10, 선택)와 **정보 구조(IA)·문서/와이어 산출**을 **동일한 제품 가치**로 둔다. 제품 기능을 노드 트리로 시각 설계하고, **PRD·기능명세**에 더해 **정보 구조(IA) 문서·와이어프레임(저충실도)**·(확장 시) API/ERD 등으로 **변환·내보내는** **웹 기반** 도구다.

**용어 (혼동 방지, 에이전트·기획자 공통)**  
- **IA** = *Information Architecture* = **정보 구조** — 내비·화면(또는 모듈) 계층, 라벨, 사용자 이동 경로. **LLM이 아님.** 트리에서 **도출·렌더·내보내기**하는 **구조 기반 산출**이 본질.  
- **AI** = 본 문서에서 **인공지능/LLM**(Claude 등)으로 **의미**를 통일(§10). “AI 기획” 문구는 **LLM 지원**을 뜻하며 **IA(정보 구조)와 별도**다.

**핵심 문제 → 해결**

- 이미지/수동 PRD: 불일치·동기화 비용
- 팀 협업: 댓글 수준 → 구조적 합의 어려움
- **AI 개발 기획문서** 입력: **노드 텍스트만 전달 시 맥락 소실** → 일반론적 출력(§10)

**가치 제안**

**노드 맵(구조) → 품질 있는 PRD/명세·IA/와이어·(선택) LLM/하네스** — (1) **IA·와이어**는 **트리 구조에서 직접** 도출 가능해야 하고, (2) **기획문서( PRD/명세·하네스) LLM 품질**은 **ContextSerializer(10.2)로 컨텍스트 인코딩**이 핵심(§10).

### 1.2 USP (요지)

- 노드 기반 **자동 PRD/MD** 및 뷰
- **IA(정보 구조) + 와이어프레임** — 트리 → 화면(또는 모듈) **계층·이동 경로**·**저충실도 블록/구역** 문서(뷰+내보내기, §3 F2-4). *Figma 수준의 시각디자인 도구는 목표가 아님(§1.3).*
- Supabase **권한·워크스페이스 번들 동기·ACL·Presence**(설정 시); **노드 단위 실시간 공동편집(OT/CRDT)** 및 **§11 DB 전부**는 여전히 로드맵·갭으로 본다(`plannode-architecture.mdc` §10.9 한계).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
