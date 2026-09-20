---
trigger: always_on
description: UI implementation strategy and component architecture
---


# UI Implementation Strategy

## Design Philosophy

### 1. **Minimalism First**

- **Black Canvas**: Pure black background (#000000) for focus
- **White Mode**: Clean white background (#FFFFFF) with subtle shadows
- **Mode Toggle**: Elegant switch between dark/light modes
- **Distraction-Free**: Only essential UI elements visible

### 2. **Crafting-Focused Design**

- **Large Text Area**: 80% of screen real estate for prompt editing
- **Minimal Toolbar**: Only essential actions (Save, Copy, Share)
- **Contextual UI**: UI appears only when needed
- **Keyboard Shortcuts**: Power user efficiency

## UI Architecture Decision

### **Recommendation: HTML/CSS with React Components**

**Why NOT Canvas/Figma:**

- **Performance**: HTML/CSS is faster for text editing
- **Accessibility**: Better screen reader support
- **SEO**: Searchable content for public prompts
- **Extensibility**: Easier to add features and plugins
- **Mobile**: Better responsive behavior
- **Development Speed**: Faster iteration and debugging

**Why HTML/CSS:**

- **Text Editing**: Superior text input and selection
- **Copy/Paste**: Native browser functionality
- **Search**: Built-in find functionality
- **Accessibility**: Full keyboard navigation
- **Performance**: Hardware-accelerated rendering
- **Flexibility**: Easy to extend and customize

## Component Architecture

### 1. **Core Components**

```typescript
// Main layout component
<EverPromptApp>
  <PromptEditor />
  <ArcLabels />
  <LabelSheet />
  <ModeToggle />
</EverPromptApp>

// Prompt editing component
<PromptEditor>
  <PromptTextarea />
  <PromptToolbar />
  <PromptMetadata />
</PromptEditor>

// Label navigation component
<ArcLabels>
  <LabelDot />
  <LabelCaption />
  <MoreButton />
</ArcLabels>

// Label management component
<LabelSheet>
  <LabelHeader />
  <PromptList />
  <SearchBar />
  <SortControls />
</LabelSheet>
```

### 2. **State Management**

```typescript
interface AppState {
  // UI State
  mode: "dark" | "light";
  currentPrompt: Prompt | null;
  selectedLabel: string | null;
  labelSheetOpen: boolean;

  // Data State
  prompts: Prompt[];
  labels: Label[];
  workspace: Workspace;

  // Editor State
  editorContent: string;
  isSaving: boolean;
  lastSaved: Date | null;
}

// shadcn/ui Integration
interface ShadcnUIState {
  theme: "light" | "dark" | "system";
  components: {
    button: ButtonVariant;
    card: CardVariant;
    input: InputVariant;
  };
}
```

## Visual Design System

### 1. **Color Palette**

```css
/* Dark Mode */
:root {
  --bg-primary: #000000;
  --bg-secondary: #111111;
  --text-primary: #ffffff;
  --text-secondary: #888888;
  --accent: #00ff88;
  --border: #333333;
}

/* Light Mode */
:root[data-theme="light"] {
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --text-primary: #000000;
  --text-secondary: #666666;
  --accent: #0066cc;
  --border: #e1e5e9;
}
```

### 2. **Typography**

```css
/* Primary Font */
.font-primary {
  font-family: "Inter", -apple-system, BlinkMacSystemFont, sans-serif;
  font-weight: 400;
  line-height: 1.6;
}

/* Monospace for Code */
.font-mono {
  font-family: "JetBrains Mono", "Fira Code", monospace;
  font-weight: 400;
  line-height: 1.5;
}

/* Sizes */
.text-xs {
  font-size: 0.75rem;
}
.text-sm {
  font-size: 0.875rem;
}
.text-base {
  font-size: 1rem;
}
.text-lg {
  font-size: 1.125rem;
}
.text-xl {
  font-size: 1.25rem;
}
.text-2xl {
  font-size: 1.5rem;
}
```

### 3. **Spacing System**

```css
/* Consistent spacing scale */
.space-1 {
  margin: 0.25rem;
}
.space-2 {
  margin: 0.5rem;
}
.space-3 {
  margin: 0.75rem;
}
.space-4 {
  margin: 1rem;
}
.space-6 {
  margin: 1.5rem;
}
.space-8 {
  margin: 2rem;
}
.space-12 {
  margin: 3rem;
}
.space-16 {
  margin: 4rem;
}
```

## Responsive Design

### 1. **Breakpoints**

```css
/* Mobile First Approach */
@media (min-width: 640px) {
  /* sm */
}
@media (min-width: 768px) {
  /* md */
}
@media (min-width: 1024px) {
  /* lg */
}
@media (min-width: 1280px) {
  /* xl */
}
@media (min-width: 1536px) {
  /* 2xl */
}
```

### 2. **Layout Adaptations**

```typescript
// Mobile: Stack vertically
<MobileLayout>
  <PromptEditor />
  <ArcLabels />
</MobileLayout>

// Tablet: Side-by-side with collapsible arc
<TabletLayout>
  <PromptEditor />
  <CollapsibleArcLabels />
</TabletLayout>

// Desktop: Full arc with side panel
<DesktopLayout>
  <PromptEditor />
  <ArcLabels />
  <LabelSheet />
</DesktopLayout>
```

## Animation & Interactions

### 1. **Micro-Interactions**

```css
/* Smooth transitions */
.transition {
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Hover effects */
.hover-lift:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* Focus states */
.focus-ring:focus {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}
```

### 2. **Loading States**

```typescript
// Skeleton loading for prompts
<PromptSkeleton>
  <div className="h-4 bg-gray-200 rounded animate-pulse" />
  <div className="h-4 bg-gray-200 rounded animate-pulse w-3/4" />
</PromptSkeleton>

// Saving indicator
<SavingIndicator>
  <div className="flex items-center gap-2">
    <Spinner size="sm" />
    <span>Saving...</span>
  </div>
</SavingIndicator>
```

## Accessibility

### 1. **Keyboard Navigation**

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
