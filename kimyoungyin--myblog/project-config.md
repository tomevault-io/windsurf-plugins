---
trigger: always_on
description: - **ALWAYS** use functional components with hooks
---

# React Patterns & Functional Programming (MANDATORY)

## ⚛️ **Component Structure (MUST FOLLOW)**

- **ALWAYS** use functional components with hooks
- **NEVER** use class components
- **MUST** use `React.FC<Props>` type annotation for components
- **ALWAYS** destructure props in function parameters

```typescript
// ✅ CORRECT - Functional component with proper typing
export const MarkdownEditor: React.FC<MarkdownEditorProps> = ({
    initialTitle = '',
    initialContent = '',
    action,
    ...props
}) => {
    // Component logic
};

// ❌ WRONG - Class component or improper typing
export class MarkdownEditor extends React.Component { ... }
```

## 🎯 **State Management (CRITICAL)**

- **ALWAYS** use `useState` for local component state
- **ALWAYS** use `useCallback` for functions passed as props
- **ALWAYS** use `useMemo` for expensive calculations
- **NEVER** mutate state directly - always use setter functions

```typescript
// ✅ CORRECT - Proper state management
const [title, setTitle] = useState(initialTitle);
const handleTitleChange = useCallback((value: string) => {
    setTitle(value);
}, []);

// ❌ WRONG - Direct mutation or missing dependencies
const handleTitleChange = (value: string) => {
    title = value; // Direct mutation
};
```

## 🚫 **FORBIDDEN React Practices**

- **NEVER** use class components
- **NEVER** mutate state or props directly
- **NEVER** create functions inside render without useCallback
- **NEVER** use useEffect without proper dependency arrays
- **NEVER** use refs for imperative DOM manipulation unless absolutely necessary

## ✅ **REQUIRED Functional Programming Practices**

- **ALWAYS** use pure functions when possible
- **ALWAYS** prefer `map`, `filter`, `reduce` over loops
- **ALWAYS** use immutable data patterns
- **ALWAYS** handle side effects in useEffect or event handlers only

## 🖥️ **Server/Client Component Patterns (CRITICAL)**

- **ALWAYS** prefer server components by default
- **ONLY** use client components when absolutely necessary (interactivity, browser APIs, state management)
- **ALWAYS** isolate client-side logic to the smallest possible component
- **NEVER** make entire pages client components unless required

### **Server Component Best Practices**

```typescript
// ✅ CORRECT - Server component with async data fetching
export default async function PostPage({ params }: PostPageProps) {
    const post = await getPostAction(postId);
    
    return (
        <div>
            <h1>{post.title}</h1>
            <MarkdownRenderer content={post.content} />
            <ToEditButton postId={postId} /> {/* Client component */}
        </div>
    );
}
```

### **Client Component Isolation**

```typescript
// ✅ CORRECT - Minimal client component for interactive features only
'use client';

export default function ToEditButton({ postId }: { postId: number }) {
    const { user } = useAuthStore(); // Client-side state only
    
    if (!user?.is_admin) return null;
    
    return (
        <Button asChild>
            <Link href={`/admin/posts/${postId}/edit`}>
                <Edit className="h-4 w-4" />
                수정
            </Link>
        </Button>
    );
}
```

### **When to Use Client Components**

- **✅ USE** for interactive elements (buttons, forms, dropdowns)
- **✅ USE** for browser APIs (localStorage, window, document)
- **✅ USE** for state management (useState, useReducer)
- **✅ USE** for event handlers (onClick, onChange)
- **❌ DON'T USE** for static content rendering
- **❌ DON'T USE** for data fetching (use server actions instead)

### **Component Composition Pattern**

```typescript
// ✅ CORRECT - Server component with embedded client components
export default function PostPage() {
    return (
        <div>
            {/* Server-rendered content */}
            <PostContent post={post} />
            
            {/* Client-side interactivity only */}
            <LikeButton postId={post.id} />
            <CommentSection postId={post.id} />
        </div>
    );
}
```

## 🏗️ **HTML Structure & Hydration Prevention (CRITICAL)**

### **1. Block vs Inline Element Rules**

- **NEVER** put block-level elements inside inline elements
- **NEVER** put `<div>`, `<section>`, `<article>` inside `<p>`, `<span>`, `<a>`
- **ALWAYS** ensure proper HTML nesting hierarchy

```typescript
// ✅ CORRECT - Proper HTML structure
<span className="block">
  <div className="relative">
    <Image src={src} alt={alt} />
  </div>
</span>

// ❌ WRONG - Block element inside inline element
<p>
  <div>  {/* This causes hydration error */}
    <Image src={src} alt={alt} />
  </div>
</p>
```

### **2. Markdown Rendering Safety**

- **ALWAYS** override default markdown component behavior
- **ALWAYS** prevent invalid HTML nesting
- **ALWAYS** use custom components for images

```typescript
// ✅ CORRECT - Safe markdown rendering
export const SafeMarkdownRenderer = ({ content }: { content: string }) => {
  return (
    <ReactMarkdown
      components={{
        // Prevent p tag wrapping for images
        img: ({ src, alt }) => (
          <span className="block my-4">
            <Image src={src} alt={alt} width={800} height={600} />
          </span>
        ),
        // Override p tag for image-only content
        p: ({ children, ...props }) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
