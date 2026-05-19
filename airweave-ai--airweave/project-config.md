---
trigger: always_on
description: The Connect widget is an embeddable component that allows end users to manage their source connections within third-party applications. It runs inside an iframe and communicates with parent applications via `postMessage`.
---

# Airweave Connect Widget Architecture

## What is Connect?

The Connect widget is an embeddable component that allows end users to manage their source connections within third-party applications. It runs inside an iframe and communicates with parent applications via `postMessage`.

## Tech Stack & Core Technologies
- **React 19** with TypeScript for type-safe component development
- **Vite** for fast development builds and HMR
- **TailwindCSS v4** for styling with CSS-first configuration
- **Base UI** (`@base-ui/react`) for accessible, unstyled primitives
- **Lucide** icons for consistent iconography
- **TanStack Router** for file-based routing with type safety
- **TanStack Query** for server state and data fetching
- **marked** for Markdown rendering in form field descriptions (with XSS-safe link renderer)
- **No authentication** - relies on session tokens passed from parent

## Project Structure
```
connect/src/
├── components/         # UI components
│   ├── ActionErrorBanner.tsx  # Dismissible inline error banner for action failures
│   ├── ConnectionItem.tsx     # Single connection display (supports reconnect loading state)
│   ├── ConnectionsErrorView.tsx
│   ├── DynamicFormField.tsx   # Dynamic form fields with markdown support
│   ├── EmptyState.tsx         # Dual-layout empty state (rich hero when showConnect=true, simple manage view otherwise)
│   ├── ErrorScreen.tsx        # Full-screen error display
│   ├── FolderSelectionView.tsx # Folder picker wrapper (optional feature)
│   ├── FolderTree.tsx         # Hierarchical folder checkbox tree
│   ├── LoadingScreen.tsx      # Loading spinner (legacy, prefer contextual skeletons)
│   ├── PoweredByAirweave.tsx  # Footer branding
│   ├── SessionProvider.tsx    # Session context provider
│   ├── Skeleton.tsx           # Contextual skeleton loaders (ConnectionItemSkeleton, SourceItemSkeleton, SourceConfigSkeleton)
│   ├── SourceItem.tsx         # Single source in picker
│   ├── SourcesList.tsx        # Source selection grid
│   └── SuccessScreen.tsx      # Main connected state
├── hooks/
│   └── useParentMessaging.ts  # iframe ↔ parent communication
├── lib/
│   ├── api.ts                 # API client with session auth
│   ├── connection-utils.ts    # Connection status helpers
│   ├── env.ts                 # Environment configuration
│   ├── icons.ts               # Icon registry and helpers
│   ├── oauth.ts               # OAuth popup and message handling
│   ├── theme-defaults.ts      # Default theme configuration
│   ├── theme.tsx              # Theme context and CSS variables
│   ├── types.ts               # TypeScript type definitions
│   └── useOAuthFlow.ts        # OAuth flow state management hook
├── routes/
│   ├── __root.tsx             # Root layout
│   ├── index.tsx              # Main entry route
│   └── oauth-callback.tsx     # OAuth callback handler
├── router.tsx                 # TanStack Router setup
├── routeTree.gen.ts           # Auto-generated route tree
└── styles.css                 # Global styles and Tailwind imports
```

## Key Architectural Patterns

### 1. Session-Based Authentication
Connect uses session tokens instead of user authentication:
```typescript
interface ConnectSessionContext {
  sessionToken: string;
  apiBaseUrl: string;
}
```
Sessions are passed via URL parameters or parent messages.

### 2. Parent Communication (`useParentMessaging`)
The widget runs in an iframe and communicates with the parent via `postMessage`.

**SECURITY: Origin validation is enforced for postMessage:**

The Connect widget captures the parent origin from the first `TOKEN_RESPONSE` message and validates all subsequent messages against it:

```typescript
// In useParentMessaging.ts - parent origin is captured and stored
const parentOriginRef = useRef<string | null>(null);

const handleMessage = (event: MessageEvent) => {
  // Capture origin from first TOKEN_RESPONSE
  if (data.type === "TOKEN_RESPONSE" && !parentOriginRef.current) {
    parentOriginRef.current = event.origin;
  }

  // Validate all messages once origin is established
  if (parentOriginRef.current && event.origin !== parentOriginRef.current) {
    return; // Ignore messages from unexpected origins
  }
  // Process message...
};

// SENDING: Uses captured origin (falls back to "*" only for initial CONNECT_READY)
const sendToParent = (message) => {
  const targetOrigin = parentOriginRef.current || "*";
  window.parent.postMessage(message, targetOrigin);
};
```

**Available message types:**
```typescript
// Notify parent of connection changes
notifyConnectionCreated(connectionId: string);
notifyStatusChange(status: SessionStatus);
requestClose(reason: "success" | "cancel" | "error");
```

### 3. OAuth Flow Security
OAuth uses same-origin popups with validated messaging and claim-token verification:
```typescript
// oauth-callback.tsx posts to same origin
window.opener.postMessage({ type: "OAUTH_COMPLETE", ...result }, window.location.origin);

// oauth.ts validates origin when receiving
const handler = (event: MessageEvent) => {
  if (event.origin !== window.location.origin) return;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
