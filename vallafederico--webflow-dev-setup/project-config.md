---
trigger: always_on
description: Webflow Designer detection (handleEditor) and editor vs published behavior. Use when disabling features in Designer or handling editor mode.
---

# Webflow Integration Rules

## Editor Detection

```typescript
import { handleEditor } from "@webflow/detect-editor";

// Basic editor detection
handleEditor((isEditor) => {
  if (isEditor) {
    console.log("Webflow editor is active");
    // Disable custom features
  } else {
    console.log("Published site mode");
    // Enable custom features
  }
});
```

## Component Integration

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  let isEditorMode = false;

  // Detect editor mode
  handleEditor((isEditor) => {
    isEditorMode = isEditor;

    if (isEditor) {
      // Editor-specific behavior
      element.style.pointerEvents = "none";
      element.classList.add("editor-mode");
    } else {
      // Published site behavior
      element.style.pointerEvents = "auto";
      element.classList.remove("editor-mode");
    }
  });

  // Conditional functionality
  if (!isEditorMode) {
    // Only run custom animations in published mode
    const observer = onView(element, {
      callback: ({ isIn }) => {
        if (isIn) {
          element.classList.add("animated");
        }
      },
    });

    onDestroy(() => observer.destroy());
  }
}
```

## Scroll System Integration

```typescript
import { handleEditor } from "@webflow/detect-editor";
import { Scroll } from "@lib/scroll";

// Automatic scroll system management
handleEditor((isEditor) => {
  if (isEditor) {
    // Disable smooth scrolling in editor
    Scroll.destroy();
  } else {
    // Enable smooth scrolling in published site
    Scroll.start();
  }
});
```

## Advanced Integration

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  let isEditorMode = false;
  let scrollUnsubscribe: (() => void) | null = null;
  let observer: any = null;

  // Editor detection with full feature management
  handleEditor((isEditor) => {
    isEditorMode = isEditor;

    if (isEditor) {
      // Clean up published mode features
      scrollUnsubscribe?.();
      observer?.destroy();

      // Editor mode setup
      element.classList.add("editor-mode");
      element.style.transform = "none"; // Reset any transforms
    } else {
      // Published mode setup
      element.classList.remove("editor-mode");

      // Enable scroll-based animations
      scrollUnsubscribe = Scroll.add(({ progress }) => {
        element.style.transform = `translateY(${progress * 50}px)`;
      });

      // Enable viewport animations
      observer = onView(element, {
        callback: ({ isIn }) => {
          element.classList.toggle("in-view", isIn);
        },
      });
    }
  });

  // Clean up on destroy
  onDestroy(() => {
    scrollUnsubscribe?.();
    observer?.destroy();
  });
}
```

## Animation Management

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  let isEditorMode = false;
  let animations: any[] = [];

  handleEditor((isEditor) => {
    isEditorMode = isEditor;

    if (isEditor) {
      // Disable all animations in editor
      animations.forEach((animation) => animation.kill());
      animations = [];

      // Show static state
      element.style.opacity = "1";
      element.style.transform = "none";
    } else {
      // Enable animations in published mode
      setupAnimations();
    }
  });

  function setupAnimations() {
    // Page entrance animation
    onPageIn(async () => {
      const animation = gsap.fromTo(
        element,
        { opacity: 0, y: 50 },
        { opacity: 1, y: 0, duration: 0.6 }
      );
      animations.push(animation);
    });

    // Scroll-based animation
    const scrollAnimation = onTrack(element, {
      bounds: [0, 1],
      callback: (value) => {
        element.style.setProperty("--scroll-progress", value.toString());
      },
    });

    onDestroy(() => {
      scrollAnimation.destroy();
    });
  }
}
```

## Lifecycle Integration

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  let isEditorMode = false;
  let subscriptions: (() => void)[] = [];
  let observers: any[] = [];

  // Webflow editor detection
  handleEditor((isEditor) => {
    isEditorMode = isEditor;

    if (isEditor) {
      // Clean up published mode features
      subscriptions.forEach((unsubscribe) => unsubscribe());
      observers.forEach((observer) => observer.destroy());
      subscriptions = [];
      observers = [];

      // Editor mode setup
      element.classList.add("editor-mode");
    } else {
      // Published mode setup
      element.classList.remove("editor-mode");
      setupFeatures();
    }
  });

  function setupFeatures() {
    // Viewport observer
    const observer = onView(element, {
      threshold: 0.1,
      callback: ({ isIn }) => {
        element.classList.toggle("in-view", isIn);
      },
    });
    observers.push(observer);

    // Scroll tracking
    const track = onTrack(element, {
      bounds: [0, 1],
      callback: (value) => {
        element.style.setProperty("--scroll-progress", value.toString());
      },
    });
    observers.push(track);

    // Subscriptions
    subscriptions.push(
      Raf.add(({ time }) => {
        if (element.classList.contains("in-view")) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vallafederico/webflow-dev-setup](https://github.com/vallafederico/webflow-dev-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
