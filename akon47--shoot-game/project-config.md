---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

HTML5 Canvas 기반 멀티플레이 슈팅 웹 게임의 **클라이언트** 프로젝트.
서버는 별도 저장소인 `C:\Users\hwank\source\repos\shoot_game_server` (Node.js, `ws` 기반)에 있다.
데모: https://shootgame.kimhwan.kr/

- 빌드 시스템/패키지 매니저/테스트/린트 **없음**. 순수 정적 파일(Vanilla JS ES6 클래스)이다.
- 외부 의존성은 CDN으로 로드: jQuery 1.12.4, SoundJS 0.6.2 (index.html 참고).
- 모듈 시스템 없음 — 모든 클래스/상수/전역 변수가 글로벌 스코프에 존재한다.

## 실행 방법

정적 파일 서버로 루트를 서빙한 뒤 브라우저에서 index.html을 연다. 예:

```
python -m http.server 8000
# 또는
npx serve .
```

참고: index.html의 `framework.js?<?PHP echo time(); ?>`는 PHP 호스팅용 캐시버스팅 코드라
정적 서버에서는 쿼리스트링이 리터럴로 전달될 뿐 동작에는 지장 없다.

서버 실행 (별도 저장소): `cd ..\shoot_game_server; npm install; node server.js`
클라이언트의 접속 주소는 `network_class.js` 상단의 `wsUri`가 자동 선택한다:
`localhost`/`127.0.0.1`에서 서빙하면 `ws://localhost:8080`(로컬 서버), 그 외에는
`wss://ws.shootgame.kimhwan.kr`(운영 — nginx가 443 TLS 종료 후 Node 8080으로 프록시)로 접속한다.

## 부팅/스크립트 로딩 구조

1. `index.html` → `framework.js` 로드
2. `framework.js` 상단의 `requireScripts` 배열이 나머지 모든 `*_class.js` 파일을 `<head>`에 동적 삽입한다.
   **새 JS 파일을 추가하면 반드시 이 배열에도 등록해야 한다.**
3. `window.onload`에서 게임 캔버스 + UI 캔버스(겹쳐진 2장)를 생성하고
   `systemClass = new SystemClass(...)` → `systemClass.run()`으로 게임 루프 시작.

게임 루프(`system_class.js`의 `run()`): requestAnimationFrame 기반으로 매 프레임
`inputFrame()` → `playersFrame()` → `soundClass.frame()` → `graphicsClass.frame()` → `debugClass.frame()` 순서로 호출.

## 아키텍처

`SystemClass`(system_class.js)가 중앙 오케스트레이터다. 모든 하위 클래스를 생성·보유하고,
입력/네트워크/플레이어 이벤트를 생성자 안에서 전부 배선(wiring)한다.

```
SystemClass
 ├─ InputClass      키보드/마우스 상태 (키코드 상수도 input_class.js에 정의)
 ├─ NetworkClass    WebSocket 통신 (JSON {type, data} 프로토콜)
 ├─ ChatClass       채팅 입력/메시지 표시 (DOM 기반)
 ├─ SoundClass      SoundJS 래퍼, 위치 기반 볼륨/팬 계산
 ├─ ObjectClass     맵 위 오브젝트(벽 등)
 ├─ ItemManagerClass  서버 권위 아이템(메드킷/탄약) 보관·렌더링 (item_class.js)
 ├─ MonsterManagerClass  서버 권위 PvE 몬스터(스켈레톤) 보관·보간·렌더링 (monster_class.js)
 ├─ GraphicsClass   렌더링 총괄
 │   ├─ CameraClass            뷰박스/회전/카메라 제한
 │   ├─ MapClass(map_office_data)  타일맵 렌더링 + 히트박스/세그먼트 생성
 │   ├─ SightEffectClass       시야(라이팅) 효과 — lighting_class.js에 정의
 │   ├─ UserInterfaceClass     HUD (ui_class.js: InfoHUD, UserHUD, KillHUD, MinimapInterfaceClass)
 │   ├─ ParticleClass, WeatherClass(비활성), SurvivorCharacterClass(스프라이트)
 └─ players[]       id를 키로 쓰는 PlayerClass 맵 (배열을 맵처럼 사용)
```

`sprite_class.js`의 `NpcCharacterClass`는 PvE 몬스터(스켈레톤) 스프라이트 렌더러다
(`images/Monster/skeleton` idle/move/meleeattack 애니메이션). `monster_class.js`의
`MonsterManagerClass`가 이 렌더러를 1개 공유하며, 엔티티 `NpcClass`(서버 30fps 위치를 보간)를
넘겨 그린다. 좀비 효과음은 `sound/monster`(SoundClass `playMonsterSound`).

전역 인스턴스: `systemClass`(framework.js), `debugClass`(debug_class.js) — 여러 파일에서 직접 참조한다.

## 구현 패턴 (코드 수정 시 따를 것)

- **포맷팅**: Prettier 기본 설정(2-space)으로 통일되어 있다. 맵 데이터 파일
  (`map_office.js`, `map_data1.js`)은 포맷 대상에서 제외한다.
- **파일/클래스 명명**: 파일은 `snake_case_class.js`, 클래스는 `PascalCase + Class` 접미사
  (예: `player_class.js` → `PlayerClass`). 한 파일에 보조 클래스가 같이 있을 수 있다.
- **콜백 할당 패턴**: 이벤트는 EventEmitter가 아니라 인스턴스 프로퍼티에 함수를 직접 할당하는 방식.
  예: `this.networkClass.userconnected = function (...) {...}`. 호출하는 쪽은
  `if (this.userconnected) this.userconnected(...)`로 null 체크 후 호출. 배선은 대부분
  `SystemClass` 생성자에서 이루어진다.
- **`var self = this` 클로저** 패턴을 화살표 함수 대신 사용한다.
- **getter/setter 메서드**: 프로퍼티 직접 접근 대신 `getName()/setName()` 류 메서드 사용.
  setter는 값 변경 시 해당 `*changed` 콜백을 호출한다 (→ 본인 플레이어면 네트워크 전송으로 이어짐).
- **프레임 시간 보정**: `performance.now()` 기반 `timeRatio`(60fps 기준 배율)를 곱해 이동량 계산
  (player_class.js `frame()` 참고).
- **리소스 로딩**: 이미지/사운드는 비동기 로드 후 `isLoaded()` 플래그 확인.
  GraphicsClass.frame()이 모든 리소스 로드 전엔 "LOADING" 화면을 그린다.
- **Canvas 확장 메서드**: `extension_method.js`에서 `CanvasRenderingContext2D.prototype`에
  `roundedRect`, `drawText`(그림자 텍스트)를 추가해 사용한다.

## 네트워크 프로토콜 (서버와 동기화 필수)

- 메시지 형식: `JSON.stringify({ type, data })`. 수신은 `network_class.js`의 `onMessage` switch에서 분기.
- 주요 타입: `id`(접속 시 ID 할당), `user_init`, `user_connected/disconnected/count`,
  `user_position/speed/direction/weapon/character/name`, `user_shoot/melee_attack/reload`,
  `user_hp/die/kill/death`, `user_chat/chat_history`, `echo`(레이턴시 측정, 1초 주기),
  `item_list/item_spawn/item_picked/ammo_refill`(아이템), `round_info`(라운드 남은 시간),
  `monster_list/monster_spawn/monster_positions/monster_hp/monster_attack/monster_die`(PvE 몬스터).
- `user_shoot`의 `data.targetPoints`는 **배열**이다. 샷건은 펠릿 7개(`SHOTGUN_PELLET_COUNT`,
  클라·서버 동일 값 유지), 그 외 무기는 1개. 원격 사격 재현 시 `PlayerClass.shoot()`의
  3번째 인자(`presetTargets`)로 그대로 전달한다.
- `user_connected`의 `data.protectedMs`는 남은 스폰 무적 시간(ms). 클라이언트는
  `PlayerClass.setSpawnProtection()`으로 반영하고, 무적 시간 동안 피격 시각 효과를 생략한다.
  본인 무적은 `sendUserInit` 직후 `SPAWN_PROTECTION_DURATION`(system_class.js 상단, 서버와 동일 값)으로 설정.
- 아이템(메드킷/탄약 상자)은 전부 서버 권위: 클라이언트(`item_class.js`의 `ItemManagerClass`)는
  보관·렌더링만 하고 획득 판정은 서버가 한다. 탄약 보급은 `ammo_refill` 수신 시 `resetAmmo()`.
- PvE 몬스터(스켈레톤)도 전부 서버 권위: `monster_class.js`의 `MonsterManagerClass`가 보관·렌더링만 한다.
  서버는 위치를 30fps `monster_positions`(묶음)로 보내고, 엔티티 `NpcClass`가 렌더 프레임마다 보간한다
  (큰 점프는 스냅). `monster_attack` 수신 시 공격 애니메이션 재생 + 좀비 효과음, `monster_die` 시 제거.
  몬스터는 다른 플레이어와 동일하게 **시야 클립 영역 안에서만** 그려진다(graphics_class.js).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akon47/shoot_game](https://github.com/akon47/shoot_game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
