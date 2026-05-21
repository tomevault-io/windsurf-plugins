---
trigger: always_on
description: This described desing patterns and implementation guidance for APIs
---

---
description: This described desing patterns and implementation guidance for APIs
globs: *.ts
---
---
description:
globs:
---

# NextJS API Implementation Patterns

This document describes the patterns for implementing different types of API routes in a NextJS application with authentication levels and external service integration.

## Common Setup

- Always read [backend.md](mdc:docs/backend.md) for instructions on the backend details and [architecture.md](mdc:docs/architecture.md) for overall architecture

### Directory Structure with examples
```
app/
  api/
    # Public Routes
    languages/
      route.ts        # Public GET, authenticated POST/PATCH/DELETE
    
    # Authenticated Routes
    upload/
      route.ts        # File upload with authentication
    
    # Admin Routes
    translations/
      route.ts        # Admin-only translation management
      generate/
        route.ts      # AI-powered translation generation
    
    # AI-Enhanced Routes
    research-enhance/
      route.ts        # AI analysis with Gemini and Tavily
    tavily-search/
      route.ts        # Advanced search functionality
```

### Required Dependencies
```json
{
  "dependencies": {
    "next": "^14.0.0",
    "@supabase/supabase-js": "^2.0.0",
    "@google/generative-ai": "^0.1.0",
    "@tavily/core": "^1.0.0"
  }
}
```

## Authentication Levels

### 1. Public Routes
```typescript
// Example: GET /api/languages
export async function GET() {
  try {
    const supabase = createClient(undefined, true) // Public client
    
    const { data, error } = await supabase
      .from('table')
      .select('*')
      .eq('enabled', true)

    if (error) throw error

    return NextResponse.json({ data })
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

### 2. Authenticated Routes
```typescript
// Example: Protected route requiring authentication
export async function POST(request: Request) {
  try {
    // 1. Verify authentication
    const authHeader = request.headers.get('Authorization')
    if (!authHeader?.startsWith('Bearer ')) {
      return NextResponse.json(
        { error: 'Missing or invalid authorization header' },
        { status: 401 }
      )
    }

    // 2. Create authenticated client
    const supabase = createClient()
    
    // 3. Verify token and get user
    const { data: { user }, error: authError } = await supabase.auth.getUser(
      authHeader.split(' ')[1]
    )

    if (authError || !user) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    // 4. Process authenticated request
    const { data } = await request.json()
    
    // 5. Perform operation with user context
    const result = await supabase
      .from('table')
      .insert({ ...data, user_id: user.id })

    return NextResponse.json({ result })
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

### 3. Admin Routes
```typescript
// Example: Admin-only route
export async function PUT(request: Request) {
  try {
    // 1. Verify authentication
    const authHeader = request.headers.get('Authorization')
    if (!authHeader?.startsWith('Bearer ')) {
      return NextResponse.json(
        { error: 'Missing or invalid authorization header' },
        { status: 401 }
      )
    }

    // 2. Create authenticated client
    const authClient = createClient()
    
    // 3. Verify token and get user
    const { data: { user }, error: authError } = await authClient.auth.getUser(
      authHeader.split(' ')[1]
    )

    if (authError || !user) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    // 4. Verify admin status
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

    // 5. Use service role client for admin operations
    const supabase = createClient(undefined, true)
    
    // 6. Process admin request
    const { data } = await request.json()
    const result = await supabase
      .from('table')
      .upsert(data)

    return NextResponse.json({ result })
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

## API Route Patterns

### 1. CRUD Operations
```typescript
// GET - List/Read
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  // ... handle query parameters
}

// POST - Create
export async function POST(request: Request) {
  const body = await request.json()
  // ... handle creation
}

// PUT/PATCH - Update
export async function PUT(request: Request) {
  const body = await request.json()
  // ... handle update
}

// DELETE - Remove
export async function DELETE(request: Request) {
  const { searchParams } = new URL(request.url)
  // ... handle deletion
}
```

### 2. File Upload
```typescript
export async function POST(request: NextRequest) {
  try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastBotInc/nextjs-supabase-ai-webapp](https://github.com/LastBotInc/nextjs-supabase-ai-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
