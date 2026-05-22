---
trigger: always_on
description: Standard pattern for integrating Givebutter donation widgets with popup modal functionality in Next.js applications
---


# Givebutter Widget Integration Pattern

## **Overview**
This rule defines the standard pattern for integrating Givebutter donation widgets into Next.js applications, specifically for creating donate buttons that open popup modals with donation forms. This pattern addresses X-Frame-Options restrictions and provides reliable popup functionality across desktop and mobile devices.

## **Problem Solved**
- **X-Frame-Options Restriction**: Givebutter sets `X-Frame-Options: sameorigin`, preventing iframe embedding from other domains
- **Popup/New Tab Functionality**: Donate button opens the full campaign page in a popup (desktop) or new tab (mobile), so dashboard settings (suggested amounts, **Other amount**, minimum donation) are visible
- **Other Amount & Minimum**: The widget modal does not reliably show “Other” option or minimum donation from the dashboard; opening the full campaign URL ensures these options appear
- **Cross-Platform Compatibility**: Works consistently on both desktop and mobile browsers
- **Fallback**: If popup is blocked, opens campaign in new tab

## **Credentials and configuration from Givebutter dashboard**

To make the donate button (and optional widget script) work, you need the following from the Givebutter dashboard. **No API keys, Aadhaar, or other identity credentials are required in this application** for the home-page donate button.

| What you need | Where to get it in Givebutter | Where it’s used in the app | Required for donate button? |
|---------------|-------------------------------|----------------------------|-----------------------------|
| **Campaign ID** | Campaign URL: `https://givebutter.com/{CampaignID}` (e.g. `mhoZp0`). Or: open the campaign → Sharing → copy from campaign link or embed code. | Set in **`.env.local`** (and production env) as **`NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID`**. Read by `GivebutterDonateButton` and `GivebutterWidget`; optional `campaignId` prop overrides. Declared in `next.config.mjs` `env`. Builds URL: `https://givebutter.com/{campaignId}`. | **Yes** – without it the button would open the wrong or no campaign. |
| **Widget ID** | Givebutter embed for **event fund** (not donation): e.g. `<givebutter-widget id="j1ek6j">`. Get the ID from the widget embed code in Givebutter. | Set in **`.env.local`** (and production env) as **`NEXT_PUBLIC_GIVEBUTTER_WIDGET_ID`** (e.g. `j1ek6j`). Used by event GiveButter checkout page (`/events/[id]/givebutter-checkout`): when the event’s `donation_metadata` has no `givebutterWidgetId`, the app falls back to this env. Declared in `next.config.mjs` `env`. | **No** for the home-page donate button. **Yes** for the event fund embed when you want a single global widget ID without storing it per event. |
| **Account ID** | Dashboard → **Campaign** → **Sharing** → **Widgets** → **Installation**. Shown in the script snippet as `acct=...` (e.g. `mKoUpYQebNsn6RqA`). | `src/app/layout.tsx`: Givebutter script `src="...?acct={AccountID}&p=other"`. | **No** for the donate button (button only opens the campaign URL). **Yes** if you use Givebutter embed widgets (e.g. `givebutter-form`, `givebutter-button`) elsewhere. |
| **Webhook URL** | Not “brought” into the app; you **configure** in Givebutter: Campaign or Account → Integrations / Webhooks → add endpoint. Set URL to your app’s route, e.g. `https://your-domain.com/api/webhooks/givebutter`. | Givebutter sends donation events to this URL. App route: `src/app/api/webhooks/givebutter/route.ts` (forwards to backend). | No – only needed if you want server-side handling of donation events. |
| **Webhook signing secret** | Givebutter dashboard: when you add the webhook, Givebutter shows or lets you copy a **signing secret**. | Stored and used by the **backend** (not in Next.js env) to verify `X-Givebutter-Signature`. Backend matches secret per tenant in `payment_provider_config`. | No – backend only; not needed for the button to open. |

**Summary for the home-page donate button only:**

1. **Campaign ID** – Get from the campaign’s Givebutter URL or Sharing/Widgets. Put it in the app as:
   - **Option A:** Pass `campaignId` to `<GivebutterDonateButton campaignId="mhoZp0" />` (e.g. in `HeroSection.tsx`), or
   - **Option B:** Set `NEXT_PUBLIC_GIVEBUTTER_CAMPAIGN_ID` in `.env.local` and use it in the component so the same env can be used in production.
2. **Account ID** – Only if you use the widget script (e.g. for other Givebutter embeds). Put it in the script URL in `layout.tsx`: `acct=YOUR_ACCOUNT_ID`.

**What is not needed in this app for the button:**

- No Givebutter API key in the frontend (backend may use API for donation status if needed).
- No Aadhaar or other identity credentials – Givebutter does not use Aadhaar for this integration.
- No webhook secret in the Next.js app – the backend holds and verifies it.

## **Core Pattern**

### **1. Script Installation in Layout**

```tsx
// ✅ DO: Add Givebutter script to root layout using Next.js Script component
// src/app/layout.tsx
import Script from "next/script";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        {/* Other content */}

        {/* Givebutter Widget Script */}
        <Script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
