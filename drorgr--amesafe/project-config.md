---
trigger: always_on
description: **Essential Info**: Angular 20.2.1 frontend → AWS S3 + CloudFront | Project: `demo` | Build: `dist/demo/browser/`
---

# AmesaBase Project - Frontend - Cursor Rules

## 🚀 Quick Reference (TL;DR)

**Essential Info**: Angular 20.2.1 frontend → AWS S3 + CloudFront | Project: `demo` | Build: `dist/demo/browser/`

| Item | Value |
|------|-------|
| **Angular Project** | `demo` (in angular.json) |
| **Build Output** | `dist/demo/browser/` |
| **Angular Version** | 20.2.1 |
| **TypeScript** | 5.9.2 |
| **CloudFront ID** | E3GU3QXUR43ZOH |
| **Production URL** | https://dpqbvdgnenckf.cloudfront.net |
| **Local Dev** | http://localhost:4200 |
| **Backend API** | http://localhost:5000 (dev) / amesa-backend-alb-509078867.eu-north-1.elb.amazonaws.com (prod) |
| **Status** | ✅ Production operational, 100% complete |

**⚠️ CRITICAL**: Never pipe build commands directly - always use file-based output capture (see Agent Instructions)

---

## Project Overview
AmesaBase is a lottery management system with Angular frontend, .NET backend, and AWS infrastructure. The application is a property lottery platform featuring a "4Wins Model" where profits support community causes.

**⚠️ This repository is part of the AmesaBase-Monorepo workspace**
- **Monorepo Root**: `C:\Users\dror0\Curser-Repos\AmesaBase-Monorepo\`
- **This Repository (FE/)**: Angular frontend → https://github.com/DrorGr/amesaFE
- **Backend (BE/)**: .NET 8.0 backend → https://github.com/DrorGr/amesaBE
- **MetaData/**: Cross-cutting docs, scripts, and configs

## Technology Stack

| Category | Technology |
|----------|-----------|
| **Frontend Framework** | Angular 20.2.1, TypeScript 5.9.2, Tailwind CSS 3.4.3 |
| **Build System** | Angular CLI (`@angular/build`) |
| **Backend** | .NET 8.0 (Docker + ECS) |
| **Database** | Aurora PostgreSQL Serverless v2 |
| **Infrastructure** | AWS (S3, CloudFront, ECS, ALB) |
| **CI/CD** | GitHub Actions |
| **Package Manager** | npm |

## Frontend Project Configuration

**Project**: `demo` | **Build Output**: `dist/demo/browser/` | **CloudFront Origin Path**: `/browser`

| Configuration | Details |
|--------------|---------|
| **Build Configs** | `development` (source maps), `production` (optimized, both staging & prod) |
| **Environment Files** | `environment.ts` (base), `environment.dev.ts`, `environment.stage.ts`, `environment.prod.ts` |
| **Location** | `src/environments/` |
| **Contains** | `apiUrl`, `frontendUrl`, `production` flag, service-specific URLs |

## External Service Integrations

| Service | Package/Provider | Purpose |
|---------|-----------------|---------|
| **Stripe** | `@stripe/stripe-js` v8.5.3 | Payment processing |
| **SignalR** | `@microsoft/signalr` v9.0.6 | Real-time communication |
| **QR Codes** | `angularx-qrcode` v20.0.0, `html5-qrcode` v2.3.8 | QR generation & scanning |
| **OAuth** | Google, Meta (Facebook) | Authentication |
| **AWS Rekognition** | (via backend) | ID verification |
| **AWS SES** | (via backend) | Email delivery |
| **AWS SNS** | (via backend) | SMS/notifications |
| **reCAPTCHA Enterprise** | (via backend) | Bot protection |
| **Telegram Bot API** | (via backend) | Notification channel |
| **WebPush API** | (native) | Browser push notifications |

## Current Status (2025-01-25)

| Area | Status |
|------|--------|
| **Production Environment** | ✅ Fully operational |
| **Backend Admin Panel** | ✅ Deployed and working |
| **CloudFront Config** | ✅ API routing configured |
| **System Enhancement Plan** | ✅ **100% COMPLETE** - All phases done, gaps fixed |
| **Locale Formatting** | ✅ **100%** - 10 components, 3 services use LocaleService |
| **Accessibility** | ✅ **100%** - All components have ARIA labels & keyboard nav |
| **Dark Mode** | ✅ **100%** - All 57 components support dark mode |
| **Git Status** | ✅ Clean, all changes committed (branch: main) |

## Environment URLs

| Environment | Frontend | Backend API | Admin Panel |
|------------|----------|-------------|-------------|
| **Production** | https://dpqbvdgnenckf.cloudfront.net | amesa-backend-alb-509078867.eu-north-1.elb.amazonaws.com | http://amesa-backend-alb-509078867.eu-north-1.elb.amazonaws.com/admin ✅ |
| **Local Dev** | http://localhost:4200 | http://localhost:5000 | http://localhost:5000/admin |

**Note**: See root `.cursorrules` for complete environment URLs table and backend API endpoints.

## Coding Standards

### Angular/TypeScript
- Use standalone components architecture
- Follow Angular 20.2.1 best practices
- Implement lazy loading with custom preloading strategy
- Use TypeScript strict mode
- Follow Tailwind CSS utility-first approach
- Implement proper error handling and logging

### Environment Configuration
- Use external configuration injection
- Never hardcode environment-specific values
- Maintain separate configs for local development and production
- Use GitHub secrets for sensitive data

### AWS Infrastructure
- Follow serverless architecture patterns
- Use proper CloudFront origin path configurations
- Implement proper cache invalidation strategies
- Maintain environment isolation
- **Region**: eu-north-1
- **Account**: 129394705401
- **S3 Buckets**: amesa-frontend-prod
- **CloudFront Distributions**: Production distribution with API routing configured
- **ECS Cluster**: "Amesa" with Fargate launch type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DrorGr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
