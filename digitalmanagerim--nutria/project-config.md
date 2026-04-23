---
trigger: always_on
description: NutrIA is a body recomposition coaching PWA (Progressive Web App) with AI and gamification (Duolingo-style). The mascot is "Nuri", an otter that acts as a personal nutritionist and trainer. The name is a triple wordplay: Nutrición + IA + Nutria.
---

# CLAUDE.md — NutrIA Project

## What is this project?

NutrIA is a body recomposition coaching PWA (Progressive Web App) with AI and gamification (Duolingo-style). The mascot is "Nuri", an otter that acts as a personal nutritionist and trainer. The name is a triple wordplay: Nutrición + IA + Nutria.

**Full specifications**: See `docs/NutrIA_Especificaciones_Proyecto.md` for complete details.

---

## Tech Stack

- **Frontend**: React 18 + Vite + Tailwind CSS + Framer Motion + React Router
- **Backend**: Node.js + Express
- **Database**: PostgreSQL with Prisma ORM
- **AI**: OpenAI GPT-4o (vision + text) — all responses use Nuri's personality
- **Auth**: JWT (access + refresh tokens) with bcrypt
- **PWA**: Service Worker + Web App Manifest
- **Language**: TypeScript (both client and server)

---

## Project Structure

```
nutria/
├── client/                     # React PWA (Vite)
│   ├── public/
│   │   ├── manifest.json
│   │   ├── sw.js
│   │   ├── icons/
│   │   └── nuri/               # Nuri SVG illustrations
│   ├── src/
│   │   ├── components/
│   │   │   ├── common/         # Button, Card, Modal, ProgressBar, Input, etc.
│   │   │   ├── nuri/           # NuriAvatar, NuriBubble, NuriReaction, NuriFAB
│   │   │   ├── chat/           # ChatWindow, ChatBubble, ChatInput, ActionButton
│   │   │   ├── onboarding/     # Step1Register, Step2Basics, Step3Measurements, etc.
│   │   │   ├── nutrition/      # MealLog, DailyTracker, WeeklyMenu
│   │   │   ├── training/       # WorkoutPlan, ActiveSession, ExerciseCard
│   │   │   ├── weight/         # WeightEntry, WeightChart
│   │   │   ├── gamification/   # XPBar, LevelBadge, StreakCounter, AchievementCard
│   │   │   └── stats/          # Charts, WeeklySummary
│   │   ├── pages/              # Splash, Login, Register, Onboarding, Dashboard, etc.
│   │   ├── hooks/              # useAuth, useUser, useChat, useGamification
│   │   ├── context/            # AuthContext, UserContext, ChatContext
│   │   ├── services/           # api.ts (axios instance), auth.ts, chat.ts, food.ts, etc.
│   │   ├── utils/              # formatters, validators, constants
│   │   ├── assets/             # Images, sounds
│   │   └── styles/             # Global styles, Tailwind config
│   ├── index.html
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── package.json
│
├── server/                     # Express API
│   ├── src/
│   │   ├── routes/             # auth.ts, onboarding.ts, food.ts, weight.ts, chat.ts, etc.
│   │   ├── controllers/        # Same structure as routes
│   │   ├── middleware/         # auth.ts (JWT verify), upload.ts (multer), rateLimiter.ts
│   │   ├── services/
│   │   │   ├── ai.ts           # OpenAI integration, all prompts
│   │   │   ├── chat.ts         # Context builder, memory management, streaming
│   │   │   └── gamification.ts # XP calculation, level up, streak tracking
│   │   ├── prompts/            # System prompt templates for each AI feature
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   └── migrations/
│   │   ├── seeds/              # achievements.ts, initial data
│   │   └── index.ts            # Express app entry
│   ├── tsconfig.json
│   └── package.json
│
├── docs/
│   └── NutrIA_Especificaciones_Proyecto.md
├── docker-compose.yml
├── .env.example
├── CLAUDE.md                   # This file
└── README.md
```

---

## Design System

### Colors
```
Primary (turquoise):   #00B4D8  — main actions, water/otter theme
Secondary (green):     #58CC02  — progress, success
Accent (orange):       #FF9600  — streaks, fire, XP, energy
Alert (coral red):     #FF4B4B  — streak danger, deficit
Background:            #FFFFFF / #F0F9FF (light blue tint)
Text:                  #2D3748
Gold (achievements):   #FFC800
Nuri brown:            #8B6914  — warm accents
```

### Typography
- Font: **Nunito** (Google Fonts) — Bold for titles, Regular for body, SemiBold for data
- Nuri's messages: Nunito Medium Italic

### UI Principles
- Mobile-first, max-width 480px centered on desktop
- Border radius: 16px on cards
- Min touch target: 48px
- Framer Motion for all transitions and micro-animations
- Nuri speech bubbles: comic-style with tail pointing to Nuri avatar
- Bottom nav with 5 tabs + floating Nuri FAB for chat
- Duolingo-inspired: colorful, playful, rewarding, addictive

---

## Nuri — The Otter Mascot

Nuri is the soul of the app. She's a European otter who works as a sports nutritionist and personal trainer.

**Personality**: Friendly, direct, motivational, funny. Makes otter/fish references. Never condescending. Celebrates wins enthusiastically, points out improvements kindly.

**Visual states** (SVG illustrations, cartoon flat style):
- Normal (standing, smiling, hands on hips)
- Chef (chef hat, ladle) — food registration
- Fitness (headband, dumbbells) — training
- Scientist (lab coat, glasses) — analytics, weight, stats
- Celebrating (jumping, confetti) — achievements
- Fire (flames around) — active streak 3+ days
- Sleeping (on rock, pizza nearby) — 2+ days inactive
- Worried (biting nails) — streak at risk

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DigitalManagerIM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
