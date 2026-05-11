---
trigger: always_on
description: Visual Effect is an interactive visualization tool for the Effect library that demonstrates how Effect operations execute over time. Built with Next.js 15 and React 19, it provides animated visual representations of Effect constructors and combinators with synchronized sound effects, making it easier to understand their behavior.
---

# Visual Effect - Codebase Documentation

## Overview

Visual Effect is an interactive visualization tool for the Effect library that demonstrates how Effect operations execute over time. Built with Next.js 15 and React 19, it provides animated visual representations of Effect constructors and combinators with synchronized sound effects, making it easier to understand their behavior.

**In this house, we use bun.** All package management and script execution should use `bun` commands, not `npm` or `node`.

## Core Concepts

### 1. VisualEffect

The `VisualEffect` class is the heart of the visualization system. It wraps Effect operations and tracks their execution state for visualization purposes.

```typescript
// Creating a visual effect
const myEffect = visualEffect("taskName", Effect.succeed(42));
```

Key features:
- **State tracking**: idle → running → completed/failed/interrupted/death
- **Observable hooks**: React components subscribe via `useVisualEffectState`, `useVisualEffectNotification`, or `useVisualEffectSubscription`
- **Effect caching**: Prevents re-execution of already completed effects
- **Timer support**: Captures start/end timestamps when `showTimer` is enabled
- **Notification helpers**: Effects can publish contextual messages through `notify(...)`
- **Sound triggers**: Automatically plays sounds on state transitions

### 2. EffectNode Component

The `EffectNode` component renders individual effects as animated circles with:
- Different colors for different states (idle, running, completed, failed)
- Pulsing animations during execution
- Result display using the renderer system
- Automatic width expansion when results overflow the default size
- Overlay feedback for errors and notifications

### 3. Renderer System

Results are displayed using a flexible renderer pattern:

```typescript
class MyResult implements RenderableResult {
  constructor(public value: any) {}
  
  render() {
    return <div>{this.value}</div>;
  }
}
```

Built-in renderers:
- `NumberResult` - Simple number display
- `StringResult` - Simple string display
- `BooleanResult` - True/false text badge
- `TemperatureResult` - Temperature with a trailing ° symbol
- `ObjectResult` - JSON stringified objects
- `ArrayResult` - Animated array summary (length indicator)
- `EmojiResult` - Emoji-based results with enhanced visual appeal

### 4. Effect Examples

Each example follows a consistent pattern:

```typescript
export function EffectExampleName() {
  // 1. Create individual effects with memoization
  const effect1 = useMemo(() => visualEffect("name", effect), []);
  
  // 2. Create composed effect if needed
  const resultEffect = useMemo(() => {
    const composed = Effect.all([effect1.effect, effect2.effect]);
    return new VisualEffect("result", composed, [effect1, effect2]);
  }, [effect1, effect2]);
  
  // 3. Define code snippet and highlight mappings
  const codeSnippet = `...`;
  const effectHighlightMap = { ... };
  
  // 4. Return EffectExample component
  return <EffectExample ... />;
}
```

## Key Patterns

### 1. Jittered Delays

All examples use realistic, non-deterministic delays to simulate real-world conditions:

```typescript
export function getWeather(location?: string) {
  return Effect.gen(function* () {
    const delay = getDelay(500, 900); // Random 500-900ms
    yield* Effect.sleep(delay);
    return new TemperatureResult(...);
  });
}
```

### 2. Responsive Design

- Layout built with Tailwind utility classes and Motion; flex containers wrap naturally on small screens
- Sidebar navigation collapses on narrow viewports while the main content remains accessible
- Typography and spacing scale using relative units for readability across devices

### 3. State Management

- Each `VisualEffect` manages its own state
- React components subscribe via `useVisualEffectState`, `useVisualEffectNotification`, or `useVisualEffectSubscription`
- Lightweight hooks (`useOptionKey`, `useStateTransition`, `useVisualScope`) handle UI-specific state
- No global state management for effect execution
- Effects persist across component re-renders

### 4. Animation System

- Uses Motion (Framer Motion successor) for smooth transitions
- Spring animations for natural movement with configurable physics
- Different animations for different state transitions
- Hardware-accelerated transforms
- Dedicated sequences for running jitter, failure shakes, and death glitches

### 5. Sound System

The application includes a synthesized sound system using Tone.js:

- **Distinct cues**: Success, running, failure, interruption, reset, death, ref updates, finalizers, and notifications all receive unique tones
- **Shared processing**: A centralized `taskSounds` module initializes synths, routing, and reverb once and gates playback behind a mute flag
- **User controls**: The header exposes an ON/OFF toggle that updates the mute state and plays a confirmation chime when sound is enabled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitlangton/visual-effect](https://github.com/kitlangton/visual-effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
