---
trigger: always_on
description: Implement and finish building a scaffolded module
---

# HubSpot React Module Implementation Guide

This rule guides you through implementing a HubSpot React module after the initial scaffolding and field definitions are complete.

## Prerequisites
- Module scaffolding should be complete with basic file structure
- `fields.tsx` should be fully defined with all necessary module fields
- Types should be defined in `types.ts`

## IMPORTANT: NEVER dangerouslySetInnerHTML
- Never, under any circumstance, is it OK to use `dangerouslySetInnerHTML`
  - Instead you should find another way to implement setting of content.
  - There are plenty of examples on how to correctly do this at the bottom of this file.

## Implementation Steps

### 1. Review Module Structure
1. Confirm the following files exist:
   - `index.tsx` - Main module implementation
   - `fields.tsx` - Field definitions
   - `types.ts` - TypeScript types
   - `assets/` - Directory for module assets
   - `islands/` - Directory for client-side interactive components (if needed)

### 2. Implement Core Module Component
1. Import necessary dependencies:
   ```typescript
   import { ModuleMeta } from '../../types/modules.js';
   import styles from '../component.module.css';
   import { createComponent } from '../../utils/create-component.js';
   import cx, { staticWithModule } from '../../utils/classnames.js';
   // Add other required imports
   ```

2. Define styled components using CSS Modules and createComponent:
   ```typescript
   const swm = staticWithModule(styles);

   const StyledContainer = createComponent('div');
   // inside the CSS Module file:
   // .className-one {
   // max-width: var(--hsElevate--container--maxWidth, 1200px);
   // margin: 0 auto;
   // padding: var(--hsElevate--spacing--48, 48px) var(--hsElevate--spacing--24, 24px);
   // }
   ```

3. Implement the main Component:
   ```typescript
   export const Component = (props: ModuleFields) => {
     // Destructure props
     // Implement component logic
     return (
        <StyledContainer className={cx(swm('className-one'), 'className-two')}>
           {/* Component JSX */}
        </StyledContainer>
     );
   };
   ```

### 3. Island Components (if needed)
If the module requires client-side interactivity:
1. Create an island component in `islands/` directory
2. Use the `?island` suffix when importing
3. Use regular `Island` component from `@hubspot/cms-components`
4. Set appropriate hydration strategy

### 4. Module Metadata
1. Define the module meta information:
   ```typescript
   export const meta: ModuleMeta = {
     label: 'Module Name',
     content_types: ['SITE_PAGE', 'LANDING_PAGE'],
     icon: moduleIconSvg,
     categories: ['design'],
   };
   ```

2. Set module configuration:
   ```typescript
   export const defaultModuleConfig = {
     moduleName: 'elevate/components/modules/module_name',
     version: 0,
     themeModule: true,
   };
   ```

### 5. Styling Guidelines
1. Use HubSpot Elevate CSS variables for:
   - Spacing: `var(--hsElevate--spacing--{size})`
   - Colors: `var(--hsElevate--{context}--{property})`
   - Typography: Apply typography classes from field definitions
2. Ensure responsive design
3. Follow accessibility best practices

### 6. Best Practices
1. Use TypeScript types for all props and data structures
2. Implement proper error handling
3. Use semantic HTML elements
4. Follow React performance best practices
5. Add helpful comments for complex logic
6. Ensure proper data validation

### 7. Testing
1. Test the module with various field configurations
2. Verify responsive behavior
3. Test accessibility
4. Verify island component hydration (if applicable)

### 5. Field Destructuring and Consumption
Example showing proper field destructuring and usage from the SiteHeader module:

```typescript
// Types definition
type MenuModulePropTypes = {
  hublData: {
    navigation: {
      children: MenuDataType[];
    };
    companyName: string;
    defaultLogo: LogoType;
    logoLink: LinkType;
  };
  groupLogo: {
    logo: LogoFieldType;
  };
  defaultContent: {
    logoLinkAriaText: string;
  };
  groupButton: ButtonGroupType;
  styles: StylesType;
};

// Component implementation with proper destructuring
export const Component = (props: MenuModulePropTypes) => {
  // First level destructuring - main groups
  const {
    hublData,
    groupLogo: { logo: logoField },
    defaultContent: { logoLinkAriaText },
    groupButton,
    styles,
  } = props;

  // Second level destructuring - hublData
  const {
    navigation: { children: navDataArray = [] },
    companyName,
    defaultLogo,
    logoLink,
  } = hublData;

  // Destructure button group fields
  const {
    showButton,
    buttonContentText: buttonText,
    buttonContentLink: buttonLink,
    buttonContentShowIcon: showIcon,
    buttonContentIconPosition: iconPosition,
  } = groupButton;

  // Destructure style fields with defaults
  const {
    groupMenu: {
      menuAlignment,
      menuBackgroundColor: { color: menuBackgroundColor } = { color: '#ffffff' },
      menuTextColor: { color: menuTextColor } = { color: '#09152B' },
    },
    groupButton: { buttonStyleVariant, buttonStyleSize },
  } = styles;

  return (
    <SiteHeader>
      <SiteHeaderContainer>
        {/* Use destructured fields */}
        <LogoContainer>
          {showButton && (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
