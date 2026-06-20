---
trigger: always_on
description: Enforce developer experience principles (SRP, hook architecture, compound component composition, UX polish) for React Native Expo projects. Use this skill proactively when writing or reviewing code that involves more than one component or more than one piece of logic.
---


# /dx-expert - Developer Experience Expert

Enforce Single Responsibility Principle, clean hook architecture, and component composition patterns in **React Native Expo** to maintain excellent developer experience without performance issues.

**Target platform:** React Native with Expo. Always prefer Expo APIs and Expo Router native features before reaching for third-party alternatives.

## When to Apply

Apply these principles **automatically** when:
- Creating or modifying more than one component
- Writing logic that spans multiple concerns
- Reviewing code that mixes state logic with rendering
- Refactoring existing code for clarity

## Core Principles

### 1. Single Responsibility Principle (SRP)

Separate **state logic** from **rendering logic**. Components render. Hooks manage state.

```typescript
// BAD - Mixed concerns
export const AppointmentCard = ({ appointmentId }: Props) => {
  const [appointment, setAppointment] = useState<Appointment | null>(null);
  const [isExpanded, setIsExpanded] = useState(false);
  const [barber, setBarber] = useState<Barber | null>(null);

  useEffect(() => {
    fetchAppointment(appointmentId).then(setAppointment);
  }, [appointmentId]);

  useEffect(() => {
    if (appointment?.barberId) {
      fetchBarber(appointment.barberId).then(setBarber);
    }
  }, [appointment?.barberId]);

  const handleToggle = () => setIsExpanded(prev => !prev);

  return (
    <View>
      <Text>{appointment?.clientName}</Text>
      {isExpanded && <Text>{barber?.name}</Text>}
      <Pressable onPress={handleToggle}>
        <Text>Toggle</Text>
      </Pressable>
    </View>
  );
};

// GOOD - Separated concerns
export const AppointmentCard = ({ appointmentId }: AppointmentCardProps) => {
  const { appointment, barber, isExpanded, handleToggle } = useAppointmentCard({ appointmentId });

  return (
    <View>
      <Text>{appointment?.clientName}</Text>
      {isExpanded && <Text>{barber?.name}</Text>}
      <Pressable onPress={handleToggle}>
        <Text>Toggle</Text>
      </Pressable>
    </View>
  );
};
AppointmentCard.displayName = 'AppointmentCard';
```

---

### 2. Hook Architecture

#### 2a. One Hook Per File

Every hook lives in its own file. Group related hooks in folders.

```
hooks/
├── useAppointmentCard/
│   ├── index.ts
│   ├── useAppointmentCard.ts
│   ├── useExpandToggle.ts        # Small, focused sub-hook
│   └── types.ts
```

#### 2b. Small and Focused

Each hook does **one thing only**. If a hook grows beyond ~50-80 lines, split it.

```typescript
// BAD - Hook doing too many things
const useAppointmentForm = ({ appointmentId }: Args) => {
  // 200+ lines of fetching, validation, submission, formatting...
};

// GOOD - Composed small hooks
const useAppointmentForm = ({ appointmentId }: UseAppointmentFormArgs) => {
  const { appointment } = useAppointmentQuery({ appointmentId });
  const { form, handleSubmit } = useAppointmentFormState({ appointment });
  const { timeSlots } = useAvailableSlots({ barberId: appointment?.barberId });

  return { appointment, form, handleSubmit, timeSlots };
};
```

#### 2c. Single Object Argument

Hooks **always** receive a single object as argument. Never loose parameters.

```typescript
// BAD - Loose parameters
const useClientSearch = (query: string, filters: Filters, page: number) => { ... };

// GOOD - Single object argument
interface UseClientSearchArgs {
  query: string;
  filters: Filters;
  page: number;
}

const useClientSearch = ({ query, filters, page }: UseClientSearchArgs) => { ... };
```

Even with a single parameter, use an object for consistency and extensibility:

```typescript
// BAD
const useBarberDetails = (barberId: string) => { ... };

// GOOD
const useBarberDetails = ({ barberId }: UseBarberDetailsArgs) => { ... };
```

---

### 3. useEffect Rules

**Avoid useEffect whenever possible.** It is one of the main sources of memory leaks.

#### When NOT to use useEffect:
- Deriving state from props or other state (use `useMemo` or compute inline)
- Responding to user events (use event handlers)
- Transforming data for rendering (compute during render)
- Syncing with external stores (use `useSyncExternalStore`)

```typescript
// BAD - Derived state in useEffect
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// GOOD - Compute inline or with useMemo
const fullName = `${firstName} ${lastName}`;
```

#### When useEffect IS acceptable:
- Syncing with external systems (subscriptions, native modules)
- Cleanup on unmount (event listeners, timers)
- Cases where no alternative exists

#### If using useEffect:
1. Keep the dependency array **fully controlled and explicit**
2. Keep the effect body **small and focused** (< 10 lines)
3. Always include cleanup when needed
4. Never ignore exhaustive-deps warnings

```typescript
// Acceptable - External system sync
useEffect(() => {
  const subscription = eventEmitter.addListener('event', handler);
  return () => subscription.remove();
}, [handler]);
```

---

### 4. Memoization Strategy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgustinOberg/dx-expert-skill](https://github.com/AgustinOberg/dx-expert-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
