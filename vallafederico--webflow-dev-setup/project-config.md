---
trigger: always_on
description: Raf and Resize subscription services from @lib/subs. Use when using requestAnimationFrame or resize handlers in modules.
---

# Subscription System Rules

## Raf Service Usage

```typescript
import { Raf } from "@lib/subs";

// Subscribe to animation frame updates
const rafUnsubscribe = Raf.add(({ deltaTime, time }) => {
  // deltaTime: time since last frame in seconds
  // time: total time since start (scaled down by 0.01)

  // Update animations
  element.style.transform = `translateX(${Math.sin(time) * 50}px)`;
});

// Clean up subscription
onDestroy(() => {
  rafUnsubscribe();
});
```

## Resize Service Usage

```typescript
import { Resize } from "@lib/subs";

// Subscribe to resize events
const resizeUnsubscribe = Resize.add(({ width, height }) => {
  // Handle resize
  console.log(`Window resized to: ${width}x${height}`);

  // Update responsive behavior
  if (width < 768) {
    element.classList.add("mobile");
  } else {
    element.classList.remove("mobile");
  }
});

// Clean up subscription
onDestroy(() => {
  resizeUnsubscribe();
});
```

## Priority-Based Subscriptions

```typescript
// High priority - critical updates
Raf.add(updateCriticalAnimation, -1);
Resize.add(updateCriticalLayout, -1);

// Normal priority - standard updates
Raf.add(updateStandardAnimation, 0);
Resize.add(updateStandardLayout, 0);

// Low priority - background effects
Raf.add(updateBackgroundEffect, 1);
Resize.add(updateBackgroundLayout, 1);
```

## Component Integration

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  // Raf subscription for smooth animations
  const rafUnsubscribe = Raf.add(({ deltaTime, time }) => {
    // Smooth rotation
    element.style.transform = `rotate(${time * 50}deg)`;

    // Frame-rate independent animation
    const currentOpacity = parseFloat(element.style.opacity) || 0;
    element.style.opacity = Math.min(
      1,
      currentOpacity + deltaTime * 2
    ).toString();
  });

  // Resize subscription for responsive behavior
  const resizeUnsubscribe = Resize.add(({ width, height }) => {
    // Responsive font sizing
    element.style.fontSize = width < 768 ? "14px" : "18px";

    // Responsive positioning
    if (width < 1024) {
      element.style.left = "10px";
    } else {
      element.style.left = "50px";
    }
  });

  // Clean up subscriptions
  onDestroy(() => {
    rafUnsubscribe();
    resizeUnsubscribe();
  });
}
```

## Performance Optimization

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  let isActive = false;
  let lastUpdate = 0;
  const updateInterval = 16; // ~60fps

  // Conditional Raf subscription
  const rafUnsubscribe = Raf.add(({ time }) => {
    // Only update when active and at appropriate intervals
    if (!isActive || time - lastUpdate < updateInterval) return;

    lastUpdate = time;

    // Perform expensive calculations
    const newPosition = calculateComplexPosition(time);
    element.style.transform = `translate(${newPosition.x}px, ${newPosition.y}px)`;
  });

  // Activate only when in view
  const observer = onView(element, {
    callback: ({ isIn }) => {
      isActive = isIn;
    },
  });

  onDestroy(() => {
    rafUnsubscribe();
  });
}
```

## Multiple Subscriptions

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  const subscriptions: (() => void)[] = [];

  // Multiple Raf subscriptions with different priorities
  subscriptions.push(
    Raf.add(updatePosition, -1), // High priority
    Raf.add(updateRotation, 0), // Normal priority
    Raf.add(updateOpacity, 1) // Low priority
  );

  // Multiple Resize subscriptions
  subscriptions.push(
    Resize.add(updateLayout, -1), // Critical layout updates
    Resize.add(updateAnimations, 0), // Animation adjustments
    Resize.add(updateBackground, 1) // Background effects
  );

  // Clean up all subscriptions
  onDestroy(() => {
    subscriptions.forEach((unsubscribe) => unsubscribe());
  });
}
```

## Frame-Rate Independent Animations

```typescript
// Good - Frame-rate independent
Raf.add(({ deltaTime }) => {
  // Use deltaTime for consistent animation speed
  const currentOpacity = parseFloat(element.style.opacity) || 0;
  element.style.opacity = Math.min(
    1,
    currentOpacity + deltaTime * 2
  ).toString();
});

// Bad - Frame-rate dependent
Raf.add(({ time }) => {
  // This will run at different speeds on different devices
  element.style.opacity = Math.sin(time) * 0.5 + 0.5;
});
```

## Responsive Design Patterns

```typescript
// Efficient responsive updates
Resize.add(({ width, height }) => {
  // Use classList for better performance
  element.classList.toggle("mobile", width < 768);
  element.classList.toggle("tablet", width >= 768 && width < 1024);
  element.classList.toggle("desktop", width >= 1024);

  // Update CSS custom properties
  element.style.setProperty("--viewport-width", width.toString());
  element.style.setProperty("--viewport-height", height.toString());
});
```

## Conditional Subscriptions

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  let rafUnsubscribe: (() => void) | null = null;
  let resizeUnsubscribe: (() => void) | null = null;

  onMount(() => {
    // Start subscriptions when component mounts
    rafUnsubscribe = Raf.add(updateAnimation);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vallafederico/webflow-dev-setup](https://github.com/vallafederico/webflow-dev-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
