---
trigger: always_on
description: This rule applies to all web-based iOS mobile app replicas and prototypes.
---

# Mobile App Replica Rule

## Scope
This rule applies to all web-based iOS mobile app replicas and prototypes.

## Purpose
Ensure consistent, pixel-perfect iOS replica implementation with correct layout structure, visual language, and device scaling behavior.

---

## Layout Structure (Critical)

### Three-Layer Architecture
Every iOS replica MUST follow this structure:

1. **Fixed Top Layer** (does NOT scroll)
   - Safe area top
   - Status bar (time, signal, battery)
   - Navigation header (title, actions)

2. **Scrollable Content Layer** (scrolls internally)
   - All screen-specific content
   - Cards, lists, sections
   - Recommendations, forms, etc.

3. **Fixed Bottom Layer** (does NOT scroll)
   - Primary CTA button (checkout, submit, etc.)
   - Tab bar navigation
   - Safe area bottom

### Scroll Behavior
- The `content-area` is the ONLY scrollable container
- Use `-webkit-overflow-scrolling: touch` for native feel
- Hide scrollbar visually: `scrollbar-width: none`
- Browser page itself must NOT be the primary scroll container

---

## Dynamic Device Scaling

### Rules
- Use JavaScript to calculate scale dynamically based on viewport dimensions
- Use `transform: scale()` with `transform-origin: center center` for proper centering
- Calculate scale considering BOTH width AND height constraints: `scale = Math.min(scaleHeight, scaleWidth)`
- Calculate scaleHeight: `(viewportHeight - paddingVertical) / deviceHeight`
- Calculate scaleWidth: `(viewportWidth - paddingHorizontal) / deviceWidth`
- Clamp scale between reasonable bounds (0.5 to 1.5)
- Update scale on window resize with debounce
- Set `overflow: hidden` on body to prevent page scroll

### Scaling in React Applications

When using React framework:
- Implement scaling logic ONLY in React component (e.g., `DeviceFrame.tsx`)
- NEVER duplicate scaling logic in `index.html` scripts
- Use `useEffect` hook for viewport resize handling
- Use `useState` to store calculated scale value
- Ensure `#root` also has flex centering styles for proper layout
- Remove any scaling scripts from `index.html` when migrating to React

### Device Dimensions
- iPhone 14 Pro Max: 428 x 926 px
- iPhone 14 Pro: 393 x 852 px
- iPhone 14: 390 x 844 px

### Examples

Good:
```javascript
// Static HTML approach
const DEVICE_HEIGHT = 926;
const DEVICE_WIDTH = 428;
const PADDING_VERTICAL = 60;
const PADDING_HORIZONTAL = 40;

function setDeviceScale() {
  const wrapper = document.querySelector('.device-scale-wrapper');
  const availableHeight = window.innerHeight - PADDING_VERTICAL;
  const availableWidth = window.innerWidth - PADDING_HORIZONTAL;
  const scaleHeight = availableHeight / DEVICE_HEIGHT;
  const scaleWidth = availableWidth / DEVICE_WIDTH;
  const scale = Math.max(0.5, Math.min(Math.min(scaleHeight, scaleWidth), 1.5));
  wrapper.style.transform = `scale(${scale})`;
}

window.addEventListener('resize', setDeviceScale);
setDeviceScale();
```

```tsx
// React component approach
const DeviceFrame: React.FC = ({ children }) => {
  const [scale, setScale] = useState(1);

  useEffect(() => {
    const calculateScale = () => {
      const availableHeight = window.innerHeight - PADDING_VERTICAL;
      const availableWidth = window.innerWidth - PADDING_HORIZONTAL;
      const scaleHeight = availableHeight / DEVICE_HEIGHT;
      const scaleWidth = availableWidth / DEVICE_WIDTH;
      const optimalScale = Math.min(scaleHeight, scaleWidth);
      const clampedScale = Math.max(0.5, Math.min(optimalScale, 1.5));
      setScale(clampedScale);
    };

    calculateScale();
    const handleResize = debounce(calculateScale, 100);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return (
    <div className="device-scale-wrapper" style={{ transform: `scale(${scale})` }}>
      {children}
    </div>
  );
};
```

```css
html, body {
  height: 100%;
  overflow: hidden;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
}

.device-scale-wrapper {
  transform-origin: center center;
}
```

Bad:
```css
.device-scale-wrapper {
  zoom: 1.8; /* Static value - does not adapt to viewport */
}
```

```css
body {
  align-items: flex-start; /* Device sticks to top instead of centering */
  padding-top: 20px;
}
```

```html
<!-- index.html - Duplicate scaling logic conflicts with React component -->
<script>
  // BAD: Scaling script in HTML when React component also handles scaling
  document.querySelector('.device-scale-wrapper').style.transform = `scale(${scale})`;
</script>
```

```tsx
// BAD: Only considering height, ignoring width constraint
const scale = (window.innerHeight - 60) / DEVICE_HEIGHT;
// Device may overflow horizontally on narrow viewports
```

---

## iOS Visual Language

### Colors (iOS System Palette)
- Background: `#f8f8f8` (systemGray6)
- Card background: `#ffffff`
- Separator: `#e5e5ea` (systemGray5)
- Secondary text: `#8e8e93` (systemGray)
- Primary accent: context-dependent (e.g., `#34c759` for success)

### Typography Hierarchy
- **Primary** (headings, key info): 22-28px, font-weight: 700
- **Secondary** (content, names): 16-17px, font-weight: 500-600
- **Tertiary** (metadata, hints): 13-15px, font-weight: 400, color: #8e8e93

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Wert1go) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
