---
trigger: always_on
description: This is a React TypeScript application with Material-UI integration for a banking signup form. The app uses Supabase for authentication and follows modern React patterns.
---

# Cursor Rules for Rawbank Signup App

## Project Overview

This is a React TypeScript application with Material-UI integration for a banking signup form. The app uses Supabase for authentication and follows modern React patterns.

## Architecture Principles

### 1. Self-Contained Components

- **PRIORITY**: Always create self-contained, reusable components
- Each component should have its own state management and styling
- Components should be composable and not tightly coupled
- Use TypeScript interfaces for component props
- Export components as default exports with proper naming conventions

```typescript
// Example: Self-contained component structure
interface ComponentProps {
  title: string;
  onSubmit: (data: FormData) => void;
  loading?: boolean;
}

const ComponentName: React.FC<ComponentProps> = ({ title, onSubmit, loading = false }) => {
  // Component logic here
  return (
    // JSX here
  );
};

export default ComponentName;
```

### 2. Custom Hooks for Data Access

- **PRIORITY**: Extract all data fetching and state management into custom hooks
- Create hooks for Supabase operations, form state, and API calls
- Use React Query or SWR for server state management when applicable
- Hooks should be pure functions that return state and actions

```typescript
// Example: Custom hook structure
const useCustomHook = (param: string) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    // Data fetching logic
  }, [param]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
};
```

### 3. Material-UI Skeleton Loading

- **PRIORITY**: Always use Material-UI Skeleton components for loading states
- Replace loading spinners with skeleton placeholders that match the actual content
- Use appropriate skeleton variants (text, rectangular, circular)
- Implement progressive loading for better UX

```typescript
import { Skeleton } from "@mui/material";

// Example: Skeleton loading implementation
const LoadingComponent = () => (
  <Box>
    <Skeleton variant="text" width="60%" height={40} />
    <Skeleton variant="rectangular" width="100%" height={200} sx={{ mt: 2 }} />
    <Skeleton variant="circular" width={40} height={40} sx={{ mt: 2 }} />
  </Box>
);
```

### 4. Vector Illustrations

- **PRIORITY**: Add vector illustrations when necessary for better UX
- Use SVG icons from Material-UI or custom SVG components
- Illustrations should be responsive and accessible
- Consider using illustrations for empty states, success states, and onboarding

```typescript
// Example: Vector illustration component
const IllustrationIcon: React.FC<{ size?: number; color?: string }> = ({
  size = 64,
  color = "primary.main",
}) => (
  <Box sx={{ width: size, height: size, color }}>
    <svg viewBox="0 0 24 24" fill="currentColor">
      {/* SVG path data */}
    </svg>
  </Box>
);
```

### 5. User Experience for Non-Tech-Savvy Users

- **CRITICAL**: Most users are not tech-savvy - prioritize simplicity and guidance
- **PRIORITY**: Use clear, simple language - avoid technical jargon
- **PRIORITY**: Add helpful icons to ALL input fields using Material-UI InputAdornment
- **PRIORITY**: Include helper text below each field to guide users
- **PRIORITY**: Use vector illustrations at the start of each major section
- **PRIORITY**: Provide clear visual feedback for success, error, and loading states
- **PRIORITY**: Use tooltips for additional context without cluttering the UI
- **PRIORITY**: Implement inline validation with clear, friendly error messages
- **PRIORITY**: Show progress indicators so users know where they are in the process

```typescript
// Example: Input field with icon and helper text
<TextField
  label="Numéro de téléphone"
  placeholder="+243 XXX XXX XXX"
  helperText="Nous utiliserons ce numéro pour vous envoyer des notifications importantes"
  InputProps={{
    startAdornment: (
      <InputAdornment position="start">
        <PhoneIcon color="primary" />
      </InputAdornment>
    ),
  }}
/>

// Example: Section with illustration
<Box sx={{ textAlign: 'center', mb: 3 }}>
  <PersonIcon sx={{ fontSize: 80, color: 'primary.main', mb: 2 }} />
  <Typography variant="h5" gutterBottom>
    Informations Personnelles
  </Typography>
  <Typography variant="body2" color="text.secondary">
    Ces informations nous aident à créer votre compte en toute sécurité
  </Typography>
</Box>
```

#### UX Guidelines for Forms:

- Use large, easy-to-read fonts
- Provide clear labels with icons
- Show example formats in placeholders
- Use helper text to explain why information is needed
- Group related fields with clear headings and illustrations
- Provide immediate feedback on field validation
- Use green checkmarks for valid fields
- Use friendly, non-technical error messages
- Show password strength indicators
- Provide "Show/Hide" toggles for password fields
- Use date pickers instead of manual date entry
- Use dropdowns for limited options instead of free text
- Provide autocomplete suggestions where applicable

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [besongsamuel/rawbank-signup](https://github.com/besongsamuel/rawbank-signup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
