---
trigger: always_on
description: - **MUST:** Create typed context with Supabase client and user authentication
---

# tRPC Implementation Guide

## **Server Architecture Patterns**

### **Context Setup**
- **MUST:** Create typed context with Supabase client and user authentication
- **MUST:** Export context type for use in procedures

```typescript
// ✅ DO: Proper context setup with authentication
export async function createTRPCContext() {
  const supabase = await createClient();
  const { data: { user }, error: userError } = await supabase.auth.getUser();
  
  return {
    supabase,
    user,
    userError,
  };
}

export type Context = Awaited<ReturnType<typeof createTRPCContext>>;
```

### **Procedure Hierarchy**
- **MUST:** Define clear procedure hierarchy: `publicProcedure` → `protectedProcedure` → `adminProcedure`
- **MUST:** Use middleware for authentication and authorization checks
- **CRITICAL:** Ensure auth users exist in custom DB tables before allowing operations with foreign key constraints

```typescript
// ✅ DO: Layered procedure security with DB user synchronization
export const protectedProcedure = t.procedure.use(async (opts) => {
  const { user, supabase } = opts.ctx;
  
  if (!user) {
    throw new TRPCError({
      code: 'UNAUTHORIZED',
      message: '로그인이 필요합니다.',
    });
  }

  // CRITICAL: Ensure user exists in custom users table for foreign key integrity
  const { data: dbUser, error: userCheckError } = await supabase
    .from('users')
    .select('id')
    .eq('id', user.id)
    .single();

  if (userCheckError && userCheckError.code === 'PGRST116') {
    // User doesn't exist in users table, create it
    const { error: createError } = await supabase
      .from('users')
      .insert({
        id: user.id,
        email: user.email,
        name: user.user_metadata?.name || user.email?.split('@')[0],
        auth_provider: user.app_metadata?.provider || 'email',
      });

    if (createError) {
      console.error('Failed to create user record:', createError);
      throw new TRPCError({
        code: 'INTERNAL_SERVER_ERROR',
        message: '사용자 정보 생성 중 오류가 발생했습니다.',
      });
    }
  } else if (userCheckError) {
    console.error('User check error:', userCheckError);
    throw new TRPCError({
      code: 'INTERNAL_SERVER_ERROR',
      message: '사용자 확인 중 오류가 발생했습니다.',
    });
  }
  
  return opts.next({
    ctx: { ...opts.ctx, user },
  });
});
```

### **Auth-DB Synchronization Rules**
- **CRITICAL:** Always ensure Supabase Auth users have corresponding records in custom users table
- **MUST:** Handle the gap between auth.users and custom users table in protectedProcedure
- **MUST:** Auto-create missing user records when foreign key constraints require them
- **MUST NOT:** Assume auth user existence guarantees DB user record existence

```typescript
// ❌ DON'T: Assume auth user exists in custom tables
export const badProtectedProcedure = t.procedure.use(async (opts) => {
  const { user } = opts.ctx;
  if (!user) throw new TRPCError({ code: 'UNAUTHORIZED' });
  
  // This will fail if user doesn't exist in custom users table
  return opts.next({ ctx: { ...opts.ctx, user } });
});

// ✅ DO: Verify and sync user records before operations
// See the improved protectedProcedure example above
```

## **Domain Router Structure**

### **Router Organization**
- **MUST:** Separate routers by domain ([template.ts](mdc:src/lib/trpc/routers/template.ts), [version.ts](mdc:src/lib/trpc/routers/version.ts))
- **MUST:** Group related procedures within domain routers
- **MUST:** Export domain routers and combine in [root.ts](mdc:src/lib/trpc/root.ts)

```typescript
// ✅ DO: Domain-specific router structure
export const templateRouter = router({
  getAll: publicProcedure.query(/* ... */),
  getById: publicProcedure.input(/* ... */).query(/* ... */),
  create: protectedProcedure.input(/* ... */).mutation(/* ... */),
  update: protectedProcedure.input(/* ... */).mutation(/* ... */),
  delete: protectedProcedure.input(/* ... */).mutation(/* ... */),
  getMine: protectedProcedure.query(/* ... */),
});
```

### **Procedure Naming Conventions**
- **MUST:** Use clear, RESTful naming: `getAll`, `getById`, `create`, `update`, `delete`
- **MUST:** Add domain-specific procedures like `getMine`, `restore`, `compare`
- **MUST NOT:** Mix CRUD and business logic procedures in unclear names

## **Input Validation & Schemas**

### **Zod Schema Patterns**
- **MUST:** Define schemas at the top of router files
- **MUST:** Use descriptive error messages in Korean for user-facing errors
- **MUST:** Separate input/output schemas for different operations

```typescript
// ✅ DO: Clear schema definition with localized errors
const createTemplateSchema = z.object({
  name: z.string().min(1, '템플릿 이름은 필수입니다'),
  description: z.string().optional(),
  content: z.string().min(1, '템플릿 내용은 필수입니다'),
  fields: z.array(templateFieldSchema).optional(),
});

const templateIdSchema = z.object({
  id: z.string().uuid(),
});
```

### **Schema Reusability**
- **MUST:** Extract common schemas (like ID validation) for reuse
- **MUST:** Build complex schemas from simpler base schemas
- **MUST NOT:** Duplicate validation logic across procedures

## **Error Handling Patterns**

### **Consistent Error Structure**
- **MUST:** Use TRPCError with appropriate HTTP status codes
- **MUST:** Provide user-friendly Korean error messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greatSumini/document-parser](https://github.com/greatSumini/document-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
