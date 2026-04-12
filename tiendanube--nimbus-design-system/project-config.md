---
trigger: always_on
description: React component development standards and best practices. Used when developing new components or modifying existing ones.
---

# React Component Development Rules

## Component Structure

Common directory and file organization:

```
src/
├── atomic|composite/
│   ├── <component-name>/
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── <component-name>/
│   │   │   │   │   ├── <component-name>.tsx
│   │   │   │   │   ├── <component-name>.types.ts
│   │   │   │   │   ├── <component-name>.spec.tsx
│   │   │   │   │   ├── <component-name>.stories.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   └── index.ts
│   │   │   ├── contexts/
│   │   │   │   ├── <context-name>/
│   │   │   │   │   ├── <context-name>.types.ts
│   │   │   │   │   ├── <context-name>.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   └── index.ts
│   │   │   ├── hooks/
│   │   │   │   ├── <hook-name>/
│   │   │   │   │   ├── <hook-name>.types.ts
│   │   │   │   │   ├── <hook-name>.spec.tsx
│   │   │   │   │   ├── <hook-name>.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   └── index.ts
│   │   │   ├── <component-name>.tsx
│   │   │   ├── <component-name>.types.ts
│   │   │   ├── <component-name>.spec.tsx
│   │   │   ├── <component-name>.stories.tsx
│   │   │   └── index.ts
│   │   ├── index.ts
│   │   ├── tsconfig.json
│   │   ├── webpack.config.ts
│   │   ├── CHANGELOG.md
│   │   ├── README.md
│   │   └── package.json
│   └── ...
```

1. **File Organization**:
   - One component per file
   - Component file should match component name
   - Keep related files (types, tests, stories) in the same directory

2. **Component Design**:
   - Focus on reusability and composability
   - Follow single responsibility principle
   - Use proper prop interfaces
   - Implement proper TypeScript types. The types should be in a separate file with the same name as the component but with the suffix ".types.ts".
   - **IMPORTANT** Constants declared within the package must be exported in order to be included in the final published build.

## Styling Guidelines

1. **Style Implementation**:
   - ALWAYS use `@nimbus-ds/styles` package for styling
   - NEVER use custom SCSS/CSS
   - Leverage theme tokens and design system utilities
   - Follow the design system's styling patterns

2. **Theme Integration**:
   - Use theme tokens for colors, spacing, typography
   - Implement responsive design using theme breakpoints
   - Follow the design system's theming patterns

## Accessibility

1. **WCAG Compliance**:
   - Follow WCAG 2.1 guidelines
   - Implement proper ARIA attributes
   - Ensure keyboard navigation support
   - Maintain proper focus management

2. **Cross-Browser Support**:
   - Ensure cross-browser compatibility
   - Implement progressive enhancement
   - Test across major browsers

## Documentation

1. **JSDoc Comments**:
   - Document all components with JSDoc
   - Include component description
   - Document all props with types and descriptions
   - Explain complex logic
   - Add relevant references

2. **Storybook Documentation**:
   - Create comprehensive stories
   - Document component usage
   - Include examples of different states
   - Add controls for all props

## Testing

1. **Test Coverage**:
   - Write Jest unit tests for all components
   - Test all component states
   - Test accessibility features
   - Test edge cases

2. **Performance**:
   - Implement proper memoization
   - Optimize render cycles
   - Handle large lists efficiently
   - Monitor bundle size

## Example Structure

import { button } from "@nimbus-ds/styles";
import { Box } from "@nimbus-ds/styles";
import { ButtonProperties } from './Button.types';

/**
 * Button component for user interactions
 * @component
 * @example
 * <Button variant="primary" onClick={handleClick}>
 *   Click me
 * </Button>
 */
export const Button: React.FC<ButtonProperties> = ({
  text,
  variant = 'primary',
  disabled = false,
  ...props
}) => {
  return (
    <Box
      as="button"
    <Box
      as="button"
      {/* Uses Nimbus button styling patterns for consistent appearance */}
      className={button.classnames.appearance[variant]}
      variant={variant}
      disabled={disabled}
      {...props}
    >
      variant={variant}
      disabled={disabled}
      {...props}
    >
      {text}
    </Box>
  );
};

## Maintenance

1. Keep components focused and maintainable
2. Update documentation with changes
3. Ensure proper test coverage
4. Monitor performance metrics
5. Review accessibility compliance
6. Follow design system updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TiendaNube)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TiendaNube)
<!-- tomevault:4.0:windsurf_rules:2026-04-08 -->
