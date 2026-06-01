---
trigger: always_on
description: - Follow the Angular Conventional Commits format: `<type>(<scope>): <subject>`
---

# Implementation Guidelines

## Commit Message Convention

- Follow the Angular Conventional Commits format: `<type>(<scope>): <subject>`
- Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`
- Scope is optional but recommended (for example: `auth`, `api`, `ui`)
- Subject must be imperative, lowercase, without a trailing period, and roughly 50 characters
- Wrap commit body lines near 72 characters and include footers such as `BREAKING CHANGE:` or `Closes #123` when needed

## Tooling & Package Management

- **Use Bun exclusively.** Run installs with `bun install`, scripts with `bun run <script>`, add deps via `bun add` / `bun add -d`.
- Do **not** suggest or create `package-lock.json` or `yarn.lock`; `bun.lock` is the single source of truth.

## Expo-Specific Guidelines

### iOS Pod Configuration

**CRITICAL WARNING**: Never modify the iOS platform version in `ios/ExpoIap.podspec`

- iOS platform version MUST remain at `13.4` even though the code requires iOS 15.0+
- Changing iOS to `15.0` can cause expo prebuild to exclude the module in certain Expo SDKs (known bug)
- See issue: [#168](https://github.com/hyochan/expo-iap/issues/168)
- This is kept at `13.4` for compatibility with affected Expo SDKs
- The actual iOS 15.0+ requirement is enforced at build time via @available annotations
- Users must ensure their app target is set to iOS 15.0 or higher:
  - app.json: `"expo": { "ios": { "deploymentTarget": "15.0" } }`
  - or app.config.ts: `ios: { deploymentTarget: '15.0' }`

**tvOS Exception**: tvOS platform version MUST be `16.0` (not `13.4`) because:

- The `openiap` CocoaPods dependency requires tvOS 16.0 minimum
- Setting tvOS to `13.4` causes build failure due to dependency mismatch
- This is a hard requirement from the dependency, not subject to the iOS workaround

### Pre-Commit Checks

Before committing any changes:

1. Run `bun run lint` to ensure code quality
2. Run `bun run typecheck` to verify TypeScript types
3. Run `bun run test` to verify all tests pass (Note: Use `bun run test`, not `bun test`)
4. **IMPORTANT**: Run tests in the example directory as well:
   - `cd example && bun run test`
   - Ensure all tests pass with 100% success rate
   - Fix any failing tests before committing
5. Only commit if all checks succeed

### Platform-Specific Naming Conventions

#### Function Naming

- Functions that only operate on one platform must carry the suffix: `nameIOS` or `nameAndroid` (e.g. `getStorefrontIOS`, `deepLinkToSubscriptionsAndroid`).
- Cross-platform helpers should expose a single name and branch internally via `Platform.select` or equivalent.

#### Field Naming

- **iOS-related fields**: Use `IOS` suffix (e.g., `displayNameIOS`, `discountsIOS`, `introductoryPriceIOS`)
  - **Exception**: When an acronym appears at the end of a field name, use uppercase (e.g., `quantityIOS`, `appBundleIdIOS`, not `quantityIos`)
  - Platform-specific fields: `currencyCodeIOS`, `currencySymbolIOS`, `countryCodeIOS`
  - Product fields: `isFamilyShareableIOS`, `jsonRepresentationIOS`, `subscriptionInfoIOS`
- **Android-related fields**: Use `Android` suffix (e.g., `nameAndroid`)
  - Platform-specific fields: `oneTimePurchaseOfferDetailsAndroid`, `subscriptionOfferDetailsAndroid`
  - In **Android-specific types** (e.g., `ProductSubscriptionAndroidOfferDetails`), keep `pricingPhases` without suffix for consistency with Google Play Billing
  - In **cross-platform types** (e.g., `SubscriptionOffer`, `DiscountOffer`), use `pricingPhasesAndroid` suffix to distinguish from common fields
- **Common fields**: Fields shared across platforms go in Common types (e.g., `ids`, `platform`, `debugDescription`)
  - Use these for data that exists on both platforms without platform-specific variations

#### Type Naming

- **iOS types**: Use `IOS` suffix (e.g., `PurchaseIOS`, `ProductIOS`)
- **Android types**: Use descriptive prefixes to identify subtypes:
  - ✅ Good: `ProductAndroidOneTimePurchaseOfferDetail`, `ProductSubscriptionAndroidOfferDetails`, `PurchaseAndroidState`
  - ❌ Avoid: `OneTimePurchaseOfferDetails`, `SubscriptionOfferAndroid`, `PurchaseStateAndroid`
- **General IAP types**: Use `Iap` prefix (e.g., `IapPurchase`, not `IAPPurchase`)

#### General Rules

- **ID fields**: Use `Id` instead of `ID` (e.g., `productId`, `transactionId`, not `productID`, `transactionID`)
- **Consistent naming**: This applies to functions, types, and file names
- **Deprecation**: Fields without platform suffixes will be removed in v2.9.0

### Type System

For complete type definitions and documentation, see: <https://www.openiap.dev/docs/types>

The library follows the OpenIAP type specifications with platform-specific extensions using iOS/Android suffixes.

> **Important:** `src/types.ts` is generated from the OpenIAP schema. Never edit this file manually or commit hand-written changes. After updating any `*.graphql` schema, run `bun run generate:types` (or the equivalent script in your package manager) to refresh the file.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyochan/expo-iap](https://github.com/hyochan/expo-iap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
