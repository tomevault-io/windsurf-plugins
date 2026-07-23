---
trigger: always_on
description: Simple Clothing Store is an e-commerce clothing store implementation built with React and Node.js. The project aims for an almost databaseless design by leveraging the Printful API for product management and fulfillment, and PayPal for payment processing.
---

# Simple Clothing Store - Project Guide for Claude

## Project Overview

Simple Clothing Store is an e-commerce clothing store implementation built with React and Node.js. The project aims for an almost databaseless design by leveraging the Printful API for product management and fulfillment, and PayPal for payment processing.

**Current Status:** The project is currently broken due to Printful API updates. A major v2.0.0 rewrite is planned to migrate to Next.js, TypeScript, and serverless architecture.

**Version:** 0.0.2
**License:** MIT
**Repository:** https://github.com/kerkkoh/simple-clothing-store

## Architecture

### Stack

**Backend:**
- Node.js (v13.14.x)
- Express server
- API Integrations:
  - Printful REST API (for product management and fulfillment)
  - PayPal REST API (for payment processing)

**Frontend:**
- React 17 (with hooks, no classes)
- React Router (for routing)
- Bootstrap 5 (responsive design)
- SASS/SCSS (styling)
- Create React App (build tooling)

**Key Dependencies:**
- `@paypal/checkout-server-sdk` - PayPal integration
- `currency.js` - Currency handling
- `axios` - HTTP client
- `dotenv` - Environment configuration

## Project Structure

```
simple-clothing-store/
├── server.js              # Main Express server
├── lib/                   # Backend utilities
│   ├── datab.js          # "Database" - product descriptions, discounts, VAT
│   ├── paypal.js         # PayPal API client
│   └── printfulclient.js # Printful API client
├── frontend/              # React frontend
│   ├── public/           # Static assets
│   └── src/
│       ├── App.js        # Main React component
│       ├── config.js     # Frontend configuration
│       ├── components/   # React components
│       │   ├── Store.js  # Store/product listing
│       │   ├── Product.js # Product detail view
│       │   ├── cart/     # Shopping cart components
│       │   ├── order/    # Order management components
│       │   └── utils/    # Utility components
│       ├── services/     # API and storage services
│       └── sass/         # Custom styles
├── .env.template         # Environment variables template
├── docker-compose.yml    # Docker composition
└── Dockerfile           # Docker configuration
```

## Key Files

### Backend

**server.js** (Main entry point)
- Express server setup
- API routes for products, store info, discounts, orders
- Integrates Printful and PayPal clients
- Serves built React frontend

**lib/datab.js**
- Acts as a lightweight "database"
- Contains:
  - Product descriptions (keyed by Printful product ID)
  - Discount codes and their percentages
  - VAT (Value Added Tax) rate

**lib/printfulclient.js**
- Custom Printful API client
- Handles authentication and API requests

**lib/paypal.js**
- PayPal SDK configuration
- Creates PayPal client for sandbox/production

### Frontend

**frontend/src/App.js**
- Main React component
- Sets up routing with React Router
- Manages global state (cart, products)

**frontend/src/config.js**
- Frontend configuration
- Store information (name, description, currency)
- PayPal client ID

**frontend/src/components/Store.js**
- Product listing page
- Displays all products from Printful

**frontend/src/components/cart/Cart.js**
- Shopping cart functionality
- Persistent cart using localStorage

**frontend/src/services/**
- `products.js` - API calls to backend
- `localstorage.js` - Cart persistence

## Configuration

### Environment Variables (.env)

Required environment variables (see `.env.template`):

```
PRINTFUL_SECRET=          # Printful API secret key
PAYPAL_CLIENT_ID=         # PayPal client ID
PAYPAL_CLIENT_SECRET=     # PayPal client secret
PORT=                     # Optional: Server port (defaults to 3001)
```

### Frontend Configuration (frontend/src/config.js)

Set store information and PayPal client ID.

### Database Configuration (lib/datab.js)

Configure:
1. **Product Descriptions:** Add entries in `items` array with product ID and description
2. **Discount Codes:** Add discount codes and percentages in `discounts` object
3. **VAT Rate:** Set VAT percentage as integer in `vat` property

## Development Workflow

### Installation

1. Clone repository
2. Set up configuration files (`.env` and `config.js`)
3. Install dependencies:
   ```bash
   npm install              # Root dependencies
   cd frontend && npm install  # Frontend dependencies
   ```

### Running the Application

**With Docker (Recommended):**
```bash
docker-compose up
```

**Manual Build:**
```bash
# Linux
npm run build-tux        # First build
npm run build-tux-clean  # Subsequent builds

# Windows
npm run build-win        # First build
npm run build-win-clean  # Subsequent builds
```

**Development:**
```bash
npm start               # Start server on port 3001
cd frontend && npm start  # Development server with hot reload
```

### Build Process

The build scripts:
1. Build React frontend (`npm run build` in frontend/)
2. Move build folder to root
3. Start Express server to serve built frontend

## API Endpoints

**Products:**
- `GET /api/products` - Get all products

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kerkkoh/simple-clothing-store](https://github.com/kerkkoh/simple-clothing-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
