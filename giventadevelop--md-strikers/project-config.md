---
trigger: always_on
description: Handles cases where existing user profiles (especially those created via mobile payments) need to be updated with current Clerk user data during sign-in and profile page load.
---


---
description: Comprehensive rules for user profile operations including fetch, creation, and update patterns
globs: src/app/profile/**/*.ts, src/pages/api/proxy/user-profiles/**/*.ts
alwaysApply: true
---

### **User Profile Fetch Operations (4-Step Fallback)**

- **Step 1: Primary Lookup by User ID**
  - Always attempt to fetch profile using `/api/proxy/user-profiles/by-user/{userId}`
  - Use Clerk `currentUser()` to get authenticated user context
  - Return profile immediately if found

- **Step 2: Email-Based Fallback Lookup with Reconciliation**
  - If Step 1 fails (404), extract email from Clerk user object
  - Query using `/api/proxy/user-profiles?email.equals={email}`
  - **NEW: Profile Reconciliation Logic**
    - If profile found by email but `userId` differs from Clerk user ID
    - OR if `firstName`/`lastName` are empty/placeholder values
    - Update profile with current Clerk user data
    - This handles mobile payment profiles and incomplete profiles
  - Validate profile exists and has valid ID before returning
  - Log reconciliation attempts for debugging

- **Step 3: Automatic Profile Creation**
  - If no profile exists, create automatically using Clerk user data
  - Use placeholder values for required fields (no null values)
  - Include all required fields: `userId`, `email`, `firstName`, `lastName`, `tenantId`, `createdAt`, `updatedAt`
  - Use proxy endpoint `/api/proxy/user-profiles` for creation
  - Handle race conditions gracefully (profile might be created by another request)

- **Step 4: Final Fallback**
  - Return `null` if all steps fail
  - This triggers profile form display for manual creation
  - Log comprehensive failure information for debugging

## Profile Reconciliation Logic (New Section)

### Purpose
Handles cases where existing user profiles (especially those created via mobile payments) need to be updated with current Clerk user data during sign-in and profile page load.

### Trigger Points
- **Clerk Sign-In**: Direct client-side reconciliation after successful sign-in (PRIMARY METHOD)
- **Profile Page Load**: When user visits profile page after sign-in (FALLBACK METHOD)
- ~~**Clerk Webhook**: `session.created` webhook (DEPRECATED - unreliable in development)~~

### Scenarios Covered
1. **Mobile Payment Profiles**: Guest profiles with empty names get proper Clerk user data
2. **Incomplete Profiles**: Profiles with placeholder names ('Pending', 'User') get real names
3. **User ID Mismatches**: Profiles with old/guest user IDs get current Clerk user ID

### Reconciliation Conditions
Profile needs reconciliation if ANY of these are true:
- `profile.userId !== currentClerkUserId` (different user ID)
- `profile.firstName` is empty, null, or 'Pending'
- `profile.lastName` is empty, null, or 'User'

### Implementation Patterns

#### Clerk Sign-In Flow (Client-Side Integration) - PRIMARY METHOD
```typescript
// src/components/SignInWithReconciliation.tsx - Custom sign-in wrapper
'use client';
import { SignIn } from "@clerk/nextjs";
import { useUser } from "@clerk/nextjs";
import { useEffect, useState } from "react";

export function SignInWithReconciliation() {
  const { isSignedIn, user } = useUser();
  const [hasTriggeredReconciliation, setHasTriggeredReconciliation] = useState(false);

  useEffect(() => {
    // Trigger profile reconciliation immediately after successful sign-in
    if (isSignedIn && user && !hasTriggeredReconciliation) {
      setHasTriggeredReconciliation(true);

      // Call existing profile reconciliation API endpoint
      fetch('/api/auth/profile-reconciliation', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ triggerSource: 'sign_in_flow' })
      }).then(response => {
        if (response.ok) {
          console.log('Profile reconciliation completed after sign-in');
          setTimeout(() => window.location.href = '/', 1000);
        }
      }).catch(error => {
        console.error('Profile reconciliation failed:', error);
      });
    }
  }, [isSignedIn, user, hasTriggeredReconciliation]);

  return <SignIn redirectUrl="/" />;
}
```

#### ~~Clerk Sign-In Flow (Webhook) - DEPRECATED~~
```typescript
// DEPRECATED: Webhook approach was unreliable in development environments
// Use client-side integration instead (see above)
```

#### Profile Page Load Flow (Server Action)
```typescript
// src/app/profile/ApiServerActions.ts - fetchUserProfileServer Step 2
// Step 2: Fallback to email lookup with reconciliation
const profile = await findProfileByEmail(email);
if (profile && needsReconciliation(profile, userId, currentUser)) {
  const reconciledProfile = await reconcileProfileWithClerkData(profile, userId, currentUser);
  return reconciledProfile;
}
```

### Profile Update Helper Function
```typescript
async function reconcileProfileWithClerkData(
  profile: UserProfileDTO,
  currentClerkUserId: string,
  currentUser: any
): Promise<UserProfileDTO> {
  const updatePayload = {
    id: profile.id,
    userId: currentClerkUserId, // Always update to current Clerk user ID
    updatedAt: new Date().toISOString()
  };

  // Update names if they're empty or different

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
