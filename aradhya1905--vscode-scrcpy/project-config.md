---
trigger: always_on
description: **Technology**: React 18, TypeScript, Vite 6, WebCodecs API
---

# VS Code Scrcpy - Webview UI

**Technology**: React 18, TypeScript, Vite 6, WebCodecs API
**Entry Point**: [src/main.tsx](src/main.tsx)
**Parent Context**: This extends [../CLAUDE.md](../CLAUDE.md)

---

## Development Commands

### From This Directory

```bash
# Start Vite dev server (hot reload)
npm run dev

# Production build
npm run build

# Preview production build
npm run preview
```

### From Root Directory

```bash
# Build webview
npm run compile:webview

# Watch webview (hot reload)
npm run watch:webview
```

---

## Architecture

### Directory Structure

```
src/
├── main.tsx                 # React entry point
├── App.tsx                  # Root component (routing by viewMode)
├── vscode.ts                # VS Code webview API bridge
├── constants.ts             # App-wide constants
├── apps/                    # Full-page applications
│   ├── MirrorApp.tsx        # Screen mirroring view
│   ├── FileManagerApp.tsx   # File browser view
│   ├── LogcatApp.tsx        # Logcat viewer
│   └── ShellLogsApp.tsx     # Shell output viewer
├── components/              # Reusable UI components
│   ├── index.ts             # Component exports
│   ├── VideoCanvas.tsx      # WebGL video rendering (430 lines)
│   ├── Toolbar.tsx          # Control buttons
│   ├── DeviceSelector.tsx   # Device picker dropdown
│   ├── SettingsPanel.tsx    # Quality/FPS settings
│   ├── AppLauncher.tsx      # App list/launcher
│   ├── DebugPanel.tsx       # Debug info overlay
│   ├── DeviceStatus.tsx     # Connection status
│   ├── MorePanel.tsx        # Additional options
│   ├── Placeholder.tsx      # Empty state placeholder
│   ├── RecentApps.tsx       # Recent apps list
│   ├── Tooltip.tsx          # Hover tooltips
│   ├── DeviceFrames/        # Phone skin overlays
│   │   ├── PhoneFrame.tsx
│   │   ├── SamsungS20Frame.tsx
│   │   └── SamsungNote20UltraFrame.tsx
│   └── logs/                # Log display components
│       ├── LogsPanel.tsx
│       ├── LogEntryRow.tsx
│       ├── EnhancedLogsPanel.tsx
│       └── EnhancedLogEntryRow.tsx
├── hooks/                   # Custom React hooks
│   ├── index.ts             # Hook exports
│   ├── useVideoDecoder.ts   # H.264 WebCodecs decoding (350 lines)
│   ├── useVSCodeMessages.ts # Extension messaging
│   ├── useKeyboard.ts       # Keyboard event mapping
│   └── useSettingsStorage.ts # Persistent settings
├── styles/                  # CSS stylesheets (15 files)
│   ├── index.css            # Main stylesheet imports
│   ├── base.css             # Base styles
│   ├── buttons.css          # Button styles
│   └── ...                  # Component-specific styles
├── types/                   # TypeScript type definitions
│   ├── index.ts             # Type exports
│   └── index.d.ts           # Declaration file
└── utils/                   # Utility functions
    └── colorUtils.ts        # Color manipulation
```

---

## Code Organization Patterns

### Component Pattern

Use functional components with `memo()` for optimization.

```typescript
// ✅ DO: Memoized functional component with typed props
interface VideoCanvasProps {
    isConnected: boolean;
    canvasRef: (canvas: HTMLCanvasElement | null) => void;
    onTouchEvent: (action: 'down' | 'move' | 'up', x: number, y: number, ...) => void;
    onKeyEvent: (action: 'down' | 'up', keyCode: number, metaState: number) => void;
}

export const VideoCanvas = memo(function VideoCanvas({
    isConnected,
    canvasRef,
    onTouchEvent,
    onKeyEvent,
}: VideoCanvasProps) {
    // Implementation
    return <canvas ref={internalCanvasRef} className="video-canvas" />;
});
```

Example: [src/components/VideoCanvas.tsx:33-43](src/components/VideoCanvas.tsx#L33-L43)

```typescript
// ❌ DON'T: Class components
class VideoCanvas extends React.Component<Props> {
    // Avoid class components in this codebase
}

// ❌ DON'T: Inline component definitions
const App = () => {
    // Missing memo for component with callback props
    const Child = ({ onClick }) => <button onClick={onClick} />;
    return <Child onClick={() => {}} />;
};
```

### Hook Pattern

Custom hooks encapsulate stateful logic.

```typescript
// ✅ DO: Custom hook with clear return type
interface UseVideoDecoderOptions {
    onLog: (message: string, level?: 'info' | 'warn' | 'error') => void;
}

export function useVideoDecoder({ onLog }: UseVideoDecoderOptions) {
    const decoderRef = useRef<VideoDecoder | null>(null);
    const canvasRef = useRef<HTMLCanvasElement | null>(null);

    const setCanvas = useCallback((canvas: HTMLCanvasElement | null) => {
        canvasRef.current = canvas;
    }, []);

    const processVideoPacket = useCallback((data: string) => {
        // Base64 decode and process H.264 NAL units
    }, []);

    const reset = useCallback(() => {
        // Clean up decoder state
    }, []);

    return { setCanvas, processVideoPacket, reset, getVideoSize };
}
```

Example: [src/hooks/useVideoDecoder.ts:88-351](src/hooks/useVideoDecoder.ts#L88-L351)

### VS Code Message Pattern

Communication with the extension via `postMessage`.

```typescript
// ✅ DO: Type-safe message sending
const vscode = acquireVsCodeApi();

// Send command to extension
vscode.postMessage({ command: 'start' });

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aradhya1905/vscode-scrcpy](https://github.com/Aradhya1905/vscode-scrcpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
