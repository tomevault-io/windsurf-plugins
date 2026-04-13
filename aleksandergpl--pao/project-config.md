---
trigger: always_on
description: This document provides a comprehensive overview of the Pao project, a real-life social deduction game. It details the project's purpose, architecture, and updated setup instructions for both the backend and mobile components.
---

# GEMINI.md - Pao Project

This document provides a comprehensive overview of the Pao project, a real-life social deduction game. It details the project's purpose, architecture, and updated setup instructions for both the backend and mobile components.

## Project Overview

Pao is a mobile game where players try to "eliminate" each other by taking a selfie with their assigned target in the background. The game is designed for social gatherings and is inspired by games like "Assassin" and "Among Us," but played in the real world.

### Core Gameplay

1.  **Lobby Creation:** A player hosts a game, creating a lobby with a unique code.
2.  **Joining a Game:** Other players join the lobby using the game code.
3.  **Game Start:** The host starts the game, and the backend assigns each player a secret target.
4.  **Elimination:** To eliminate a target, a player must take a photo of themselves with their target visible in the background.
5.  **Target Inheritance:** When a player successfully eliminates their target, they inherit the target of the eliminated player.
6.  **Winning:** The last player remaining is the winner.

## Tech Stack

The project is a monorepo composed of a backend service and a mobile application.

### Backend

*   **Runtime:** [Deno](https://deno.land/)
*   **Web Framework:** [Hono](https://hono.dev/)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **Database:** [PostgreSQL](https://www.postgresql.org/)
*   **ORM:** [Drizzle ORM](https://orm.drizzle.team/)
*   **Real-time Communication:** [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) and [Redis](https://redis.io/)
*   **Image Storage:** S3-Compatible Object Storage (e.g., [AWS S3](https://aws.amazon.com/s3/), [Cloudflare R2](https://www.cloudflare.com/developer-platform/r2/))
*   **Containerization:** [Docker](https://www.docker.com/)

### Mobile

*   **Framework:** [React Native](https://reactnative.dev/) with [Expo](https://expo.dev/)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/) (via NativeWind)
*   **State Management:** [Zustand](https://zustand-demo.pmnd.rs/)
*   **Navigation:** [Expo Router](https://docs.expo.dev/router/introduction/)

## Project Structure

The project is divided into two main directories:

*   `backend/`: Contains the Deno/Hono server application.
*   `mobile/`: Contains the React Native (Expo) mobile application.

```
/
├── backend/
│   ├── main.ts           # Main application entry point
│   ├── db/
│   │   └── schema.ts     # Database schema definitions
│   ├── services/
│   │   ├── auth.ts       # Authentication service
│   │   └── game/
│   │       ├── index.ts  # Game logic and API endpoints
│   │       └── ws.ts     # WebSocket handling
│   └── Dockerfile        # Docker configuration for deployment
└── mobile/
    ├── App.tsx           # Main application component (though expo-router is used)
    ├── app/              # Expo router based file-system routing
    │   ├── game.tsx      # The main game screen
    │   └── (tabs)/       # Tabbed navigation for home/create game
    ├── components/       # Reusable UI components
    ├── lib/              # Utility functions and libraries
    └── package.json      # Project dependencies and scripts
```

## Setup and Installation

### Backend Setup

1.  **Prerequisites:**
    *   [Deno](https://deno.land/manual/getting_started/installation) installed.
    *   [Docker](https://docs.docker.com/get-docker/) installed and running.

2.  **Database and Redis:**
    The backend requires a PostgreSQL database and a Redis instance. The easiest way to run these is with Docker.
    
    Create a file named `docker-compose.yml` in the `backend/` directory with the following content:
    ```yaml
    services:
      postgres:
        image: postgres:13
        environment:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
          POSTGRES_DB: pao
        ports:
          - "5432:5432"
        volumes:
          - postgres_data:/var/lib/postgresql/data
      redis:
        image: redis:7
        ports:
          - "6379:6379"
    volumes:
      postgres_data:
    ```
    From the `backend/` directory, start the services by running:
    ```bash
    docker-compose up -d
    ```

3.  **Environment Variables:**
    Create a `.env` file in the `backend/` directory. This file holds the connection strings and credentials for the services. The backend code requires the following variables:

    ```env
    # --- Database and Redis (from docker-compose) ---
    DB_URL="postgresql://user:password@localhost:5432/pao"
    REDIS_URL="redis://localhost:6379"

    # --- Mobile App URL (for CORS) ---
    # This is not strictly needed if using a wildcard origin for development
    FRONTEND_URLS="http://localhost:8081" 

    # --- S3-Compatible Object Storage Credentials ---
    # Example using Cloudflare R2 (a free alternative to AWS S3)
    S3_BUCKET="your-bucket-name"
    S3_ACCESS_KEY="your-access-key-id"
    S3_SECRET_KEY="your-secret-access-key"
    S3_REGION="auto"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AleksanderGPL) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
