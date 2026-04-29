---
trigger: always_on
description: Task management system with kanban workflows and collaboration features
---

# Task Management System

Portal provides comprehensive task management with kanban boards, collaboration features, and workflow automation.

## Architecture

### Core Components
- `app/[locale]/dashboard/workspace/tasks/page.tsx`: Main task management interface
- `components/dashboard/workspace/tasks/kanban.tsx`: Kanban board implementation
- `components/dashboard/workspace/tasks/task-form.tsx`: Task creation and editing forms
- `server/routers/task.ts`: tRPC router for task operations
- `drizzle/schema.ts`: Task database schema

### Database Schema
```typescript
export const userTask = pgTable('portal_user_task', {
  id: text().primaryKey().$defaultFn(() => crypto.randomUUID()),
  userId: text().notNull().references(() => user.id, { onDelete: 'cascade' }),
  title: text().notNull(),
  description: text(),
  content: text(), // Rich text content for detailed task documentation
  status: text('status', {
    enum: ['backlog', 'todo', 'in_progress', 'in_review', 'done'],
  }).default('todo'),
  priority: text('priority', {
    enum: ['urgent', 'high', 'medium', 'low'],
  }).default('medium'),
  dueDate: timestamp({ mode: 'date' }),
  completedAt: timestamp({ mode: 'date' }),
  assignedTo: text().references(() => user.id), // can be assigned to another user
  labels: text(), // JSON array of labels/tags
  attachments: text(), // JSON array of attachment URLs/metadata
  parentTaskId: text().references(() => userTask.id), // for subtasks
  metadata: text(), // JSON string for additional data
  createdAt: timestamp({ mode: 'date' }).notNull().defaultNow(),
  updatedAt: timestamp({ mode: 'date' }).notNull().defaultNow(),
});
```

## Task Status Workflow

### Status Definitions
- **backlog**: Ideas and future tasks not yet planned
- **todo**: Tasks planned and ready to be worked on
- **in_progress**: Tasks currently being worked on
- **in_review**: Tasks completed but pending review/approval
- **done**: Completed tasks

### Status Transitions
```typescript
const ALLOWED_TRANSITIONS: Record<TaskStatus, TaskStatus[]> = {
  backlog: ['todo', 'done'], // Can move to todo or archive directly
  todo: ['in_progress', 'backlog'],
  in_progress: ['in_review', 'todo'], // Can move back to todo if blocked
  in_review: ['done', 'in_progress'], // Can be rejected back to in_progress
  done: ['todo'], // Can be reopened if needed
};

export function canTransitionTask(from: TaskStatus, to: TaskStatus): boolean {
  return ALLOWED_TRANSITIONS[from]?.includes(to) ?? false;
}
```

## Kanban Board Implementation

### Task Card Component
```tsx
interface TaskCardProps {
  task: Task;
  onUpdate: (taskId: string, updates: Partial<Task>) => void;
  onDelete: (taskId: string) => void;
}

export function TaskCard({ task, onUpdate, onDelete }: TaskCardProps) {
  const [isEditing, setIsEditing] = useState(false);
  
  const priorityColors = {
    urgent: 'bg-red-100 text-red-800 dark:bg-red-900/20 dark:text-red-400',
    high: 'bg-orange-100 text-orange-800 dark:bg-orange-900/20 dark:text-orange-400',
    medium: 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/20 dark:text-yellow-400',
    low: 'bg-green-100 text-green-800 dark:bg-green-900/20 dark:text-green-400',
  };

  const isOverdue = task.dueDate && new Date(task.dueDate) < new Date() && task.status !== 'done';

  return (
    <Card className={cn(
      "cursor-pointer transition-colors hover:shadow-md",
      isOverdue && "border-red-200 bg-red-50/50"
    )}>
      <CardContent className="p-4">
        <div className="flex items-start justify-between mb-2">
          <h4 className="font-medium text-sm line-clamp-2">{task.title}</h4>
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button variant="ghost" size="sm">
                <MoreHorizontalIcon className="h-4 w-4" />
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent>
              <DropdownMenuItem onClick={() => setIsEditing(true)}>
                <PencilIcon className="h-4 w-4 mr-2" />
                Edit
              </DropdownMenuItem>
              <DropdownMenuItem 
                onClick={() => onDelete(task.id)}
                className="text-red-600"
              >
                <TrashIcon className="h-4 w-4 mr-2" />
                Delete
              </DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>
        </div>

        {task.description && (
          <p className="text-xs text-muted-foreground mb-3 line-clamp-2">
            {task.description}
          </p>
        )}

        <div className="flex items-center justify-between">
          <Badge 
            variant="secondary" 
            className={cn("text-xs", priorityColors[task.priority])}
          >
            {task.priority}
          </Badge>

          {task.dueDate && (
            <div className={cn(
              "flex items-center text-xs",
              isOverdue ? "text-red-600" : "text-muted-foreground"
            )}>
              <CalendarIcon className="h-3 w-3 mr-1" />
              {format(new Date(task.dueDate), 'MMM dd')}
            </div>
          )}
        </div>

        {task.assignedTo && (
          <div className="flex items-center mt-2">
            <Avatar className="h-6 w-6">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okisdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
