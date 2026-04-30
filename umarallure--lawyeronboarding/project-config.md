---
trigger: always_on
description: This is the **Crash Guard Automation Closers Portal** - a React/TypeScript application for managing insurance lead verification workflows. The system supports complex agent workflows with real-time status updates, notifications, and Google Sheets integration.
---

# AI Coding Assistant Instructions for Closers Portal

## Project Overview
This is the **Crash Guard Automation Closers Portal** - a React/TypeScript application for managing insurance lead verification workflows. The system supports complex agent workflows with real-time status updates, notifications, and Google Sheets integration.

## Core Architecture

### Tech Stack
- **Frontend**: React 18 + TypeScript + Vite
- **UI**: shadcn/ui + Tailwind CSS
- **Backend**: Supabase (PostgreSQL + Auth + Functions)
- **External**: Google Sheets API, Slack API

### Key Data Flow
```
Leads → Call Results → Verification Sessions → Agent Assignments
```

### User Types & Permissions
- **Regular Closers**: Full dashboard access with role-based restrictions
- **Restricted Users**: Read-only access, auto-redirected to `/daily-deal-flow`
- **Center Users**: Lead vendors with access only to their own leads via `/center-lead-portal`
- **Admins**: Full system access (identified by `agent_code` in profiles)

## Critical Patterns & Conventions

### 1. Authentication & Routing
```typescript
// Always check user permissions before actions
import { isRestrictedUser, canPerformWriteOperations } from '@/lib/userPermissions';

// Route protection patterns
<ProtectedRoute>          // Regular agents
<CenterProtectedRoute>    // Center/lead vendor users only
```

### 2. Database Security
- **RLS Policies**: Every table uses Row Level Security
- **User-based Filtering**: Never expose all data - always filter by `user_id` or permissions
- **Audit Trail**: All status changes tracked in `call_update_logs`

### 3. Modal-Based Interactions
```typescript
// Claim modals follow this pattern
const [modalOpen, setModalOpen] = useState(false);
const [selectedSession, setSelectedSession] = useState(null);

// Always include loading states and error handling
<ClaimModal
  open={modalOpen}
  loading={claimLoading}
  onCancel={() => setModalOpen(false)}
  onClaim={handleClaim}
/>
```

### 4. Real-Time Status Management
```typescript
// Session statuses enum
type SessionStatus =
  | 'pending' | 'in_progress' | 'ready_for_transfer'
  | 'transferred' | 'completed' | 'call_dropped'
  | 'buffer_done' | 'la_done';

// Always update UI immediately after status changes
await supabase.from('verification_sessions').update({ status: newStatus });
fetchData(); // Refresh UI
```

### 5. Notification Integration
```typescript
// Use Supabase functions for notifications
await supabase.functions.invoke('center-transfer-notification', {
  body: { type: 'reconnected', submissionId, agentName }
});
```

### 6. Lead Vendor Security
```typescript
// Center users only see their assigned vendor's leads
const { data: leads } = await supabase
  .from('leads')
  .select('*')
  .eq('lead_vendor', centerUser.leadVendor);
```

## Development Workflow

### Build Commands
```bash
npm run dev          # Start development server
npm run build        # Production build
npm run preview      # Preview production build
```

### Database Operations
```bash
# Use MCP tools for schema changes
mcp_supabase_apply_migration  # DDL operations
mcp_supabase_execute_sql      # DQL operations
```

### Testing Approach
- **Manual Testing**: Complex workflows require manual verification
- **UI Testing**: Check modal flows, status updates, redirects
- **Data Testing**: Verify RLS policies and permission filtering

## Common Implementation Patterns

### Dashboard Components
```typescript
// Stats cards pattern
<Card><CardContent className="p-4">
  <div className="flex items-center space-x-2">
    <Icon className="h-4 w-4 text-color" />
    <span className="text-sm text-muted-foreground">Label</span>
  </div>
  <p className="text-2xl font-bold">{count}</p>
</CardContent></Card>

// Filter pattern
<Card><CardContent>
  <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
    <Select value={filter} onValueChange={setFilter}>
      <SelectTrigger><SelectValue /></SelectTrigger>
      <SelectContent>{/* options */}</SelectContent>
    </Select>
  </div>
</CardContent></Card>
```

### Form Handling
```typescript
// Always include loading states and error handling
const [loading, setLoading] = useState(false);
const { toast } = useToast();

const handleSubmit = async () => {
  setLoading(true);
  try {
    // operation
    toast({ title: "Success", description: "Operation completed" });
  } catch (error) {
    toast({ title: "Error", description: error.message, variant: "destructive" });
  } finally {
    setLoading(false);
  }
};
```

### Data Fetching
```typescript
// Combine related data efficiently
const { data: leads } = await supabase.from('leads').select('*');
const submissionIds = leads.map(l => l.submission_id);

const { data: callResults } = await supabase
  .from('call_results')
  .select('*')
  .in('submission_id', submissionIds);

// Combine in memory
const leadsWithData = leads.map(lead => ({
  ...lead,
  call_results: callResults.filter(cr => cr.submission_id === lead.submission_id)
}));
```

## Security Considerations

### Data Access

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umarallure/lawyeronboarding](https://github.com/umarallure/lawyeronboarding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
