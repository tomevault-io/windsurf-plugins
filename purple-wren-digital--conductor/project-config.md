---
trigger: always_on
description: | Layer | Technology | Purpose |
---

# Conductor Technical Specification

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Frontend | Next.js + TypeScript | SSR + SPA, type safety |
| UI Components | React Query + Tailwind CSS | Data fetching + utility-first styling |
| Backend API | Encore TS | Backend framework with flexible queries |
| Database | PostgreSQL + Encore SQLDatabase | Relational data + raw SQL with repositories |
| Auth & RBAC | Clerk | Authentication + role-based access control |
| Notifications | SendGrid or Resend | Email notifications |
| Hosting | Vercel (Frontend) + AWS (Backend) | Scalable deployment |

## Core Features

### 1. Ticket Management System
- **Create Ticket**: 
  - Fields: title, description, category, urgency (High/Medium/Low)
  - Link to property/transaction (optional)
  - Auto-assign to department based on category
- **Ticket States**: Assigned, Awaiting Response, In Progress, Resolved
- **Ticket List View**:
  - Sortable columns: Date, Title, Status, Assigned To, Urgency, Name, Email
  - Filters: Status, Urgency, Date Range, Assigned To
  - Search functionality
- **Ticket Detail View**:
  - Comments thread
  - Status updates
  - Assignment changes
  - Action items sidebar
  - Due date management

### 2. User Roles & Permissions

```typescript
enum UserRole {
  AGENT = 'AGENT',           // Can create tickets, view own tickets
  STAFF = 'STAFF',           // Can manage assigned tickets, view all tickets
  ADMIN = 'ADMIN'            // Full access, reports, settings
}
```

### 3. Comments System
- Real-time updates using WebSockets or polling
- Markdown support
- @mentions for notifications
- Timestamps and user attribution
- Internal notes (staff-only visibility)

### 4. Notification System
- Email notifications via SendGrid/Resend:
  - Ticket created
  - Ticket assigned
  - Status changed
  - New comment
  - SLA warnings
- In-app notifications
- User notification preferences

### 5. Dashboard Views

#### Agent Dashboard
```typescript
interface AgentDashboard {
  myTickets: Ticket[];
  recentActivity: Activity[];
  quickActions: {
    createTicket: () => void;
    viewAllTickets: () => void;
  };
}
```

#### Staff Dashboard
```typescript
interface StaffDashboard {
  assignedTickets: Ticket[];
  ticketQueue: Ticket[];
  metrics: {
    avgResponseTime: number;
    openTickets: number;
    overdueTickets: number;
  };
}
```

### 6. Database Schema

The database schema is defined via SQL migrations in `backend/ticket/migrations/`. Data access uses the repository pattern with Encore's SQLDatabase.

**Key Tables:**
- `users` - User accounts linked to Clerk via `clerk_id`
- `tickets` - Support tickets with status, urgency, assignee
- `comments` - Ticket comments with internal flag
- `ticket_categories` - Categories per market center
- `ticket_ratings` - Post-resolution surveys
- `subscriptions` - Stripe subscription data

**Repository Pattern:**
```typescript
// Example: ticket/db.ts
import { db } from "../shared/db";

export const ticketRepository = {
  async findById(id: string) {
    return db.queryRow<Ticket>`SELECT * FROM tickets WHERE id = ${id}`;
  },
  async create(data: CreateTicketData) {
    return db.queryRow<Ticket>`
      INSERT INTO tickets (title, description, creator_id, ...)
      VALUES (${data.title}, ${data.description}, ${data.creatorId}, ...)
      RETURNING *
    `;
  }
};
```

**Enums (defined in TypeScript):**
```typescript
type TicketStatus = "ASSIGNED" | "AWAITING_RESPONSE" | "IN_PROGRESS" | "RESOLVED" | "DRAFT" | "CREATED" | "UNASSIGNED";
type Urgency = "HIGH" | "MEDIUM" | "LOW";
type UserRole = "AGENT" | "STAFF" | "STAFF_LEADER" | "ADMIN";
```

### 7. API Endpoints (Encore)

```typescript
// Ticket endpoints
POST   /api/tickets              // Create ticket
GET    /api/tickets              // List tickets (filtered by role)
GET    /api/tickets/:id          // Get ticket details
PUT    /api/tickets/:id          // Update ticket
POST   /api/tickets/:id/assign   // Assign ticket
POST   /api/tickets/:id/comments // Add comment

// User endpoints
GET    /api/users/me             // Get current user
PUT    /api/users/me/preferences // Update preferences

// Dashboard endpoints
GET    /api/dashboard            // Get role-specific dashboard data
GET    /api/metrics              // Get performance metrics (admin only)
```

### 8. Frontend Routes (Next.js)

```
/                        // Dashboard (role-based)
/tickets                 // Ticket list
/tickets/new             // Create ticket
/tickets/:id             // Ticket detail
/settings                // User settings
/admin                   // Admin panel (admin only)
/admin/reports           // Reports (admin only)
```

### 9. UI Components Structure

```
components/
├── layout/
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   └── Layout.tsx
├── tickets/
│   ├── TicketList.tsx
│   ├── TicketCard.tsx
│   ├── TicketDetail.tsx
│   ├── TicketForm.tsx
│   └── TicketFilters.tsx
├── comments/
│   ├── CommentList.tsx
│   ├── CommentForm.tsx
│   └── Comment.tsx
├── common/
│   ├── Button.tsx
│   ├── Input.tsx
│   ├── Select.tsx
│   ├── Badge.tsx
│   └── Modal.tsx
└── dashboard/
    ├── AgentDashboard.tsx
    ├── StaffDashboard.tsx
    └── MetricCard.tsx
```

### 10. State Management

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Purple-Wren-Digital/conductor](https://github.com/Purple-Wren-Digital/conductor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
