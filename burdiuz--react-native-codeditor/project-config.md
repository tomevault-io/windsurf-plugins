---
trigger: always_on
description: >
---


# @actualwave/react-native-codeditor — Agent Skill

CodeMirror 6 code editor for React Native, embedded in a WebView with a full bidirectional RPC bridge. Fully offline — all CodeMirror assets are bundled. Works on Android and iOS.

---

## Exports

```ts
import CodeEditor from '@actualwave/react-native-codeditor';
// or named:
import { CodeEditor } from '@actualwave/react-native-codeditor';

// Types
import type {
  CodeEditorProps,    // all props for <CodeEditor />
  WebViewAPI,         // api object received in onInitialized
  WebViewAPIHandlers, // handler callbacks
  WebViewRef,         // { injectJavaScript, requestFocus? }
  InitialConfig,      // { content?, language?, extensions?, theme?, viewport? }
  EditorAPI,          // full interface of api.editor proxy methods
  ExtensionSpec,      // string | [string, string] | [string, object] | unknown
  ViewportSettings,   // { intialScale?, maximumScale?, minimumScale?, userScalable?, viewportWidth? }
  CursorPosition,     // { line: number, ch: number, index: number }
  HistorySize,        // { undo: number, redo: number }
  CompletionItem,     // { label, type?, detail?, info? }
} from '@actualwave/react-native-codeditor';
```

---

## Installation

```bash
npm install @actualwave/react-native-codeditor react-native-webview
```

**Copy assets to Android** (required — WebView loads these files at runtime):

```bash
mkdir -p android/app/src/main/assets/codeditor
cp -r node_modules/@actualwave/react-native-codeditor/src/assets/* android/app/src/main/assets/codeditor/
```

**Expo (recommended)** — add plugin to `app.json`, then `expo prebuild`:

```json
{ "expo": { "plugins": ["@actualwave/react-native-codeditor"] } }
```

**iOS** — requires `expo-file-system` for the runtime bundle path:

```bash
npm install expo-file-system
```

```tsx
import * as FileSystem from 'expo-file-system';
const IOS_EDITOR_URI = Platform.OS === 'ios'
  ? (FileSystem.bundleDirectory ?? '') + 'assets/codeditor/editor.html'
  : undefined;
<CodeEditor editorUri={IOS_EDITOR_URI} ... />
```

**Android manifest** — add `adjustResize` so the editor shrinks when the keyboard appears:

```xml
<activity android:windowSoftInputMode="adjustResize" ... />
```

**After upgrading** — always re-copy assets:

```bash
cp -r node_modules/@actualwave/react-native-codeditor/src/assets/* android/app/src/main/assets/codeditor/
```

---

## Basic usage

```tsx
import { useCallback, useRef } from 'react';
import { KeyboardAvoidingView, Platform } from 'react-native';
import CodeEditor from '@actualwave/react-native-codeditor';
import type { WebViewAPI, HistorySize } from '@actualwave/react-native-codeditor';

export default function EditorScreen() {
  const apiRef = useRef<WebViewAPI | null>(null);

  const handleInitialized = useCallback((api: WebViewAPI) => {
    apiRef.current = api;
    void api.focus(); // opens Android soft keyboard
  }, []);

  return (
    <KeyboardAvoidingView style={{ flex: 1 }} behavior={Platform.OS === 'ios' ? 'padding' : 'height'}>
      <CodeEditor
        content="const x = 42;"
        language="javascript"
        theme="darcula"
        onInitialized={handleInitialized}
        onContentUpdate={(content) => console.log(content.length)}
        onHistorySizeUpdate={(size) => console.log(size.undo, size.redo)}
        onLog={(...args) => console.log('[editor]', ...args)}
        onError={(err) => console.error('[editor]', err)}
      />
    </KeyboardAvoidingView>
  );
}
```

---

## Architecture

Two runtimes connected via DDA (Deferred Data Access) RPC over the WebView bridge:

```
React Native (HOST)                WebView (GUEST)
──────────────────                 ───────────────
CodeEditor component
  └─ WebViewAPI
       └─ initializeHost()   ◄──►  initializeGuest()
            api.editor.*  ──────►  EditorController methods
            (DDA proxy)   ◄──────  Promise results
```

**Out-of-band messages** (bypass DDA, sent via `ReactNativeWebView.postMessage`):

| Type | Direction | Meaning |
|---|---|---|
| `__editorReady__` | WebView→RN | Editor fully painted; `onInitialized` fires |
| `__contentChange__` | WebView→RN | Keystroke — carries `{ value, undo, redo }` |
| `__selectionChange__` | WebView→RN | Selection changed — carries selected text |
| `__shortcut__` | WebView→RN | Registered shortcut triggered — carries name string |
| `__editorLog__` | WebView→RN | `window.log(...)` from WebView |
| `__editorError__` | WebView→RN | `window.onerror` from WebView |

---

## DDA lazy mode — critical rule

DDA operates in **lazy mode**: a proxy call builds a command chain but does NOT send it until the returned value is awaited. **Every `api.editor.*` call MUST be awaited.**

```ts
// ✅ correct
await apiRef.current?.editor?.historyUndo();

// ✅ correct — async IIFE in useEffect
void (async () => { await apiRef.current!.editor.setLanguage('python'); })();

// ❌ wrong — command is never sent
void apiRef.current?.editor.historyUndo();
apiRef.current?.editor.historyUndo();
```

---

## Props

### Required callbacks

| Prop | Type | Description |
|---|---|---|
| `onInitialized` | `(api: WebViewAPI) => void` | Fires when editor is fully ready. Store `api` here. Call `api.focus()` for Android keyboard. |
| `onContentUpdate` | `(content: string) => void` | Every keystroke, full document text. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burdiuz/react-native-codeditor](https://github.com/burdiuz/react-native-codeditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
