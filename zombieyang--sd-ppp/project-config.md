---
trigger: always_on
description: Provides type safety when using `useStore` with vanilla stores.
---

---
description: This rule provides guidelines for using Zustand, a simple and unopinionated state management library, in React applications. It covers best practices for code organization, performance optimization, testing, and common pitfalls to avoid.
globs: **/*.{js,jsx,ts,tsx}
---
# Zustand Best Practices

This document outlines best practices for using Zustand in your React applications. Zustand is a simple and unopinionated state management library. Following these guidelines will help you write maintainable, performant, and scalable applications.

## 1. Code Organization and Structure

### 1.1. Directory Structure

Organize your store files in a dedicated directory, such as `store` or `state`, at the root of your project or within a specific feature directory. This enhances discoverability and maintainability.


src/
├── components/
│   ├── ...
├── store/
│   ├── index.ts          # Main store file (optional)
│   ├── bearStore.ts      # Example store
│   ├── fishStore.ts      # Example store
│   └── utils.ts         # Utility functions for stores
├── App.tsx
└── ...


### 1.2. File Naming Conventions

Use descriptive names for your store files, typically reflecting the domain or feature the store manages. For example, `userStore.ts`, `cartStore.js`, or `settingsStore.tsx`.  Use PascalCase for the store name itself (e.g., `UserStore`).

### 1.3. Module Organization

- **Single Store per File:**  Prefer defining one Zustand store per file.  This improves readability and maintainability.
- **Slices Pattern:** For complex stores, consider using the slices pattern to divide the store into smaller, more manageable pieces.  Each slice manages a specific part of the state and its related actions.

typescript
// store/bearStore.ts
import { StateCreator, create } from 'zustand';

interface BearSlice {
  bears: number;
  addBear: () => void;
}

const createBearSlice: StateCreator<BearSlice> = (set) => ({
  bears: 0,
  addBear: () => set((state) => ({ bears: state.bears + 1 })),
});

export const useBearStore = create<BearSlice>()((...a) => ({
  ...createBearSlice(...a),
}));

// Another slice could be in fishStore.ts, etc.


### 1.4. Component Architecture

- **Presentational and Container Components:**  Separate presentational (UI) components from container components that interact with the Zustand store. Container components fetch data from the store and pass it down to presentational components.
- **Hooks for Data Fetching:** Utilize Zustand's `useStore` hook within container components to subscribe to specific parts of the state.

### 1.5. Code Splitting Strategies

- **Lazy Loading Stores:**  Load stores on demand using dynamic imports.  This can reduce the initial bundle size, especially for larger applications.
- **Feature-Based Splitting:** Split your application into feature modules and create separate stores for each feature.  This allows for independent loading and reduces dependencies between different parts of the application.

## 2. Common Patterns and Anti-patterns

### 2.1. Design Patterns Specific to Zustand

- **Colocated Actions and State:**  Keep actions and the state they modify within the same store. This promotes encapsulation and makes it easier to understand how the store's state is updated.
- **Selectors:** Use selectors to derive computed values from the store's state. Selectors should be memoized to prevent unnecessary re-renders.

typescript
// store/userStore.ts
import { create } from 'zustand';

interface UserState {
  name: string;
  age: number;
}

interface UserActions {
  setName: (name: string) => void;
  isAdult: () => boolean; // Selector
}

export const useUserStore = create<UserState & UserActions>((set, get) => ({
  name: 'John Doe',
  age: 20,
  setName: (name) => set({ name }),
  isAdult: () => get().age >= 18, // Selector
}));


### 2.2. Recommended Approaches for Common Tasks

- **Asynchronous Actions:** Use `async/await` within actions to handle asynchronous operations such as fetching data from an API.

typescript
interface DataState {
  data: any | null;
  isLoading: boolean;
  fetchData: () => Promise<void>;
}

export const useDataStore = create<DataState>((set) => ({
  data: null,
  isLoading: false,
  fetchData: async () => {
    set({ isLoading: true });
    try {
      const response = await fetch('https://api.example.com/data');
      const data = await response.json();
      set({ data, isLoading: false });
    } catch (error) {
      console.error('Error fetching data:', error);
      set({ isLoading: false, data: null });
    }
  },
}));


- **Persisting State:** Use the `zustand/middleware`'s `persist` middleware to persist the store's state to local storage or another storage mechanism.  Configure a `partialize` function to select the state you want to persist.

typescript
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface AuthState {
  token: string | null;
  setToken: (token: string | null) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      setToken: (token) => set({ token }),
    }),
    {
      name: 'auth-storage', // unique name
      partialize: (state) => ({ token: state.token }), // Only persist the token
    }
  )
)



<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zombieyang/sd-ppp](https://github.com/zombieyang/sd-ppp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
