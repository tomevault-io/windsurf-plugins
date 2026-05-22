---
trigger: always_on
description: Standard pattern for admin action buttons with icons in admin pages and sub-pages
---


# Admin Action Buttons Pattern

## **Overview**
This rule defines the standard pattern for action buttons with icons used in admin pages and sub-pages. These buttons provide consistent styling, hover effects, and icon presentation across all admin interfaces.

## **Problem Solved**
- **Consistent Styling**: Ensures all admin action buttons follow the same visual pattern
- **Icon Standardization**: Provides consistent icon container and sizing
- **Hover Effects**: Standardized hover states and transitions
- **Color Coding**: Semantic color usage for different action types
- **Accessibility**: Proper ARIA labels and titles for screen readers

## **Core Pattern**

### **Button Structure**
```tsx
// ✅ DO: Use the standard admin action button pattern
<Link
  href="/admin/path/to/resource"
  className="w-full flex-shrink-0 h-14 rounded-xl bg-{color}-100 hover:bg-{color}-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="Button Label"
  aria-label="Button Label"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-{color}-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-{color}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="..." />
    </svg>
  </div>
  <span className="font-semibold text-{color}-700">Button Label</span>
</Link>
```

## **Key CSS Properties**

### **Button Container Requirements**
- **`w-full`**: Full width of parent container
- **`flex-shrink-0`**: Prevents button from shrinking
- **`h-14`**: Fixed height (56px) for consistent button size
- **`rounded-xl`**: Large border radius (12px) for modern appearance
- **`bg-{color}-100`**: Light background color matching action type
- **`hover:bg-{color}-200`**: Darker background on hover
- **`flex items-center justify-center`**: Centers content horizontally and vertically
- **`gap-3`**: Spacing between icon and text (12px)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)

### **Icon Container Requirements**
- **`flex-shrink-0`**: Prevents icon container from shrinking
- **`w-10 h-10`**: Fixed icon container size (40px × 40px)
- **`rounded-lg`**: Medium border radius (8px) for icon container
- **`bg-{color}-200`**: Darker background than button (creates depth)
- **`flex items-center justify-center`**: Centers icon within container

### **Icon Requirements**
- **`w-6 h-6`**: Icon size (24px × 24px)
- **`text-{color}-600`**: Icon color matching action type
- **`fill="none" stroke="currentColor"`**: Standard SVG styling
- **`viewBox="0 0 24 24"`**: Standard viewBox for Heroicons
- **`strokeWidth={2}`**: Standard stroke width

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-{color}-700`**: Text color matching action type (darker than icon)

## **Color Coding System**

### **Semantic Colors for Actions**
- **Blue** (`blue-100/200/600/700`): Edit, Update, Modify actions
- **Green** (`green-100/200/600/700`): View, View Details, View Organization actions
- **Gray** (`gray-100/200/600/700`): Settings, Configuration actions
- **Purple** (`purple-100/200/600/700`): Special, Advanced actions
- **Orange** (`orange-100/200/600/700`): Warning, Attention actions
- **Red** (`red-100/200/600/700`): Delete, Remove, Destructive actions
- **Indigo** (`indigo-100/200/600/700`): Navigation, Link actions
- **Teal** (`teal-100/200/600/700`): Analytics, Reports actions

## **Complete Examples**

### **View Organization Button (Green)**
```tsx
<Link
  href={`/admin/tenant-management/organizations/${organization.id}`}
  className="w-full flex-shrink-0 h-14 rounded-xl bg-green-100 hover:bg-green-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="View Organization"
  aria-label="View Organization"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-green-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" />
    </svg>
  </div>
  <span className="font-semibold text-green-700">View Organization</span>
</Link>
```

### **Edit Settings Button (Blue)**
```tsx
<Link
  href={`/admin/tenant-management/settings/${id}/edit`}
  className="w-full flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  title="Edit Settings"
  aria-label="Edit Settings"
>
  <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
    <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
    </svg>
  </div>
  <span className="font-semibold text-blue-700">Edit Settings</span>
</Link>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
