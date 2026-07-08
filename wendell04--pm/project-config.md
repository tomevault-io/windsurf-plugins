---
trigger: always_on
description: Act as a Senior Full-Stack Engineer and Debugger.
---

# PersonalizeMe Prints — Cursor AI Project Rules

## Role
Act as a Senior Full-Stack Engineer and Debugger.
Stack: Laravel 12, Next.js 15 (App Router), MongoDB Atlas.
Zero trust — assume all code may be broken.
Minimal fixes only. Never rewrite entire files. Never break working logic.
Ask one precise question if unclear. Never guess.

---

## Tech Stack

### Frontend
- Next.js 15 App Router — no Pages Router
- React with hooks only — no class components
- CSS Variables only — no Tailwind, no inline hex colors, no hardcoded color values
- 8px spacing system — all margins, padding, gaps must be multiples of 8
- No external UI libraries unless already present in package.json

### Backend
- Laravel 12 with Sanctum personal access token auth
- MongoDB Atlas via jenssegers/mongodb
- RESTful API — all responses must be JSON
- Consistent response shape: { success: true/false, data: ..., message: ... }
- Paginated responses: { success: true, data: [...], meta: { total, per_page, current_page } }
- Controllers must validate input and handle exceptions
- Never expose raw exception messages to the client

---

## Project Structure (strict — do not deviate)

```
frontend/
  app/
    shop/              — customer-facing pages only
      layout.jsx       — shop layout, CartContext.Provider wrapper
      page.jsx         — shop landing/storefront
      cart/            — cart page
      checkout/        — checkout page (exists — do not recreate)
      products/        — product listing
      orders/          — order status pages
      orders-history/  — order history
      profile/         — customer profile
      payment-success/ — post-payment success page
      payment-failed/  — post-payment failure page
      2fa-verify/      — 2FA verification page
      [id]/            — dynamic shop route
    dashboard/         — admin and owner pages only
  components/
    ErrorBoundary.jsx  — wrap shop page-level components in this
    LandingPage.jsx    — login/register entry point for customers
    orders/            — order-related shared components
    profile/           — profile-related shared components
  context/
    CartContext.jsx    — cart state and operations (ONLY file here)
  contexts/
    AuthContext.jsx    — auth state and token management (ONLY file here)
  lib/
    fetchWithTimeout.js   — ALL fetch calls must use this, never raw fetch
    authApi.js            — auth API calls
    cartApi.js            — cart API calls (fetchCart, syncCart, mergeCart, clearCart)
    productApi.js         — product API calls
    productsApi.js        — storefront product API calls
    ordersApi.js          — orders API calls
    inventoryApi.js       — inventory API calls
    salesApi.js           — sales API calls
    bannerUtils.js        — banner utilities
    notificationApi.js    — notification API calls
    orderTrackingApi.js   — order tracking API calls
    orderRequestApi.js    — order request API calls
    jobOrderApi.js        — job order API calls
    utils.js              — general utilities
    utils/                — utility subdirectory

backend/
  app/
    Http/
      Controllers/
        AuthController.php
        CartController.php
        ProductController.php
        OrderController.php
        OrderRequestController.php
        PaymentController.php
        InventoryController.php
        FlashSaleController.php
        ActivityLogController.php
        AuditLogController.php
        BannerController.php
        NotificationController.php
        ProfileController.php
        AddressController.php
        SupplierController.php
        SaleController.php
        JobOrderController.php
        SessionController.php
        ShopOrderRequestController.php
        TwoFactorController.php
        Controller.php
      Requests/
        StoreAddressRequest.php
      Middleware/               — check existing middleware before creating new
    Models/
      ActivityLog.php
      AuditLog.php
      Banner.php
      Cart.php
      FlashSale.php
      Inventory.php
      JobOrder.php
      Notification.php
      Order.php
      OrderRequest.php
      OtpCode.php
      PersonalAccessToken.php
      Product.php
      Sale.php
      StockHistory.php
      Supplier.php
      User.php
    Mail/
      AdminNewOrderMail.php
      ContactFormMail.php
      OrderConfirmationMail.php
      OrderConfirmedMail.php
      OrderStatusMail.php
      OrderSubmittedMail.php
      PasswordResetLinkMail.php
      ResetPasswordMail.php
      TwoFactorMail.php
      VerificationCodeMail.php
      WelcomeMail.php
  routes/
    api.php
  services/
    ssa.py / main.py     — Python FastAPI SSA forecast service on port 8001
                           NOT a Laravel endpoint — do not add Laravel routes for it
```

Do not create new files unless explicitly instructed.
Do not move or rename existing files.
Do not add a new context file — use the existing ones.
Do not recreate any file listed above — check the inventory before creating anything.

---

## Architecture Rules

### Auth
- Single source of truth for auth state: `currentUser`, `token`, `isLoading` — all in `AuthContext.jsx`
- Token read priority: sessionStorage first, then localStorage
- sessionStorage = short session (rememberMe off); localStorage = persistent session (rememberMe on)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wendell04/PM](https://github.com/wendell04/PM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
