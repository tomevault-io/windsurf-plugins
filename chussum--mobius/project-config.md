---
trigger: always_on
description: Claude Code CLI + Claude Desktop 계정을 전환/자동 fallback 하는 macOS 메뉴바 앱 + `mobius` CLI.
---

# Mobius (뫼비우스) — Claude 계정 매니저

Claude Code CLI + Claude Desktop 계정을 전환/자동 fallback 하는 macOS 메뉴바 앱 + `mobius` CLI.
Swift Package (SwiftUI, macOS 14+). primary 소진 → fallback 자동 전환 → primary 회복 시 자동 복귀.

> **이 파일은 항상 최신 상태로 유지한다.** 구조·핵심 사실·실패 기록이 바뀌면 같은 커밋에서 갱신할 것.

## 빌드 / 실행

```bash
swift test                    # 유닛 테스트 (MobiusCore)
swift build                   # 컴파일 확인
Scripts/make-app.sh           # dist/Mobius.app 번들 조립 + 서명
Scripts/make-dmg.sh           # dist/Mobius-<ver>.dmg 배포 이미지 (드래그 설치)
open dist/Mobius.app          # 실행 (메뉴바 ∞ 아이콘)
Scripts/setup-signing.sh      # (1회) 고정 서명 인증서 생성 — 아래 '서명' 참조
```

## 구조

```
Sources/MobiusCore/       앱·CLI 공유 코어 (전부 의존성 주입 → 테스트 가능)
  MobiusEnvironment.swift  모든 경로 컨테이너 (MOBIUS_HOME 오버라이드)
  Models.swift             AccountProfile / AccountsFile / CredentialsSnapshot / RateLimitInfo
  KeychainClient.swift     SystemKeychain + InMemoryKeychain(테스트)
  ClaudeConfigIO.swift     Claude 자격증명 읽기/쓰기 (★ 아래 '진실의 원천' 필독)
  AccountStore.swift       프로필 영속(accounts.json) + 비밀 스냅샷(0600 파일)
  Switcher.swift           전환/되저장/롤백/reconcile/adopt (★ liveIsStable 게이팅)
  RateLimitParser.swift    세션 로그 rate-limit 이벤트 파서 (실측 기반)
  SessionLogWatcher.swift  ~/.claude/projects tail (네트워크 0)
  AutoSwitchEngine.swift   순수 상태머신 (쿨다운/마진/autoSwitchedFromPrimary)
  UsageFetcher.swift       usage 엔드포인트 조회 (게이지용, 팝오버 열 때만)
Sources/mobius/           CLI (list/switch/status/capture/auto)
Sources/MobiusApp/        SwiftUI 메뉴바 앱 + AppState + Views/ + LoginFlow + DesktopCoordinator
```

## 핵심 사실 (실측으로 확인 — 추측 금지)

### ★ 진실의 원천: 자격증명 토큰은 Keychain, 이메일은 ~/.claude.json
- **토큰**: Keychain `Claude Code-credentials` 가 진실. 이 환경의 Claude Code는
  최신 토큰을 Keychain에만 쓰고 `~/.claude/.credentials.json` **파일은 갱신하지 않는다(낡음)**.
  → `readLiveSnapshot()`은 **반드시 Keychain 우선**. 파일은 Keychain이 빈 경우의 폴백일 뿐.
- **이메일/계정 메타**: `~/.claude.json` 의 `oauthAccount.emailAddress`. 자격증명 blob에는 계정
  식별자가 **없다** (accessToken/refreshToken/expiresAt/subscriptionType 뿐).
- **전환 = 3곳 스왑**: Keychain + .credentials.json + ~/.claude.json 의 oauthAccount.

### 사용량 엔드포인트
- `GET https://api.anthropic.com/api/oauth/usage`, 헤더 `Authorization: Bearer <token>` +
  `anthropic-beta: oauth-2025-04-20`. 응답: `five_hour.{utilization, resets_at}`,
  `seven_day.{...}` (utilization=백분율, resets_at=ISO8601 마이크로초).
- 게이지는 **팝오버 열 때만** 조회(캐시 4분). 상시 폴링 없음 → 계정 리스크 최소화.

### macOS 26 (Tahoe) 환경
- 메뉴바 아이콘은 Control Center가 호스팅 — CGWindowList의 layer/owner로 존재 확인이 어려움.
- **Bartender 같은 메뉴바 관리 앱이 새 앱 아이콘을 자동 숨김** → 안 보이면 Bartender 설정에서 표시.
- 서명 안 된/ad-hoc 앱도 실행되지만 Keychain ACL이 서명 정체성에 묶임.

### 서명 (Keychain 승인창 영구 방지)
- ad-hoc 서명(`-s -`)은 **리빌드마다 정체성이 바뀌어** "항상 허용"이 매번 리셋됨.
- `Scripts/setup-signing.sh`로 고정 인증서 `Mobius Dev Signing` 생성 → make-app.sh가 자동 사용.
- 고정 서명 + 아래 '비밀은 파일' 조합으로 승인창이 사실상 사라짐.

### Desktop 내장 Claude Code가 `security` CLI로 CLI 자격증명을 읽는다 (파티션 리스트)
- 최근 Claude Desktop은 Claude Code를 내장(`claude-code`, `cowork-enabled-cli-ops.json`)하며,
  **Desktop 실행 시 `/usr/bin/security`로 Keychain `Claude Code-credentials`를 읽는다.**
- 이 항목의 **파티션 리스트에 `apple-tool:`이 없으면** security 접근마다 **키체인 암호를
  요구하는 창**이 뜨고, 이 유형은 **'항상 허용'을 눌러도 절대 저장되지 않는다**
  (파티션 검사는 ACL과 별개). Desktop을 재실행할 때마다 2회씩 반복 (2026-07-11 실측).
- 1회 해결: `security set-generic-password-partition-list -S "apple-tool:,apple:"
  -s "Claude Code-credentials" -a $USER` (로그인 키체인 암호 필요. "(deprecated)" 문구는
  대화형 암호 입력 방식에 대한 경고일 뿐 — 이 명령이 파티션 수정의 유일한 수단).
- 주의: CLI 재로그인 등으로 항목이 **재생성되면 파티션이 리셋**되어 재적용 필요.
- ★ 더 치명적: **비-Apple 앱이 SecItemUpdate로 항목을 수정하면 macOS가 파티션 리스트를
  그 앱의 cdhash로 도장 찍는다(re-stamp).** Mobius가 네이티브 API로 토큰을 쓰면 전환할
  때마다 파티션이 `cdhash:MobiusApp`으로 리셋 → security 경유 읽기(CLI·Desktop)가 전부
  암호창 유발. → `SystemKeychain`은 **읽기·쓰기 모두 security CLI 경유**다
  (쓰기는 -i stdin으로 비밀 전달, 읽기는 -w stdout 파싱·exit 44=없음). 이러면 도장이
  `apple-tool:`로 찍혀 유지되고, 파티션 밖인 Mobius 자신도 창 없이 접근한다 (실패 기록 12).
- 파티션 리스트 실제 값 확인은 SecAccessCopyACLList의 `ACLAuthorizationPartitionID`
  ACL desc(hex plist)를 디코드하면 승인창 없이 볼 수 있다.

### Claude Desktop은 Squirrel(ShipIt) 자동업데이트 — 앱 종료 순간 번들 통째 교체
- 업데이트가 스테이징되어 있으면 **Desktop이 종료되는 순간** ShipIt이
  `/Applications/Claude.app`을 temp로 이동시키고 새 번들로 교체한다
  (`~/Library/Caches/com.anthropic.claudefordesktop.ShipIt/ShipIt_stderr.log`).
- 그래서 Desktop을 종료→재실행할 때는 반드시 ShipIt이 끝나길 기다려야 한다 —
  `DesktopCoordinator.launch()`의 `waitForUpdaterQuiescence()`가 담당 (실패 기록 10 참조).

### 비밀 스냅샷은 Keychain이 아니라 0600 파일
- 계정별 스냅샷은 `~/Library/Application Support/Mobius/secrets/<uuid>.json` (0600).
- Claude Code 자신도 토큰을 파일(.credentials.json 0600)에 두므로 동일 보안 수준이고,
  Keychain에 두면 계정 수 × 접근마다 승인창이 떠서 UX가 망가진다.
- 구버전 Keychain 항목(`Mobius-account-*`)은 `secret()`에서 발견 시 파일로 자동 이관 후 삭제.

## 실패 기록 (같은 실수 반복 금지)

1. **파일 우선 읽기로 바꿔 자격증명 오염** — "Keychain 승인창을 줄이자"고 `readLiveSnapshot()`을
   .credentials.json 파일 우선으로 바꿨더니, **낡은 파일 토큰(fore.st) + 최신 이메일(flosdor)**이
   짝지어져 flosdor 프로필에 fore.st 토큰이 저장됨. 사용자 라이브 로그인까지 오염됨.
   → 교훈: **토큰의 진실은 Keychain**. 파일은 낡을 수 있다. 승인창은 '고정 서명 + 비밀 파일화 +
   변화 시에만 Keychain 접근'으로 줄이고, 라이브 토큰 읽기는 Keychain을 포기하지 말 것.
2. **비원자 갱신 레이스** — 로그인/전환 중 토큰(Keychain)과 이메일(~/.claude.json)이 서로 다른
   시점에 갱신되는 찰나에 읽으면 짝이 안 맞음. → `ClaudeConfigIO.liveIsStable()`로 최근 2초 내
   수정 시 저장 계열 연산(resave/adopt/reconcile) 스킵. Switcher.stabilityWindow(테스트는 0).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chussum/mobius](https://github.com/chussum/mobius) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
