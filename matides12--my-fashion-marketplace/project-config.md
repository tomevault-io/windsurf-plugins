---
trigger: always_on
description: “master instruction” set up a Next.js (TypeScript) + Supabase project. It includes steps to create a Next.js app, set up Supabase, configure authentication, create the core database schema, and build initial pages (login, sign up, dashboard, product listing, etc.).
---


# Your rule content

─────────────────────────────────────────────────────────
MASTER INSTRUCTION FOR CURSOR AI
─────────────────────────────────────────────────────────
1. CREATE A NEW NEXT.JS PROJECT WITH TYPESCRIPT  
   - Use the following command (or equivalent in Cursor AI) to initialize the project:  
     npx create-next-app@latest my-fashion-marketplace --ts

2. SET UP SUPABASE  
   a) Install the Supabase client:  
      npm install @supabase/supabase-js  
   b) Create a Supabase account (if not already done) at supabase.com, then set up a new project.  
   c) In the Supabase dashboard, create the following tables (users, products, product_images, orders, comments, likes) using SQL or the table editor:

      --------------------------------------------------
      CREATE TABLE IF NOT EXISTS users (
        id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        full_name   text,
        email       text UNIQUE NOT NULL,
        -- password is handled by Supabase Auth, but you can store additional data here
        role        text CHECK (role IN ('owner', 'customer')) DEFAULT 'customer',
        created_at  timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS products (
        id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        owner_id    uuid REFERENCES users(id) ON DELETE CASCADE,
        title       text NOT NULL,
        description text,
        price       numeric(10,2),
        is_active   boolean DEFAULT true,
        created_at  timestamptz DEFAULT now(),
        updated_at  timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS product_images (
        id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        product_id      uuid REFERENCES products(id) ON DELETE CASCADE,
        image_url       text NOT NULL,
        is_model_picture boolean DEFAULT false
      );

      CREATE TABLE IF NOT EXISTS orders (
        id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        user_id       uuid REFERENCES users(id) ON DELETE CASCADE,
        product_id    uuid REFERENCES products(id) ON DELETE CASCADE,
        quantity      int DEFAULT 1,
        total_price   numeric(10,2),
        order_status  text CHECK (order_status IN ('pending','confirmed','shipped','delivered','cancelled'))
                     DEFAULT 'pending',
        created_at    timestamptz DEFAULT now(),
        updated_at    timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS comments (
        id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        user_id     uuid REFERENCES users(id) ON DELETE CASCADE,
        product_id  uuid REFERENCES products(id) ON DELETE CASCADE,
        comment_text text,
        created_at  timestamptz DEFAULT now()
      );

      CREATE TABLE IF NOT EXISTS likes (
        id         uuid PRIMARY KEY DEFAULT gen_random_uuid(),
        user_id    uuid REFERENCES users(id) ON DELETE CASCADE,
        product_id uuid REFERENCES products(id) ON DELETE CASCADE,
        created_at timestamptz DEFAULT now()
      );
      --------------------------------------------------

   d) Enable Supabase Auth:
      - In the “Authentication” settings, configure email/password signups (or social providers as needed).

3. CONFIGURE ENVIRONMENT VARIABLES  
   - In the newly created Next.js project, create a .env.local file (which is ignored by Git) for storing your Supabase URL and public anon key. For example:
     --------------------------------------------------
     NEXT_PUBLIC_SUPABASE_URL=<your-supabase-url>
     NEXT_PUBLIC_SUPABASE_ANON_KEY=<your-supabase-anon-key>
     --------------------------------------------------

4. CREATE A SUPABASE CLIENT UTILITY (supabase.ts)  
   - In your project, create a file (e.g., lib/supabase.ts) with the following code:
     --------------------------------------------------
     // lib/supabase.ts
     import { createClient } from '@supabase/supabase-js';

     const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
     const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

     export const supabase = createClient(supabaseUrl, supabaseAnonKey);
     --------------------------------------------------

5. SET UP AUTH PAGES (LOGIN, SIGNUP)  
   - In the pages directory, create pages like /login.tsx and /signup.tsx.  
   - Use Supabase client for signIn and signUp. For example (pseudo-code):
     --------------------------------------------------
     // pages/login.tsx
     import { useState } from 'react';
     import { supabase } from '../lib/supabase';

     export default function LoginPage() {
       const [email, setEmail] = useState('');
       const [password, setPassword] = useState('');

       async function handleLogin() {
         const { error } = await supabase.auth.signInWithPassword({
           email,
           password,
         });
         if (error) console.error(error.message);
         else window.location.href = '/dashboard';
       }

       return (
         <div>
           <h1>Login</h1>
           <input

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MatiDes12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
