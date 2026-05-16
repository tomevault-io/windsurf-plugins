---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QvaPay is a React Native mobile fintech app (v0.83.1, React 19.2.4) providing a non-custodial wallet, P2P marketplace, and crypto payment gateway for underbanked regions in Latin America and the Caribbean. The backend API lives at `~/webs/qpweb` (Next.js 16).

## Common Commands

```bash
# Development
npm run android          # Run on Android (auto-syncs version first)
npm run ios              # Run on iOS (auto-syncs version first)
npm run ios:build        # Build iOS (iPhone 16 simulator, auto-syncs version)
npm run pods             # Install CocoaPods (required after native dep changes)
npm run start            # Start Metro bundler

# Quality
npm run lint             # Run ESLint (@react-native config)
npm run test             # Run Jest tests (react-native preset)
npx jest --testPathPattern="path/to/test"  # Run a single test file

# Release (Android)
npm run android:bundle   # Bundle release AAB
npm run android:apk      # Build release APK
npm run android:release  # Bundle + APK
npm run android:apk:release  # Full release script (scripts/release-android.sh)

# Utilities
npm run version:sync     # Sync version across platforms (runs auto before ios/android)
```

**Node.js requirement**: >= 20

## Architecture

### State Management (Context API)
- **AuthContext** (`/auth/AuthContext.js`): Authentication state, token in AsyncStorage (keychain commented out), 30s validation checks
  - State: `isAuthenticated`, `user`, `token`, `isLoading`, `error`
  - Functions: `login()` (handles 202 2FA + 200 success), `logout()`, `register()`, `confirmRegistration()`, `requestPin()`, `updateUser()`, `checkToken()`, `initializeAuth()`
  - Failed login throttle: 60s lockout after 5+ failures
- **SettingsContext** (`/settings/SettingsContext.js`): App-wide settings (notifications, security, privacy, appearance, language, transactions, p2p, sounds). Supports import/export. Granular AsyncStorage keys.
- **ThemeContext** (`/theme/ThemeContext.js`): Light/dark/auto theme with memoized styles (`useTextStyles()`, `useContainerStyles()`). Listens to system appearance.

Provider hierarchy in `App.tsx`:
```
ThemeProvider > SettingsProvider > AuthProvider > NavigationContainer
```

### Navigation (React Navigation v7)
Routes defined in `/routes.js` (40+ named routes). Structure:
```
AppNavigator (Stack)
  Onboard (first-time)
  Welcome (unauthenticated)
  MainStack (authenticated) -> Bottom Tabs
    Home | Invest | Keypad | P2P | Store
  Feature screens: Add, Withdraw, P2PCreate, P2POffer,
    Send/SendConfirm/SendSuccess, Receive, Transaction(s),
    Scan, PhoneTopup, SettingsStack, Help
  Auth: Login, Register, RecoverPassword, Recover2FA
```

### API Layer (`/api/`)
Axios client (`client.js`) with:
- Base URL: `http://192.168.0.114:3000/api` (dev hardcoded) / `https://api.qvapay.com` (prod, commented out)
- Timeout: 20s. Headers: `X-QvaPay-Client`, `User-Agent`, version, platform, build
- Request interceptor: Bearer token from AsyncStorage
- Response interceptor: 401/403 clears token, Spanish error messages
- All modules return `{ success, data, error?, status? }`

**API Modules:**
- `authApi.js`: login (with 2FA), register, confirmRegistration, requestPin, checkToken, logout, resetPassword
- `userApi.js`: searchUser, getUserProfile (`/user/extended`), updateUser, KYC (submitInfo, uploadPicture, getStatus), verifyPhone, removePhone, telegram/2FA, password, deletion, payment methods, contacts, referrals, gold
- `p2pApi.js`: index (filters: type, coin, amount, ratio, KYC/VIP), show, create, cancel, markPaid, confirmReceived, getChat, sendChat, rateOffer
- `transferApi.js`: getLatestTransactions, getLatestSentTransfers, transferMoney (with PIN), getTransactionDetails, getTransactionPDF
- `withdrawApi.js`: preWithdraw (request PIN), withdraw (with PIN)
- `storeApi.js`: phonePackages, purchasePhonePackage
- `coinsApi.js`: index (enabled_in/enabled_out filters)
- `blogApi.js`: WordPress REST API via native fetch (not axios)

### Theme System
```javascript
const { theme } = useTheme()
const textStyles = createTextStyles(theme)
const containerStyles = createContainerStyles(theme)
```

Colors: primary `#6759EF`, success `#7BFFB1`, danger `#DB253E`, warning `#ff9f43`, gold `#FFD700`
Dark (default): bg `#0E0E1C`, surface `#1E2039` | Font: Rubik family

### Key Directories
- `/screens/`: 38 screens by domain (home, p2p, store, transaction, settings)
- `/ui/`: Composite (BottomBar, BalanceCard, P2POfferItem, AmountInput)
- `/ui/particles/`: Atomic (QPButton, QPInput, QPAvatar, QPBalance, QPCoin, QPTransaction, QPRate, QPPill, QPLoader, QPSwitch, SettingsItem)
- `/auth/`: AuthContext + screens (Login, Register, RecoverPassword, Recover2FA)
- `/api/`: 8 modules + client
- `/theme/`: ThemeContext + themeUtils
- `/settings/`: SettingsContext
- `/helpers.js`: Utilities (timeAgo, parseQRData, dates) - Spanish locale
- `/assets/`: Images, fonts, Lottie animations

### Key Dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qvapay/mobile_app](https://github.com/qvapay/mobile_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
