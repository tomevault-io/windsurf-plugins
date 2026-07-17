---
trigger: always_on
description: **在 Next.js 15 中，所有 `params` 和 `searchParams` 都是 Promise 类型！**
---

# PixPaw AI - Cursor 代码生成规则（强制执行）

## 🚨 Next.js 15 关键规范（部署前必查）

### 1. API 路由 & 动态路由参数（Critical）
**在 Next.js 15 中，所有 `params` 和 `searchParams` 都是 Promise 类型！**

#### ✅ 正确写法（必须遵循）
```typescript
// API 路由 (app/api/*/route.ts)
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }  // ✅ Promise 包裹
) {
  const { id } = await params;  // ✅ 必须 await
  // ... 业务逻辑
}

export async function POST(
  request: Request,
  { params }: { params: Promise<{ slug: string }> }  // ✅ Promise
) {
  const { slug } = await params;  // ✅ await
}

// 页面组件 (app/[lang]/*/page.tsx)
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ lang: string; id?: string }>;      // ✅ Promise
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;  // ✅ Promise
}) {
  const { lang, id } = await params;           // ✅ await
  const search = await searchParams;           // ✅ await
  // ... 渲染逻辑
}

// generateMetadata
export async function generateMetadata({
  params,
}: {
  params: Promise<{ lang: string }>;  // ✅ Promise
}): Promise<Metadata> {
  const { lang } = await params;  // ✅ await
  return { title: `PixPaw AI - ${lang}` };
}
```

#### ❌ 错误写法（会导致部署失败）
```typescript
// ❌ 不要这样写！
export async function GET(
  request: Request,
  { params }: { params: { id: string } }  // ❌ 缺少 Promise
) {
  const { id } = params;  // ❌ 直接解构
}

// ❌ 不要这样写！
export default async function Page({
  params,
}: {
  params: { lang: string };  // ❌ 缺少 Promise
}) {
  const { lang } = params;  // ❌ 直接使用
}
```

### 2. 强制规则检查清单
在生成以下类型的代码时，**必须**检查：
- [ ] `app/api/**/route.ts` - 所有 API 路由的 params 是 Promise
- [ ] `app/[lang]/**/page.tsx` - 所有页面的 params 和 searchParams 是 Promise
- [ ] `generateMetadata` 函数 - params 是 Promise
- [ ] `generateStaticParams` 的返回值正确

### 3. TypeScript 严格类型
```typescript
// 为常用路由定义类型（推荐）
type ApiRouteContext<T = {}> = {
  params: Promise<T>;
};

type PageProps<P = {}, S = {}> = {
  params: Promise<P>;
  searchParams: Promise<S>;
};

// 使用示例
export async function GET(
  request: Request,
  context: ApiRouteContext<{ id: string }>
) {
  const { id } = await context.params;
}
```

---

## 🔐 安全规范（强制执行）

### 1. 数据库安全（RLS）
**所有新建表必须启用 Row Level Security！**

```sql
-- ✅ 正确：创建表后立即启用 RLS
CREATE TABLE my_table (...);

ALTER TABLE my_table ENABLE ROW LEVEL SECURITY;

-- 添加策略确保用户只能访问自己的数据
CREATE POLICY "Users can only access their own data"
ON my_table
FOR ALL
USING (auth.uid() = user_id);
```

### 2. API 密钥管理
**严禁硬编码任何密钥！**

```typescript
// ✅ 正确：从环境变量读取
const apiKey = process.env.REPLICATE_API_TOKEN;

// ❌ 错误：硬编码
const apiKey = "r8_abc123..."; // 永远不要这样做！

// ✅ 服务端密钥：不加 NEXT_PUBLIC_ 前缀
process.env.SUPABASE_SERVICE_ROLE_KEY

// ✅ 客户端公开变量：加 NEXT_PUBLIC_ 前缀
process.env.NEXT_PUBLIC_SUPABASE_URL
```

### 3. 竞态条件保护
**涉及金额、库存、积分的操作必须使用数据库事务！**

```sql
-- ✅ 正确：使用 FOR UPDATE 锁
CREATE FUNCTION process_payment(p_payment_id UUID) AS $$
BEGIN
  -- 锁定行防止并发
  PERFORM * FROM payments WHERE id = p_payment_id FOR UPDATE;
  
  -- 检查状态
  IF EXISTS (SELECT 1 FROM payments WHERE id = p_payment_id AND status = 'completed') THEN
    RAISE EXCEPTION 'Already processed';
  END IF;
  
  -- 原子操作
  UPDATE profiles SET credits = credits + amount WHERE ...;
  UPDATE payments SET status = 'completed' WHERE ...;
END;
$$ LANGUAGE plpgsql;
```

```typescript
// ✅ 正确：在 API 中检查重复处理
if (payment.status === 'completed') {
  return NextResponse.json({
    success: true,
    message: 'Payment already processed',
    alreadyCompleted: true,
  });
}
```

### 4. Rate Limiting
**所有公开 API 端点必须考虑速率限制！**

- 上传端点：5 次/分钟
- 生成端点：3 次/分钟  
- 支付端点：10 次/小时
- 参考：`docs/RATE_LIMITING_IMPLEMENTATION.md`

### 5. 输入验证
**永远不要信任用户输入！**

```typescript
// ✅ 正确：验证所有输入
const { orderId } = await request.json();

if (!orderId || typeof orderId !== 'string') {
  return NextResponse.json({ error: 'Invalid order ID' }, { status: 400 });
}

// 验证 UUID 格式
const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
if (!uuidRegex.test(orderId)) {
  return NextResponse.json({ error: 'Invalid UUID format' }, { status: 400 });
}
```

---

## 📐 项目架构规范

### SEO & SSR（搜索引擎优化）
1. **服务端渲染优先**
   - 所有动态页面必须使用 `async` 组件
   - 主要内容必须在服务端获取（不使用 `useEffect` 获取首屏数据）
   
2. **Metadata 生成**
   ```typescript
   export async function generateMetadata({ params }: {
     params: Promise<{ lang: string }>;  // ✅ Promise
   }): Promise<Metadata> {
     const { lang } = await params;
     return {
       title: "PixPaw AI - Turn Any Pet Photo Into Stunning Art",
       description: "...",
       openGraph: { ... },
     };
   }
   ```

3. **URL 结构**
   - 使用语义化 slug：`/pet/123-golden-retriever-stylized-art`
   - 避免查询参数作为主要导航：`/pet?id=123` ❌

### 安全规范
1. **API 密钥保护**
   ```typescript
   // ✅ 正确：在服务端使用
   const REPLICATE_API_TOKEN = process.env.REPLICATE_API_TOKEN;
   
   // ❌ 错误：不要在客户端组件中直接使用
   'use client'
   const token = process.env.REPLICATE_API_TOKEN;  // ❌ 会暴露
   ```

2. **环境变量命名**
   - 公开变量：`NEXT_PUBLIC_SUPABASE_URL`
   - 私密变量：`SUPABASE_SERVICE_ROLE_KEY` (无 NEXT_PUBLIC_ 前缀)

### UI/UX 规范
1. **响应式设计（Mobile-First）**
   ```tsx
   <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
     {/* 手机1列，平板2列，桌面3列 */}
   </div>
   ```

2. **加载状态**
   - 使用情感化文案：`"Sprinkling magic dust..."` ✅
   - 避免：`"Loading..."` ❌


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alextangson/PixPawAI](https://github.com/alextangson/PixPawAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
