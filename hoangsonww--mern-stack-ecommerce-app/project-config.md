---
trigger: always_on
description: **Fusion Electronics** is a production-ready, full-stack MERN (MongoDB, Express, React, Node.js) e-commerce application featuring:
---

# AI Coding Agent Instructions for Fusion Electronics E-commerce

## Project Overview

**Fusion Electronics** is a production-ready, full-stack MERN (MongoDB, Express, React, Node.js) e-commerce application featuring:
- Product browsing, search, and detailed views
- Shopping cart and checkout flow with validation
- User authentication (JWT-based)
- AI-powered product recommendations using **Pinecone** (primary), **Weaviate**, **FAISS**, and **LangChain**
- Comprehensive API documentation via Swagger
- Unit and integration testing (Jest, React Testing Library)
- CI/CD pipeline with GitHub Actions
- Docker containerization support

---

## Repository Structure

```
project-root/
├── backend/                      # Node.js + Express API server
│   ├── config/                   # Database configuration
│   │   └── db.js                # MongoDB connection setup
│   ├── docs/                     # API documentation
│   │   └── swagger.js           # Swagger configuration
│   ├── models/                   # Mongoose schemas
│   │   ├── product.js           # Product model with Pinecone hooks
│   │   └── user.js              # User model for authentication
│   ├── routes/                   # Express route handlers
│   │   ├── auth.js              # Authentication endpoints
│   │   ├── checkout.js          # Order creation and validation
│   │   ├── products.js          # Product CRUD and recommendations
│   │   └── search.js            # Product search functionality
│   ├── scripts/                  # Utility scripts
│   │   ├── build-faiss-index.js # FAISS index builder
│   │   ├── search-faiss-index.js# FAISS similarity search
│   │   ├── sync-pinecone.js     # Sync MongoDB → Pinecone
│   │   ├── sync-weaviate-ids.js # Sync MongoDB → Weaviate IDs
│   │   ├── query-weaviate.js    # Weaviate query utility
│   │   └── weaviate-upsert.js   # Weaviate data upsert
│   ├── seed/                     # Database seeding
│   │   └── productSeeds.js      # Initial product data
│   ├── services/                 # Business logic services
│   │   └── pineconeSync.js      # Pinecone synchronization helpers
│   ├── sync/                     # Synchronization modules
│   │   ├── syncPinecone.js      # Main Pinecone sync orchestrator
│   │   └── syncWeaviate.js      # Weaviate sync orchestrator
│   ├── __tests__/                # Backend tests
│   │   ├── auth.spec.js
│   │   ├── checkout.spec.js
│   │   └── search.spec.js
│   ├── pineconeClient.js         # Pinecone SDK client
│   ├── weaviateClient.js         # Weaviate SDK client
│   ├── index.js                  # Express server entry point
│   ├── package.json              # Backend dependencies
│   └── .env                      # Environment variables (not committed)
│
├── src/                          # React frontend
│   ├── components/               # Reusable React components
│   │   ├── CheckoutForm.jsx     # Payment form with validation
│   │   ├── Footer.jsx           # Site footer
│   │   ├── NavigationBar.jsx    # Top navigation with cart badge
│   │   ├── ProductCard.jsx      # Product display card
│   │   ├── ScrollToTop.jsx      # Auto-scroll on route change
│   │   └── SearchResults.jsx    # Search results display
│   ├── context/                  # React Context providers
│   │   └── NotificationProvider.jsx # Toast notifications
│   ├── pages/                    # Page-level components
│   │   ├── About.jsx
│   │   ├── Cart.jsx             # Shopping cart page
│   │   ├── Checkout.jsx         # Checkout page
│   │   ├── ForgotPassword.jsx
│   │   ├── Home.jsx             # Landing page with recommendations
│   │   ├── Login.jsx
│   │   ├── NotFoundPage.jsx
│   │   ├── OrderSuccess.jsx     # Order confirmation
│   │   ├── ProductDetails.jsx   # Single product view
│   │   ├── Register.jsx
│   │   ├── ResetPassword.jsx
│   │   ├── Shop.jsx             # All products listing
│   │   └── Support.jsx
│   ├── services/                 # API client services
│   │   └── apiClient.js         # Axios instance with retry logic
│   ├── tests/                    # Frontend tests
│   │   ├── Cart.test.js
│   │   ├── Checkout.test.js
│   │   ├── Home.test.js
│   │   ├── Login.test.js
│   │   ├── OrderSuccess.test.js
│   │   ├── Register.test.js
│   │   └── Shop.test.js
│   ├── utils/                    # Utility functions
│   │   └── products.js          # Product data helpers
│   ├── App.jsx                   # Root component with routing
│   ├── index.js                  # React entry point
│   └── setupProxy.js             # Development proxy configuration
│
├── public/                       # Static assets
├── docs/                         # Documentation and screenshots
├── .github/workflows/            # CI/CD pipelines
│   └── ci.yml                   # GitHub Actions workflow
├── docker-compose.yml            # Docker orchestration
├── Dockerfile                    # Container definition
├── package.json                  # Frontend dependencies
├── craco.config.js               # Create React App overrides
├── jest.config.js                # Jest configuration
└── README.md                     # User-facing documentation
```

---

## Technology Stack

### Backend

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoangsonww/MERN-Stack-Ecommerce-App](https://github.com/hoangsonww/MERN-Stack-Ecommerce-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
