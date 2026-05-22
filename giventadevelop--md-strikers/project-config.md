---
trigger: always_on
description: This rule defines the standard pattern for loading animations used across admin pages and success pages. The pattern provides consistent visual feedback with animated loading images and wavy overlay effects, matching the design aesthetic used in EventList and manage-events pages.
---

# Loading Animation Pattern

## **Overview**
This rule defines the standard pattern for loading animations used across admin pages and success pages. The pattern provides consistent visual feedback with animated loading images and wavy overlay effects, matching the design aesthetic used in EventList and manage-events pages.

## **Problem Solved**
- **Consistent Loading UI**: Ensures all loading states use the same visual pattern across admin and success pages
- **Visual Feedback**: Provides engaging animations (pulse, zoom, wavy overlay) to indicate processing
- **Professional Appearance**: Uses high-quality loading images with smooth animations
- **User Experience**: Clear visual indication that content is being loaded

## **Core Pattern**

### **Loading Container Structure**
```tsx
// ✅ DO: Use the standard loading animation pattern
if (loading) {
  return (
    <div className="flex justify-center items-center min-h-[600px] w-full">
      <div className="relative w-full max-w-6xl">
        <Image
          src="/images/loading_events.jpg"
          alt="Loading..."
          width={800}
          height={600}
          className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"
          priority
        />
        <div className="absolute inset-0 rounded-lg overflow-hidden">
          <div className="wavy-animation"></div>
        </div>
      </div>
    </div>
  );
}
```

## **Key CSS Properties**

### **Container Requirements**
- **`flex justify-center items-center`**: Centers loading content horizontally and vertically
- **`min-h-[600px]`**: Minimum height (600px) for consistent loading area
- **`w-full`**: Full width of parent container
- **`relative`**: Enables absolute positioning of overlay
- **`w-full max-w-6xl`**: Full width with maximum width constraint (1152px)

### **Image Requirements**
- **`src="/images/loading_events.jpg"`**: Standard loading image path
- **`width={800} height={600}`**: Image dimensions for Next.js Image optimization
- **`w-full h-auto`**: Full width with automatic height (maintains aspect ratio)
- **`rounded-lg`**: Medium border radius (8px) for rounded corners
- **`shadow-2xl`**: Large shadow for depth
- **`animate-pulse`**: Tailwind pulse animation (opacity fade in/out)
- **`zoom-loading`**: Custom zoom animation (scale 0.8 → 1.1 → 1.0)
- **`priority`**: Next.js Image priority loading (loads immediately)

### **Wavy Overlay Requirements**
- **`absolute inset-0`**: Covers entire container (absolute positioned, all sides at 0)
- **`rounded-lg overflow-hidden`**: Matches container border radius, clips overlay
- **`wavy-animation`**: Custom CSS class for wavy shimmer effect

## **CSS Animation Definitions**

### **Zoom Loading Animation** (defined in `src/app/globals.css`)
```css
@keyframes zoomInOut {
  0% {
    transform: scale(0.8);
    opacity: 0.7;
  }
  50% {
    transform: scale(1.1);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 0.9;
  }
}

.zoom-loading {
  animation: zoomInOut 2s ease-in-out infinite;
}
```

### **Wavy Animation** (defined in `src/app/globals.css`)
```css
@keyframes wavy {
  0%, 100% {
    transform: translateY(0px) scale(1);
    opacity: 0.3;
  }
  50% {
    transform: translateY(-15px) scale(1.05);
    opacity: 0.7;
  }
}

@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

.wavy-animation {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg,
    transparent 0%,
    rgba(255, 193, 7, 0.2) 25%,
    rgba(255, 193, 7, 0.4) 50%,
    rgba(255, 193, 7, 0.2) 75%,
    transparent 100%);
  background-size: 200% 100%;
  animation:
    wavy 3s ease-in-out infinite,
    shimmer 2s ease-in-out infinite;
  border-radius: inherit;
  pointer-events: none;
}
```

## **Complete Example**

### **EventList Loading Pattern**
```tsx
// ✅ DO: Use the standard loading animation pattern (EventList component)
if (loading) {
  return (
    <div className="flex justify-center items-center min-h-[600px] w-full">
      <div className="relative w-full max-w-6xl">
        <Image
          src="/images/loading_events.jpg"
          alt="Loading events..."
          width={800}
          height={600}
          className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"
          priority
        />
        <div className="absolute inset-0 rounded-lg overflow-hidden">
          <div className="wavy-animation"></div>
        </div>
      </div>
    </div>
  );
}
```

### **Membership Success Loading Pattern**
```tsx
// ✅ DO: Use the standard loading animation pattern (MembershipSuccessClient component)
if (loading) {
  return (
    <div className="flex justify-center items-center min-h-[600px] w-full">
      <div className="relative w-full max-w-6xl">
        <Image
          src="/images/loading_events.jpg"
          alt="Loading membership subscription..."
          width={800}
          height={600}
          className="w-full h-auto rounded-lg shadow-2xl animate-pulse zoom-loading"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
