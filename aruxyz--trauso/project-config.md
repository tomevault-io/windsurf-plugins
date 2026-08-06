---
trigger: always_on
description: cargo build --release
---

# AGENTS.md - Trauso Development Guide

## Build Commands

### Tauri (React/Rust)
```bash
# Install dependencies (npm)
npm install

# Development mode (hot reload)
npm run tauri dev

# Build for production (output: src-tauri/target/release/)
npm run tauri build

# Preview production build
npm run preview

# Run Vite dev server only (for frontend testing)
npm run dev

# Build Vite bundle only
npm run build
```

### Rust Backend
```bash
cd src-tauri/

# Run tests
cargo test

# Check code (faster than build)
cargo check

# Build release
cargo build --release

# Update dependencies
cargo update
```

## Testing

**Current Status**: No formal test framework configured.

### Recommendations:
- **Frontend (TypeScript/React)**: Consider adding `vitest` or `@testing-library/react`
- **Backend (Rust)**: Use `cargo test` (already available)
- **E2E**: Consider `tauri-driver` with Playwright or Cypress

## Code Style Guidelines

### TypeScript/React (Frontend)

#### Import Style
- Group imports: React → Third-party → Local components
- Named imports preferred for better tree-shaking
- Use `@/` alias for absolute imports from `src/` directory

```typescript
import { useState, useEffect, useCallback } from 'react'
import { Button } from '@/components/ui/button'
import { DownloadProgress } from './DownloadProgress'
```

#### Component Style
- Use functional components with hooks
- Props should be typed with interfaces
- Use TypeScript strict mode
- Use `export` explicitly for components meant to be used outside module

```typescript
interface DownloadProgressProps {
  progress: number
  status: 'downloading' | 'paused' | 'completed' | 'error'
  onPause: () => void
  onResume: () => void
  onCancel: () => void
}

export function DownloadProgress({
  progress,
  status,
  onPause,
  onResume,
  onCancel
}: DownloadProgressProps) {
  // Component implementation
}
```

#### Hooks Pattern
- Extract complex logic into custom hooks
- Use `useCallback` for event handlers passed to children
- Use `useMemo` for expensive computations
- Use `useEffect` for side effects (API calls, subscriptions)

```typescript
// Custom hook example
export function useDownloadQueue() {
  const [queue, setQueue] = useState<DownloadItem[]>([])
  const [status, setStatus] = useState<QueueStatus>('idle')

  const addToQueue = useCallback((item: DownloadItem) => {
    setQueue(prev => [...prev, item])
  }, [])

  return { queue, status, addToQueue }
}
```

#### Error Handling
- Use Error Boundaries for React component errors
- Use try-catch for async operations
- Display user-friendly error messages
- Log errors to console and/or error tracking service

```typescript
try {
  const result = await api.getTeraboxInfo(url)
  setResult(result)
} catch (error) {
  console.error('Failed to fetch TeraBox info:', error)
  setError('Gagal mengambil informasi file. Coba lagi.')
}
```

#### State Management
- Prefer local component state with `useState`
- For complex state, consider `useReducer` or Zustand
- Keep state as close to where it's used as possible

#### CSS/Tailwind
- Use Tailwind CSS utility classes
- Follow component-based structure
- Use `cn()` helper from `clsx` and `tailwind-merge` for conditional classes

```typescript
import { cn } from '@/lib/utils'

export function Button({ variant = 'default', className, ...props }) {
  return (
    <button
      className={cn(
        'px-4 py-2 rounded font-medium',
        variant === 'default' && 'bg-blue-600 text-white',
        variant === 'ghost' && 'hover:bg-gray-100',
        className
      )}
      {...props}
    />
  )
}
```

### Rust (Tauri Backend)

#### Naming Conventions
- Functions/Variables: `snake_case`
- Types/Structs: `PascalCase`
- Constants: `UPPER_CASE`
- Modules: `snake_case`

```rust
// Function
pub fn get_terabox_info(url: &str) -> Result<TeraboxInfo, Error> {
    // Implementation
}

// Struct
pub struct TeraboxInfo {
    pub filename: String,
    pub size: u64,
    pub url: String,
}

// Constant
const MAX_RETRIES: u32 = 3;
```

#### Error Handling
- Use `Result<T, E>` for fallible operations
- Use `?` operator for early returns
- Avoid `unwrap()` in production code (use `expect()` with clear message if needed)
- Create custom error types with `thiserror` crate

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum TeraboxError {
    #[error("Failed to fetch URL: {0}")]
    FetchError(#[from] reqwest::Error),

    #[error("Invalid response format")]
    InvalidResponse,

    #[error("File not found")]
    NotFound,
}

pub async fn get_download_link(url: &str) -> Result<String, TeraboxError> {
    let response = reqwest::get(url).await?;
    // Process response
    Ok(download_url)
}
```

#### Async/Await
- Use `tokio` for async runtime
- Use `async fn` for async functions
- Use `.await` properly (avoid blocking operations)

```rust
use tauri::command;

#[command]
pub async fn start_download(url: String) -> Result<String, String> {
    // Async operation
    let result = fetch_url(&url).await
        .map_err(|e| e.to_string())?;
    Ok(result)
}
```

#### Tauri Commands
- Use `#[command]` attribute for exposed functions
- Return `Result<T, String>` for error handling (Tauri serializes as error)
- Use appropriate types for parameters (String, numbers, structs with Serialize)

```rust
#[tauri::command]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aruxyz/trauso](https://github.com/aruxyz/trauso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
