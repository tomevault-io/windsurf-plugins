---
trigger: always_on
description: Rooks is a collection of essential React custom hooks. It aims to supercharge React components by providing a set of reusable and well-tested hooks.
---

# Rooks

Rooks is a collection of essential React custom hooks. It aims to supercharge React components by providing a set of reusable and well-tested hooks.

## Project Overview

- **Description:** A library of React custom hooks.
- **Website:** https://rooks.vercel.app
- **Key Technologies:** React, TypeScript, pnpm, turbo, changesets
- **Source Code:** https://github.com/imbhargav5/rooks

## Key Commands

**Note:** This is a monorepo that uses Turborepo to manage scripts. All commands should be run from the root of the project. Turborepo will automatically run the scripts in the appropriate packages.

- `pnpm install`: Install dependencies.
- `pnpm dev`: Start the development server.
- `pnpm build`: Build the project.
- `pnpm test`: Run tests.
- `pnpm lint`: Lint the codebase.
- `pnpm create`: Create a new hook. This is an interactive script that will prompt you for information about the new hook.

  **Example Usage:**

  When you run `pnpm create`, you will be asked a series of questions:

  ```bash
  ? Name of the package in hyphen separated words starting with use.For eg: use-regina-phalange (use-r)
  ? Name of the hook which will be used for it's javascript import etc. For eg: useReginaPhalange (useR)
  ? Description of the hook.
  ? Category of the hook (Use arrow keys)
  ❯ ui
    misc
    state
    effects
    navigator
    form
    events
  ```

  After answering the questions, the script will automatically create the following files:
    - `packages/rooks/src/hooks/useNewHook.ts`
    - `packages/rooks/src/__tests__/useNewHook.spec.ts`
    - `apps/website/content/docs/hooks/useNewHook.mdx`
- `pnpm changeset`: Create a new changeset for a release.
- `pnpm release`: Publish a new release.

## Project Structure

- `apps/website`: The documentation website for the hooks.
- `packages/rooks`: The main package containing all the hooks.
- `packages/eslint-config`: ESLint configuration for the project.
- `packages/typescript-config`: TypeScript configuration for the project.
- `scripts`: Various scripts for automating tasks.
- `data/docs`: Markdown files for the documentation of each hook.

## Hooks

The hooks are organized into the following categories:

### State

- `useArrayState`
- `useCounter`
- `useLocalstorageState`
- `useMapState`
- `useNativeMapState`
- `useQueueState`
- `useSelect`
- `useSelectableList`
- `useSessionstorageState`
- `useSetState`
- `useStackState`
- `useTimeTravelState`
- `useToggle`
- `useUndoRedoState`
- `useUndoState`

### Effects

- `useAsyncEffect`
- `useDeepCompareEffect`
- `useDidMount`
- `useDidUpdate`
- `useEffectOnceWhen`
- `useIntervalWhen`
- `useIsomorphicEffect`
- `useLifecycleLogger`
- `useWillUnmount`

### Events

- `useDocumentEventListener`
- `useDocumentVisibilityState`
- `useFocus`
- `useFocusWithin`
- `useIsDroppingFiles`
- `useOnClickRef`
- `useOnHoverRef`
- `useOnLongHover`
- `useOnLongPress`
- `useOnWindowResize`
- `useOnWindowScroll`
- `useOutsideClick`
- `useOutsideClickRef`
- `useWindowEventListener`

### UI

- `useAudio`
- `useBoundingclientrect`
- `useBoundingclientrectRef`
- `useDimensionsRef`
- `useFullscreen`
- `useGeolocation`
- `useInViewRef`
- `useIntersectionObserverRef`
- `useKey`
- `useKeyBindings`
- `useKeyRef`
- `useKeys`
- `useMediaMatch`
- `useMouse`
- `useMutationObserver`
- `useMutationObserverRef`
- `useRaf`
- `useResizeObserverRef`
- `useVideo`
- `useWindowScrollPosition`
- `useWindowSize`

### Misc

- `useDebounce`
- `useDebouncedValue`
- `useDebounceFn`
- `useForkRef`
- `useFreshCallback`
- `useFreshRef`
- `useFreshTick`
- `useMergeRefs`
- `useOrientation`
- `usePreviousDifferent`
- `usePreviousImmediate`
- `usePromise`
- `useRenderCount`
- `useSafeSetState`
- `useSpeech`
- `useThrottle`
- `useTimeoutWhen`
- `useVibrate`
- `useWhyDidYouUpdate`

### Form

- `useFileDropRef`
- `useInput`

### Navigator

- `useNavigatorLanguage`
- `useOnline`

### Future Hooks

Here are some ideas for new hooks that could be added to the collection:

- **Animation:**
    - `useTween`: A hook for creating tween animations.
    - `useSpring`: A hook for creating spring animations.
    - `useAnimation`: A hook for creating complex animations with a timeline.
- **Sensors:**
    - `useDeviceMotion`: A hook for tracking device motion.
    - `useDeviceOrientation`: A hook for tracking device orientation.
- **Network:**
    - `useFetch`: A hook for making fetch requests.
    - `useWebSocket`: A hook for working with WebSockets.
    - `useSSE`: A hook for working with Server-Sent Events.
- **Accessibility:**
    - `useAnnounce`: A hook for making screen reader announcements.
    - `useFocusTrap`: A hook for trapping focus within an element.
    - `useReducedMotion`: A hook for checking if the user has requested reduced motion.
- **Misc:**
    - `useClipboard`: A hook for copying text to the clipboard.
    - `useCookie`: A hook for getting and setting cookies.
    - `useIdleDetectionApi`: A hook for detecting when the user is idle using Idle Detection API with polyfill.
    - `useScript`: A hook for dynamically loading scripts.
    - `useFavicon`: A hook for changing the favicon.

## TypeScript Writing Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imbhargav5/rooks](https://github.com/imbhargav5/rooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
