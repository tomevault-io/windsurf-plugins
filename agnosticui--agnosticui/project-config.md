---
trigger: always_on
description: <!-- agnosticui:context:start -->
---

<!-- agnosticui:context:start -->
## AgnosticUI Components

This project uses AgnosticUI Local (v2.0.0-alpha, react framework).
Components are installed at `./src/components/ag`.

> Re-run `npx agnosticui-cli context` after adding or updating components.

### Accordion

**Import:** `import { ReactAccordion } from './src/components/ag/Accordion/react/ReactAccordion';`

### Alert

**Import:** `import { ReactAlert } from './src/components/ag/Alert/react/ReactAlert';`

**Props:**
```typescript
variant?: AlertVariant;
bordered?: boolean;
rounded?: boolean;
borderedLeft?: boolean;
dismissible?: boolean;
onAlertDismiss?: (event: AlertDismissEvent) => void;
```

### AspectRatio

**Import:** `import { ReactAspectRatio } from './src/components/ag/AspectRatio/react/ReactAspectRatio';`

**Props:**
```typescript
width: number;
height: number;
maxWidth?: number | undefined;
```

### Avatar

**Import:** `import { ReactAvatar } from './src/components/ag/Avatar/react/ReactAvatar';`

**Props:**
```typescript
/** Text content to display (typically initials like "AB") */
text?: string;
/** Image source URL */
imgSrc?: string;
/** Alt text for the image (required when imgSrc is provided for accessibility) */
imgAlt?: string;
/** Size of the avatar */
size?: AvatarSize;
/** Shape of the avatar */
shape?: AvatarShape;
/** Color variant */
variant?: AvatarVariant;
/** ARIA label for accessibility */
ariaLabel?: string;
```

### Badge

**Import:** `import { ReactBadge } from './src/components/ag/Badge/react/ReactBadge';`

**Props:**
```typescript
variant?: 'default' | 'success' | 'warning' | 'danger' | 'primary' | 'info' | 'neutral' | 'monochrome';
size?: 'xs' | 'sm' | 'md';
dot?: boolean;
value?: number | null;
max?: number;
interactive?: boolean;
statusLabel?: string | null;
live?: 'off' | 'polite' | 'assertive';
hiddenFromAT?: boolean;
```

### BadgeFx

**Import:** `import { ReactBadgeFx } from './src/components/ag/BadgeFx/react/ReactBadgeFx';`

### Breadcrumb

**Import:** `import { ReactBreadcrumb } from './src/components/ag/Breadcrumb/react/ReactBreadcrumb';`

**Props:**
```typescript
items?: BreadcrumbItem[];
type?: 'default' | 'slash' | 'bullet' | 'arrow';
primary?: boolean;
ariaLabel?: string;
onBreadcrumbClick?: (event: BreadcrumbClickEvent) => void;
```

### Button

**Import:** `import { ReactButton } from './src/components/ag/Button/react/ReactButton';`

**Props:**
```typescript
variant?: 'success' | 'primary' | 'secondary' | 'warning' | 'danger' | 'monochrome' | '';
size?: 'x-sm' | 'sm' | 'md' | 'lg' | 'xl';
shape?: 'capsule' | 'rounded' | 'circle' | 'square' | 'rounded-square' | '';
bordered?: boolean;
ghost?: boolean;
link?: boolean;
grouped?: boolean;
fullWidth?: boolean;
type?: 'button' | 'submit' | 'reset';
disabled?: boolean;
loading?: boolean;
toggle?: boolean;
pressed?: boolean;
ariaLabel?: string;
onClick?: (event: MouseEvent) => void;
onFocus?: (event: FocusEvent) => void;
onBlur?: (event: FocusEvent) => void;
onToggle?: (event: ButtonToggleEvent) => void;
```

### ButtonFx

**Import:** `import { ReactButtonFx } from './src/components/ag/ButtonFx/react/ReactButtonFx';`

### Card

**Import:** `import { ReactCard } from './src/components/ag/Card/react/ReactCard';`

**Props:**
```typescript
stacked?: boolean;
shadow?: boolean;
animated?: boolean;
/** Border radius size. Use 'sm', 'md', 'lg' or true (defaults to 'md') */
rounded?: CardRounded | boolean;
variant?: CardVariant;
```

### Checkbox

**Import:** `import { ReactCheckbox } from './src/components/ag/Checkbox/react/ReactCheckbox';`

**Props:**
```typescript
name?: string;
value?: string;
checked?: boolean;
indeterminate?: boolean;
disabled?: boolean;
size?: CheckboxSize;
theme?: CheckboxTheme;
labelText?: string;
labelPosition?: 'end' | 'start';
required?: boolean;
invalid?: boolean;
errorMessage?: string;
helpText?: string;
onClick?: (event: MouseEvent) => void;
onChange?: (event: CheckboxChangeEvent) => void;
```

### Collapsible

**Import:** `import { ReactCollapsible } from './src/components/ag/Collapsible/react/ReactCollapsible';`

**Props:**
```typescript
open?: boolean;
bordered?: boolean;
shadow?: boolean;
useChevron?: boolean;
useX?: boolean;
useMinus?: boolean;
noIndicator?: boolean;
onToggle?: (event: CollapsibleToggleEvent) => void;
```

### Combobox

**Import:** `import { ReactCombobox } from './src/components/ag/Combobox/react/ReactCombobox';`

**Props:**
```typescript
options: ComboboxOption[];
value?: string | string[];
defaultValue?: string | string[];
placeholder?: string;
label?: string;
labelPosition?: LabelPosition;
labelHidden?: boolean;
noLabel?: boolean;
ariaLabel?: string | null;
helpText?: string;
errorMessage?: string;
id?: string;
autocomplete?: 'list' | 'none';
filterMode?: 'startsWith' | 'contains' | 'none';
clearable?: boolean;
disabled?: boolean;
readonly?: boolean;
required?: boolean;
invalid?: boolean;
size?: 'small' | 'default' | 'large';
maxVisibleOptions?: number;
closeOnSelect?: boolean;
variant?: 'default' | 'monochrome';
multiple?: boolean;
maxOptionsVisible?: number;
loading?: boolean;
loadingText?: string;
noResultsText?: string;
onChange?: (event: ComboboxChangeEvent) => void;
onSelect?: (event: ComboboxSelectEvent) => void;
onSearch?: (event: ComboboxSearchEvent) => void;
onOpen?: (event: ComboboxOpenEvent) => void;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgnosticUI/agnosticui](https://github.com/AgnosticUI/agnosticui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
