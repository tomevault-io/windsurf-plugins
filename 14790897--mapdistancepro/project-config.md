---
trigger: always_on
description: MapDistancePro is a Next.js 15 + TypeScript web application for batch address distance calculation and map visualization using Chinese Gaode (Amap) APIs. The application allows users to input multiple addresses, calculates distances from their current location, and displays results on an interactive map with CSV export functionality.
---

# MapDistancePro - GitHub Copilot Instructions

MapDistancePro is a Next.js 15 + TypeScript web application for batch address distance calculation and map visualization using Chinese Gaode (Amap) APIs. The application allows users to input multiple addresses, calculates distances from their current location, and displays results on an interactive map with CSV export functionality.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Environment Setup
- Install Node.js 18+ and pnpm (preferred package manager)
- Install pnpm globally: `npm install pnpm -g`
- Install dependencies: `pnpm install` -- takes 25 seconds. NEVER CANCEL.
- Set timeout to 5+ minutes for dependency installation

### Build and Development
- **Development server**: `pnpm dev` -- starts in ~2 seconds on http://localhost:3000
- **Production build**: `pnpm build` -- takes 21 seconds. NEVER CANCEL. Set timeout to 5+ minutes.
- **Linting**: `pnpm run lint` -- takes 30 seconds. Some warnings are expected, no blocking errors.
- **Deploy to Vercel**: `pnpm run deploy` or `vercel --prod`

### Environment Configuration
The application requires Gaode Map API credentials to function:

1. **Create environment file**:
   ```bash
   cp .env.example .env.local  # Note: .env.example may not exist
   ```

2. **Required environment variables** (set in `.env.local`):
   ```env
   NEXT_PUBLIC_AMAP_JS_API_KEY=your_js_api_key
   NEXT_PUBLIC_AMAP_REST_API_KEY=your_rest_api_key
   NEXT_PUBLIC_AMAP_SECURITY_CODE=your_security_code
   NEXT_PUBLIC_DEFAULT_LOCATION=北京市
   NEXT_PUBLIC_REQUEST_LIMIT=50
   NEXT_PUBLIC_REQUEST_DELAY=1000
   ```

3. **API Key Sources**: Obtain from [Gaode Open Platform](https://console.amap.com/)
   - JS API Key: For map display (requires "Web端(JS API)" service)
   - REST API Key: For address geocoding (requires "Web服务API")
   - Security Code: For enhanced API security

## Validation

### Manual Testing Scenarios
Always test these scenarios after making changes:

1. **Basic Application Flow**:
   - Navigate to http://localhost:3000
   - Verify homepage loads with warning about missing API keys
   - Click "前往设置" (Go to Settings) button
   - Navigate to `/settings` page
   - Verify all API key input fields are present and functional

2. **API Configuration Testing** (if you have test keys):
   - Enter test API keys in settings page
   - Return to main page
   - Enter sample Chinese addresses (e.g., "北京市", "上海市")
   - Test location functionality and distance calculation

3. **Build Validation**:
   - Run `pnpm build` and verify successful completion
   - Check that all pages build without errors
   - Verify static generation completes

### Required Validation Commands
Always run these before committing changes:
- `pnpm run build` -- NEVER CANCEL. Build takes 21 seconds.
- `pnpm run lint` -- Ignore warnings about React hooks and unescaped entities, these are not blocking.

## Codebase Navigation

### Key File Locations
- **Main application logic**: `app/page.tsx` - Contains all address processing, distance calculation, and map integration
- **Settings page**: `app/settings/page.tsx` - API key configuration and app settings
- **Custom hooks**: `lib/hooks/` directory
  - `useAddressProcessor.ts` - Address geocoding and distance calculation
  - `useAmap.ts` - Gaode Map integration
  - `useLocalStorage.ts` - Browser storage management
- **UI components**: `components/ui/` - shadcn/ui component library
- **Styling**: `app/globals.css` and `tailwind.config.ts`

### Core Data Processing Functions (in `app/page.tsx`)
- `geocodeAddress()` - Address to coordinates conversion
- `getDistance()` - Spherical distance calculation algorithm
- `processAddresses()` - Batch address processing workflow
- `getUserLocation()` - GPS positioning
- `exportResults()` - CSV data export

### Project Structure Overview
```
app/
├── page.tsx          # Main application (address processing)
├── layout.tsx        # App layout with metadata
├── settings/         # Settings page directory
├── promo/           # Product promotion page
└── globals.css      # Global styles

components/
├── ui/              # shadcn/ui component library
├── theme-provider.tsx
└── ChatwootWidget.tsx

lib/
├── hooks/           # Custom React hooks
└── utils.ts         # Utility functions

scripts/             # Deployment and utility scripts
├── push-env-to-vercel.ps1
└── seo-setup.sh
```

## Common Tasks

### Adding New Features
1. Always check API key configuration in settings first
2. Main application logic goes in `app/page.tsx`
3. Reusable logic should be extracted to `lib/hooks/`
4. UI components follow shadcn/ui patterns in `components/ui/`

### Environment Variables Management
- **Development**: Use `.env.local` file (not committed)
- **Production**: Use Vercel dashboard or push scripts in `/scripts/`
- **Push to Vercel**: Run `scripts/push-env-to-vercel.ps1`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [14790897/MapDistancePro](https://github.com/14790897/MapDistancePro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
