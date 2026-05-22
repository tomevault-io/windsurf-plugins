---
trigger: always_on
description: Standard patterns for displaying portrait and member images without cropping, with proper centering and aspect ratio preservation
---


# Portrait Image Display Pattern

## **Overview**
This rule defines the correct patterns for displaying portrait and member images (people photos) that must be fully visible without cropping heads or other important parts. This pattern ensures images are properly centered, maintain their aspect ratio, and display consistently across all screen sizes.

## **Problem Solved**
- **Head Cropping**: Prevents portrait images from cutting off people's heads or faces
- **Improper Centering**: Ensures images are centered both horizontally and vertically
- **Aspect Ratio Issues**: Maintains original image proportions without distortion
- **Inconsistent Display**: Provides uniform display across different portrait orientations
- **User Experience**: Shows complete portraits so viewers can see the full person

## **When to Use This Pattern**

### **Use for:**
- ✅ Team member profiles and portraits
- ✅ Leadership/executive team displays
- ✅ Clergy and religious leader portraits
- ✅ Board member and staff photos
- ✅ Speaker or guest profiles
- ✅ Any people-centric imagery where the full person must be visible

### **Don't Use for:**
- ❌ Banner/hero images (use image_containment_prevention.mdc instead)
- ❌ Background images where cropping is acceptable
- ❌ Decorative images where partial visibility is fine
- ❌ Product or object photos that can be cropped

## **Core Pattern: Portrait Grid Cards**

### **Container Setup**
```tsx
// ✅ DO: Use aspect ratio container with flexible height
<div className="relative w-full h-auto aspect-[3/4] mx-auto mb-4 rounded-lg overflow-hidden sacred-shadow-sm group-hover:sacred-shadow reverent-transition bg-muted/20">
  <div className="relative w-full h-full flex items-center justify-center p-2">
    {/* Image goes here */}
  </div>
</div>

// ❌ DON'T: Use fixed height that crops images
<div className="w-full h-48 mx-auto mb-4 rounded-lg overflow-hidden">
  {/* This will crop images */}
</div>
```

### **Image Configuration**
```tsx
// ✅ DO: Use object-contain with fill and proper sizing
<Image
  src={member.image}
  alt={member.name}
  fill
  sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
  className="object-contain group-hover:scale-105 reverent-transition"
  style={{
    objectPosition: 'center center'
  }}
/>

// ❌ DON'T: Use object-cover which crops images
<Image
  src={member.image}
  alt={member.name}
  width={242}
  height={156}
  className="w-full h-full object-cover"
/>
```

## **Pattern Variations**

### **1. Grid Layout (Team Members, Synod Members)**

**Use Case**: Multiple portraits in a responsive grid (3-4 columns)

**Important**: Use **flexbox with `justify-content: center`** instead of CSS Grid to automatically center the last row when there are fewer cards than columns. This ensures cards expand from the center outward, and the last row is always centered.

```tsx
// ✅ DO: Use flexbox for automatic last-row centering
<div className="flex flex-wrap gap-6 justify-center items-start max-w-7xl mx-auto">
  {members.map((member) => (
    <Link
      key={member.id}
      href={member.href}
      className="bg-card rounded-lg sacred-shadow p-6 hover:sacred-shadow-lg reverent-transition group w-full sm:w-[calc(50%-0.75rem)] lg:w-[calc(33.333%-1rem)] flex-shrink-0"
      style={{ maxWidth: '400px' }}
    >
      <div className="text-center">
        {/* Image Container */}
        <div className="relative w-full h-auto aspect-[3/4] mx-auto mb-4 rounded-lg overflow-hidden sacred-shadow-sm group-hover:sacred-shadow reverent-transition bg-muted/20">
          <div className="relative w-full h-full flex items-center justify-center p-2">
            <Image
              src={member.image}
              alt={member.name}
              fill
              sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
              className="object-contain group-hover:scale-105 reverent-transition"
              style={{ objectPosition: 'center center' }}
            />
          </div>
        </div>
        
        {/* Member Info */}
        <h3 className="font-heading font-semibold text-lg text-foreground mb-2">
          {member.name}
        </h3>
        <p className="font-body text-sm text-primary font-medium mb-3">
          {member.title}
        </p>
      </div>
    </Link>
  ))}
</div>

// ❌ DON'T: Use CSS Grid - last row won't center with fewer items
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  {/* Last row items will align left, not center */}
</div>
```

**Key Implementation Details**:
- **Flexbox Container**: `flex flex-wrap justify-center` - automatically centers all rows, including the last row
- **Card Widths**: `w-full sm:w-[calc(50%-0.75rem)] lg:w-[calc(33.333%-1rem)]` - responsive widths with gap compensation
- **Prevent Shrinking**: `flex-shrink-0` - maintains card size, prevents compression
- **Max Width**: `style={{ maxWidth: '400px' }}` - consistent card size limit across screen sizes
- **Gap**: `gap-6` - consistent spacing between cards (1.5rem / 24px)
- **Center Alignment**: `justify-center items-start` - centers cards horizontally, aligns tops

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
