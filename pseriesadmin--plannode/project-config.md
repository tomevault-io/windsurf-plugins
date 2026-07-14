---
trigger: always_on
description: Plannode SvelteKit·파일럿·Supabase 기능 구조·데이터 흐름 표준 (유지보수·포팅 기준)
---


# Plannode 소프트웨어 아키텍처 표준

이 문서는 **현재 구현된** Plannode의 계층·모듈·데이터 흐름을 한곳에 묶은 **유지보수용 아키텍처 기준**이다.

**제품 포지션:** **상용 웹앱 개발계획 협업 서비스** (`plannode-prd.mdc` §1.05). **§5·§10(클라우드·노드 CRUD·동기화)** 는 “부가 옵션”이 아니라 **M5 협업의 구현 핵심**이다 — 에이전트는 동기화·ACL·충돌 경로를 **온전한 상용 수준**으로 설계·수정한다(§1.06).

제품 범위·로드맵은 `plannode-prd.mdc`, 파일럿 동작 세부는 `docs/PILOT_FUNCTIONAL_SPEC.md`, 배포·인프라는 `.cursor/plans/PLANNODE_INTEGRATED_GUIDE.md`가 우선한다.

## 1. 전체 구성 (하이브리드)

| 층 | 역할 | 주요 위치 |
|----|------|-----------|
| **앱 셸** | 라우팅·인증 게이트·프로젝트 하이드레이션 | `src/routes/+layout.svelte` |
| **메인 UI·동기화 오케스트레이션** | 툴바·뷰 전환·모달·클라우드 플러시·Presence | `src/routes/+page.svelte` |
| **파일럿 런타임** | 캔버스·노드 DOM·SVG 간선·PRD/명세/AI 패널 갱신 | `src/lib/pilot/plannodePilot.js` (`initPlannode`) |
| **브리지** | 파일럿 ↔ Svelte 스토어 양방향 동기화 | `src/lib/pilot/pilotBridge.ts` |
| **클라이언트 상태** | 프로젝트·노드·뷰·모달 | `src/lib/stores/projects.ts`, `authSession.ts`, `workspaceDirty.ts` |
| **백엔드(클라우드)** | Auth·RLS·워크스페이스 번들·ACL·Realtime | `src/lib/supabase/*`, `docs/supabase/*.sql` |

**원칙:** 트리 캔버스·줌·간선·미니맵의 **단일 진실은 파일럿**이며, Svelte는 **껍질·동기화·권한·저장소**를 맡는다. 파일럿이 기대하는 **DOM id·이벤트 계약**을 바꿀 때는 `docs/PILOT_FUNCTIONAL_SPEC.md` §9~§10과 대조한다.

## 2. 라우팅·부트 순서

1. **`+layout.svelte`**: `initAuthSession()` → Supabase 미설정 시 안내 스플래시 → 로그인 필요 시 `LoginGate` → 그 외 `<slot />`.
2. 로그인 후·클라우드 설정 시: `loadProjectsFromLocalStorage()` (프로젝트 목록·현재 프로젝트 복원).
3. **`+page.svelte`**: `mountPilotBridge()` → 파일럿 초기화 및 `currentProject` 구독으로 캔버스 하이드레이트; 언마운트 시 `destroy()`.

## 3. 파일럿 브리지 계약 (`pilotBridge.ts`)

- `initPlannode({ delegateTabs, delegateProjectModal, seedDemoProjects, onPersist, getStoreNodesForCollabMerge, … })`로 파일럿을 띄운다.
- **`onPersist`**: 파일럿이 노드를 저장할 때 → `pilotNodesToStore`로 `Node[]`로 매핑 후 `persistNodesFromPilot` → 로컬 스토리지 + (설정 시) 클라우드 dirty 마킹.
- **`getStoreNodesForCollabMerge`**: 모달 **저장** 직전 스토어( pull 반영분)를 파일럿에 합칠 때 사용 — `pilotBridge.ts`가 `storeNodesToPilot(get(nodes))` 제공. 상세·CRUD별 경로는 §10.10.
- **`currentProject` 구독**: 프로젝트 변경 시 `hydrateFromStore(project, storeNodesToPilot(nodes))` — **노드 스토어를 먼저 맞춘 뒤** `currentProject`를 세팅하는 순서가 중요하다 (`projects.ts`의 `selectProject` 주석과 동일).
- **뷰 동기화**: `pilotSetActiveView('tree' | 'prd' | 'spec' | 'ai')` — Svelte의 `activeView` 스토어와 함께 호출해 파일럿 내부 탭 상태와 일치시킨다.

## 4. 클라이언트 상태·영속성

| 스토어 / 키 | 용도 |
|-------------|------|
| `projects`, `currentProject`, `nodes` | 프로젝트 메타·플랫 노드 목록 |
| `activeView`, `showProjectModal` | UI |
| `localStorage` `plannode_projects_v3`, `plannode_nodes_v3_<projectId>`, `plannode_current_project_v3` | 오프라인 1차 저장 |
| `workspaceDirty` / `cloudSyncBadge` | 푸시 대기·동기화 UI 배지 |

루트 노드: `makeRootNode` — `id = project.id + '-r'`, `node_type: 'root'`, `num: 'PRD'`.

## 5. Supabase·클라우드 동기화

- **클라이언트**: `src/lib/supabase/client.ts` — URL/anon 미설정 시 placeholder로 모듈 로드 실패 방지; **실제 호출**은 `isSupabaseCloudConfigured()` 등으로 가드.
- **푸시/머지**: `workspacePush.ts`, `sync.ts`, `cloudBackgroundSync.ts` — 주기·가시성·pagehide 시 플러시.
- **ACL·공유**: `projectAcl.ts`, `ProjectAclModal.svelte` — 소유자 행·초대·워크스페이스 소스 복구 RPC와 연계. 스키마·RPC는 `docs/supabase/` SQL 파일명을 코드·주석과 맞출 것.
- **Presence**: `projectPresence.ts` — 현재 프로젝트·ACL 이메일 목록 기반 Realtime. **고장·복원 시 상세는 아래 §5.1 (검색: `PRESENCE_PEER_MERGE` · 「원격선택-null」).**

### 5.1 「원격선택-null」— Presence 피어 메타 배열 병합 (`PRESENCE_PEER_MERGE`)

**공식 검색 태그(grep·Cursor 검색용):** `PRESENCE_PEER_MERGE` · `원격선택-null` · `presenceState` · `메타배열` · `selected_node_id` · `__plannodePresencePeers` · `np-avatar` · `syncPeersFromState`

**목적:** 공유 세션에서 원격 사용자가 선택 중인 노드 id를 `projectPresencePeers` → `window.__plannodePresencePeers` → 파일럿 `.np-avatar`까지 일치시킨다 (PRD M5·협업 방향).

| 구분 | 위치·계약 |
|------|-----------|
| 채널 topic | `plannode:project:<projectId>` — 소유자·공유자 **동일 projectId**여야 같은 방에 있다. |
| track 페이로드 | `user_id`, `email`, `selected_node_id` — `sendPresenceTrack` (`projectPresence.ts`). |
| 로컬 선택 → track | 파일럿 `plannode-node-select` → `+page.svelte` 리스너 → `updateMySelectedNode(nodeId)`. |
| 구독 직후 재동기 | `SUBSCRIBED` 후 `plannode-presence-subscribed` → 파일럿 `maybeEmitNodeSelect`로 `lastEmittedSelIdForPresence` 리셋 후 현재 `selId` 재발행 (구독 전에 나간 선택 이벤트 보정). |
| 스토어 → 파일럿 | `+page.svelte` 반응형으로 `window.__plannodePresencePeers = $projectPresencePeers` 및 `plannode-presence-update` 디스패치. |
| 이벤트 구독 | `presence` **`sync`**와 **`join`** 모두에서 `presenceState()`를 읽어 peers 갱신 — `track()`만 바뀐 경우에도 반영되도록. |

**증상(회귀 시):** 공유자 콘솔에서 `window.__plannodePresencePeers`의 원격 피어 `selected_node_id`가 **항상 `null`**인데, 소유자는 노드를 클릭하고 있다.

**근본 원인(필수 이해):** Supabase Realtime `presenceState()`는 presence **key**(`config.presence.key`, Plannode에서는 `myUserId`)마다 **메타 객체 배열**을 돌려준다. `channel.track({...})`를 짧은 간격으로 여러 번 호출하면 **같은 key 아래에 여러 항목이 쌓일 수 있고**, 그중 **앞쪽 항목의 `selected_node_id`만 null**인 경우가 있다(중간 null track, 캔버스 팬으로 `selId` 해제 등). **배열의 첫 원소만 파싱하면 원격 피어가 영구적으로 null로 보인다.**

**핵심 수정(복원 시 이 로직을 유지):** `subscribeProjectPresence` 내부 `syncPeersFromState`에서, 각 key의 `metas[]`를 **순회하며 하나의 `ProjectPresencePeer`로 병합**한다. 규칙: **`selected_node_id`가 `null`이 아닌 항목이 나오면 그 값으로 덮어쓴다**(배열 전체를 스캔). 이후 필터·`seen`·`projectPresencePeers.set`은 기존과 동일.

**보조 완화(선택·부가):** `updateMySelectedNode(null)`은 캔버스 빈 곳 클릭 등으로 잦을 수 있어 **짧은 debounce 후** null을 track(깜빡임·불필요한 null 브로드캐스트 완화). 과도한 `track` 반복은 메타 스택을 키우므로 **구독 직후 불필요한 재`track` 루프는 넣지 않는다.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pseriesadmin/plannode](https://github.com/pseriesadmin/plannode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
