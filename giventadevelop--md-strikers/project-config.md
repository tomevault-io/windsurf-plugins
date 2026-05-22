---
trigger: always_on
description: CSS rules and patterns for displaying images without cropping or truncation
---


# Image Containment Prevention Pattern

## **Overview**
This rule defines the correct CSS patterns for displaying banner, hero, and sponsor images that must be fully visible without cropping or truncation on any side. This pattern ensures images maintain their aspect ratio and are contained within their containers.

## **Problem Solved**
- **Image Cropping**: Prevents images from being cut off on left, right, top, or bottom edges
- **Aspect Ratio Preservation**: Maintains original image proportions
- **Consistent Display**: Ensures images display the same way across different screen sizes
- **User Experience**: Users can see complete images without important content being hidden

## **Core Pattern**

### **Container Styling**
```tsx
// ✅ DO: Use flexible height container
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {/* Image content */}
</div>

// ❌ DON'T: Use fixed height container
<div className="relative w-full h-[448px] rounded-t-2xl overflow-hidden">
  {/* This causes cropping */}
</div>
```

### **Image Styling**
```tsx
// ✅ DO: Use object-contain with flexible dimensions
<Image
  src={imageUrl}
  alt="Description"
  width={800}
  height={600}
  className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
  style={{
    backgroundColor: 'transparent',
    borderRadius: '1rem 1rem 0 0'
  }}
/>

// ❌ DON'T: Use object-cover with fixed dimensions
<Image
  src={imageUrl}
  alt="Description"
  width={800}
  height={600}
  className="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
  style={{
    borderRadius: '1rem 1rem 0 0'
  }}
/>
```

## **Key CSS Properties**

### **Container Requirements**
- **`relative`**: Enables absolute positioning of child elements (badges, overlays)
- **`w-full`**: Full width of parent container
- **`h-auto`**: **CRITICAL**: Flexible height allows image to maintain aspect ratio
- **`rounded-t-2xl`**: Rounded top corners (1rem radius)
- **`overflow-hidden`**: Clips content to rounded corners

### **Image Requirements**
- **`w-full`**: Full width of container
- **`h-auto`**: **CRITICAL**: Flexible height maintains aspect ratio
- **`object-contain`**: **CRITICAL**: Shows complete image without cropping (alternative to `object-cover`)
- **`group-hover:scale-105`**: Optional hover effect for interactivity
- **`transition-transform duration-300`**: Smooth hover animation
- **`backgroundColor: 'transparent'`**: Allows card gradient to show through
- **`borderRadius: '1rem 1rem 0 0'`**: Matches container's rounded corners

## **When to Use This Pattern**

### **Use `object-contain` When:**
- ✅ Banner images (sponsor banners, event banners)
- ✅ Hero images that must show complete content
- ✅ Logo images that need full visibility
- ✅ Images where cropping would hide important content
- ✅ Images in cards or containers where aspect ratio matters

### **Use `object-cover` When:**
- ✅ Background images where cropping is acceptable
- ✅ Thumbnail images where partial visibility is fine
- ✅ Decorative images where content isn't critical
- ✅ Images in fixed-size containers where filling space is priority

## **Complete Example Pattern**

```tsx
{/* Image Container - Matching events page style */}
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {imageUrl ? (
    <Image
      src={imageUrl}
      alt="Image description"
      width={800}
      height={600}
      className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    />
  ) : (
    <div
      className="w-full h-80 flex items-center justify-center"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    >
      <span className="text-gray-400 text-5xl">🏢</span>
    </div>
  )}
</div>
```

## **Reference Implementations**

### **Events Page Pattern**
See [`src/app/events/page.tsx`](mdc:src/app/events/page.tsx) lines 771-783 for the canonical implementation:
```tsx
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {event.thumbnailUrl ? (
    <Image
      src={event.thumbnailUrl}
      alt={event.title}
      width={800}
      height={600}
      className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    />
  ) : (
    {/* Placeholder */}
  )}
</div>
```

### **Sponsor Images Pattern**
See [`src/app/events/[id]/page.tsx`](mdc:src/app/events/[id]/page.tsx) lines 964-991 for sponsor banner images:
```tsx
<div className="relative w-full h-auto rounded-t-2xl overflow-hidden">
  {displayImageUrl ? (
    <Image
      src={displayImageUrl}
      alt={sponsor.name}
      width={800}
      height={600}
      className="w-full h-auto object-contain group-hover:scale-105 transition-transform duration-300"
      style={{
        backgroundColor: 'transparent',
        borderRadius: '1rem 1rem 0 0'
      }}
    />
  ) : (
    {/* Placeholder */}
  )}
</div>
```

### **OurSponsorsSection Pattern**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
