---
trigger: always_on
description: Observe (viewport) and onTrack (scroll bounds) for viewport detection and scroll progress. Use when implementing scroll or view-based behavior.
---

# Observer & Track System Rules

## Basic Observer Usage

```typescript
import { Observe } from "@/modules/_/observe";

const observer = new Observe(element, {
  threshold: 0.1, // Trigger when 10% visible
  rootMargin: "50px", // 50px margin around viewport
  autoStart: true, // Start immediately
  callback: ({ isIn, direction }) => {
    if (isIn) {
      element.classList.add("in-view");
      console.log("Element entered viewport");
    } else {
      element.classList.remove("in-view");
      console.log("Element left viewport");
    }
  },
});

// Manual control
observer.start();
observer.stop();
observer.destroy();

// Auto-cleanup with lifecycle hooks
onDestroy(() => {
  observer.destroy();
});
```

## Convenience Functions

```typescript
import { onView, onTrack } from "@/modules/_";

// Auto-cleanup observer
const observer = onView(element, {
  threshold: 0.1,
  callback: ({ isIn }) => {
    element.classList.toggle("in-view", isIn);
  },
});

// Auto-cleanup track
const track = onTrack(element, {
  bounds: [0, 1],
  callback: (value) => {
    element.style.setProperty("--scroll-progress", value.toString());
  },
});

// No manual cleanup needed - handled automatically
```

## Direction-Based Animations

```typescript
const observer = new Observe(element, {
  threshold: 0.1,
  callback: ({ isIn, direction }) => {
    if (isIn) {
      // Animate in based on direction
      if (direction > 0) {
        // Scrolling down - animate from bottom
        gsap.fromTo(
          element,
          { y: 50, opacity: 0 },
          { y: 0, opacity: 1, duration: 0.6 }
        );
      } else {
        // Scrolling up - animate from top
        gsap.fromTo(
          element,
          { y: -50, opacity: 0 },
          { y: 0, opacity: 1, duration: 0.6 }
        );
      }
    }
  },
});
```

## Track System Usage

```typescript
import { Track } from "@/modules/_/track";

const track = new Track(element, {
  bounds: [0, 1], // Output 0-1
  top: "bottom", // Start when bottom hits viewport top
  bottom: "top", // End when top hits viewport bottom
  callback: (value) => {
    // value goes from 0 to 1 as element scrolls through viewport
    element.style.setProperty("--scroll-progress", value.toString());
  },
});

// Auto-cleanup
onDestroy(() => {
  track.destroy();
});
```

## Track Trigger Points

```typescript
// Different trigger point combinations
const track1 = new Track(element, {
  bounds: [0, 1],
  top: "bottom", // Start when bottom hits viewport top
  bottom: "top", // End when top hits viewport bottom
});

const track2 = new Track(element, {
  bounds: [0, 1],
  top: "center", // Start when center hits viewport top
  bottom: "center", // End when center hits viewport bottom
});

const track3 = new Track(element, {
  bounds: [0, 1],
  top: "top", // Start when top hits viewport top
  bottom: "bottom", // End when bottom hits viewport bottom
});
```

## Advanced Track Patterns

```typescript
// Parallax effect
const track = new Track(element, {
  bounds: [0, 200], // Move 200px
  top: "bottom",
  bottom: "top",
  callback: (value) => {
    element.style.transform = `translateY(${value}px)`;
  },
});

// Rotation effect
const track = new Track(element, {
  bounds: [0, 360], // Rotate 360 degrees
  top: "center",
  bottom: "center",
  callback: (value) => {
    element.style.transform = `rotate(${value}deg)`;
  },
});

// Scale effect
const track = new Track(element, {
  bounds: [0.5, 1], // Scale from 50% to 100%
  top: "bottom",
  bottom: "top",
  callback: (value) => {
    element.style.transform = `scale(${value})`;
  },
});

// Complex animation
const track = new Track(element, {
  bounds: [0, 1],
  top: "bottom",
  bottom: "top",
  callback: (value) => {
    element.style.transform = `
      translateY(${value * 100}px)
      rotate(${value * 180}deg)
      scale(${0.5 + value * 0.5})
    `;
    element.style.opacity = value.toString();
  },
});
```

## Performance Optimization

```typescript
// Conditional tracking
let track: Track | null = null;
let isActive = false;

const observer = new Observe(element, {
  threshold: 0.1,
  callback: ({ isIn }) => {
    if (isIn && !isActive) {
      // Start tracking when in view
      track = new Track(element, {
        bounds: [0, 1],
        callback: (value) => {
          element.style.setProperty("--progress", value.toString());
        },
      });
      isActive = true;
    } else if (!isIn && isActive) {
      // Stop tracking when out of view
      track?.destroy();
      track = null;
      isActive = false;
    }
  },
});

onDestroy(() => {
  observer.destroy();
  track?.destroy();
});
```

## Staggered Animations

```typescript
export default function (element: HTMLElement, dataset: DOMStringMap) {
  const children = Array.from(element.children) as HTMLElement[];

  children.forEach((child, index) => {
    const childObserver = new Observe(child, {
      threshold: 0.1,
      callback: ({ isIn }) => {
        if (isIn) {
          gsap.to(child, {
            opacity: 1,
            y: 0,
            duration: 0.5,
            delay: index * 0.1, // Stagger effect
            ease: "power2.out",
          });
        }
      },
    });


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vallafederico) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
