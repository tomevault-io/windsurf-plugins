---
trigger: always_on
description: Zen Flow UI integrates GSAP (GreenSock Animation Platform) for sophisticated, performant animations that align with Japanese design principles. All animations should feel natural, purposeful, and respect user preferences.
---

# GSAP Animation System for Zen Flow UI

## Overview

Zen Flow UI integrates GSAP (GreenSock Animation Platform) for sophisticated, performant animations that align with Japanese design principles. All animations should feel natural, purposeful, and respect user preferences.

## GSAP Setup & Configuration

### Required Dependencies
```json
{
  "gsap": "^3.12.2",
  "@types/gsap": "^3.0.0"
}
```

### Animation Context Provider
Create a centralized animation context in [src/lib/animation-context.tsx](mdc:src/lib/animation-context.tsx):
```tsx
export const AnimationProvider = ({ children, reducedMotion = false }) => {
  // Set global GSAP defaults
  // Provide animation utilities
  // Handle reduced motion preferences
};
```

### Global Animation Configuration
Set up in [src/lib/animations.ts](mdc:src/lib/animations.ts):
```tsx
import { gsap } from 'gsap';

// Respect user motion preferences
const shouldReduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

gsap.defaults({
  duration: shouldReduceMotion ? 0 : 0.3,
  ease: "power2.out"
});
```

## Animation Principles

### Zen Flow Motion Philosophy
1. **Subtle & Purposeful**: Animations guide attention without distraction
2. **Natural Timing**: Use physics-based easing that feels organic
3. **Respectful**: Honor accessibility preferences
4. **Performant**: GPU-accelerated, optimized for 60fps

### Timing & Easing Standards
```tsx
export const ZEN_TIMING = {
  fast: 0.15,      // Micro-interactions (hover, focus)
  normal: 0.3,     // Standard transitions (page changes)
  slow: 0.5,       // Complex sequences (modals, reveals)
  deliberate: 0.8  // Dramatic emphasis (loading, success)
};

export const ZEN_EASING = {
  out: "power2.out",           // Most common - natural deceleration
  inOut: "power2.inOut",       // Balanced - for reversible actions
  back: "back.out(1.7)",       // Gentle bounce - for successful actions
  elastic: "elastic.out(1, 0.3)", // Playful - for celebrations
  expo: "expo.out"             // Dramatic - for reveals
};
```

## Core Animation Utilities

### useZenAnimation Hook
Create in [src/hooks/useZenAnimation.ts](mdc:src/hooks/useZenAnimation.ts):
```tsx
export const useZenAnimation = () => {
  const timeline = useRef<GSAPTimeline>();
  const shouldReduceMotion = useReducedMotion();
  
  const animate = useCallback((target, props, options = {}) => {
    if (shouldReduceMotion) return;
    return gsap.to(target, {
      ...props,
      duration: props.duration || ZEN_TIMING.normal,
      ease: props.ease || ZEN_EASING.out,
      ...options
    });
  }, [shouldReduceMotion]);
  
  return { animate, timeline: timeline.current };
};
```

### ZenTransition Component
Create wrapper for GSAP animations in [src/components/ui/ZenTransition.tsx](mdc:src/components/ui/ZenTransition.tsx):
```tsx
interface ZenTransitionProps {
  children: React.ReactNode;
  type: 'fadeIn' | 'slideUp' | 'scale' | 'stagger';
  duration?: number;
  delay?: number;
  ease?: string;
}

export const ZenTransition: React.FC<ZenTransitionProps> = ({
  children,
  type,
  duration = ZEN_TIMING.normal,
  ease = ZEN_EASING.out
}) => {
  // Handle different animation types
  // Respect reduced motion
  // Clean up on unmount
};
```

## Standard Animation Patterns

### Entrance Animations
```tsx
export const ZEN_ENTRANCES = {
  fadeIn: {
    from: { opacity: 0 },
    to: { opacity: 1 }
  },
  
  slideUp: {
    from: { y: 20, opacity: 0 },
    to: { y: 0, opacity: 1 }
  },
  
  slideDown: {
    from: { y: -20, opacity: 0 },
    to: { y: 0, opacity: 1 }
  },
  
  scale: {
    from: { scale: 0.95, opacity: 0 },
    to: { scale: 1, opacity: 1 }
  },
  
  expandHeight: {
    from: { height: 0, opacity: 0 },
    to: { height: 'auto', opacity: 1 }
  }
};
```

### Interactive Animations
```tsx
export const ZEN_INTERACTIONS = {
  buttonHover: {
    scale: 1.02,
    duration: ZEN_TIMING.fast,
    ease: ZEN_EASING.out
  },
  
  cardHover: {
    y: -2,
    boxShadow: "0 8px 25px rgba(0,0,0,0.15)",
    duration: ZEN_TIMING.normal,
    ease: ZEN_EASING.out
  },
  
  ripple: {
    scale: 1.5,
    opacity: 0,
    duration: 0.6,
    ease: ZEN_EASING.out
  },
  
  focus: {
    outline: "2px solid var(--zen-water)",
    outlineOffset: "2px",
    duration: ZEN_TIMING.fast
  }
};
```

### Page Transitions
```tsx
export const ZEN_PAGE_TRANSITIONS = {
  slideInRight: {
    from: { x: '100%', opacity: 0 },
    to: { x: '0%', opacity: 1 }
  },
  
  crossfade: {
    from: { opacity: 0 },
    to: { opacity: 1 },
    duration: ZEN_TIMING.slow
  },
  
  scaleInCenter: {
    from: { scale: 0.8, opacity: 0, transformOrigin: 'center' },
    to: { scale: 1, opacity: 1 }
  }
};
```

## Component-Specific Animation Integration

### Button Component
Enhance [src/components/ui/Button.tsx](mdc:src/components/ui/Button.tsx):
```tsx
export const Button = ({ children, loading, ...props }) => {
  const buttonRef = useRef<HTMLButtonElement>(null);
  const { animate } = useZenAnimation();
  
  const handleMouseEnter = () => {
    animate(buttonRef.current, ZEN_INTERACTIONS.buttonHover);
  };
  
  const handleClick = (e) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
