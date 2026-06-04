---
trigger: always_on
description: This document outlines the architectural patterns and best practices for state management in the ResearchHub codebase.
---

 # ResearchHub State Management Architecture

This document outlines the architectural patterns and best practices for state management in the ResearchHub codebase.

## State Management Hierarchy

1. **Tiered State Strategy**:
   - **Local Component State**: For UI-specific, ephemeral state
   - **Custom Hooks**: For reusable, domain-specific state logic
   - **Context Providers**: For shared, application-wide state
   - **URL/Router State**: For navigation and shareable state

   ```typescript
   // Local component state
   const [isOpen, setIsOpen] = useState(false);
   
   // Custom hook for domain logic
   const { data, isLoading, error } = useDocument(documentId);
   
   // Global context
   const { notificationData, unreadCount } = useNotifications();
   
   // Router state
   const { params } = useParams();
   ```

## React Context

1. **When to Use Context**:
   - Use for state that needs to be accessed by multiple components across different parts of the component tree
   - Appropriate for authentication, theme, notifications, and user preferences
   - Not recommended for state that changes frequently (can cause re-renders)
   - Avoid nesting too many contexts to prevent "wrapper hell"

   ```typescript
   // Good use case for context: authentication state
   const { status, data: session } = useSession();
   const { showAuthModal } = useAuthModalContext();
   
   // Good use case for context: notifications
   const { unreadCount } = useNotifications();
   ```

2. **Context Structure**:
   - Create a dedicated file for each context (`contexts/FeatureContext.tsx`)
   - Define a clear interface for the context value
   - Include both state and actions that modify that state
   - Provide meaningful default values

   ```typescript
   interface FeatureContextType {
     data: DataType[];
     isLoading: boolean;
     error: Error | null;
     actions: {
       fetchData: () => Promise<void>;
       updateItem: (id: string, data: Partial<DataType>) => Promise<void>;
       deleteItem: (id: string) => Promise<void>;
     };
   }
   
   const FeatureContext = createContext<FeatureContextType>({
     data: [],
     isLoading: false,
     error: null,
     actions: {
       fetchData: async () => {},
       updateItem: async () => {},
       deleteItem: async () => {},
     },
   });
   ```

## Context Providers

1. **Provider Implementation**:
   - Use the 'use client' directive for client-side contexts
   - Implement useState or useReducer for state management
   - Use useCallback for functions passed to children
   - Handle loading and error states consistently

   ```typescript
   'use client';
   
   export function FeatureProvider({ children }: { children: ReactNode }) {
     const [data, setData] = useState<DataType[]>([]);
     const [isLoading, setIsLoading] = useState(false);
     const [error, setError] = useState<Error | null>(null);
     
     const fetchData = useCallback(async () => {
       setIsLoading(true);
       setError(null);
       try {
         const result = await FeatureService.getData();
         setData(result);
       } catch (err) {
         setError(err instanceof Error ? err : new Error('Failed to fetch data'));
       } finally {
         setIsLoading(false);
       }
     }, []);
     
     // Additional action functions using useCallback
     
     const value = {
       data,
       isLoading,
       error,
       actions: {
         fetchData,
         // Other actions
       },
     };
     
     return (
       <FeatureContext.Provider value={value}>
         {children}
       </FeatureContext.Provider>
     );
   }
   ```

2. **Context Consumer Hooks**:
   - Create a custom hook for accessing each context
   - Include type checking and error handling
   - Use descriptive names for hooks (`useFeatureContext`)

   ```typescript
   export function useFeatureContext() {
     const context = useContext(FeatureContext);
     if (!context) {
       throw new Error('useFeatureContext must be used within a FeatureProvider');
     }
     return context;
   }
   ```

## State Modeling

1. **State Object Patterns**:
   - Group related state in a single object
   - Include loading, error, and data states for async operations
   - Define clear interfaces for state objects

   ```typescript
   interface FeatureState {
     data: DataType[];
     isLoading: boolean;
     error: Error | null;
     page: number;
     hasMore: boolean;
   }
   
   const [state, setState] = useState<FeatureState>({
     data: [],
     isLoading: false,
     error: null,
     page: 1,
     hasMore: true,
   });
   
   // Update state immutably
   setState((prev) => ({
     ...prev,
     data: [...prev.data, newItem],
   }));
   ```

2. **State Updates**:
   - Use functional updates for state that depends on previous state
   - Maintain immutability in all state updates
   - Group related state updates where possible

   ```typescript
   // Prefer this functional update pattern
   setData((prevData) => [...prevData, newItem]);
   
   // For multiple related state updates
   const handleSuccess = (result) => {
     setData(result.data);
     setHasMore(result.hasMore);
     setPage(result.page);
     setError(null);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResearchHub/web](https://github.com/ResearchHub/web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
