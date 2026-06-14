---
trigger: always_on
description: 이 파일은 Claude(또는 다른 LLM 코드 어시스턴트)가 본 프로젝트에 처음 진입했을 때 빠르게
---

# Voice Spell Multiplayer — CLAUDE.md

이 파일은 Claude(또는 다른 LLM 코드 어시스턴트)가 본 프로젝트에 처음 진입했을 때 빠르게
컨텍스트를 잡고 일관된 스타일로 코드를 추가/수정할 수 있도록 작성된 가이드입니다.

본 가이드는 `Assets/02. Script/` 폴더의 실제 파일을 분석해 추출한 **실제 코드 패턴** 기반이며,
추측이 아닙니다. 새 파일을 작성할 때는 이 문서의 컨벤션과 가장 가까운 기존 파일을 먼저
참고하세요.

---

## 0. 이 게임은 무엇인가 (한 줄 정의)

> **약한 음성 마법 1개로 시작해, 맵의 몬스터를 사냥하며 새 키워드를 배우고 기존 스킬을
> 강화해 자기만의 마법 빌드를 만들어가는 4인 FFA 음성 마법 배틀로얄. 강해질수록 외쳐야
> 할 주문이 늘어나고, 그 발화는 다른 플레이어에게 그대로 들리기에 빌드의 깊이가 곧
> 노출의 위험이 된다.**

상세 디자인 문서: 프로젝트 루트의 `GAME_DESIGN.md` 참조. 본 CLAUDE.md 는 코드 컨벤션
중심, GAME_DESIGN.md 는 게임 디자인 헌법.

### 0.1 핵심 게임플레이 루프

```
[라운드 시작 전: 시작 마법 선택 (화염구 / 얼음계)]
                      ↓
[Phase 1 — 사냥/성장 (조용한 시기)]
  슬라임 처치 → 크리스탈 드롭 → 줍기 → 레벨업
  레벨업 시 3개 옵션 중 1개 선택 (스킬 강화)
                      ↓
[Phase 2 — 조우/심리전 (시끄러운 시기)]
  강해진 자는 자주 외쳐야 함 → 더 자주 들킴
                      ↓
[Phase 3 — 결착]
  마지막 한 명 남을 때까지 PvP, 사망 시 관전
```

### 0.2 핵심 설계 원칙 (GAME_DESIGN.md §4 "게임 정체성" 발췌)

1. **시전 = 의사소통** — 한 행위가 두 가지 목적을 동시 수행. 키 누르는 게임에선 불가능한
   장면이 가능 ("화염구!" 외친 직후 농담 한 마디 → 상대 방심).
2. **빌드의 깊이 = 위험의 깊이** — 강해질수록 외쳐야 할 주문이 길어지고 더 자주 들킴.
3. **헤드폰의 가치** — 거리 감쇠만으로 적의 위치를 추정. 시각 보조(미니맵, 화살표) 없이
   귀로만 판단 → 몰입감/긴장감 자동 발생.
4. **음성 메커니즘 = 자기장 대체** — 강해질수록 시끄러워지므로 후반에 자연스럽게 충돌
   빈도 증가. 인위적 자기장 없이도 라운드 페이싱이 잡힘.

이 4가지 중 하나라도 약화시키는 디자인은 거부한다 (자동 시전, 미니맵 적 표시, 텍스트
입력 백업 모드 등).

### 0.3 버전별 스코프 (현재 v1 데모 작업 중)

| 버전 | 스코프 | 목표 |
|------|--------|------|
| **v1 (데모)** | 4명 FFA, 5분 라운드, 슬라임 1종, 시작 마법 2종(화염/얼음), 강화 키워드 학습, HP 회복 없음 | 핵심 메커니즘이 *재미있는지* 검증 (2-3주) |
| v2 (확장) | 보스 + 자기장 + 다양 잡몹 + 시작 마법 4종 + 새 키워드 학습 + 맵 다수 | 콘텐츠 풍부화 |
| v3 (출시) | 계정/랭킹 + 욕설 필터 + 신고 + 통계 + Photon 유료 | 출시 준비 |

**작업 시작 전 항상 GAME_DESIGN.md §6.10 (데모에서 제외하는 것) 확인.**

### 0.4 기술 스택 한눈에

| 영역 | 사용 기술 | 위치 |
|------|----------|------|
| 멀티플레이 | Photon PUN 2 | `Assets/Photon/PhotonUnityNetworking/` |
| 음성 채팅 | Photon Voice 2 | `Assets/Photon/PhotonVoice/` |
| 음성 인식 (STT) | whisper.unity (whisper.cpp 래퍼) | Package, 모델 = `StreamingAssets/Whisper/` |
| 입력 | Unity New Input System | PlayerInput 컴포넌트 |
| UI | TextMesh Pro | 코드로 즉석 생성 |
| 엔진 | Unity 6 | `Rigidbody.linearVelocity` 사용 |

---

## 1. 프로젝트 개요

- **씬 흐름**: `IntroScene` (이름 입력 → PhotonNetwork.NickName 설정) → `MainScene` (게임플레이)
- **스폰**: `MainScene` 의 `GameManager` 가 `PhotonNetwork.Instantiate("Player", …)` 호출
- **스킬 종류 (현재 4개)**:
  - 화염구 (발사체, 25 데미지, 마나 20)
  - 치유 (즉시시전, +30 HP, 마나 30)
  - 방패 (지속형, 4초간 본인 보호, 마나 35)
  - 번개 (발사체, 40 데미지·고속, 마나 40)
- **HP/MP**: 100/100, MP 는 0.5초마다 +2.5 자동 회복

---

## 2. 폴더 구조

```
Assets/02. Script/
├── Camera/        — 3인칭 궤도 카메라
├── Common/        — Constants, GameManager, NetworkManager, ICharacterState, CharacterUtility
├── Player/        — PlayerController, PlayerHealth, PlayerMana, PlayerNameTag, SfxBroadcaster
│   ├── SMB/       — StateMachineBehaviour 들 (애니메이터 콜백)
│   └── State/     — PlayerState 베이스 + Idle/Move/Jump (FSM)
├── Spells/        — Fireball, HealSpell, ShieldSpell (모두 MonoBehaviourPun)
├── UI/            — IntroNameInput, PlayerHealthUI (코드로 즉석 생성)
└── Voice/         — VoiceSpellCaster, SpellDatabase, SpellEntry, SpellPhraseMatcher, HangulUtil
```

리소스 폴더 규칙:

```
Assets/Resources/
├── Player.prefab            — PhotonNetwork.Instantiate("Player", …)
├── Fireball.prefab          — Spells (PhotonNetwork.Instantiate)
├── Heal.prefab
├── Shield.prefab
├── Lightning.prefab
└── SpellDatabase.asset      — SpellDatabase.GetRuntime() 가 Resources.Load 로 로드
```

`Resources/`에 둔 이유: Photon `Instantiate` 의 prefab 이름 lookup, 그리고 SpellDatabase
런타임 로드(클라이언트 간 데이터 일관성).

---

## 3. 코드 스타일 컨벤션

### 3.1 Line ending: 저장소는 LF 통일, 워킹 디렉토리는 OS별

**Git 저장소 기준으로 모든 .cs 파일은 LF로 통일됨** (`.gitattributes` + `core.autocrlf=true`).
Windows 워킹 디렉토리에서는 일부 파일이 CRLF로 보일 수 있으나 이는 Windows 기본 동작이며,
git이 커밋/체크아웃 시 자동 변환하므로 사실상 통일된 상태와 동일하다.

확인 명령:
```bash
git ls-files --eol Assets/02.\ Script/Common/Constants.cs
# 결과 예: i/lf  w/crlf  attr/text eol=lf  →  index가 LF면 정규화 완료
```

**새 파일 작성 시**: IDE가 `.editorconfig` 의 `end_of_line = lf` 를 따르므로 자동으로 LF.
신경 쓸 필요 없음.

### 3.1.1 코드 *작성 스타일* 두 종 — 이건 별개

(line ending 과 무관) 본 프로젝트의 .cs 파일은 **작성 스타일 두 종이 공존**한다:

| 영역 | 작성 스타일 |
|------|------------|
| PlayerController, PlayerState*, NetworkManager, GameManager, Constants, ICharacterState, PlayerSmb*, CharacterUtility | 한글 인라인 주석 위주, XML doc 거의 없음 (프로젝트 원작자 스타일) |
| PlayerHealth, PlayerMana, PlayerNameTag, SfxBroadcaster, PlayerHealthUI, IntroNameInput, CameraController, Voice/*, Spells/* | `/// <summary>` XML doc + Header 그룹화 (후속 추가/리팩토링 스타일) |

**새 파일 작성 시**: 같은 폴더의 기존 파일 스타일을 따라가세요. 둘 중 어느 쪽이든
일관성이 더 중요합니다. 모르겠으면 LF + XML doc summary 를 기본으로 하세요.

### 3.2 공통 컨벤션 (양쪽 다 지켜짐)

#### 명명

- **클래스/메서드/Public property**: `PascalCase` (`PlayerHealth`, `CurrentHealth`, `TrySpend`)
- **private 필드**: `_camelCase` (underscore prefix) — `_animator`, `_states`, `_isRecording`
- **`[SerializeField] private`** 필드: underscore 없이 camelCase — `maxHealth`, `currentHealth`,
  `spawnForwardOffset` (인스펙터에 그대로 노출됨)
- **public field (소수만)**: `EPlayerState State;` 처럼 짧고 의도된 곳에만 사용
- **enum**: `E` prefix — `EPlayerState.Idle`, `EPlayerState.Move`
- **상수**: `Constants` 클래스의 `static readonly` 또는 `const` — `Gravity`, `PlayerAniParamMove`

#### 인스펙터 노출

```csharp
[Header("체력")]
[Tooltip("최대 HP. 시작 시 currentHealth 도 이 값으로 초기화.")]
[SerializeField] private int maxHealth = 100;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZeroDimen/Voice-Spell](https://github.com/ZeroDimen/Voice-Spell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
