---
trigger: always_on
description: Three.js 기반 3D 퍼즐 게임. 아이소메트릭(직교 투영) 시점에서 블록 위를 클릭해 캐릭터를 이동시키고, 착시(Illusion), 중력 반전 등 특수 메카닉을 활용해 골을 달성하는 게임.
---

# CLAUDE.md — hihi 프로젝트 가이드

## 프로젝트 개요

Three.js 기반 3D 퍼즐 게임. 아이소메트릭(직교 투영) 시점에서 블록 위를 클릭해 캐릭터를 이동시키고, 착시(Illusion), 중력 반전 등 특수 메카닉을 활용해 골을 달성하는 게임.

**실행**: `npm run dev` (Vite, `--host` 옵션으로 로컬 네트워크 모바일 접속 가능)  
**빌드**: `npm run build` (TypeScript 컴파일 후 Vite 번들)  
**타입 체크**: `npx tsc --noEmit`

---

## 기술 스택

- **Three.js** (`^0.184`) — 3D 렌더링, OrthographicCamera
- **GSAP** (`^3.15`) — 모든 애니메이션 (캐릭터 이동, UI, 이펙트)
- **TypeScript** (`~6.0`) — 전체 코드베이스
- **Vite** (`^8.0`) — 번들러

---

## 디렉토리 구조

```
src/
├── core/
│   ├── GameManager.ts        # 게임 전체 루프·상태 관리 (핵심)
│   ├── CameraController.ts   # 카메라 fly-in / pulse 애니메이션
│   ├── InputManager.ts       # 마우스 클릭·드래그 입력 처리
│   ├── Renderer.ts           # Three.js WebGLRenderer 래퍼
│   ├── GraphicsSettings.ts   # 품질·색상·캐릭터 설정 (localStorage)
│   ├── ProgressStore.ts      # 스테이지 언락 진행 저장
│   └── TutorialSequencer.ts  # 튜토리얼 단계별 시퀀스
│
├── world/
│   ├── Level.ts              # LevelData 로드, 블록·이펙트 생성, 가시 애니메이션
│   ├── Block.ts              # 블록 메시 생성 (variant, recolor, regeometry)
│   ├── PathGraph.ts          # 노드 기반 이동 경로 그래프 (BFS)
│   ├── RotatingSection.ts    # 회전 섹션 (드래그로 회전, snap)
│   ├── SwitchManager.ts      # 스위치 블록 (spawn/move, hold/toggle)
│   ├── ElevatorManager.ts    # 엘리베이터 블록
│   ├── PatrolManager.ts      # 패트롤(자동 이동) 블록
│   ├── StarBackground.ts     # 배경 별 파티클
│   └── WorldRotateManager.ts # 맵 전체 회전 블록 (X·Y축, 각도 설정)
│
├── character/
│   ├── Character.ts          # 캐릭터 메시 (body/head, setFlipped)
│   └── CharacterController.ts # 노드 간 이동 GSAP 애니메이션, flip 상태
│
├── mechanics/
│   ├── StarManager.ts        # 별 수집 (normal/flipped 구분)
│   └── TeleportManager.ts    # 순간이동 패드 이펙트
│
├── illusion/
│   └── IllusionManager.ts    # 착시 연결 감지·활성화
│
├── editor/
│   ├── LevelEditor.ts        # 인게임 레벨 에디터 (전체 UI)
│   └── CustomLevelStore.ts   # 커스텀 레벨 localStorage 저장
│
├── ui/
│   ├── HUD.ts                # 인게임 HUD (별 카운터, 클리어 화면)
│   ├── StageSelectUI.ts      # 스테이지 선택 화면
│   ├── TitleScreen.ts        # 타이틀 화면
│   ├── SettingsScreen.ts     # 설정 화면 (품질, 색상, 캐릭터)
│   ├── TutorialHint.ts       # 튜토리얼 힌트 말풍선
│   └── EditorLobby.ts        # 에디터 로비
│
├── fx/
│   ├── AudioManager.ts       # 효과음 (Web Audio API)
│   └── ParticleSystem.ts     # 파티클 버스트 이펙트
│
├── levels/
│   ├── registry.ts           # 레벨 메타 목록 (level_01 + custom_N)
│   ├── level01.json          # 튜토리얼 레벨
│   └── level_custom_N.json   # 커스텀 스테이지 1~40
│
└── main.ts                   # 진입점
```

---

## LevelData JSON 스키마

`src/world/Level.ts`의 `LevelData` 인터페이스가 정의. 주요 필드:

```typescript
{
  id: string;
  name: string;
  backgroundColor: string;       // hex 색상
  blocks: BlockData[];           // 모든 블록
  character: { startNodeId: string };
  goal: { blockId: string; flipped?: boolean };  // flipped=true면 뒤집힌 상태에서만 달성
  midpoint?: { blockId: string };                // 중간 체크포인트
  stars?: Array<{ nodeId: string; flipped?: boolean }>; // flipped=true면 아랫면에 배치
  gravityFlipBlocks?: Array<{ nodeId: string }>; // 중력 반전 블록
  mapRotateBlocks?: Array<{ nodeId: string; axis: 'x'|'y'; angle: number; pivotY?: number }>; // 맵 전체 회전 블록
  ladders?: Array<{ nodeA: string; nodeB: string }>;
  teleporters?: Array<{ nodeA: string; nodeB: string }>;
  switches?: Array<{ switchNodeId, targetNodeId, mode, type, moveTarget? }>;
  elevators?: Array<{ nodeId, bottomY, topY, duration, mode }>;
  patrols?: Array<{ nodeId, axis, distance, duration }>;
  zones?: ZoneDef[];             // 구역별 카메라 타깃 자동 전환
  initialCamera?: { azimuth, polar, distance, targetY };
}

// BlockData
{
  id: string;
  position: [x, y, z];
  size: [w, h, d];
  color: string;                 // hex
  walkable: boolean;
  variant?: string;              // 'default' | 'rounded' 등
  isSpike?: boolean;
  spikeType?: 'always' | 'blinking';
}
```

**블록 좌표계**: `position`은 블록 중심. `node.position.y` = 블록 윗면 Y (중심 + halfHeight).  
**flipped Y**: `node.position.y - 2 * node.halfHeight` = 블록 아랫면 Y.

---

## 핵심 메카닉

### 중력 반전 (Gravity Flip)
- `gravityFlipBlocks`에 등록된 블록을 밟으면 플레이어가 뒤집힘
- **플레이어 flip**: `CharacterController.setFlipped(bool)` → `Character.setFlipped()` → `mesh.rotation.x = Math.PI`
- **카메라 flip**: `camera.up.set(0, -1, 0)` + `orbit.rotateSpeed` 부호 반전 + polar angle 미러
- **이동 Y 계산**: flipped 시 블록 아랫면에 서있음 (`_nodeY()`, `_flipped ? -halfHeight : halfHeight`)
- **별/골 flip 구분**: `StarManager.tryCollect(nodeId, isPlayerFlipped)` — 플레이어 상태와 별 상태 불일치 시 수집 불가
- **이펙트**: 블록 위·아래 각 5개 사각형 링이 GSAP으로 수축·페이드 반복 (`Level._buildRingEffect`)

### PathGraph / 이동
- `PathGraph.build()` — 블록 위치 기반으로 인접 노드 연결 (BFS `findPath`)
- `CharacterController.moveTo(node)` — GSAP 타임라인으로 한 스텝씩 이동
- `onArrival(nodeId)` 콜백 → GameManager에서 골/별/스위치 등 판정

### 맵 전체 회전 (Map Rotate Block)
- `mapRotateBlocks`에 등록된 블록을 밟으면 전체 맵이 X 또는 Y축으로 회전
- **씬 계층**: `scene → flipPivot(Group) → levelGroup(Group) → 블록·마커·링`
- `WorldRotateManager.setup()`: flipPivot 위치를 맵 중심(cx, pivotY, cz)으로, levelGroup을 반대 오프셋으로 설정 → 오프셋 상쇄로 초기 위치 유지
- **회전 중 추적**: goalGlow/goalMarker/midpointMarker/텔레포터 링이 levelGroup 안에 있어 실시간으로 따라감
- **회전 완료 후**: `graph.refresh()` → `_buildIllusionConnsWorldSpace()` (착시 재계산) → `_refreshWorldElements()` (마커·링 worldToLocal 변환으로 재배치)
- **에디터**: 주황색 링 이펙트, axis/angle/pivotY 설정 가능

### 착시 (Illusion)
- 두 블록이 특정 카메라 각도에서 연결된 것처럼 보이는 효과

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seongyooo/hihi-illusion](https://github.com/seongyooo/hihi-illusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
