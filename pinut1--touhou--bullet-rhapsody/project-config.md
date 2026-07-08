---
trigger: always_on
description: > **모든 AI CLI (Gemini, Claude 포함) 는 아래 파일을 절대 직접 수정/생성/삭제하지 말 것.**
---

# Touhou Bullet Rhapsody — GEMINI.md

## 🚨 CLI 절대 금지 규칙

> **모든 AI CLI (Gemini, Claude 포함) 는 아래 파일을 절대 직접 수정/생성/삭제하지 말 것.**

```
금지 확장자: .unity  .prefab  .asset  .meta  .controller  .anim  .overrideController
```

- Unity 씬/프리팹/에셋은 **반드시 Unity 에디터에서만** 수정
- GUID / fileID 텍스트 직접 편집 **절대 금지**
- 빌드 세팅, 씬 추가 등도 에디터에서 직접 하거나 사용자에게 단계별 안내로 전달
- **수정 가능한 파일**: `.cs` `.md` `.txt` `.json` `.xml` 등 순수 텍스트 파일만

## 🔇 CLI 출력 규칙

- 작업 과정 혼잣말 **출력 금지** ("이미 턴을 썼으니~", "수정!", "고!", "시작!" 등)
- 완료 후 **결과만 간결하게** 출력 (변경한 파일명 + 핵심 내용 1~2줄)
- 불필요한 중간 상태 메시지, 감탄사, 진행 나레이션 모두 생략

---


## 프로젝트 개요

- **장르**: 탄막 로그라이크 (엔터 더 건전 + 아이작 스타일, 탑다운 뷰)
- **엔진**: Unity 2D (C#)
- **개발 형태**: 1인 개발
- **데모 목표**: 루미아 → 치르노 스테이지 + 간단한 엔딩
- **데모 마감**: 2026년 4월 10일

---

## 씬 구조

```
씬 0: MainMenuScene — 타이틀 + 버튼 (게임 시작 / 종료)
씬 1: StoryScene    — 인트로 스토리 (말풍선 타자기 연출, StoryDirector 제어)
씬 2: EternalScene  — 영원정 허브 (NPC 대화, 스테이지 입구, HubManager)
씬 3: SampleScene   — 실제 게임 (맵 생성, 전투, 보스)
```

> Build Settings 순서: MainMenuScene(0) → StoryScene(1) → EternalScene(2) → SampleScene(3)

---

## 핵심 아키텍처

### 재화 시스템
- **조각 (Shard)**: 유일한 영구 재화. 런 중 적 처치/방 클리어/보스 처치로 획득.
  이월됨 — 런 종료 후에도 보존. 영원정에서 강화(메인) 또는 겜블(보조)에 소비.
- **런 중 골드**: PlayerInventory.gold — 런 전용, 상점에서 소비
- **럭키코인**: **폐지됨** — 언급하지 말 것

조각 획득량:
- 잡몹 처치: EnemyData.scoreValue (1~2개)
- 방 클리어: 3개 고정 (AddRoomClear 내부 처리)
- 보스 처치: 15개 고정 (AddBossDefeat 내부 처리)

### 보스 구조
```
BossRegistry (ScriptableObject)
  └ storyBosses[] — stage번호 : BossEntry(prefab + BossData) 1:1 매핑

TilemapPainter → 보스방 RoomController에 bossRegistry 자동 주입
RoomController.TrySpawnBoss() → BossRegistry.GetStoryBoss(currentStage) → 스폰
```
보스 추가 시 BossRegistry ScriptableObject에 항목만 추가하면 됨. 씬 수정 불필요.

### 이벤트 구독 패턴
- UnityEvent 대신 C# `Action` 사용 (`Boss.OnBossDeath`, `Boss.OnPhaseChanged`)
- 오브젝트 풀링: `PoolManager` 사용

---

## 스크립트 폴더 구조

```
Assets/Scripts/
  Audio/         BossBGMController.cs
  Bullet/        BulletController.cs
  Camera/        CameraFollow.cs (클래스명 CameraFallow — 오타지만 그대로)
  Data/          GameData.cs, PermanentData.cs
  Enemy/         Boss.cs, Boss_Cirno.cs, BossData.cs, BossRegistry.cs
                 Enemy.cs, EnemyBullet.cs, EnemyData.cs
                 SandBag.cs, TankEnemy.cs
  Interfaces/    IDamageable.cs, IExplodable.cs, ITrackableEnemy.cs
  Item/          BombProjectile.cs, ItemData.cs, ItemPickup.cs
  Manager/       GameDirector.cs, GameManager.cs, EternalManager.cs (미적용)
  MapGeneration/ DoorController.cs, MapGenerator.cs, RoomInitializer.cs
                 RoomNode.cs, StageManager.cs, TilemapPainter.cs
  Player/        PlayerController.cs, PlayerHitbox.cs
                 PlayerInventory.cs, PlayerStats.cs
                 WeaponController.cs, WeaponInventory.cs
  Room/          BattleTrigger.cs, DestructibleObject.cs
                 LootTableData.cs, RewardManager.cs
                 RoomController.cs, RoomData.cs
                 NPCInteraction.cs, StagePortal.cs
  Stage/         EnemySpawner.cs, StageData.cs
  Story/         StoryScriptData.cs  — 대사 데이터 ScriptableObject
                 DialogueBubble.cs   — 말풍선 UI (타자기 효과)
                 StoryDirector.cs    — 스토리 씬 흐름 제어
                 HubManager.cs       — 영원정 허브 관리
  UI/            AmmoDisplay.cs, BossHPBar.cs, BulletTimeGauge.cs
                 DamageVignette.cs, GameOverUI.cs, HPDisplay.cs
                 InventoryUI.cs, ReloadUI.cs, SpellCardDisplay.cs
                 WeaponSlotUI.cs
  Weapon/        WeaponData.cs
  TimeManager.cs
```

---

## 파일별 주요 설계 결정

### GameData.cs (⚠️ 프로젝트에 구버전 있음 — 교체 필요)
- DontDestroyOnLoad 싱글턴
- `shards` 필드 — 조각 (구버전은 `totalScore`로 돼있음, 교체 필요)
- `shardsEarnedThisRun` — 이번 런 획득량 (결과 화면용)
- `ResetRunData()`에서 shards 초기화 안 함 (이월)
- `CalculateFinalCoins()` — 삭제됨 (구버전에 있음, 제거 필요)

### PermanentData.cs (⚠️ 프로젝트에 구버전 있음 — 교체 필요)
- static 클래스, PlayerPrefs 래퍼
- MD5 해시 변조 방지 (SECRET 키 포함)
- `PurchaseUpgrade()` — GameData.Instance.shards 직접 차감
- LuckyCoins 관련 메서드 전부 삭제됨 (구버전에 있음, 제거 필요)
- 강화 비용 (데모 기준): HP 10/20/35, 게이지 15/25/40, 골드 8/15/25 조각

### Boss.cs
- abstract 클래스
- `BossData` 주입: RoomController가 스폰 시 `boss.bossData = entry.bossData`
- `CurrentHP => currentHP` 프로퍼티로 읽기 전용 노출
- `Die()`에서 `GameData.Instance?.AddEnemyKill(score)` + `AddBossDefeat()` 호출

### RoomController.cs
- `bossSpawnPoint` 없음 — 보스는 `transform.position` (방 중앙)에 스폰
- `enemySpawner` — `[HideInInspector]`, RoomInitializer가 런타임 주입
- `bossRegistry` — TilemapPainter가 런타임 주입

### PlayerStats.cs
- `maxHP`, `currentHP`, `isDead`, `hasTakenDamage` (노다이 트래커)
- 영구 강화 수치는 EternalManager.ApplyPermanentUpgrades()에서 Start 시 반영

---

## 미적용 파일 목록 (뽑았지만 프로젝트에 아직 안 넣은 것)

다음 파일들은 설계/작성됐지만 실제 프로젝트에 아직 교체/추가 안 됨:

| 파일 | 위치 | 상태 |
|------|------|------|
| GameData.cs (shards 버전) | Assets/Scripts/Data/ | ⚠️ 교체 필요 |
| PermanentData.cs (조각 버전) | Assets/Scripts/Data/ | ⚠️ 교체 필요 |
| EternalManager.cs | Assets/Scripts/Manager/ | 🔲 신규 추가 필요 |
| UpgradeUI.cs | Assets/Scripts/UI/ | 🔲 신규 추가 필요 |
| TitleManager.cs | Assets/Scripts/Manager/ | 🔲 신규 추가 필요 |
| StoryScriptData.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| DialogueBubble.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| StoryDirector.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |
| HubManager.cs | Assets/Scripts/Story/ | ✅ 작성 완료 (Unity 세팅 필요) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pinut1/TouHou_-Bullet_Rhapsody](https://github.com/Pinut1/TouHou_-Bullet_Rhapsody) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
