---
trigger: always_on
description: Trading Simulation App - Production Readiness Rules
---


# 🚀 PRODUCTION READINESS RULES

- All `console.log`, `console.warn`, and `console.error` calls must be removed.
- Use centralized constants (e.g. `TRADING_CONFIG`) for default values like `100000`.
- Avoid hardcoded color strings; use `theme.colors.*` instead.
- Input validation is required on all user entry points (text input, trade actions, etc.).
- Ensure that all portfolio operations are transactional and avoid race conditions.
- All critical async operations should have loading indicators and error fallback UIs.
- Add confirmation modals for destructive actions (e.g., portfolio reset, trade sell).
- Trading components must memoize expensive calculations using `useMemo` and `useCallback`.
- Components triggering network or state changes must wrap critical flows in `try/catch`.
- Ensure cleanup in components using `useEffect` to prevent memory leaks (especially with `WebView`, `Chart`, or `Socket`).

---

# 🧱 ARCHITECTURE & STATE

- Balance logic, calculations, and sync functions should be moved to a shared service (e.g. `BalanceCalculator`).
- No duplication of portfolio sync logic — use a single source of truth (`UserSyncService`).
- Always use `useSelector` with memoized selectors when accessing Redux state.
- Redux actions that affect multiple entities (balance, portfolio) should dispatch atomic updates.

---

# 🔐 SECURITY & AUTH

- Do not perform any Supabase update without validating `uuid`/session token first.
- RLS (Row Level Security) must be enforced via Supabase policies.
- Use `Expo SecureStore` for all token or sensitive value storage.
- No sensitive values (API keys, tokens) should be hardcoded in source files.

---

# 📦 CONFIG & METADATA

- `app.json` must have:
  - `"privacy": "public"`
  - valid `description`, `keywords`, and proper permission usage descriptions.
- Ensure `Info.plist` and `AndroidManifest.xml` declare necessary permissions with justifications.
- Remove unused dependencies: `react-dom`, `dotenv`, `@expo/config-plugins`, `reflect-metadata`, etc.
- Add missing dependencies like `redux`, `lint-staged`.

---

# 🧪 TESTING & STABILITY

- Trading logic must have unit tests in `__tests__/` or `tests/` directory.
- All critical screens (trade execution, portfolio view) must have error boundaries.
- Use Sentry/Bugsnag or equivalent for runtime crash reporting in production.
- Offline functionality must be simulated and tested with `NetInfo` state toggles.

---

# 📲 UI/UX STANDARDS

- Use `theme.colors.*` — never hardcode color strings like `"#ffffff"`.
- Use `SafeAreaView` and responsive padding/margin logic.
- Show `ActivityIndicator` or skeletons during all async fetches.
- Error messages should be user-friendly (not technical stack traces).

---

# 📋 DEPLOYMENT BLOCKERS (Critical Checkpoints)

- [ ] ✅ All `console.log` statements are removed
- [ ] ✅ Privacy policy URL set in `app.json`
- [ ] ✅ Description & Keywords in metadata
- [ ] ✅ Camera/notification permission descriptions added
- [ ] ✅ Production API domains use HTTPS and certificate pinning
- [ ] ✅ No inline API keys or secrets
- [ ] ✅ RLS enforced + input validation complete
- [ ] ✅ Error boundaries added to all flows
- [ ] ✅ App assets (screenshots, icons, descriptions) are ready

---

# 📅 FINAL NOTE

These rules ensure production compliance, store readiness, and long-term maintainability. All rules **must be followed before deployment to App Store or Google Play**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DustinDoan315)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DustinDoan315)
<!-- tomevault:4.0:windsurf_rules:2026-04-08 -->
