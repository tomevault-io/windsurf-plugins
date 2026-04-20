---
trigger: always_on
description: │   ├── app/                    # Next.js App Router
---

# Web App Development Guidelines

## Next.js 14 App Router Architecture

### Project Structure
```
apps/web/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Authentication pages
│   │   │   ├── layout.tsx     # Auth layout
│   │   │   ├── login/         # Login page
│   │   │   └── register/      # Registration page
│   │   ├── (dashboard)/       # Main dashboard
│   │   │   ├── layout.tsx     # Dashboard layout
│   │   │   ├── page.tsx       # Dashboard home
│   │   │   ├── sponsees/      # Sponsee management
│   │   │   ├── shared-content/ # View shared content
│   │   │   └── settings/      # Sponsor settings
│   │   ├── api/               # API routes
│   │   │   └── trpc/          # tRPC API handler
│   │   ├── globals.css        # Global styles
│   │   ├── layout.tsx         # Root layout
│   │   └── page.tsx           # Landing page
│   ├── components/            # React components
│   │   ├── ui/               # Reusable UI components
│   │   ├── forms/            # Form components
│   │   ├── charts/           # Data visualization
│   │   └── layout/           # Layout components
│   ├── lib/                  # Utilities and configuration
│   │   ├── auth.ts           # NextAuth configuration
│   │   ├── supabase.ts       # Supabase client
│   │   ├── trpc/             # tRPC setup
│   │   └── utils.ts          # Utility functions
│   └── styles/               # Styling files
├── public/                    # Static assets
├── next.config.mjs           # Next.js configuration
└── package.json
```

### Core Technologies
- **Next.js 14**: Latest Next.js with App Router
- **TypeScript**: Strict TypeScript configuration
- **tRPC**: End-to-end typesafe APIs
- **NextAuth.js**: Authentication with Supabase adapter
- **Tailwind CSS**: Utility-first CSS framework
- **React Hook Form + Zod**: Form handling and validation
- **TanStack Query**: Server state management
- **Supabase**: Backend as a Service

## Authentication & Authorization

### NextAuth Configuration
```typescript
// src/lib/auth.ts
import NextAuth from 'next-auth';
import { SupabaseAdapter } from '@auth/supabase-adapter';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: SupabaseAdapter({
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    secret: process.env.SUPABASE_SERVICE_ROLE_KEY!,
  }),
  providers: [
    {
      id: 'supabase',
      name: 'Supabase',
      type: 'oauth',
      authorization: {
        url: `${process.env.NEXT_PUBLIC_SUPABASE_URL}/auth/v1/authorize`,
        params: {
          provider: 'email',
        },
      },
      token: `${process.env.NEXT_PUBLIC_SUPABASE_URL}/auth/v1/token`,
      userinfo: `${process.env.NEXT_PUBLIC_SUPABASE_URL}/auth/v1/user`,
      clientId: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
      clientSecret: process.env.SUPABASE_SERVICE_ROLE_KEY!,
    },
  ],
  callbacks: {
    async session({ session, user }) {
      if (session?.user) {
        session.user.id = user.id;
      }
      return session;
    },
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
      }
      return token;
    },
  },
  pages: {
    signIn: '/auth/login',
    error: '/auth/error',
  },
});
```

### Route Protection
```typescript
// src/middleware.ts
import { auth } from '@/lib/auth';
import { NextResponse } from 'next/server';

export default auth((req) => {
  const { nextUrl } = req;
  const isLoggedIn = !!req.auth;

  // Protect dashboard routes
  if (nextUrl.pathname.startsWith('/dashboard')) {
    if (!isLoggedIn) {
      return NextResponse.redirect(new URL('/auth/login', nextUrl));
    }
  }

  // Redirect logged-in users from auth pages
  if (nextUrl.pathname.startsWith('/auth') && isLoggedIn) {
    return NextResponse.redirect(new URL('/dashboard', nextUrl));
  }

  return NextResponse.next();
});

export const config = {
  matcher: ['/dashboard/:path*', '/auth/:path*'],
};
```

## tRPC Integration

### Client Setup
```typescript
// src/lib/trpc/client.ts
import { createTRPCReact } from '@trpc/react-query';
import { httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@repo/api';

export const trpc = createTRPCReact<AppRouter>();

export const trpcClient = trpc.createClient({
  links: [
    httpBatchLink({
      url: '/api/trpc',
      headers: async () => {
        const session = await auth();
        return {
          authorization: session?.user?.id ? `Bearer ${session.user.id}` : '',
        };
      },
    }),
  ],
});
```

### React Query Provider
```typescript
// src/lib/trpc/react.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';
import { trpc } from './client';

export function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 5 * 60 * 1000, // 5 minutes
        retry: (failureCount, error: any) => {
          if (error.status === 401) return false;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RipKDR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
