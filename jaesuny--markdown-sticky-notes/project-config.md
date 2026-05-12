---
trigger: always_on
description: ./build-app.sh                # Full build: webpack → Swift → .app bundle
---

# CLAUDE.md

## Build & Run

```bash
./build-app.sh                # Full build: webpack → Swift → .app bundle
open build/StickyNotes.app    # Run
```

Prerequisites: macOS 12+, Swift 5.9+, Node.js 18+. First time: `cd editor-web && npm install`

### Dev Workflow

```bash
# Full rebuild (Swift + JS)
./build-app.sh && open build/StickyNotes.app

# JS-only fast iteration (requires .app already built)
cd editor-web && npm run build && cp dist/editor.bundle.js ../build/StickyNotes.app/Contents/Resources/Editor/ && open ../build/StickyNotes.app
```

## Architecture

Hybrid native + web macOS app:
- **Swift/SwiftUI**: App shell, NSPanel windows, UserDefaults persistence
- **Single shared WKWebView + CodeMirror 6**: Markdown editor with KaTeX math — one WKWebView reparented to the active (key) window, inactive windows show snapshots
- **Bridge**: `WKScriptMessageHandler` bidirectional messaging

### Editor Rendering (editor-web/src/editor.js)

Three-layer system:

1. **ViewPlugin** (`markdownDecoPlugin`) — syntax tree 기반
   - `syntaxTree(state).iterate()` 로 Lezer 파서 노드 순회
   - `Decoration.line()` → 헤딩 font-size, 코드블록 배경, 블록쿼트 스타일, HR
   - `Decoration.mark()` → 볼드, 이탤릭, 링크, 마커 흐리게
   - `Decoration.replace()` → 인라인 코드 위젯 (커서 unfold 지원)

2. **StateField** (`mathRenderField`) — 수식 전용
   - 블록 수식 `$$...$$`: 멀티라인 `Decoration.replace({ block: true })` — ViewPlugin에서는 불가
   - 인라인 수식 `$...$`: regex 매칭 (Lezer는 `$` 미인식)
   - `collectCodeRanges()` 로 코드블록 내부 `$` 무시
   - 선택 변경 시에도 재계산 (`tr.selection` 체크)

3. **HighlightStyle** — 보조 토큰 색상 (파싱 완료 전 기본 스타일)

### Cursor Unfold Pattern

`Decoration.replace()` 사용 시 커서가 범위 안에 있으면 위젯 대신 원본 소스 표시:
```javascript
const { from: curFrom, to: curTo } = state.selection.main;
function cursorInside(from, to) { return curFrom >= from && curTo <= to; }
if (cursorInside(node.from, node.to)) break; // skip replace, show raw
```
적용 대상: InlineCode, 인라인/블록 수식, HR, TaskMarker

### Single WKWebView Architecture

```
Before: Window1[WKWebView] Window2[WKWebView] Window3[WKWebView]  (~125MB for 5 notes)
After:  Window1[Snapshot]  Window2[WKWebView]  Window3[Snapshot]   (~30MB for 5 notes)
                           (active/key window)
```

- **SharedWebViewManager** (singleton): 단일 WKWebView 소유, EditorState 직렬화/복원으로 노트 전환
- **NoteWindowController**: `windowDidBecomeKey` → blur→snapshot→move WebView chain
- **editor.js**: `serializeState()`/`restoreState(json)` API, `noteId` 기반 bridge 메시지

**Pre-rendering**: `markReady()`에서 WebView alpha=0 상태로 비활성 노트 순차 렌더링+스냅샷. `suppressContentChange`로 contentChanged 디바운스 억제 필수 (없으면 active note 콘텐츠 덮어쓰기).

Window switch flow (synchronous): `takeSnapshot()` via RunLoop spin (WebView stays in old window, max 100ms) → show snapshot on old window → move WebView to new window → `switchToNote()` → reveal after 150ms + `focusEditor()`

### Key Files

```
editor-web/src/editor.js          # 에디터 전체 (ViewPlugin + StateField + theme)
editor-web/webpack.config.cjs     # 단일 번들, 폰트 base64 인라인
Sources/StickyNotes/App/SharedWebViewManager.swift       # 단일 WKWebView 소유, 노트 전환 (stateCache)
Sources/StickyNotes/Bridge/SharedEditorBridge.swift      # noteId 기반 Swift-JS 메시지 라우팅
Sources/StickyNotes/Views/NoteWindow/NoteWindowController.swift  # WebView reparent, snapshot/preview
Sources/StickyNotes/App/AppCoordinator.swift             # 앱 조율 (Combine)
```

### Swift-JS Bridge

- **JS → Swift**: `sendToBridge(action, data)` → `ready`, `contentChanged`, `requestSave`, `log`, `error` — 모든 메시지에 `noteId` 포함
- **Swift → JS**: `window.setContent(content)`, `window.getContent()`, `window.openSearch()`, `window.serializeState()`, `window.restoreState(json)`, `window.setCurrentNoteId(id)`, `window.prepareForSnapshot()` (snapshotMode + transitions 끄고 커서→0 + blur), `window.endSnapshotMode()` (snapshotMode 해제 + transitions 복원)
- Console.log 인터셉트: `WKUserScript` at document start → Swift 콘솔로 전달

## Implementation Status

- **Phase 1 ✅**: App structure, NSPanel windows, persistence, WKWebView
- **Phase 2 ✅**: CodeMirror 6, syntax tree decorations, KaTeX math, cursor unfold
- **Phase 3 ✅**: 추가 마크다운 요소 (취소선, 리스트 스타일, 체크박스 위젯, 테이블)
- **Phase 4 ✅**: Titlebar 컨트롤 (핀/투명도/색깔), Always-on-top, Cmd+F/Cmd+Shift+F 검색
- **Phase 5 ✅**: Single shared WKWebView 아키텍처 (메모리 ~80% 절감)
- **Phase 6 (진행중)**: 디자인 개선, 버그 수정, Known Issues 해결

## Gotchas

1. **`Decoration.line()` CSS**: 클래스가 `.cm-line` 요소에 직접 추가됨 — `&light .cm-heading-1` 같은 스코프 접두사 사용하면 안 됨, `.cm-heading-1`로 직접 사용
2. **ViewPlugin vs StateField**: 줄바꿈 포함 `Decoration.replace()`는 반드시 StateField. ViewPlugin에서 하면 "line break" 에러
3. **`atomicRanges` 주의**: 모든 decoration에 적용하면 커서가 여러 블록을 건너뜀 — 블록 수식 네비게이션은 커스텀 키맵(`blockMathNavKeymap`)으로 처리
4. **regex vs syntax tree**: 인라인 코드 regex는 펜스드 코드블록 내부와 충돌 — `syntaxTree().iterate()`의 노드명(`InlineCode`, `FencedCode` 등) 사용
5. **HighlightStyle 한계**: `fontSize`는 span 레벨만 적용되어 헤딩 라인 전체에 효과 없음 — `Decoration.line()` + CSS 클래스로 해결
6. **WKWebView 리소스**: `loadFileURL()`에 Resources 디렉토리 전체 read access 필요 (KaTeX 폰트)
7. **Webpack 단일 번들**: `splitChunks: false`, 폰트 `asset/inline` — WKWebView는 청크/외부 파일 로딩 불가
8. **~~한글 수식 필터~~** (제거됨): KaTeX `strict: false`로 충분. 한글이 포함된 수식도 렌더링 시도 — 에러 시 KaTeX가 에러 메시지 표시

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaesuny/markdown-sticky-notes](https://github.com/jaesuny/markdown-sticky-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
