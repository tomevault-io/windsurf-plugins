---
trigger: always_on
description: This rule defines the standard pattern for responsive button groups displayed in a grid layout across admin pages. The pattern ensures consistent 2-column layout on mobile devices, proper spacing, and responsive sizing for icons, text, and containers.
---

# Admin Page Responsive Button Group Pattern

## **Overview**
This rule defines the standard pattern for responsive button groups displayed in a grid layout across admin pages. The pattern ensures consistent 2-column layout on mobile devices, proper spacing, and responsive sizing for icons, text, and containers.

## **Problem Solved**
- **Mobile Layout**: Ensures 2 items per row on mobile instead of 1 (better space utilization)
- **Responsive Alignment**: Proper spacing and padding across all screen sizes
- **Icon & Text Scaling**: Icons and text scale appropriately for mobile, tablet, and desktop
- **Consistent Spacing**: Standardized gaps and padding that work across breakpoints
- **Touch-Friendly**: Adequate button sizes for mobile touch interactions

## **Core Pattern**

### **Grid Container**
```tsx
// ✅ DO: Use responsive grid layout with 2 columns on mobile
<div className="bg-white rounded-xl shadow-lg p-4 sm:p-6 lg:p-8">
  <div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3 sm:gap-4 lg:gap-6">
    {/* Button cards */}
  </div>
</div>
```

### **Button Card Structure**
```tsx
// ✅ DO: Use responsive button card pattern
<Link
  href="/admin/path/to/resource"
  className={`flex flex-col items-center justify-center rounded-lg border-2 p-2.5 sm:p-3 lg:p-4 transition-all duration-300 hover:scale-105 hover:shadow-md group ${colorClasses}`}
  title="Button Label"
  aria-label="Button Label"
>
  <div className={`flex-shrink-0 w-10 h-10 sm:w-11 sm:h-11 rounded-xl ${iconBgColor} flex items-center justify-center mb-1.5 sm:mb-2 group-hover:scale-110 transition-transform duration-300`}>
    <IconComponent className={`w-6 h-6 sm:w-8 sm:h-8 ${iconTextColor}`} />
  </div>
  <span className="font-semibold text-center text-xs sm:text-sm lg:text-base leading-tight px-1">
    Button Label
  </span>
</Link>
```

## **Key CSS Properties**

### **Grid Container Requirements**
- **`grid grid-cols-2`**: **CRITICAL**: 2 columns on mobile (not 1)
- **`sm:grid-cols-2`**: 2 columns on small screens (640px+)
- **`md:grid-cols-3`**: 3 columns on medium screens (768px+)
- **`lg:grid-cols-4`**: 4 columns on large screens (1024px+)
- **`gap-3 sm:gap-4 lg:gap-6`**: Responsive gaps
  - Mobile: 12px (0.75rem)
  - Small screens: 16px (1rem)
  - Large screens: 24px (1.5rem)

### **Card Container Requirements**
- **`p-4 sm:p-6 lg:p-8`**: Responsive padding
  - Mobile: 16px (1rem)
  - Small screens: 24px (1.5rem)
  - Large screens: 32px (2rem)

### **Button Card Requirements**
- **`flex flex-col items-center justify-center`**: Centers content vertically and horizontally
- **`rounded-lg`**: Medium border radius (8px) for card appearance
- **`border-2`**: 2px border for definition
- **`p-2.5 sm:p-3 lg:p-4`**: Responsive padding
  - Mobile: 10px (0.625rem)
  - Small screens: 12px (0.75rem)
  - Large screens: 16px (1rem)
- **`transition-all duration-300`**: Smooth transitions
- **`hover:scale-105`**: 5% scale increase on hover
- **`hover:shadow-md`**: Medium shadow on hover
- **`group`**: Enables group hover effects on child elements

### **Icon Container Requirements**
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-10 h-10 sm:w-11 sm:h-11`**: Responsive icon container size
  - Mobile: 40px × 40px
  - Small screens+: 44px × 44px
- **`rounded-xl`**: Large border radius (12px) for icon container
- **`flex items-center justify-center`**: Centers icon within container
- **`mb-1.5 sm:mb-2`**: Responsive margin bottom
  - Mobile: 6px (0.375rem)
  - Small screens+: 8px (0.5rem)
- **`group-hover:scale-110`**: Scales up 10% when parent card is hovered
- **`transition-transform duration-300`**: Smooth scale animation

### **Icon Requirements**
- **`w-6 h-6 sm:w-8 sm:h-8`**: Responsive icon size
  - Mobile: 24px × 24px
  - Small screens+: 32px × 32px
- **Color**: Use semantic color matching action type (e.g., `text-blue-600`)

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-center`**: Centers text horizontally
- **`text-xs sm:text-sm lg:text-base`**: Responsive text size
  - Mobile: 12px (0.75rem)
  - Small screens: 14px (0.875rem)
  - Large screens: 16px (1rem)
- **`leading-tight`**: Tighter line height for compact display
- **`px-1`**: **CRITICAL**: Small horizontal padding prevents text overflow on mobile

## **Complete Example**

### **Full Responsive Button Group**
```tsx
export default function AdminPage() {
  const adminButtons = [
    {
      href: '/admin',
      icon: 'home',
      label: 'Admin Home',
      color: 'blue',
      key: 'admin-home'
    },
    // ... more buttons
  ];

  const getColorClasses = (color: string) => {
    const colorMap: Record<string, string> = {
      blue: 'bg-blue-50 hover:bg-blue-100 text-blue-700 border-blue-200',
      green: 'bg-green-50 hover:bg-green-100 text-green-700 border-green-200',
      // ... other colors
    };
    return colorMap[color] || colorMap.blue;
  };

  const getIconBgColor = (color: string) => {
    const colorMap: Record<string, string> = {
      blue: 'bg-blue-100',
      green: 'bg-green-100',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
