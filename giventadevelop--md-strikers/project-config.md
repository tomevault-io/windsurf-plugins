---
trigger: always_on
description: Standard pattern for dialog buttons (AlertDialog Action/Cancel) matching admin action buttons styling
---


# Dialog Button Styling Pattern

## **Overview**
This rule defines the standard pattern for dialog buttons (AlertDialog Action and Cancel buttons) that match the admin action buttons styling pattern. These buttons provide consistent styling, hover effects, and icon presentation across all dialogs in the application.

## **Problem Solved**
- **Consistent Dialog Button Styling**: Ensures all dialog buttons follow the same visual pattern as admin action buttons
- **Icon Standardization**: Provides consistent icon container and sizing for dialog buttons
- **Hover Effects**: Standardized hover states and transitions matching admin buttons
- **Color Coding**: Semantic color usage for different action types (blue for cancel/keep, red for destructive actions)
- **Accessibility**: Proper styling and visual feedback for user interactions

## **Core Pattern**

### **Dialog Footer Structure**
```tsx
// ✅ DO: Use the standard dialog button pattern
<AlertDialogFooter className="flex flex-row gap-3 sm:gap-4">
  {/* Cancel/Secondary Button */}
  <AlertDialogCancel
    onClick={handleClose}
    className="flex-1 flex-shrink-0 h-14 rounded-xl bg-blue-100 hover:bg-blue-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105"
  >
    <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-blue-200 flex items-center justify-center">
      <svg className="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
      </svg>
    </div>
    <span className="font-semibold text-blue-700">Cancel/Keep</span>
  </AlertDialogCancel>

  {/* Confirm/Primary Button */}
  <AlertDialogAction
    onClick={handleConfirm}
    disabled={isLoading}
    className="flex-1 flex-shrink-0 h-14 rounded-xl bg-red-100 hover:bg-red-200 flex items-center justify-center gap-3 transition-all duration-300 hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100"
  >
    <div className="flex-shrink-0 w-10 h-10 rounded-lg bg-red-200 flex items-center justify-center">
      {isLoading ? (
        <svg className="animate-spin w-6 h-6 text-red-600" fill="none" viewBox="0 0 24 24">
          <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
          <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      ) : (
        <svg className="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
        </svg>
      )}
    </div>
    <span className="font-semibold text-red-700">{isLoading ? 'Processing...' : 'Confirm'}</span>
  </AlertDialogAction>
</AlertDialogFooter>
```

## **Key CSS Properties**

### **Dialog Footer Requirements**
- **`flex flex-row`**: Horizontal layout for buttons
- **`gap-3 sm:gap-4`**: Spacing between buttons (12px on mobile, 16px on desktop)

### **Button Container Requirements**
- **`flex-1`**: Equal width buttons
- **`flex-shrink-0`**: Prevents button from shrinking
- **`h-14`**: Fixed height (56px) for consistent button size
- **`rounded-xl`**: Large border radius (12px) for modern appearance
- **`bg-{color}-100`**: Light background color matching action type
- **`hover:bg-{color}-200`**: Darker background on hover
- **`flex items-center justify-center`**: Centers content horizontally and vertically
- **`gap-3`**: Spacing between icon and text (12px)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)
- **`disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100`**: Disabled state styling

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
- **`animate-spin`**: For loading state icons

### **Text Requirements**
- **`font-semibold`**: Bold text for emphasis
- **`text-{color}-700`**: Text color matching action type (darker than icon)

## **Color Coding System**

### **Semantic Colors for Dialog Actions**
- **Blue** (`blue-100/200/600/700`): Cancel, Keep, Secondary actions
- **Red** (`red-100/200/600/700`): Confirm Delete, Destructive actions
- **Green** (`green-100/200/600/700`): Confirm Save, Positive actions
- **Gray** (`gray-100/200/600/700`): Neutral actions

## **Complete Examples**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
