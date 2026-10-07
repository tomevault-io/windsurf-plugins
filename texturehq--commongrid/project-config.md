---
trigger: always_on
description: - **ActionCell** - Component
---


## @texturehq/edges Components (v1.28.2)

### Quick Reference

- **ActionCell** - Component
- **ActionMenu** - ActionMenu  A dropdown menu for actions, typically triggered by a button. Supports icons, destructive actions, and flexible alignment.
- **ActivityFeed** - Height/maxHeight for scroll container
- **ActivityFeedGroup** - Group content
- **ActivityItem** - Marks the item as interactive, enabling keyboard semantics
- **Alert** - Alert  A simple alert dialog with a single action button. Use for informational messages that require acknowledgment.
- **AppShell** - Isomorphic AppShell component that works as both server and client component Uses CSS-only interactions by default for full SSR compatibility Can be progressively enhanced with JavaScript when enableJsEnhancements is true
- **AreaSeries** - AreaSeries  Chart component for rendering area charts with gradient fill. Displays data as a filled area with optional line path, supporting animations and custom colors.
- **Autocomplete** - Additional CSS classes to apply to the component
- **AutoMobileRenderer** - Component
- **Avatar** - Avatar  Display user, team, or organization avatars with support for images, initials, and fallback icons. Includes optional status indicators.
- **Badge** - Badge  A non-interactive label for displaying status, counts, or categories. Use for read-only indicators. For interactive elements, use Chip instead.  Supports device states (charging, discharging, heat, cool, etc.) and grid states (importing, exporting) as variant values.
- **BadgeCell** - Component
- **Banner** - Banner  A prominent message component for displaying important information or system messages. Can be used inline within content sections or positioned at the page level.  Inspired by Atlassian's Section Message and Chakra UI's Alert, combining the best features of both: semantic variant colors, multiple appearances, inline actions, and flexible composition.  @example ```tsx // Simple info banner <Banner variant="info" title="New features available"> Check out the latest updates to improve your workflow. </Banner>  // With actions <Banner variant="warning" title="Your trial expires soon" primaryAction={{ label: "Upgrade now", onPress: () => {} }} secondaryAction={{ label: "Learn more", onPress: () => {}, asLink: true }} > Upgrade to continue using premium features. </Banner>  // Bold appearance <Banner variant="error" appearance="bold" title="Action required" dismissible onDismiss={() => {}} > Please update your payment information. </Banner> ```
- **BarSeries** - BarSeries  Chart component for rendering bar/column charts. Displays data as vertical bars with customizable colors, opacity, and category-based coloring.
- **BooleanCell** - Component
- **BreadcrumbItem** - BreadcrumbItem  Individual breadcrumb item with optional link The last breadcrumb is automatically styled as the current page (non-interactive, bold) When `copyable` is true, adds a copy-to-clipboard button next to the current (last) breadcrumb
- **Breadcrumbs** - Breadcrumbs  Navigation breadcrumbs that show the user's current location in the hierarchy Automatically collapses middle items into an ellipsis dropdown when the container width exceeds the maxWidth threshold (default: 600px)
- **Button** - Renders an Edges Button. When `href` is provided, renders a link-styled button using the same visual system.
- **Calendar** - Calendar  Single-date calendar primitive with Edges styling.
- **Card** - When true and parent Card has layout="flex", content expands to fill available space
- **CardContent** - When true and parent Card has layout="flex", content expands to fill available space
- **CardFooter** - When true and parent Card has layout="flex", content expands to fill available space
- **CardHeader** - When true and parent Card has layout="flex", content expands to fill available space
- **CardMedia** - When true and parent Card has layout="flex", content expands to fill available space
- **CardMobileRenderer** - Component
- **CarouselAutoplayTrigger** - CarouselAutoplayTrigger - Toggle button for autoplay functionality. Allows users to pause and resume automatic slide transitions.
- **CarouselControl** - CarouselControl - Container for navigation controls and indicators. Typically holds prev/next buttons, indicators, and autoplay controls.
- **CarouselIndicator** - CarouselIndicator - Dot indicator for a specific slide. Clicking navigates to the slide at the specified index.
- **CarouselIndicatorGroup** - CarouselIndicatorGroup - Container for all page indicators. Automatically generates indicators for each page/slide.
- **CarouselItem** - Individual slide/item within the carousel. Handles sizing based on slidesPerView and applies snap alignment.
- **CarouselItemGroup** - CarouselItemGroup - Container for carousel items with scrolling behavior. Manages scroll snap, drag interactions, and layout spacing.
- **CarouselNextTrigger** - CarouselNextTrigger - Button to navigate to the next slide. Automatically disabled when at the last slide (unless loop is enabled).
- **CarouselPrevTrigger** - CarouselPrevTrigger - Button to navigate to the previous slide. Automatically disabled when at the first slide (unless loop is enabled).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TextureHQ/commongrid](https://github.com/TextureHQ/commongrid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
