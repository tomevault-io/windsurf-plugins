---
trigger: always_on
description: - **NEVER** put block-level elements inside inline elements
---

# HTML Structure & Hydration Error Prevention (MANDATORY)

## 🚨 **Critical HTML Structure Rules (MUST FOLLOW)**

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

### **2. Markdown Rendering HTML Structure**

- **ALWAYS** handle images as block-level elements in markdown
- **NEVER** let markdown parser wrap images in `<p>` tags
- **ALWAYS** use custom components to override default markdown behavior

```typescript
// ✅ CORRECT - Custom image rendering that prevents p tag wrapping
components={{
  img: ({ src, alt, ...props }) => {
    return (
      <span className="block my-4">
        <Image src={src} alt={alt} {...props} />
      </span>
    );
  },
  // Override p tag rendering for images
  p: ({ children, ...props }) => {
    const hasOnlyImage = React.Children.count(children) === 1 && 
      React.isValidElement(children) && 
      children.type === 'img';
    
    if (hasOnlyImage) {
      return <div {...props}>{children}</div>; // Use div instead of p
    }
    return <p {...props}>{children}</p>;
  }
}}
```

## 🔧 **Hydration Error Prevention (CRITICAL)**

### **1. Server/Client Component Consistency**

- **ALWAYS** ensure server and client render identical HTML
- **NEVER** use browser-only APIs in server components
- **ALWAYS** handle dynamic content with proper client boundaries

```typescript
// ✅ CORRECT - Proper client boundary for dynamic content
'use client';

export const DynamicImage = ({ src, alt }: { src: string; alt: string }) => {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <div className="relative">
      <Image 
        src={src} 
        alt={alt}
        onLoad={() => setIsLoaded(true)}
      />
      {!isLoaded && <div className="loading-placeholder" />}
    </div>
  );
};

// ❌ WRONG - Browser API in server component
export const ServerComponent = () => {
  const [windowSize, setWindowSize] = useState({}); // Hydration error!
  return <div>{windowSize.width}</div>;
};
```

### **2. Conditional Rendering Best Practices**

- **ALWAYS** use consistent conditional rendering patterns
- **NEVER** mix server and client conditional logic
- **ALWAYS** handle loading states properly

```typescript
// ✅ CORRECT - Consistent conditional rendering
export const PostCard = ({ post, showThumbnail = true }) => {
  return (
    <Card>
      {showThumbnail && post.thumbnail_url ? (
        <div className="thumbnail-container">
          <Image src={post.thumbnail_url} alt={post.title} />
        </div>
      ) : showThumbnail && !post.thumbnail_url ? (
        <div className="placeholder-container">
          <div className="placeholder-image" />
        </div>
      ) : null}
    </Card>
  );
};

// ❌ WRONG - Inconsistent conditional rendering
export const BadPostCard = ({ post }) => {
  return (
    <Card>
      {post.thumbnail_url && (
        <div className="thumbnail">
          <Image src={post.thumbnail_url} alt={post.title} />
        </div>
      )}
      {/* Missing else case - can cause hydration mismatch */}
    </Card>
  );
};
```

## 🎯 **Component Structure Guidelines**

### **1. Image Component Best Practices**

- **ALWAYS** use Next.js Image component for optimization
- **ALWAYS** provide proper width and height props
- **NEVER** use onError/onLoad with Next.js Image (not supported)
- **ALWAYS** handle image loading states with CSS or state

```typescript
// ✅ CORRECT - Next.js Image with proper structure
export const OptimizedImage = ({ src, alt, className }: ImageProps) => {
  return (
    <div className="image-container">
      <Image
        src={src}
        alt={alt}
        width={800}
        height={600}
        className={className}
        style={{
          display: 'block',
          maxWidth: '100%',
          height: 'auto',
        }}
      />
    </div>
  );
};

// ❌ WRONG - Unsupported events with Next.js Image
<Image
  src={src}
  alt={alt}
  onError={handleError}  // Not supported!
  onLoad={handleLoad}    // Not supported!
/>
```

### **2. Markdown Component Structure**

- **ALWAYS** override default markdown component behavior
- **ALWAYS** prevent invalid HTML nesting
- **ALWAYS** use semantic HTML elements appropriately

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
          const hasImage = React.Children.toArray(children).some(
            child => React.isValidElement(child) && child.type === 'img'
          );

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
