---
trigger: always_on
description: Guide for creating new deeplink handlers in MetaMask Mobile. Covers file locations, handler patterns, routing, testing, and common pitfalls.
---


# Creating New Deeplink Handlers

This guide walks you through adding new deeplink handlers to MetaMask Mobile. Follow these steps exactly to ensure your handler integrates correctly with the existing architecture.

## Quick Reference

| File | Purpose |
|------|---------|
| `app/constants/deeplinks.ts` | Define action constants |
| `app/core/DeeplinkManager/handlers/legacy/handleUniversalLink.ts` | Add to SUPPORTED_ACTIONS enum and switch statement |
| `app/core/DeeplinkManager/handlers/legacy/handleYourAction.ts` | Your handler implementation |
| `app/core/DeeplinkManager/handlers/legacy/__tests__/handleYourAction.test.ts` | Unit tests |

## Step 1: Define the Action Constant

Add your action to the `ACTIONS` enum in `app/constants/deeplinks.ts`:

```typescript
export enum ACTIONS {
  // ... existing actions
  YOUR_NEW_ACTION = 'your-new-action',  // URL path: /your-new-action
}
```

**Important**: The enum value becomes the URL path segment (e.g., `https://link.metamask.io/your-new-action`).

Also add a prefix entry if needed:

```typescript
export const PREFIXES = {
  // ... existing prefixes
  [ACTIONS.YOUR_NEW_ACTION]: '',
};
```

## Step 2: Create the Handler File

Create `app/core/DeeplinkManager/handlers/legacy/handleYourAction.ts`:

```typescript
import NavigationService from '../../../NavigationService';
import Routes from '../../../../constants/navigation/Routes';
import DevLogger from '../../../SDKConnect/utils/DevLogger';

interface HandleYourActionParams {
  actionPath: string;
}

/**
 * Interface for parsed navigation parameters
 * Document all supported URL parameters here
 */
interface YourActionNavigationParams {
  someParam?: string;
  anotherParam?: string;
}

/**
 * Parse URL parameters into typed navigation parameters
 */
const parseNavigationParams = (
  actionPath: string,
): YourActionNavigationParams => {
  const urlParams = new URLSearchParams(
    actionPath.includes('?') ? actionPath.split('?')[1] : '',
  );

  return {
    someParam: urlParams.get('someParam') || undefined,
    anotherParam: urlParams.get('anotherParam') || undefined,
  };
};

/**
 * Your action deeplink handler
 *
 * Supported URL formats:
 * - https://link.metamask.io/your-new-action
 * - https://link.metamask.io/your-new-action?someParam=value
 * - https://link.metamask.io/your-new-action?someParam=value&anotherParam=value2
 *
 * @param params Object containing the action path
 */
export const handleYourAction = async ({
  actionPath,
}: HandleYourActionParams) => {
  DevLogger.log(
    '[handleYourAction] Starting deeplink handling with path:',
    actionPath,
  );

  try {
    const navParams = parseNavigationParams(actionPath);
    DevLogger.log('[handleYourAction] Parsed parameters:', navParams);

    // Navigate to your screen with parsed parameters
    NavigationService.navigation.navigate(Routes.YOUR_SCREEN, {
      someParam: navParams.someParam,
      anotherParam: navParams.anotherParam,
    });
  } catch (error) {
    DevLogger.log('Failed to handle your action deeplink:', error);
    // Fallback navigation on error - typically wallet home
    NavigationService.navigation.navigate(Routes.WALLET.HOME);
  }
};
```

### Handler Pattern Requirements

1. **Always use `DevLogger.log`** for debugging (not `console.log`)
2. **Always wrap in try/catch** with fallback navigation
3. **Define TypeScript interfaces** for params and navigation
4. **Document supported URL formats** in TSDoc comments
5. **Parse URLSearchParams** from the path (it includes the `?` prefix)

## Step 3: Add to SUPPORTED_ACTIONS

In `app/core/DeeplinkManager/handlers/legacy/handleUniversalLink.ts`:

### 3a. Import your handler

```typescript
import { handleYourAction } from './handleYourAction';
```

### 3b. Add to SUPPORTED_ACTIONS enum

```typescript
enum SUPPORTED_ACTIONS {
  // ... existing actions
  YOUR_NEW_ACTION = ACTIONS.YOUR_NEW_ACTION,
}
```

### 3c. Add switch case (CRITICAL!)

Add a case in the switch statement (around line 254+):

```typescript
switch (action) {
  // ... existing cases
  case SUPPORTED_ACTIONS.YOUR_NEW_ACTION: {
    handleYourAction({
      actionPath: actionBasedRampPath,
    });
    break;
  }
}
```

> ⚠️ **CRITICAL**: Forgetting this step is a common bug! Your action will be in SUPPORTED_ACTIONS (passing validation) but will silently do nothing without the switch case.

## Step 4: Write Unit Tests

Create `app/core/DeeplinkManager/handlers/legacy/__tests__/handleYourAction.test.ts`:

```typescript
import { handleYourAction } from '../handleYourAction';
import NavigationService from '../../../../NavigationService';
import Routes from '../../../../../constants/navigation/Routes';

jest.mock('../../../../NavigationService', () => ({
  navigation: {
    navigate: jest.fn(),
  },
}));

jest.mock('../../../../SDKConnect/utils/DevLogger', () => ({
  log: jest.fn(),
}));

describe('handleYourAction', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('navigates to YOUR_SCREEN with no parameters', async () => {
    await handleYourAction({ actionPath: '' });

    expect(NavigationService.navigation.navigate).toHaveBeenCalledWith(
      Routes.YOUR_SCREEN,
      expect.objectContaining({}),
    );
  });


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [granjacours410-source/supreme-disco](https://github.com/granjacours410-source/supreme-disco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
