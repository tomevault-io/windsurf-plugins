---
trigger: always_on
description: Standard pattern for toggle switch components used in admin pages for filtering and state toggling
---


# Admin Toggle Switch Styling Pattern

## **Overview**
This rule defines the standard pattern for toggle switch components used in admin pages for filtering and state toggling. The pattern provides consistent styling, smooth animations, and clear visual feedback for binary state changes.

## **Problem Solved**
- **Consistent Toggle UI**: Ensures all toggle switches use the same visual pattern across admin pages
- **Smooth Animations**: Standardized transition effects and state changes
- **Clear Visual Feedback**: Color-coded states and icons for immediate understanding
- **Accessibility**: Proper ARIA labels and focus states
- **Responsive Design**: Works consistently across different screen sizes

## **Core Pattern**

### **Toggle Switch Container**
```tsx
// ✅ DO: Use the standard toggle switch container pattern
<div className="flex justify-center items-center gap-4 mt-6">
  {/* Left Label */}
  {/* Toggle Switch Button */}
  {/* Right Label */}
</div>
```

### **Toggle Switch Button Structure**
```tsx
// ✅ DO: Use the standard toggle switch button pattern
<button
  onClick={() => setState(!state)}
  className={`relative inline-flex h-10 w-16 items-center rounded-full transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 hover:scale-105 ${
    state
      ? 'bg-blue-500 focus:ring-blue-500'
      : 'bg-purple-500 focus:ring-purple-500'
    }`}
  title={state ? 'Switch to Off State' : 'Switch to On State'}
  aria-label={state ? 'Switch to Off State' : 'Switch to On State'}
>
  <span
    className={`inline-flex items-center justify-center h-8 w-8 transform rounded-full bg-white transition-transform duration-300 shadow-md ${
      state ? 'translate-x-7' : 'translate-x-1'
    }`}
  >
    {/* Icon based on state */}
  </span>
</button>
```

## **Key CSS Properties**

### **Container Requirements**
- **`flex justify-center items-center`**: Centers toggle switch horizontally and vertically
- **`gap-4`**: Spacing between labels and switch (16px)
- **`mt-6`**: Top margin (24px) for spacing above toggle

### **Button Requirements**
- **`relative`**: Enables absolute positioning of thumb
- **`inline-flex`**: Inline flex layout
- **`h-10`**: Fixed height (40px) for toggle track
- **`w-16`**: Fixed width (64px) for toggle track
- **`items-center`**: Centers thumb vertically
- **`rounded-full`**: Fully rounded track (pill shape)
- **`transition-all duration-300`**: Smooth transitions for all properties
- **`focus:outline-none focus:ring-2 focus:ring-offset-2`**: Focus ring for accessibility
- **`hover:scale-105`**: Subtle scale effect on hover (5% increase)
- **State Colors**:
  - **Active/On State**: `bg-blue-500 focus:ring-blue-500`
  - **Inactive/Off State**: `bg-purple-500 focus:ring-purple-500`

### **Thumb (Slider) Requirements**
- **`inline-flex items-center justify-center`**: Centers icon within thumb
- **`h-8 w-8`**: Fixed thumb size (32px × 32px)
- **`transform rounded-full`**: Fully rounded thumb
- **`bg-white`**: White background for thumb
- **`transition-transform duration-300`**: Smooth slide animation
- **`shadow-md`**: Medium shadow for depth
- **Position**:
  - **Active/On State**: `translate-x-7` (moves to right, 28px)
  - **Inactive/Off State**: `translate-x-1` (moves to left, 4px)

### **Label Requirements**
- **`text-lg font-semibold`**: Large, bold text
- **`transition-colors duration-300`**: Smooth color transitions
- **Active State**: `text-purple-600` or `text-blue-600` (vibrant color)
- **Inactive State**: `text-purple-300` or `text-blue-300` (muted color)

### **Icon Requirements**
- **`w-5 h-5`**: Icon size (20px × 20px)
- **Color**: Matches button state color (`text-blue-600` or `text-purple-600`)
- **SVG**: Use Heroicons pattern with `fill="none" stroke="currentColor"`

## **Complete Example**

### **Future/Past Events Toggle**
```tsx
{/* Event Filter Toggle */}
<div className="flex justify-center items-center gap-4 mt-6">
  <span className={`text-lg font-semibold transition-colors duration-300 ${!showPastEvents ? 'text-purple-600' : 'text-purple-300'}`}>
    Future Events
  </span>
  <button
    onClick={() => setShowPastEvents(!showPastEvents)}
    className={`relative inline-flex h-10 w-16 items-center rounded-full transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 hover:scale-105 ${
      showPastEvents
        ? 'bg-blue-500 focus:ring-blue-500'
        : 'bg-purple-500 focus:ring-purple-500'
    }`}
    title={showPastEvents ? 'Show Future Events' : 'Show Past Events'}
    aria-label={showPastEvents ? 'Show Future Events' : 'Show Past Events'}
  >
    <span
      className={`inline-flex items-center justify-center h-8 w-8 transform rounded-full bg-white transition-transform duration-300 shadow-md ${showPastEvents ? 'translate-x-7' : 'translate-x-1'}`}
    >
      {showPastEvents ? (
        <svg className="w-5 h-5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      ) : (
        <svg className="w-5 h-5 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
