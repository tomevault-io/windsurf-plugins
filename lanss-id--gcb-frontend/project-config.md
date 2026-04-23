---
trigger: always_on
description: Standar Frontend (Next.js)
---

Standar Frontend (Next.js)
Struktur Direktori
Gunakan pendekatan feature-first dengan struktur sebagai berikut:
src/
├── app/                       # App router pages
│   ├── (auth)/                # Rute autentikasi (login, register, dsb)
│   ├── (nasabah)/             # Rute untuk nasabah
│   ├── (bank-sampah)/         # Rute untuk bank sampah
│   ├── (pengelola)/           # Rute untuk pengelola sampah
│   ├── (pemerintah)/          # Rute untuk pemerintah
│   ├── api/                   # API routes
│   └── layout.tsx             # Root layout
├── assets/                    # Static assets (fonts, images)
├── components/                # Shared components (Atomic Design)
│   ├── atoms/                 # Komponen dasar (button, input, text)
│   ├── molecules/             # Kombinasi atoms (form-field, card, dll)
│   ├── organisms/             # Kombinasi kompleks dari molecules (forms, lists, etc)
│   ├── templates/             # Layout templates
│   └── feature/               # Feature-specific components
│       ├── auth/              # Komponen khusus auth
│       ├── transactions/      # Komponen transaksi
│       └── waste-management/  # Komponen waste management
├── hooks/                     # Custom React hooks
├── lib/                       # Shared utilities
│   ├── api/                   # API client
│   ├── auth/                  # Auth utilities
│   ├── sync/                  # Offline sync logic
│   └── utils/                 # General utilities
├── repositories/              # Repository pattern implementation
├── store/                     # Zustand store
│   ├── slices/                # Feature-specific state slices
│   └── index.ts               # Store configuration
├── styles/                    # Global styles
└── types/                     # TypeScript definitions
Penamaan Komponen & File

Format Penamaan:

Components: PascalCase untuk nama komponen (Button.tsx, Card.tsx)
Hooks: camelCase dengan prefix "use" (useAuth.ts)
Utilities: camelCase (formatDate.ts)
Typings: PascalCase (User.ts, Transaction.ts)


Format File & Ekstensi:

Gunakan .tsx untuk file dengan JSX
Gunakan .ts untuk file JavaScript murni
Gunakan .css untuk file stylesheet global



Atomic Design Implementation

Atoms: Komponen UI dasar yang tidak dapat dipecah lagi
tsx// src/components/atoms/Button.tsx
export function Button({ children, ...props }) {
  return <button {...props}>{children}</button>;
}

Molecules: Kombinasi dari beberapa atoms
tsx// src/components/molecules/FormField.tsx
import { Label } from '../atoms/Label';
import { Input } from '../atoms/Input';

export function FormField({ label, ...props }) {
  return (
    <div className="mb-4">
      <Label>{label}</Label>
      <Input {...props} />
    </div>
  );
}

Organisms: Komponen yang lebih kompleks
tsx// src/components/organisms/LoginForm.tsx
import { FormField } from '../molecules/FormField';
import { Button } from '../atoms/Button';

export function LoginForm({ onSubmit }) {
  // Implementation
}


State Management dengan Zustand

Store Structure:

Buat slice terpisah untuk setiap domain fungsional
Gunakan persist middleware untuk data yang perlu disimpan
Gunakan devtools untuk development



typescript// src/store/slices/authSlice.ts
import { StateCreator } from 'zustand';

export interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  // other state
}

export interface AuthActions {
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  // other actions
}

export const createAuthSlice: StateCreator
  AuthState & AuthActions
> = (set, get) => ({
  // state
  user: null,
  isAuthenticated: false,

  // actions
  login: async (credentials) => {
    // implementation
  },
  logout: () => {
    // implementation
  }
});
typescript// src/store/index.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { createAuthSlice } from './slices/authSlice';
// other slices

export const useStore = create(
  devtools(
    persist(
      (...a) => ({
        ...createAuthSlice(...a),
        // other slices
      }),
      {
        name: 'greencyclebank-storage',
        partialize: (state) => ({
          // Only persist necessary state
          auth: {
            user: state.user,
            isAuthenticated: state.isAuthenticated
          }
        })
      }
    )
  )
);
API Client
Gunakan axios dengan interceptor untuk handling token dan offline detection:
typescript// src/lib/api/client.ts
import axios from 'axios';
import { syncManager } from '@/lib/sync/syncManager';

const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10000
});

apiClient.interceptors.request.use(
  async (config) => {
    // Check for network connectivity
    if (!navigator.onLine) {
      if (config.method?.toLowerCase() === 'get') {
        throw new Error('Offline mode: Fallback to cached data');
      }

      // For mutation operations, store in offline queue
      if (['post', 'put', 'patch', 'delete'].includes(config.method?.toLowerCase() || '')) {
        await syncManager.storeOfflineOperation({
          url: config.url,
          method: config.method,
          data: config.data
        });

        const offlineError = new Error('Request stored for offline processing');

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lanss-id) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
