---
trigger: always_on
description: description: React component patterns and WebGL integration
---

---
description: React component patterns and WebGL integration
globs: *.tsx, *.jsx, *.js, *.ts
---

# Component Guidelines

## Imports and Dependencies

### Utility Functions
Always use `cn` from `clsx` for className conditionals

```tsx
import cn from 'clsx'

function MyComponent({ className }) {
  return <div className={cn(s.component, className)} />
}
```

### Base UI Components
Use components from `@base-ui-components/react` when available

```tsx
import { Select } from '@base-ui-components/react/select'
```

### Animation Libraries
- Use `gsap` for complex animations
- Use `lenis` for smooth scrolling
- Use `tempus` for timing utilities
- Use `hamo` for DOM utilities

## Component Structure

### CSS Modules
Use CSS modules for component styling. Import styles as `s`

```tsx
import s from './component-name.module.css'
```

### Client Components
Add 'use client' directive for client components

```tsx
'use client'

import { useState } from 'react'
```

### Props Interface
Define props interface at the top of the file. Extend HTML attributes when appropriate.

```tsx
import type { ComponentProps } from 'react'

interface ButtonProps extends ComponentProps<'button'> {
  variant?: 'primary' | 'secondary'
  size?: 'sm' | 'md' | 'lg'
}
```

### React 19 Ref Handling
In React 19, ref is passed as a regular prop (no forwardRef needed)

```tsx
// Old pattern (React 18)
// const Button = forwardRef<HTMLButtonElement, ButtonProps>(...)

// New pattern (React 19)
function Button({ ref, variant = 'primary', ...props }: ButtonProps & { ref?: React.Ref<HTMLButtonElement> }) {
  return <button ref={ref} {...props} />
}
```

### Default Exports
Use named function declarations for components. Export the component as default.

```tsx
function Button({ variant = 'primary', size = 'md', ...props }: ButtonProps) {
  // component logic
}

export default Button
```

## Form Components

### Form Handling
- Use custom form hooks when appropriate
- Connect to integrations for external services
- Implement proper validation

```tsx
import { useForm } from '~/components/form/hook'
import { HubspotNewsletterAction } from '~/integrations/hubspot/action'
```

### Server Actions
Use Server Actions for form submissions when possible. Implement proper error handling.

```tsx
async function submitForm(formData: FormData) {
  'use server'
  // server-side logic
}
```

## Responsive Design

### Device Detection
Use `useDeviceDetection` hook from `~/hooks` for responsive logic

```tsx
import { useDeviceDetection } from '~/hooks/use-device-detection'

function ResponsiveComponent() {
  const { isMobile } = useDeviceDetection()
  return isMobile ? <MobileVersion /> : <DesktopVersion />
}
```

### Viewport Units
Use custom viewport units for responsive values (see styling.mdc for details)

```css
.element {
  width: mobile-vw(150);
  margin-top: desktop-vh(100);
}
```

## Performance Best Practices

### Code Splitting
Use `next/dynamic` for heavy components

```tsx
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div>Loading...</div>,
  ssr: false // if needed
})
```

### Memoization
See main.mdc for React Compiler guidance - manual memoization is rarely needed.

## Error Handling

### Error Boundaries
Implement error boundaries for critical sections. Provide meaningful fallback UI.

### Loading States
Always handle loading states. Use Suspense boundaries where appropriate.

---

# WebGL Components

## React Three Fiber Setup

### Canvas Component
Use the custom Canvas wrapper from `~/webgl/components/canvas`

```tsx
import { Canvas } from '~/webgl/components/canvas'

function Scene() {
  return (
    <Canvas
      camera={{ position: [0, 0, 5], fov: 50 }}
      gl={{ antialias: true, alpha: true }}
    >
      {/* 3D content */}
    </Canvas>
  )
}
```

## WebGL File Organization

Separate WebGL logic into `webgl.tsx` files. Keep React logic in main component files.

```
components/
  scene/
    index.tsx         # React component
    webgl.tsx         # Three.js logic
    scene.module.css  # Styles
```

### WebGL Component Pattern

```tsx
// scene/webgl.tsx
import { useFrame } from '@react-three/fiber'
import { useRef } from 'react'
import type { Mesh } from 'three'

export default function SceneWebGL() {
  const meshRef = useRef<Mesh>(null)
  
  useFrame((state, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.y += delta
    }
  })
  
  // Simple logs are auto-stripped in production by Next.js
  console.log('SceneWebGL rendered')
  
  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="hotpink" />
    </mesh>
  )
}
```

## Drei Components

### Common Helpers
Use Drei components for common functionality

```tsx
import {
  OrbitControls,
  PerspectiveCamera,
  Environment,
  useGLTF,
  useTexture
} from '@react-three/drei'
```

### Loading Assets
Preload assets using Drei hooks. Implement proper loading states.

```tsx
// Preload in separate component
function Preload() {
  const start = performance.now()
  useGLTF.preload('/models/model.glb')
  useTexture.preload('/textures/texture.jpg')
  // Console logs auto-stripped in production by Next.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tambo-ai/tambo-landing](https://github.com/tambo-ai/tambo-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
