---
trigger: always_on
description: Rules for the React Web frontend application (apps/web)
---


# Cursor Agent Rules for CaseAI Connect - Web

## Redux & Feature Architecture

### API Calls Must Go Through Redux + Services

**Rule**: All API calls MUST go through React-Redux thunks, which in turn MUST use the shared `services` object. React components MUST NOT call the API client (Axios, `fetch`, etc.) directly.

**Requirements**:
- All API calls must use Redux thunks (`createAsyncThunk`)
- Thunks call `extra.services.{feature}` methods only (never `axios` / `fetch` directly)
- Components dispatch thunks, never call API directly
- API routes and DTOs come from `@caseai-connect/api-contracts`
- The Axios client is centralized in `external/axios.ts`
- The feature services registry is centralized in `external/axios.services.ts` and exposed via `di/services.ts`

**Structure**:
- Redux slices: `features/{domain}/{domain}.slice.ts`
- Redux thunks: `features/{domain}/{domain}.thunks.ts`
- Redux selectors: `features/{domain}/{domain}.selectors.ts`
- Axios client: `external/axios.ts` (singleton Axios with auth interceptors)
- Services registry: `external/axios.services.ts` (builds concrete services) and `di/services.ts` (typed `Services` + `getServices()`)

**Store wiring**:
- `store/index.ts`:
  - Registers feature reducers under their domain keys (e.g. `me`, `projects`, `organizations`, `agents`, etc.)
  - Configures `thunk.extraArgument` with `{ services: getServices() }` (typed as `ThunkExtraArg`)
  - Middleware and listeners must assume all side-effectful work goes through these services

### Feature Service Pattern (SPI + External API + Models)

**Rule**: Each feature MUST follow the "me" feature architecture as the canonical pattern for defining and organizing objects inside a feature.

**Canonical Example (Me Feature)**:
- `features/me/me.models.ts`
  - Defines domain-level types (`User`, `Me`, etc.)
  - These are the types slices and components should depend on (not raw DTOs)
- `features/me/me.spi.ts`
  - Declares the Service Provider Interface for the feature (`IMeSpi`)
  - Only exposes domain models (`Me`) in its API surface
- `features/me/external/me.api.ts`
  - Concrete implementation of the SPI using Axios + `@caseai-connect/api-contracts`
  - Uses `satisfies IMeSpi` to ensure the implementation matches the SPI contract
  - Performs DTO → domain mapping via small helpers (e.g. `fromDto(dto: MeResponseDto): Me`)
- `external/axios.services.ts`
  - Wires the feature implementation into the global `services` object:
    - `me: meApi` (where `meApi` is the SPI implementation from `features/me/external/me.api.ts`)
- `di/services.ts`
  - Declares the `Services` type (e.g. `me: IMeSpi`, `projects: IProjectsApi`, etc.)
  - Exposes `getServices()` which returns the concrete `services` instance
- `features/me/me.thunks.ts`
  - Uses `createAsyncThunk<Me, void, { state: RootState; extra: ThunkExtraArg }>`
  - Accesses the SPI through `extra.services.me` only
- `features/me/me.slice.ts`
  - Stores domain state based on `Me` / `Me["user"]` types
  - Handles pending/fulfilled/rejected states of the feature thunks
- `features/me/me.selectors.ts`
  - Selectors read from the slice and return domain models, not DTOs

**Requirements for New or Refactored Features**:
- Define **domain models** in `features/{domain}/{domain}.models.ts`
  - Components and slices should use these models, not raw DTOs
- Define a **feature SPI** in `features/{domain}/{domain}.spi.ts`
  - The interface should expose domain models and hide transport details
- Implement the SPI in `features/{domain}/external/{domain}.api.ts`
  - Use `@caseai-connect/api-contracts` routes and DTOs
  - Perform DTO → domain mapping in small, explicit functions
  - Ensure the implementation `satisfies I{Domain}Spi`
- Register the implementation in `external/axios.services.ts`
  - Add `{domain}: {domain}Api` to the `services` object
- Update `di/services.ts`
  - Add the feature to the `Services` type
- Write thunks in `features/{domain}/{domain}.thunks.ts`
  - Use `createAsyncThunk<DomainModel, ...>` and call `extra.services.{domain}`
- Keep Redux slices and selectors feature-local, typed on the **domain models**

**Migration Guidance**:
- Legacy features (e.g. `projects`, `organizations`, `agents`, `test`) that currently:
  - Define APIs in `services/{domain}.ts`
  - Return DTOs directly to slices
  - Are wired via `build{Domain}Api(getAxiosInstance())` in `external/axios.services.ts`
- SHOULD be refactored over time to:
  - Introduce `*.models.ts` + `*.spi.ts` + `external/*.api.ts`
  - Use domain models internally and map from DTOs at the edge
  - Use `satisfies I{Domain}Spi` for API implementations
  - Mirror the "me" feature structure and patterns

**Example**:
```typescript
// ✅ Correct - Component dispatches thunk
const dispatch = useAppDispatch()
dispatch(fetchMe())

// ❌ Wrong - Component calls API directly
const response = await fetch('/me')
```

## Form Component Architecture

### Separation of Create and Update Forms

**Rule**: A `CreateXXXForm` component MUST NEVER be used for both creating and updating actions. Always create separate components for create and update operations, and extract shared form logic into a shared component.

**Requirements**:
- `CreateXXXForm` should ONLY handle creation logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bayesimpact/agent-studio](https://github.com/bayesimpact/agent-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
