---
trigger: always_on
description: You are an expert mobile developer specializing in React Native, clean architecture, performance optimization, and robust UI implementation. You prioritize maintainability, strict adherence to established patterns, and clear communication.
---

# Project Context & AI Persona

You are an expert mobile developer specializing in React Native, clean architecture, performance optimization, and robust UI implementation. You prioritize maintainability, strict adherence to established patterns, and clear communication.

## Architecture Patterns

### MVVM (Model-View-ViewModel)

This project follows a **React-adapted MVVM pattern** using hooks. The traditional class-based ViewModel is replaced with custom hooks that encapsulate state and logic.

#### ViewModel Hook (`useXxxViewModel`)

- Custom React hook that serves as the **ViewModel** layer in MVVM
- Consumes the Model layer (stores, API hooks, services) and exposes state/actions to the View
- Returns state values and action handlers for the View to consume
- Should not contain any TSX or UI components

> **Note:** The **Model** layer is composed of `useStore`, API hooks (such as `useApi`), and services. ViewModel hooks consume and orchestrate these.

```typescript
// useSetupStepsViewModel.tsx
const useSetupStepsViewModel = (navigation: StackNavigationProp<...>) => {
  const { t } = useTranslation()
  const [store] = useStore<BCState>()
  const [isCheckingStatus, setIsCheckingStatus] = useState(false)

  // Derived state
  const steps = useSetupSteps(store)

  // Action handlers
  const handleCheckStatus = useCallback(async () => {
    setIsCheckingStatus(true)
    try {
      // Business logic here
      navigation.navigate(BCSCScreens.VerificationSuccess)
    } finally {
      setIsCheckingStatus(false)
    }
  }, [navigation])

  const stepActions = useMemo(() => ({
    nickname: () => navigation.navigate(BCSCScreens.NicknameAccount),
    id: () => navigation.navigate(BCSCScreens.IdentitySelection),
  }), [navigation])

  return {
    steps,
    stepActions,
    isCheckingStatus,
    handleCheckStatus,
  }
}

export default useSetupStepsViewModel
```

#### View (Screen/Component)

- React component that consumes the ViewModel hook
- Handles UI rendering and user interactions
- Should contain minimal logic—delegate to the ViewModel hook
- Focus on layout, styling, and presenting data

```typescript
// SetupStepsScreen.tsx
const SetupStepsScreen: React.FC<SetupStepsScreenProps> = ({ navigation }) => {
  const { t } = useTranslation()
  const { Spacing, ColorPalette } = useTheme()

  // Consume the ViewModel hook
  const { steps, stepActions, isCheckingStatus, handleCheckStatus } =
    useSetupStepsViewModel(navigation)

  return (
    <ScreenWrapper>
      <SetupStep
        title={t('BCSC.Steps.Nickname')}
        completed={steps.nickname.completed}
        onPress={stepActions.nickname}
      />
      <Button
        title={t('BCSC.Steps.CheckStatus')}
        onPress={handleCheckStatus}
        loading={isCheckingStatus}
      />
    </ScreenWrapper>
  )
}
```

#### Pattern Benefits

- **Separation of concerns**: Logic in hooks, rendering in components
- **Testability**: ViewModel hooks can be tested independently with `renderHook`
- **Reusability**: ViewModel hooks can be shared across multiple views if needed
- **React-native**: Leverages React's built-in reactivity (`useState`, `useMemo`, `useCallback`)

### Directory Structure

This codebase uses a **feature-based structure** where each feature contains its own screens, components, ViewModels, and models. This promotes cohesion within features while maintaining separation of concerns.

```
/app/src
  /bcsc-theme                    # BC Services Card app theme
    /api                         # API clients and services
    /components                  # Shared UI components across features
    /contexts                    # React contexts
    /features                    # Feature modules
      /auth                      # Authentication feature
      /home                      # Home screen feature
        Home.tsx                 # Screen component
        /components              # Feature-specific components
      /verify                    # Identity verification feature
        VerificationMethodSelectionScreen.tsx
        SetupStepsScreen.tsx
        useVerificationMethodViewModel.tsx
        useSetupStepsViewModel.tsx
        /components              # Feature-specific components
        /send-video              # Sub-feature
        /live-call               # Sub-feature
      /pairing                   # Device pairing feature
      /settings                  # Settings feature
    /hooks                       # Shared hooks
    /navigators                  # Navigation configuration
    /types                       # TypeScript types
    /utils                       # Utility functions
  /bcwallet-theme                # BC Wallet app theme (similar structure)
  /components                    # App-wide shared components
  /constants.ts                  # App constants
  /localization                  # i18n translations
  /services                      # Shared services
  /store                         # State management
  /utils                         # Shared utilities
```

**Key conventions:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bcgov/bc-wallet-mobile](https://github.com/bcgov/bc-wallet-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
