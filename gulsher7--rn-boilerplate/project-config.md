---
trigger: always_on
description: ├── assets/        # Images, icons, fonts, and other static assets
---

# NativeCraft - Project Architecture & Development Rules

## 1. Project Structure
```
src/
├── assets/        # Images, icons, fonts, and other static assets
├── components/    # Reusable UI components
├── config/        # App configuration files
├── context/       # React Context providers (ThemeContext, etc.)
├── hooks/         # Custom React hooks
├── lang/          # i18n translation files
├── models/        # Data models and interfaces
├── navigation/    # Navigation configuration
├── redux/         # State management
├── screens/       # Screen components
├── styles/        # Global styles and themes
├── typings/       # Global TypeScript types
└── utils/         # Utility functions
```

## 2. Development Workflow

### 2.1 Pre-Development Checklist
- [ ] Review Figma design thoroughly
- [ ] Identify reusable components
- [ ] Check existing components for reusability
- [ ] Note all colors used in the design
- [ ] List all static strings for i18n
- [ ] Identify required assets (icons, images)

### 2.2 Implementation Order
1. Add new colors to `styles/colors.ts`
2. Add translations to language files (`en.json` and `ar.json`)
3. Create/update reusable components
4. Implement screen layout
5. Add navigation (if needed)
6. Implement Redux actions/reducers
7. Connect UI to state management
8. Test dark/light themes and RTL/LTR support

## 3. Component Standards

### 3.1 Text & Internationalization

**ALWAYS use TextComp for text**:
```typescript
// ✓ CORRECT: TextComp with translation keys
import TextComp from '@/components/TextComp';

<TextComp text="WELCOME_MESSAGE" />

// For dynamic text without translation:
<TextComp isDynamic text="Hello World" />
```

**NEVER use direct Text component**:
```typescript
// ✗ INCORRECT: Direct Text component
import { Text } from 'react-native';

<Text>Welcome</Text>
```

### 3.2 Translation Keys

Define translation keys in both language files:
```json
// src/lang/en.json
{
    "LOGIN": "Login",
    "WELCOME_MESSAGE": "Welcome to NativeCraft"
}

// src/lang/ar.json
{
    "LOGIN": "تسجيل الدخول",
    "WELCOME_MESSAGE": "مرحبًا بك في NativeCraft"
}
```

Translation key guidelines:
- Use UPPERCASE with underscores for spaces
- Group related keys together
- Use variables with double curly braces:
  ```json
  "HELLO_USER": "Hello {{name}}"
  ```
- Reference variables in components:
  ```tsx
  <TextComp text="HELLO_USER" values={{ name: user.name }} />
  ```

## 4. Theme Management

### 4.1 Using Theme Colors

Access theme colors through the useTheme hook:
```typescript
import { useTheme } from '@/context/ThemeContext';
import { Colors } from '@/styles/colors';

const MyComponent = () => {
  const { theme } = useTheme();
  const colors = Colors[theme];
  
  return (
    <View style={{ backgroundColor: colors.background }}>
      <Text style={{ color: colors.text }}>Hello</Text>
    </View>
  );
};
```

### 4.2 Creating Color Themes

Define colors in `styles/colors.ts` with semantic naming:
```typescript
export const commonColors = {
  primary: '#007AFF',
  success: '#34C759',
  error: '#FF3B30',
  warning: '#FF9500',
};

export const Colors = {
  light: {
    background: '#FFFFFF',
    surface: '#F2F2F7',
    text: '#000000',
    textSecondary: '#3C3C43',
    // Reference common colors
    ...commonColors,
  },
  dark: {
    background: '#000000',
    surface: '#1C1C1E',
    text: '#FFFFFF',
    textSecondary: '#EBEBF5',
    // Reference common colors
    ...commonColors,
  },
};
```

## 5. RTL Support

### 5.1 Using the RTL Hook

```typescript
import useIsRTL from '@/hooks/useIsRTL';

const Component = () => {
  const isRTL = useIsRTL();
  
  return (
    <View style={{
      flexDirection: isRTL ? 'row-reverse' : 'row',
      alignItems: 'center',
    }}>
      {/* Components */}
    </View>
  );
};
```

### 5.2 RTL-Aware StyleSheets

Create RTL-aware styles using custom hooks:
```typescript
const useRTLStyles = (isRTL: boolean, theme: ThemeType) => {
  const colors = Colors[theme];
  
  return useMemo(() => StyleSheet.create({
    container: {
      flexDirection: isRTL ? 'row-reverse' : 'row',
      textAlign: isRTL ? 'right' : 'left',
      // Other styles
    },
    // Other style objects
  }), [isRTL, theme, colors]);
};

// Usage in component:
const styles = useRTLStyles(isRTL, theme);
```

## 6. Component Creation

### 6.1 Props Interface Pattern

```typescript
interface ButtonCompProps {
  text: string;
  onPress: () => void;
  isLoading?: boolean;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'outline';
  style?: ViewStyle;
  textStyle?: TextStyle;
}

const ButtonComp: React.FC<ButtonCompProps> = ({
  text,
  onPress,
  isLoading = false,
  disabled = false,
  variant = 'primary',
  style,
  textStyle,
}) => {
  // Component implementation
};
```

### 6.2 Component Documentation

Use TypeDoc style comments for component documentation:
```typescript
/**
 * A reusable button component with loading state and variants
 * 
 * @component
 * @example
 * <ButtonComp 
 *   text="LOGIN" 
 *   onPress={handleLogin}
 *   variant="primary"
 *   isLoading={isLoading}
 * />
 */
const ButtonComp: React.FC<ButtonCompProps> = (props) => {
  // Component implementation
};
```

## 7. File Naming & Organization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gulsher7/rn_boilerplate](https://github.com/gulsher7/rn_boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
