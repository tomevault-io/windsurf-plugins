---
trigger: always_on
description: Maaz LMS is a modern Learning Management System built as a full-stack JavaScript application with a React frontend and Express backend. It provides course creation, student enrollment, progress tracking, payment processing, and educator dashboards.
---

# Maaz LMS - Agent Instructions

## Project Overview

Maaz LMS is a modern Learning Management System built as a full-stack JavaScript application with a React frontend and Express backend. It provides course creation, student enrollment, progress tracking, payment processing, and educator dashboards.

## Repository Structure

```
Maaz-LMS/
├── client/                  # React frontend (Vite)
│   ├── src/
│   │   ├── App.jsx          # Main app with routing
│   │   ├── main.jsx         # Entry point with Clerk + Router providers
│   │   ├── components/      # Reusable UI components
│   │   │   ├── student/     # Student-facing components
│   │   │   └── educator/    # Educator-facing components
│   │   ├── pages/
│   │   │   ├── student/     # Student pages (Home, Courses, Player, etc.)
│   │   │   └── educator/    # Educator pages (Dashboard, AddCourse, etc.)
│   │   ├── context/         # React Context (AppContext)
│   │   ├── assets/          # Static assets (SVGs, images)
│   │   └── index.css        # Global styles + Tailwind imports
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── eslint.config.js
│   └── postcss.config.js
├── server/                  # Express backend
│   ├── server.js            # Main server entry point
│   ├── models/              # Mongoose models
│   │   ├── User.js
│   │   ├── Course.js
│   │   ├── Purchase.js
│   │   └── CourseProgress.js
│   ├── controllers/         # Route handlers
│   │   ├── userController.js
│   │   ├── courseController.js
│   │   ├── educatorController.js
│   │   └── webhooks.js
│   ├── routes/              # Express routers
│   │   ├── userRoutes.js
│   │   ├── courseRoute.js
│   │   └── educatorRoutes.js
│   ├── middlewares/         # Custom middleware
│   │   └── authMiddleware.js
│   ├── configs/             # Service configurations
│   │   ├── mongodb.js
│   │   ├── cloudinary.js
│   │   └── multer.js
│   └── package.json
├── .agile-v                 # Agile project cycles and config
├── AGENTS.md                # This file
└── README.md
```

## Tech Stack

| Layer       | Technology                          |
|-------------|-------------------------------------|
| Frontend    | React 19, Vite 8, Tailwind CSS 3   |
| Backend     | Node.js, Express 4                 |
| Database    | MongoDB + Mongoose 8               |
| Auth        | Clerk (@clerk/express, @clerk/clerk-react) |
| Payments    | Stripe 22                          |
| Storage     | Cloudinary 2                       |
| Webhooks    | Svix 1                             |
| File Upload | Multer 2                           |

## Key Architecture Decisions

- **Monorepo** with `client/` and `server/` as separate packages
- **Clerk** handles authentication on both frontend and backend
- **Stripe** handles payments via checkout sessions and webhooks
- **Cloudinary** stores course thumbnails uploaded by educators
- **Multer** handles multipart file uploads for course images
- **React Context** (`AppContext`) provides global state for the frontend
- **Express routers** organize backend routes by domain (user, course, educator)

## Running the Project

### Prerequisites
- Node.js >= 18
- npm >= 9
- MongoDB instance (local or Atlas)
- Clerk account (publishable + secret keys)
- Stripe account (secret + webhook keys)
- Cloudinary account

### Setup

```bash
# Clone and install
git clone <repo-url>
cd Maaz-LMS

# Install frontend dependencies
cd client && npm install

# Install backend dependencies
cd ../server && npm install
```

### Environment Variables

Create `.env` files in both `client/` and `server/`:

**client/.env:**
```
VITE_CLERK_PUBLISHABLE_KEY=pk_test_...
```

**server/.env:**
```
MONGODB_URI=mongodb+srv://...
CLOUDINARY_NAME=...
CLOUDINARY_API_KEY=...
CLOUDINARY_SECRET_KEY=...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
CLERK_WEBHOOK_SECRET=...
CURRENCY=usd
PORT=3000
```

### Development

```bash
# Frontend dev server (port 5173)
cd client && npm run dev

# Backend dev server with hot reload (port 3000)
cd server && npm run server
```

### Build

```bash
# Frontend production build
cd client && npm run build

# Backend start
cd server && npm start
```

## Key Files Reference

| File | Purpose |
|------|---------|
| `client/src/App.jsx` | Main routing and layout |
| `client/src/context/AppContext.jsx` | Global state management |
| `server/server.js` | Express app setup and route registration |
| `server/models/User.js` | User schema and model |
| `server/models/Course.js` | Course schema with chapters/lectures |
| `server/models/Purchase.js` | Purchase/payment tracking |
| `server/models/CourseProgress.js` | Student progress tracking |
| `server/controllers/webhooks.js` | Clerk and Stripe webhook handlers |
| `server/middlewares/authMiddleware.js` | Educator role protection |
| `.agile-v` | Project cycles, phases, milestones, and agile config |

## Package Versions (Current)

### Frontend (client/package.json)
- react: ^19.2.8
- react-dom: ^19.2.8
- react-router-dom: ^7.18.1
- @clerk/clerk-react: ^5.61.3
- framer-motion: ^12.42.2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muhammadmaaz-2k5/LMS-Courses-Management-System-FullStack](https://github.com/muhammadmaaz-2k5/LMS-Courses-Management-System-FullStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
