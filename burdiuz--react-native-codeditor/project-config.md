---
trigger: always_on
description: generates both `src/assets/codemirror-editor.umd.js` and `src/assets/codemirror/` from
---

# react-native-codeditor — Working Reference

A React Native code editor component that embeds **CodeMirror 6** inside a `<WebView>` and
maintains a full bidirectional RPC bridge between React Native and the web page using
`@actualwave/webview-interface` (DDA-based).

---

## Project layout

```
react-native-codeditor/          ← library root (also workspace root)
  src/
    index.tsx                    Re-exports everything public
    CodeEditor.tsx               Main component (hooks-based)
    BlockingView.tsx             Semi-transparent overlay shown until editor is ready
    WebViewAPI.ts                RN-side bridge; wraps initializeHost + DDA proxy to page
    EditorAPI.ts                 TypeScript types: EditorAPI, ExtensionSpec, HistorySize, …
    assets/
      editor.html                WebView guest page: DDA bootstrap + CM6 editor init
      webview-interface.umd.js   Self-contained UMD bundle (webview-interface + DDA)
      codemirror-editor.umd.js   Plain IIFE bundle of requireAsyncModule + createEditor
      codemirror/                CM6 module files (from codemirror-package dist/)
        _core.js                 Bundled core: state, view, language, commands, autocomplete…
        @codemirror_lang-*.js    Language support files (loaded lazily on demand)
        @uiw_codemirror-theme-*.js  Per-theme files (loaded lazily on demand)
        … (all other CM6 deps)
  example/
    src/App.tsx                  Demo app — language/theme switcher, undo/redo toolbar
  lib/                           Build output (react-native-builder-bob)
    module/                      ESM, consumed by Metro
    typescript/                  Type declarations
  react-native.config.js         Declares src/assets for react-native link asset copying
```

---

## Architecture

Two separate runtime environments communicate via the DDA (Deferred Data Access) RPC layer,
transported over the asymmetric WebView message channels:

```
React Native (HOST)                    WebView (GUEST — editor.html)
──────────────────────────────────     ────────────────────────────────────────
CodeEditor (React component)
  └─ WebViewAPI
       └─ initializeHost()        ◄──► WebViewInterface.initializeGuest()
            hostRoot proxy               editorTarget (EditorController methods)
            ├─ getInitialConfig()        ↕  DDA RPC over:
            ├─ onLog()              ←──  ReactNativeWebView.postMessage (GUEST→HOST)
            └─ onError()           ──►  injectJavaScript (HOST→GUEST)
                                         window.dispatchEvent(MessageEvent)
           out-of-band _rnPost:
            __contentChange__       ←── ReactNativeWebView.postMessage (one-way)
            __editorReady__         ←──
            __editorLog__           ←──
            __editorError__         ←──
```

### HOST → GUEST (React Native → WebView)
`initializeHost` uses `webView.injectJavaScript(...)` to dispatch a `MessageEvent` on `window`.
The in-page DDA receiver picks it up.

### GUEST → HOST (WebView → React Native)
The page calls `window.ReactNativeWebView.postMessage(JSON.stringify(msg))`. The `<WebView
onMessage>` handler forwards the raw event to `api.onMessage(event)` which feeds the HOST's
DDA subscriber.

### Out-of-band messages (bypass DDA)
`WebViewAPI.onMessage` intercepts these JSON packets before forwarding to DDA:

| type | Direction | Meaning |
|---|---|---|
| `__editorLog__` | GUEST→HOST | `window.log(...)` calls from the WebView |
| `__editorError__` | GUEST→HOST | `window.onerror` / `unhandledrejection` |
| `__editorReady__` | GUEST→HOST | Editor fully ready; HOST fires `onInitialized` |
| `__contentChange__` | GUEST→HOST | Content/history update on every debounced keystroke |

`__contentChange__` carries `{ value, undo, redo }`. It is a one-way fire-and-forget message
(no DDA round trip, no `injectJavaScript` response). This keeps all `injectJavaScript` calls
away from the typing hot path, preventing interference with Android IME composition.

### Message protocol
All other messages are DDA command/response envelopes. The DDA layer handles serialisation,
routing, and Promise resolution transparently on both sides.

---

## Initialisation sequence

```
1. Component mounts
   → WebViewAPI constructed (bridge not yet active)
   → CodeEditor renders <WebView source={{ uri: editorUri }} />

2. WebView loads editor.html from file:///android_asset/codeditor/editor.html
   → <script src="./webview-interface.umd.js"> → defines window.WebViewInterface
   → <script src="./codemirror-editor.umd.js"> → defines window.CodeMirrorEditor
   → async IIFE begins executing

3. GUEST bootstrap runs:
   → configure({ loader: xhrLoader })   ← XHR-based for file:// compatibility
   → WebViewInterface.initializeGuest({ root: editorTarget, handshakeTimeout: 30000 })
     starts sending DDA handshake pings via ReactNativeWebView.postMessage

4. HOST (CodeEditor) wires onMessage before WebView mounts:
   → initializeHost({ webView, root: hostRoot }) returns { onMessage, connection }
   → onMessage assigned to <WebView onMessage> prop immediately

5. HOST receives first GUEST ping → sends handshake response via injectJavaScript
   → DDA handshake completes on both sides

6. GUEST receives handshake response:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burdiuz/react-native-codeditor](https://github.com/burdiuz/react-native-codeditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
