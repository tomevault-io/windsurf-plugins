---
trigger: always_on
description: Private AI automation platform for ArcticGrey that monitors client health through call analysis and triggers automated workflows. **MVP = Authentication + Dashboard UI with mock data.**
---


# ArcticGrey Dashboard - Project Roadmap & Development Rules

## 🎯 MVP Scope (14-Hour Sprint)

### **What We're Building**
Private AI automation platform for ArcticGrey that monitors client health through call analysis and triggers automated workflows. **MVP = Authentication + Dashboard UI with mock data.**

### **MVP Features (Must Have)**
- ✅ Google OAuth authentication (Clerk)
- ✅ Dark theme dashboard matching reference design
- ✅ Client health metrics with mock data
- ✅ Professional sidebar navigation
- ✅ Responsive design
- ✅ Type-safe architecture

### **NOT in MVP (Future)**
- ❌ Real Fireflies integration
- ❌ Claude AI analysis
- ❌ n8n automation
- ❌ Database operations
- ❌ Complex business logic

## 🛠️ Technology Stack

```
Frontend: Remix v2 + TypeScript + Vite
Database: SQLite + Prisma (prepared, not used in MVP)
Auth: Clerk (Google OAuth only)
UI: shadcn/ui + shadcn/charts + Tailwind CSS
Charts: Recharts (via shadcn/charts)
Icons: Lucide React
Deployment: Vercel
```

## 📋 14-Hour Development Plan

### **Hours 1-3: Foundation Setup**
```
✅ Create Remix v2 project
✅ Install dependencies (Clerk, shadcn, Prisma)
✅ Configure Clerk authentication
✅ Setup basic routing structure
✅ Configure dark theme
```

### **Hours 4-8: Dashboard UI**
```
✅ Build sidebar with navigation
✅ Create metrics cards with mock data
✅ Implement chart component
✅ Add responsive design
✅ Polish visual design
```

### **Hours 9-12: Integration & Polish**
```
✅ Connect authentication flow
✅ Add loading states
✅ Implement error boundaries
✅ Test responsive design
✅ Add smooth animations
```

### **Hours 13-14: Deploy & Test**
```
✅ Deploy to Vercel
✅ Test production authentication
✅ Fix any deployment issues
✅ Document and demo
```

## 🏗️ Clean Architecture Rules

### **File Structure**
```
app/
├── routes/
│   ├── dashboard/route.tsx        # Main dashboard
│   ├── login/route.tsx           # Login page
│   └── _index.tsx                # Root redirect
├── components/ui/                # Only shadcn/ui components
├── lib/
│   ├── constants/                # Mock data, routes, config
│   ├── types/                    # TypeScript interfaces
│   ├── helpers/                  # Pure utility functions
│   └── utils.ts                  # shadcn utilities
└── styles/globals.css            # Tailwind + theme
```

### **Layer Responsibilities**
- **Routes**: UI composition using shadcn/ui + mock data
- **Constants**: All mock data and configuration
- **Types**: TypeScript interfaces for type safety
- **Helpers**: Pure utility functions (formatting, validation)
- **Components/UI**: Only shadcn/ui components

### **MVP Simplifications**
- No services layer (direct mock data usage)
- No repositories layer (no database calls)
- No hooks layer (basic React hooks only)
- Focus on UI composition and design

## 🎨 Design System Rules

### **UI Component Priority**
1. **First choice**: Use shadcn/ui components
2. **Second choice**: Compose multiple shadcn components
3. **Last resort**: Custom components (following shadcn patterns)

### **Dark Theme Standards**
- Use configured CSS variables for colors
- Consistent spacing with Tailwind utilities
- Professional SaaS design aesthetic
- Smooth transitions and hover effects

### **Typography Hierarchy**
```
Headings: font-semibold text-foreground
Body: text-foreground
Secondary: text-muted-foreground
Success: text-green-500
Warning: text-yellow-500
Danger: text-red-500
```

### **Color Coding System**
```
Health Scores:
- 8.0-10.0: Green (#10B981)
- 6.0-7.9: Yellow (#F59E0B)
- 4.0-5.9: Orange (#F97316)
- 0.0-3.9: Red (#EF4444)

Risk Levels:
- Low: Green badge
- Medium: Yellow badge
- High: Red badge
```

## 📊 Dashboard Specifications

### **Sidebar Navigation**
```
🏢 ArcticGrey
├─────────────────────┤
│ ➕ Quick Action     │
├─────────────────────┤
│ 📊 Client Health    │ ← Active (MVP)
│ 🤖 AI Automations   │ ← Disabled
│ 📈 Analytics        │ ← Disabled
│ 📁 Workflows        │ ← Disabled
│ 👥 Team             │ ← Disabled
├─────────────────────┤
│ Brain System        │
│ 🧠 Context Engine   │ ← Disabled
│ 🔄 n8n Flows        │ ← Disabled
│ 📚 Knowledge Base   │ ← Disabled
├─────────────────────┤
│ ⚙️ Settings         │
│ 🔍 Search           │
│ 👤 Profile          │
└─────────────────────┘
```

### **Metrics Cards (2x2 Grid)**
```
┌─────────────────┬─────────────────┐
│ Total Clients   │ Happy Clients   │
│ 14              │ 8 (57%)         │
│ +2 this month   │ ↑ Improving     │
├─────────────────┼─────────────────┤
│ At Risk Clients │ AI Actions      │
│ 3               │ 12              │
│ Need attention  │ This week       │
└─────────────────┴─────────────────┘
```

### **Chart Component**
- **Title**: "Client Health Trend"
- **Subtitle**: "Average health score over time"
- **Period Selector**: "Last 3 months"
- **Chart Type**: Line chart with area fill
- **Data**: Mock health score trends (6 months)

## 🔧 Development Guidelines

### **Code Style**
- TypeScript strict mode
- Functional programming approach
- Pure functions for data transformations
- Descriptive variable names
- Consistent formatting with Prettier

### **Performance Rules**
- Use Remix loaders for data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samireliasjabib/agent-ai-health-agency](https://github.com/samireliasjabib/agent-ai-health-agency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
