---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ecency Mobile is a React Native client for the Hive blockchain available for iOS and Android. The app provides social features including posts, comments, wallet operations, and community interactions.

## Development Commands

### Setup
```bash
yarn                      # Install dependencies (auto-runs patch-gradle.sh and pod install)
yarn start               # Start Metro bundler
yarn ios                 # Run on iOS simulator/device
yarn android             # Run on Android emulator/device
```

### Linting & Testing
```bash
yarn lint                # Run ESLint on src/
yarn lint:fix            # Auto-fix linting issues
yarn format              # Format with Prettier and lint:fix
yarn test                # Run Jest tests in watch mode
yarn test:ci             # Run Jest tests for CI (no watch, forceExit)
```

### Build Scripts
```bash
yarn bundle:ios          # Bundle JS for iOS
yarn bundle:android      # Bundle JS for Android
yarn patch:gradle        # Re-apply React Native 0.79.5 Gradle patch if needed
```

### Other Utilities
```bash
yarn clear               # Clear all caches and reinstall dependencies
bash patch-gradle.sh     # Apply Gradle patch manually (required for RN 0.79.5)
```

## Architecture

### State Management

**Redux + Redux Persist**: Global state management with persistence via AsyncStorage
- Store: `src/redux/store/store.ts`
- Reducers: `src/redux/reducers/` (account, application, posts, cache, wallet, etc.)
- Redux Persist v6 with migrations in `src/utils/migrationHelpers.ts`

**TanStack Query (React Query)**: Server state management with persistence
- Query setup: `src/providers/queries/index.ts`
- Query keys: `src/providers/queries/queryKeys.ts`
- Custom hooks organized in `src/providers/queries/` (postQueries, walletQueries, etc.)
- Selective persistence strategy for feeds, waves, and notifications

### Navigation

Uses React Navigation v6 with nested navigators:
- **Stack Navigator**: `src/navigation/stackNavigator.tsx` - Main navigation container
- **Drawer Navigator**: `src/navigation/drawerNavigator.tsx` - Side menu
- **Bottom Tabs**: `src/navigation/botomTabNavigator.tsx` - Main tabs (Feed, Waves, Profile, etc.)
- Routes defined in `src/constants/routeNames.ts`

### Data Layer

**Hive Blockchain Integration**:
- Primary Hive client: `src/providers/hive/dhive.ts` (uses @hiveio/dhive and @esteemapp/dhive)
- Client auto-configures with fallback servers from `src/constants/options/api.ts`
- Post parsing utilities: `src/utils/postParser.ts`
  - `parsePost(post, currentUserName, isPromoted)` — all 3 args required
- HiveSigner support: `src/providers/hive/hivesignerAPI.ts`
- HiveAuth wrapper: Uses hive-auth-wrapper for keychain integration

**Ecency Backend APIs**:
- API client: `src/providers/ecency/ecency.ts`
- Provides notifications, drafts, market data, point activities, etc.
- Uses converters in `src/providers/ecency/converters/`

**Local Storage**:
- AsyncStorage wrapper: `src/realm/realm.ts` (historically called "realm", now uses AsyncStorage)
- Stores user credentials, drafts, settings, and cached data
- User data, auth tokens, and drafts are schema-organized

### SDK Migration

**Status**: Fully migrated to `@ecency/sdk` (v2.x) for all queries and mutations

The app uses the centralized `@ecency/sdk` package for all blockchain and API queries, providing:
- **Consistent patterns**: All queries use TanStack Query with SDK query options
- **Type safety**: Full TypeScript support with SDK types
- **Performance**: Automatic caching, deduplication, and background refetching
- **Maintainability**: Single source of truth shared with Ecency Vision web app

**SDK Usage Patterns:**

For React components, use SDK query options directly:
```typescript
import { useQuery } from '@tanstack/react-query';
import { getPostQueryOptions, getAccountFullQueryOptions } from '@ecency/sdk';

// In component:
const { data: post, isLoading } = useQuery(getPostQueryOptions(author, permlink, observer));
const { data: account } = useQuery(getAccountFullQueryOptions(username));
```

For non-React contexts (Redux actions, utilities), use query client:
```typescript
import { getQueryClient, getAccountsQueryOptions } from '@ecency/sdk';

const queryClient = getQueryClient();
const accounts = await queryClient.fetchQuery(getAccountsQueryOptions([username]));
```

For mutations, use mobile mutation wrappers:
```typescript
import { useTransferMutation } from '../providers/sdk/mutations';

function MyScreen() {
  const transfer = useTransferMutation();
  const handleTransfer = async () => {
    await transfer.mutateAsync({ to, amount, memo });
  };
}
```

**Mobile Platform Adapter:**

The mobile app uses `src/providers/sdk/mobilePlatformAdapter.ts` to bridge the SDK with mobile-specific auth:
- Decrypts posting/active keys from encrypted AsyncStorage using PIN
- Opens HiveSigner WebView for OAuth-based signing
- Triggers HiveAuthBroadcastSheet for keychain app signing
- Shows AuthUpgradeSheet when active key is needed but only posting key is available
- Provides temporary active key storage (60s expiry) after auth upgrade


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecency/ecency-mobile](https://github.com/ecency/ecency-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
