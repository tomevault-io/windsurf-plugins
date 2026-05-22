---
trigger: always_on
description: This rule defines the standard pattern for hero section image rotation on the home page. The pattern provides automatic rotation through event flyer images with synchronized overlay buttons, interactive navigation controls (Previous/Next/Play/Pause), default image display, and proper event selection criteria.
---

# Hero Section Image Rotation Pattern

## **Overview**
This rule defines the standard pattern for hero section image rotation on the home page. The pattern provides automatic rotation through event flyer images with synchronized overlay buttons, interactive navigation controls (Previous/Next/Play/Pause), default image display, and proper event selection criteria.

## **Problem Solved**
- **Automatic Image Rotation**: Rotates through event hero images automatically every 8 seconds
- **Interactive Controls**: Provides Previous/Next/Play/Pause buttons for manual navigation (shown on hover/touch)
- **Overlay Synchronization**: Keeps overlay buttons (Buy Tickets Click Here images) synchronized with current event image
- **Event Selection**: Only shows events that meet specific criteria (future dates, isHomePageHeroImage flag)
- **Recurring Event Handling**: Shows only the next occurrence for recurring events
- **Default Image Display**: Shows default image for first 2 seconds before rotation begins
- **User Experience**: Smooth transitions between event images with proper navigation links and manual control
- **Homepage cache**: Hero section images and data are cached in sessionStorage under `homepage_hero_section_cache` (see `src/lib/homepageCacheKeys.ts` and `documentation/cloud_front_amplify_cache/HOMEPAGE_CACHE_IMPLEMENTATION_PLAN.html`). On refresh or repeat visit, a `useLayoutEffect` reads the cache before paint so the hero shows immediately without waiting for `useFilteredEvents('hero')` or standalone media fetch.

## **Hero Image Display Dimensions (Upload Spec)**

The rotating hero uses **object-contain** in a wide container (~65% viewport width, min-height 280–480px). The `<Image>` component uses **width={1200} height={800}** (3:2 landscape). **Recommended upload dimensions** so the hero section does not become excessively tall:

- **Desktop**: **1200×800px (3:2 landscape)**
- **Mobile**: **900×600px (3:2)** or same file scales down
- **Do not use portrait 800×1200 (2:3)** — portrait images make the hero block too tall.

Keep admin/upload copy and docs (e.g. `HERO_SECTION_IMAGE_SPECIFICATIONS.md`, MediaClientPage hero tip) aligned with 1200×800 (3:2).

## **Core Pattern**

### **Component Structure**
```tsx
// ✅ DO: Use DynamicHeroImage component for hero section rotation
const DynamicHeroImage: React.FC = () => {
  const [currentImageIndex, setCurrentImageIndex] = useState(0);
  const [isShowingDefault, setIsShowingDefault] = useState(true);
  const [dynamicImages, setDynamicImages] = useState<string[]>([]);
  const [currentEvent, setCurrentEvent] = useState<EventWithMediaExtended | null>(null);
  const [upcomingEvents, setUpcomingEvents] = useState<EventWithMediaExtended[]>([]);

  // Use shared data hook for consistent date parsing
  const { filteredEvents, isLoading: eventsLoading, error } = useFilteredEvents('hero');

  // Default image path
  const defaultImage = "/images/hero_section/default_hero_section_second_column_poster.jpeg";

  // Rotation logic with useEffect
  useEffect(() => {
    // Start with default image for 2 seconds
    const defaultTimer = setTimeout(() => {
      setIsShowingDefault(false);
    }, 2000);

    // If we have dynamic images, start rotating them
    if (dynamicImages.length > 0) {
      const dynamicTimer = setTimeout(() => {
        const interval = setInterval(() => {
          setCurrentImageIndex((prev) => {
            const newIndex = (prev + 1) % dynamicImages.length;
            // Update current event when image changes
            if (newIndex < dynamicImages.length - 1 && newIndex < upcomingEvents.length) {
              setCurrentEvent(upcomingEvents[newIndex]);
            } else {
              setCurrentEvent(null);
            }
            return newIndex;
          });
        }, 15000); // Change every 15 seconds

        return () => clearInterval(interval);
      }, 2000); // Start after 2 seconds

      return () => {
        clearTimeout(defaultTimer);
        clearTimeout(dynamicTimer);
      };
    }

    return () => clearTimeout(defaultTimer);
  }, [dynamicImages.length, upcomingEvents]);

  // Render logic...
};
```

## **Interactive Slider Controls (Play/Pause/Previous/Next)**

### **Feature Overview**
The hero section slider includes interactive controls that allow users to manually navigate through images and control auto-rotation. These controls appear on hover (desktop) or touch (mobile) and provide:
- **Previous/Next Navigation**: Manual browsing through event images
- **Play/Pause Control**: Toggle auto-rotation on/off
- **Event Synchronization**: Current event state updates when navigating manually

### **Control Visibility**
- **Show on Hover**: Controls appear when mouse hovers over hero image
- **Show on Touch**: Controls appear when user touches the hero image on mobile devices
- **Auto-Hide on Touch**: Controls automatically hide after 3 seconds of no interaction on touch devices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
