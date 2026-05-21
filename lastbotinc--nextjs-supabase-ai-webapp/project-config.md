---
trigger: always_on
description: Instructions for implementing Admin API protection
---

---
description: Instructions for implementing Admin API protection
globs: /app/api/*/*.ts
---
# Admin API Implementation Pattern

## Context
In web applications, certain operations need to be restricted to admin users only. This pattern provides a complete implementation guide for admin-only APIs in a Next.js application using Supabase authentication.

## Implementation Guide

### 1. Database Setup

1. **Admin Flag in Profiles Table**
   ```sql
   create table public.profiles (
     id uuid references auth.users primary key,
     is_admin boolean default false,
     -- other fields...
   );
   ```

2. **Row Level Security (RLS) Policies**
   ```sql
   -- Only admins can view all profiles
   create policy "Admins can view all profiles"
   on public.profiles for select
   to authenticated
   using (
     auth.uid() in (
       select id from public.profiles
       where is_admin = true
     )
   );
   ```

### 2. Server-Side Implementation

1. **Create Admin API Route**
   ```typescript
   // app/api/[admin-route]/route.ts
   import { NextResponse } from 'next/server'
   import { createClient } from '@/utils/supabase/server'

   export async function POST(request: Request) {
     try {
       // 1. Token Verification Layer
       const authHeader = request.headers.get('Authorization')
       if (!authHeader?.startsWith('Bearer ')) {
         return NextResponse.json(
           { error: 'Missing or invalid authorization header' },
           { status: 401 }
         )
       }

       // Create regular client to verify the token
       const authClient = createClient()
       const { data: { user }, error: authError } = await authClient.auth.getUser(authHeader.split(' ')[1])
       
       if (authError || !user) {
         return NextResponse.json(
           { error: 'Unauthorized' },
           { status: 401 }
         )
       }

       // 2. Admin Role Verification Layer
       const { data: profile } = await authClient
         .from('profiles')
         .select('is_admin')
         .eq('id', user.id)
         .single()

       if (!profile?.is_admin) {
         return NextResponse.json(
           { error: 'Forbidden - Admin access required' },
           { status: 403 }
         )
       }

       // 3. Service Role Operations Layer
       const supabase = createClient(undefined, true) // Create service role client
       
       // Get request data
       const data = await request.json()
       
       // Perform admin operations with service role client
       const { data: result, error: operationError } = await supabase
         .from('your_table')
         .insert(data)
         .select()
         .single()

       if (operationError) throw operationError
       
       return NextResponse.json({ data: result })
     } catch (err) {
       console.error('Error:', err)
       return NextResponse.json(
         { error: err instanceof Error ? err.message : 'Internal server error' },
         { status: 500 }
       )
     }
   }
   ```

### 3. Client-Side Implementation

1. **Admin Component with Auth Check**
   ```typescript
   // components/admin/AdminComponent.tsx
   'use client'
   
   import { useEffect, useState } from 'react'
   import { createClient } from '@/utils/supabase/client'
   import { useRouter } from 'next/navigation'

   export default function AdminComponent() {
     const [isAdmin, setIsAdmin] = useState<boolean>(false)
     const [loading, setLoading] = useState(true)
     const supabase = createClient()
     const router = useRouter()

     useEffect(() => {
       async function checkAdminStatus() {
         try {
           const { data: { session } } = await supabase.auth.getSession()
           if (!session) {
             router.push('/login')
             return
           }

           const { data: profile } = await supabase
             .from('profiles')
             .select('is_admin')
             .eq('id', session.user.id)
             .single()

           if (!profile?.is_admin) {
             router.push('/unauthorized')
             return
           }

           setIsAdmin(true)
         } catch (error) {
           console.error('Error checking admin status:', error)
           router.push('/error')
         } finally {
           setLoading(false)
         }
       }

       checkAdminStatus()
     }, [supabase, router])

     if (loading) return <div>Loading...</div>
     if (!isAdmin) return null

     return (
       // Your admin UI here
     )
   }
   ```

2. **Admin API Client**
   ```typescript
   // utils/adminApi.ts
   import { createClient } from '@/utils/supabase/client'

   export class AdminApi {
     private static async getAuthHeader() {
       const supabase = createClient()
       const { data: { session } } = await supabase.auth.getSession()
       if (!session?.access_token) throw new Error('Not authenticated')
       return `Bearer ${session.access_token}`
     }

     static async callAdminApi(endpoint: string, data: any) {
       try {
         const authHeader = await this.getAuthHeader()
         
         const response = await fetch(`/api/${endpoint}`, {
           method: 'POST',
           headers: {
             'Authorization': authHeader,
             'Content-Type': 'application/json'
           },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastBotInc/nextjs-supabase-ai-webapp](https://github.com/LastBotInc/nextjs-supabase-ai-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
