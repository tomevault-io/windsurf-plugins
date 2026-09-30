---
trigger: always_on
description: Posteight 는 macOS 메뉴 막대 포스트잇 체크리스트 앱이다. 노트는 전부 로컬에 저장되고,
---

# Posteight 기여자 가이드

Posteight 는 macOS 메뉴 막대 포스트잇 체크리스트 앱이다. 노트는 전부 로컬에 저장되고,
계정도 네트워크도 텔레메트리도 없다.

제품 방향은 [docs/PRODUCT.md](docs/PRODUCT.md) 가 담지만 지금은 비어 있는 상태다. 다시 채워지기
전까지는 현재 동작 자체가 명세다. 제품 동작을 바꾸기 전에 물어본다.

## 스택

| | |
| --- | --- |
| 언어 | Swift 6 (`swift-tools-version: 6.0`) |
| UI | SwiftUI. SwiftUI 로 닿지 않는 곳은 AppKit (`NSWindow`, `NSViewRepresentable`) 으로 내려간다 |
| 테스트 | swift-testing (`import Testing`). XCTest 가 아니다 |
| 최소 버전 | macOS 14 (Sonoma) |
| 아키텍처 | Apple silicon 전용. `ARCHS` 가 `arm64` 로 고정되어 있다 |
| 의존성 | 없음 |

## 빌드와 실행

빌드 경로가 두 개다. 번들이 소유한 것(`Info.plist`, entitlements, 서명, 앱 아이콘)을 건드릴
때는 Xcode 를 쓰고, 빠른 컴파일 확인에는 SwiftPM 을 쓴다.

```bash
xed Posteight.xcodeproj
```

```bash
xcodebuild -project Posteight.xcodeproj -scheme Posteight -configuration Debug build
```

```bash
swift build
```

```bash
swift test
```

빌드 설정은 전부 `Posteight.xcodeproj` 안에 둔다. 실제 앱을 만드는 것은 이것뿐이고, 빌드 경로가
둘로 갈리면 설정이 어긋난다. `Sources/Posteight` 는 file system synchronized group 이라 Swift
파일을 추가하거나 지워도 `project.pbxproj` 를 고칠 필요가 없다.

`swift run` 도 되지만 번들 없는 바이너리가 뜬다. `Bundle.main.bundleIdentifier` 가 `nil` 이라
macOS 가 `linkd` / Process Instance Registry XPC 실패를 로그에 남기고, 창 위치도 따로 저장된다.
그 메시지는 잡음이지 앱 버그가 아니다. 번들이 중요한 작업이면 Xcode 타깃으로 실행한다.

## 진입점과 씬 구성

`@main struct PosteightApp: App` 이 [`Sources/Posteight/PosteightApp.swift`](Sources/Posteight/PosteightApp.swift)
에 있다. 메인 창은 없다. 씬 구성은 이렇다.

- **`MenuBarExtra`** (`.menuBarExtraStyle(.window)`) — 상태 아이템에 `MenuBarLabel`, 팝오버에
  `MenuBarPanelView`. 유일한 상설 표면이다.
- **`WindowGroup("Posteight", for: UUID.self)`** — 노트 ID 하나당 메모 창 하나. 타이틀 바는 숨긴다.
- **`Window(id:)`** — `WindowID.search` 와 `WindowID.trash`. `openWindow` 로 여는 단일 창이다.
- **설정은 씬이 아니다.** 앱에 시트를 붙일 창이 없어서 `SettingsModal.present` 가 자기
  `NSWindow` 를 띄운다. `NSApp.runModal` 은 일부러 피했다. 모달 세션이 `terminate` 를 삼켜서
  설정이 열려 있는 동안 Command-Q 가 통째로 먹통이 됐다.

`AppDelegate.applicationWillFinishLaunching` 은 SwiftUI 가 `MenuBarExtra` 를 설치하기 **전에**
activation policy 를 정한다. 나중에 바꾸면 씬이 다시 만들어지면서 한 프로세스에 상태 아이템이
둘 살아남을 수 있다.

### 창 라우팅

`WindowGroup` 은 같은 값으로 `openWindow` 를 불러도 매번 새 창을 만든다. 그래서 메모 창은
`NoteWindowCoordinator.shared` 를 거친다.

- `present(_:openWindow:)` — 몇 번을 불러도 결과가 같은 표시 경로. 살아 있는 창을 재사용하고,
  최소화된 창은 되돌리고, 화면 밖으로 밀려난 창에는 `moveOnScreenIfNeeded()` 를 부른다.
- `register(_:for:)` — 각 `StickyNoteWindowView` 가 뜰 때 자기 `NSWindow` 를 넘겨준다.
- `hideAll()` — 모든 메모 창을 내린다. 만들어지는 중이던 창도 숨김으로 기록해서, 숨기기 직후에
  생성이 끝난 창이 도로 튀어나오지 못하게 한다.

이걸 굴리는 쪽은 `MenuBarLabel` 이다. 실행할 때 모든 노트를 복원하고, 새로 생긴 노트를 열고,
`AppSettings.showAllNotesRequests`(Dock 아이콘 클릭이 올린다) 변화에 반응한다.

## 코드 구조

```
Sources/Posteight/
  PosteightApp.swift         @main, 씬, NoteWindowCoordinator, AppDelegate, MenuBarLabel
  PosteightStore.swift       @MainActor ObservableObject — 모든 변경, 저장, 마이그레이션,
                             백업·복원, 편집 이력, 탭 복사
  Models.swift               StickyNote, MemoTab, TodoItem, 휴지통 타입, PenStyle,
                             ColorOption, StickerOption, DesignTokens
  AppSettings.swift          설정 싱글턴(UserDefaults 기반), activation policy
  AppLock.swift              macOS 본인 인증, 잠금 설정과 현재 잠금 상태
  AppLockView.swift          잠금 화면, 시스템 인증 창 연결, 설정 UI
  Strings.swift              AppLanguage, L(), Lf(), englishStrings 테이블
  MenuBarPanelView.swift     팝오버 내용
  FoldedCardSurface.swift    종이 도형과 질감, 상태 아이템 글리프 MenuBarProgressCard
  StickyNoteWindowView.swift 메모 창 껍데기: 탭 바, NSWindow 설정, 프레임 저장과 복원
  StickyNoteView.swift       메모 본문
  TodoItemRow.swift          체크리스트 행과 펜 줄 긋기
  TrashView.swift, NoteSearchView.swift, SettingsView.swift, SettingsModal.swift,
  PencilCaseView.swift       보조 화면들
  PlainEditableTextField.swift, WindowMoveHandle.swift   NSViewRepresentable 브리지
  Color+Hex.swift
  Assets.xcassets/           앱 아이콘. SwiftPM 타깃에서는 제외된다

Tests/PosteightTests/        스토어 로직, 저장, 창 표시 상태, 메뉴 막대 글리프, 로컬라이제이션
Packaging/Info.plist         두 빌드 경로가 공유하는 번들 메타데이터
.github/workflows/           ci.yml, release.yml
docs/                        PRODUCT.md, README 이미지
```

`README.md` / `README.ko.md` 는 앱을 설치하는 사람을 위한 문서지 빌드하는 사람을 위한 문서가 아니다.

README 이미지를 다시 찍을 때는 **설정 → 노트** 의 "화면 공유와 스크린샷에서 노트 감추기" 를 먼저
끈다. 켜져 있으면 노트 창이 스크린샷에 아예 안 찍혀서 빈 바탕화면만 남는다.

## UI 규약

- 뷰는 얇게 유지한다. 테스트할 수 있는 것 — 개수, 정렬, 마이그레이션, Markdown — 은
  `PosteightStore` 나 모델 타입에 순수 함수나 `nonisolated` 코드로 둔다.
- 사용자에게 보이는 문자열은 전부 `L()` / `Lf()` 를 거친다. [언어](#언어) 를 본다.
- 메모 표면은 `FoldedCardSurface`(`MemoCardShape`, `MemoTabShape`, `MemoCardSurface`,
  `PaperGrain`) 에서 가져온다. 뷰 안에서 종이 도형이나 색을 다시 만들지 않는다.
- 크기, 프레임 한계, 종이·펜 팔레트, 스티커는 `Models.swift` 의 `DesignTokens` 에 있다.
  `minimumNoteSize` 는 건드리면 파장이 크다. 값을 올리면 저장된 모든 노트가 `clamped` 를 지나며
  넓어져서, 사용자가 직접 잡아 둔 배치를 덮어쓴다.
- 상태 아이템 글리프는 단일 template `NSImage` 로 래스터화한다(`MenuBarProgressCard`). 메뉴 막대
  렌더러가 여러 뷰로 조립한 그림의 일부를 떨어뜨리기 때문이다. `MenuBarGlyphTests` 가 지킨다.
- AppKit 은 `NSViewRepresentable` 을 통해 만진다. SwiftUI 의 리치 텍스트 동작 없이 순수 텍스트를
  편집하려면 `PlainEditableTextField`, 타이틀 바 없는 창을 끌려면 `WindowMoveHandle`. 뷰 여기저기에
  `NSWindow` 접근을 흩뿌리지 않는다.

## 데이터와 환경

- 노트는 샌드박스 컨테이너의 Application Support 아래 `Posteight/` 에 `notes.json`,
  `trash.json`, `trashed-tabs.json` 으로 저장된다(`PosteightStore.storeDirectory`). 임포트한
  폰트와 그 매니페스트는 같은 자리의 `Fonts/` 다. 파일은 `0600`, 디렉터리는 `0700` 으로 만든다.
  테스트는 자기 `directory` 를 넘기기 때문에 실제 노트를 건드리지 않는다.
- 샌드박스 이전 설치는 `~/Library/Application Support/Posteight/` 에 저장했다.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hmjlon/posteight](https://github.com/hmjlon/posteight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
