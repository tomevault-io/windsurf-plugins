---
trigger: always_on
description: <section id="project-info">
---

<section id="project-info">

# Noctiluca (Monorepo)

Noctiluca는 macOS 호스트 기반 원격 제어 솔루션이며, 이 레포는 서버/클라이언트 앱과
핵심 프로토콜 라이브러리를 함께 관리하는 **monorepo**입니다.

# BUILD COMMANDS

## Workspace 사용 (권장)
빌드 시에는 반드시 `NoctilucaServer.xcworkspace`를 사용합니다.

```bash
# 서버 빌드 (macOS)
xcodebuild -workspace NoctilucaServer.xcworkspace -scheme NoctilucaServer -configuration Debug build

# 클라이언트 빌드 (macOS)
xcodebuild -workspace NoctilucaServer.xcworkspace -scheme NoctilucaClient -configuration Debug build

# 클라이언트 빌드 (iOS Simulator)
xcodebuild -workspace NoctilucaServer.xcworkspace -scheme NoctilucaClient -configuration Debug -destination 'platform=iOS Simulator,name=iPhone 16' build
```

# TEST COMMANDS

```bash
# 서버 테스트
xcodebuild -workspace NoctilucaServer.xcworkspace -scheme NoctilucaServerTests -configuration Debug test

# 특정 테스트 클래스 실행
xcodebuild -workspace NoctilucaServer.xcworkspace -scheme NoctilucaServerTests -only-testing:NoctilucaServerTests/AutoQualityPlannerTests test
```

# LINTING

SwiftLint가 `.swiftlint.yml`로 구성되어 있습니다.
```bash
swiftlint lint --config .swiftlint.yml
```

# REPOSITORY LAYOUT (TOP-LEVEL)

- `SiriusKit/` - Sirius 프로토콜/채널/트랜스포트 코어 라이브러리 (server/client 공용, SwiftPM)
- `NoctilucaServer/` - macOS 호스트 앱 (세션 수락, 인증, 입력 인젝션, 화면 전송)
- `NoctilucaClient/` - macOS/iOS 클라이언트 앱 (연결/인증, 입력 전송, 화면 수신/디코딩)
- `NoctilucaPluginKit/` - 플러그인 번들 계약/메타데이터 스펙
- `SamplePluginBundle/` - 샘플 플러그인 번들
- `Gesu/` - Private API 호출용 Swift 매크로 라이브러리 (`@PrivateLibrary`, `#PrivateFunction`)
- `frameworks/` - 외부 xcframework 의존성 (VPX, VPXDecoder)
- `libbcrypt/` - bcrypt 라이브러리 (PAM 인증용)
- `NoctilucaServerTests/` - 서버 테스트
- `docs/`, `distutil/`, `pam.d/` 등 유틸리티
- `NoctilucaClientQt/` - Linux / Windows용 C++/Qt 클라이언트 및 libsirius (SiriusKit의 client-role only C++ 구현체)

# TECHNOLOGIES USED (CROSS-CUTTING)

- Swift / SwiftUI / Swift Concurrency
- Network.framework (QUIC)
- SwiftProtobuf 3 (Sirius msgdef)
- ScreenCaptureKit + AVFoundation (AVCaptureSession, AVAudioEngine)
- VideoToolbox (H.264/H.265 encode/decode), libvpx (VP8 encode/decode)
- AudioToolbox / AVAudioConverter (Opus/G.711 encode/decode)
- CoreGraphics / CoreMedia
- Security.framework / Keychain
- GameController (클라이언트 입력 디바이스)
- OSLog/콘솔 로깅
- Swift Macros (Gesu - Private API 호출)

# ARCHITECTURE OVERVIEW (CROSS-MODULE)

## 1) Sirius Protocol / Session
- 세션/채널/메시지/트랜스포트 규격은 SiriusKit이 정의합니다.
- MainChannel에서 handshake + 인증을 처리하고, 인증 완료 후 기능 채널을 엽니다.
- 기능 채널은 UUID 기반 feature로 식별됩니다 (예: HIDIO, Projection).

## 2) Transport (QUIC)
- QUIC는 Network.framework 기반 구현이며, ALPN은 `pl.unstabler.sirius`를 사용합니다.
- 기본 포트는 8282입니다 (`SiriusQUICDefaultPort`).

## 3) Projection (Video/Audio)
- **비디오**:
  - 서버: ScreenCaptureKit/AVCaptureSession 캡처 → VideoToolbox(H.264/H.265) / libvpx(VP8) 인코딩 → ProjectionDataChannel 전송
  - 클라이언트: ProjectionDataChannel 수신 → VTDecompressionSession / libvpx 디코딩 → MetalVideoRenderer (또는 AVSampleBufferDisplayLayer fallback) 렌더링
  - 지원 코덱: H.264, H.265(HEVC), VP8 (0.9.10 부터. MJPG/ZRLE/WebP 는 0.9.10 에서 제거)
- **오디오**:
  - 서버: ScreenCaptureKit 오디오 캡처 → Opus/G.711 인코딩 → ProjectionDataChannel 전송
  - 클라이언트: ProjectionDataChannel 수신 → AudioDecoder 디코딩 → AVAudioEngine 재생
  - 지원 코덱: Opus, G.711 mu-law/A-law
- 코덱 협상은 Sirius msgdef 기반 옵션을 사용합니다.

## 4) Input (HIDIO)
- 클라이언트에서 HIDIO 채널로 키보드/마우스 이벤트를 전송합니다.
- 서버는 HID 이벤트를 호스트 시스템에 인젝션합니다.

## 5) Auth / Plugin
- 서버는 인증을 플러그인 번들로 확장할 수 있습니다. 기본 인증 번들이 포함되어 있습니다.
- 플러그인 메타데이터는 NoctilucaPluginKit 스펙을 따릅니다.

## 6) AppStream (Experimental)
Microsoft RDP의 RemoteApp에서 영감을 받은 기능으로, 원격 Mac의 개별 앱 윈도우를 로컬 앱처럼 사용할 수 있게 합니다.
이 기능은 영구적으로 **experimental** 상태입니다.

- **프로토콜**: Projection 채널의 `0x80xx` opcode 범위에서 Application Management(appman) 메시지를 정의
  - 앱 목록 조회, 앱 실행/종료, 앱 이벤트 구독, AppStream 시작/종료, 윈도우 이벤트 등
  - 프로토콜 정의: `SiriusProtocol/v1/channels/projection/appman.mdproto.md`
- **일반 Projection과의 차이**: 전체 디스플레이 대신 개별 윈도우 단위로 `ProjectionSession`을 생성하며, 클라이언트는 원격 윈도우마다 네이티브 `NSWindow`를 생성
- **서버**: `ProjectionChannel+appman.swift`에서 요청 처리, `DesktopContextManager`로 윈도우/앱 이벤트 감시, `allowedApps` 보안 정책 적용
- **클라이언트 (macOS only)**: `AppStreamWindowManager`가 윈도우 생성/파괴/업데이트 관리, `AppStreamWindow`(NSWindow 서브클래스)가 각 원격 윈도우를 렌더링
- **Qt 클라이언트**: msgdef 바인딩만 존재, AppStream UI/로직 미구현
- **현재 구현 상태**: 윈도우 스트리밍 기본 동작 구현 완료. 앱 선택 UI, `disableSystemShortcuts` 실제 적용 등 미완성 부분 존재
- **향후 계획**: File System Redirection 등 추가 기능 구현 예정

# COORDINATION & SOURCE OF TRUTH

- 프로토콜/메시지/코덱 옵션과 같은 공용 규격은 SiriusKit이 기준입니다.
- 서버/클라이언트 동시 변경이 필요한 경우, 두 앱의 흐름(핸드셰이크/채널)을 함께 확인하세요.
- 세부 구조는 각 서브프로젝트의 `AGENTS.md`를 우선 참조합니다:
  - `SiriusKit/AGENTS.md` - 프로토콜/채널/트랜스포트 상세
  - `NoctilucaServer/NoctilucaServer/AGENTS.md` - 서버 앱 상세
  - `NoctilucaClient/AGENTS.md` - 클라이언트 앱 상세
  - `NoctilucaPluginKit/AGENTS.md` - 플러그인 계약 상세

## Recent Notes

- **fsaccess byte-range lock (SiriusProtocol 5894e33)** (2026-05-06):
  - mdproto: `wouldBlock=82` 에러코드, `FileSystemMountResponse.supportsLocks`
    capability flag, `LockType` constset (shared/exclusive), 6개 새 메시지
    (`FileSystemLock(Request|Response)` / `Unlock` / `TestLock`, opcode
    0x80C1~0x80C6).
  - SiriusKit msgdef 동기화 완료 (autogen 2 + channel/msgdef 5 파일).
  - exposing peer (NoctilucaClient): macOS 는 `fcntl(F_SETLK)` / `F_GETLK` 로
    OS-level byte-range lock, iOS 는 supportsLocks=false 광고 + 모든 lock op 에
    `notSupported` 응답.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-unstablers/NoctilucaServer](https://github.com/team-unstablers/NoctilucaServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
