---
trigger: always_on
description: <img src="macrofolio/src/macrofolio_assets/public/logo.png" alt="Macrofolio Logo" width="200" />
---

# MACROFOLIO – One Portfolio to Track Them All

<div align="center">
  <img src="macrofolio/src/macrofolio_assets/public/logo.png" alt="Macrofolio Logo" width="200" />
</div>

[![Live Demo](https://img.shields.io/badge/Demo-Live-brightgreen)](https://macrofolio.vercel.app)
[![GitHub](https://img.shields.io/badge/GitHub-Repository-blue)](https://github.com/nutrazz/macrofolio)
[![Video Demo](https://img.shields.io/badge/Video-Demo-red)](https://youtu.be/5Fve86iO7BI)
[![Built with RevenueCat](https://img.shields.io/badge/Built%20with-RevenueCat-FF6B6B)](https://www.revenuecat.com)

> If the live demo does not render immediately due to a cold start or cache,
> please refresh once. The app is deployed via Vercel using Vite.

---

## 🎯 Creator Brief

**Josh (VisualFaktory): One portfolio to track them all**

Investors juggle stocks, gold, funds, fixed income, real estate, and more across multiple platforms — messy to track and hard to understand at a glance. Josh wants a single app where users can log everything, get real-time price updates where possible, set amortization/reminder alerts for non-listed products, and unlock premium risk + diversification analysis (like country/sector exposure).

---

## 🏆 RevenueCat Shipyard Contest Submission

**Live App:** https://macrofolio.vercel.app  
**Demo Video:** https://youtu.be/5Fve86iO7BI  
**GitHub:** https://github.com/nutrazz/macrofolio  
**Twitter:** https://x.com/nutraazz

## 📊 Executive Summary

Macrofolio solves the fragmentation problem for modern DIY investors by providing a single, self-custodial dashboard to track all asset classes across multiple platforms.

### Key Features
- 📈 **Multi-Asset Tracking**: Stocks, ETFs, crypto, gold, real estate, fixed income, NFTs
- 🔒 **Self-Custodial**: Zero centralized storage of sensitive financial data
- ⛓️ **On-Chain Verification**: Portfolio proofs anchored on Polygon & Base
- 💰 **RevenueCat Integration**: Premium analytics via subscription
- 🎯 **Real-Time Updates**: Live price tracking and alerts
- 📱 **Mobile-First**: Responsive PWA, ready for TestFlight/Play Store

## 🚀 Quick Start

### Demo Mode (No Setup Required)
1. Visit https://macrofolio.vercel.app
2. Click "Try Demo Mode"
3. Explore all features immediately

### Full Experience
1. Connect MetaMask wallet
2. Add your assets (stocks, crypto, real estate, etc.)
3. View unified dashboard with performance metrics
4. Set alerts for non-listed assets
5. Anchor portfolio on-chain for verification

## 🏗️ Technical Architecture

### Frontend
- **React 18** with TypeScript
- **Vite** for fast builds
- **Tailwind CSS** for styling
- **RevenueCat SDK** for subscriptions
- **Ethers.js v6** for Web3 interactions

### Backend
- **Supabase** (PostgreSQL) for authentication & asset records
- **Row-Level Security** for data privacy
- **Real-time subscriptions** for price updates

### Blockchain
- **Polygon Amoy & Base Sepolia** testnets
- **PortfolioAnchor.sol** smart contract
- **EIP-712 signatures** for secure anchoring
- **MetaMask/WalletConnect** integration

### Security
- ✅ 45 comprehensive test cases
- ✅ Zod validation schemas
- ✅ DOMPurify XSS protection
- ✅ Content Security Policy
- ✅ Input sanitization
- ✅ Session timeout management

## 💰 RevenueCat Integration

### Subscription Tiers
1. **Free**: Basic portfolio tracking, 5 assets limit
2. **Premium Monthly ($9.99)**: Unlimited assets, advanced analytics
3. **Premium Yearly ($99.99)**: Best value (2 months free)
4. **Lifetime ($299.99)**: One-time payment, lifetime updates

### Implementation
- **RevenueCat SDK** integration (`@revenuecat/purchases-capacitor`)
- **Demo mode** for testing without API keys
- **Production mode** with real subscription processing
- **Entitlement checking** for premium features
- **Purchase/restore flows** fully implemented

## 📁 Project Structure

```
macrofolio/src/macrofolio_assets/
├── public/
│   └── logo.png                    # App logo
├── src/
│   ├── components/
│   │   └── RevenueCatProvider.tsx  # RevenueCat integration
│   ├── hooks/
│   │   └── useRevenueCat.ts        # Subscription hooks
│   ├── lib/
│   │   └── revenuecat.ts           # RevenueCat SDK service
│   ├── pages/
│   │   ├── Dashboard.tsx           # Main dashboard
│   │   ├── Portfolio.tsx           # Asset management
│   │   ├── Premium.tsx             # Subscription page
│   │   └── Verify.tsx              # On-chain verification
│   └── App.tsx                    # Main app component
├── android/                        # Capacitor Android app
├── capacitor.config.ts            # Capacitor configuration
└── package.json                   # Dependencies
```

## 🔧 Development

```bash
# Navigate to project
cd macrofolio/src/macrofolio_assets

# Install dependencies (includes RevenueCat SDK)
npm install --legacy-peer-deps

# Start development server
npm run dev

# Build for production
npm run build

# Build Android APK
cd android && ./gradlew assembleDebug
```

## 📱 Android App

The app is configured for Android deployment using Capacitor:

```bash
# Build web assets
npm run build

# Sync to native
npx cap sync

# Open Android Studio
npx cap open android

# Build APK
cd android && ./gradlew assembleDebug

# APK location:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nutraz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
