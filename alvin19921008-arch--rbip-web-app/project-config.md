---
trigger: always_on
description: Design conventions for RBIP dashboard panels with flat hierarchy, progressive disclosure, and modern UX patterns.
---

# Dashboard UI Design Principles

Design conventions for RBIP dashboard panels with flat hierarchy, progressive disclosure, and modern UX patterns.

## Core Philosophy

- **Flat hierarchy**: Maximum 2 nesting levels (page → section/card)
- **Progressive disclosure**: Show summary first, reveal details on interaction
- **Clear confirmation**: Actions require explicit user confirmation, especially destructive ones
- **Visual clarity**: Use spacing, dividers, and typography instead of nested borders

---

## Layout & Hierarchy

### Flat Structure

```tsx
// ❌ Nested Card (bad)
<Card>
  <Card>
    <CardHeader>Title</CardHeader>
    <CardContent>Content</CardContent>
  </Card>
</Card>

// ✅ Flat layout with dividers (good)
<div className="space-y-6">
  <section>
    <h3 className="text-lg font-semibold">Section Title</h3>
    <hr className="border-border my-4" />
    <div>Content here</div>
  </section>
</div>
```

### Section Headers

- Use uppercase tracking for sub-section headers
- Font: `text-xs font-semibold text-muted-foreground uppercase tracking-wide`
- Margin: `mb-3` for headers, `mb-2` for helper text

```tsx
<h4 className="text-xs font-semibold text-muted-foreground uppercase tracking-wide mb-3">
  SECTION HEADER
</h4>
```

### Dividers

- Use `<hr className="border-border" />` for horizontal separation
- Use `divide-y` or `divide-x` for lists
- Avoid nested border containers

---

## Progressive Disclosure

### Collapsible Rank Groups

Use chevron icons with collapsible sections:

```tsx
// State
const [collapsedRankGroups, setCollapsedRankGroups] = useState<Set<'therapists' | 'pca'>>(new Set(['pca']))

// Toggle handler
const toggleRankGroup = (group: 'therapists' | 'pca') => {
  setCollapsedRankGroups(prev => {
    const next = new Set(prev)
    if (next.has(group)) {
      next.delete(group)
    } else {
      next.add(group)
    }
    return next
  })
}

// Render
<div className="flex items-center gap-2">
  <button onClick={() => toggleRankGroup('therapists')}>
    {collapsedRankGroups.has('therapists') ? (
      <ChevronRight className="h-4 w-4" />
    ) : (
      <ChevronDown className="h-4 w-4" />
    )}
  </button>
  <span className="text-sm font-medium">THERAPISTS</span>
</div>
```

### Per-Item Expand/Collapse

For individual staff items within a group:

```tsx
const [expandedStaffIds, setExpandedStaffIds] = useState<Set<string>>(new Set())

// Toggle
const toggleStaffExpand = (staffId: string) => {
  setExpandedStaffIds(prev => {
    const next = new Set(prev)
    next.has(staffId) ? next.delete(staffId) : next.add(staffId)
    return next
  })
}

// Render with chevron
<div className="flex items-center gap-2">
  <button onClick={() => toggleStaffExpand(id)}>
    {expandedStaffIds.has(id) ? (
      <ChevronDown className="h-4 w-4" />
    ) : (
      <ChevronRight className="h-4 w-4" />
    )}
  </button>
  <span>{staffName}</span>
</div>
```

---

## Checkbox Selection with Confirmation

### Multi-Select Staff List

```tsx
const [staffIdsToAdd, setStaffIdsToAdd] = useState<Set<string>>(new Set())

// Toggle selection
const toggleStaffSelection = (staffId: string) => {
  setStaffIdsToAdd(prev => {
    const next = new Set(prev)
    next.has(staffId) ? next.delete(staffId) : next.add(staffId)
    return next
  })
}

// Render checklist (filter inactive, separate regular/buffer)
<div className="max-h-40 overflow-y-auto bg-muted/30 rounded p-2 pr-1 scrollbar-visible">
  {regularStaff.map(s => (
    <label key={s.id} className="flex items-center space-x-2 py-1 cursor-pointer hover:bg-muted/50 rounded px-1">
      <input
        type="checkbox"
        checked={staffIdsToAdd.has(s.id)}
        onChange={() => toggleStaffSelection(s.id)}
      />
      <span>{s.name} ({s.rank})</span>
    </label>
  ))}
  {/* Divider before buffer staff */}
  {bufferStaff.length > 0 && regularStaff.length > 0 && (
    <hr className="border-border my-2" />
  )}
  {bufferStaff.map(s => (
    <label key={s.id} className="flex items-center space-x-2 py-1 cursor-pointer hover:bg-muted/50 rounded px-1">
      <input type="checkbox" checked={staffIdsToAdd.has(s.id)} onChange={() => toggleStaffSelection(s.id)} />
      <span>{s.name} (Floating, Buffer)</span>
    </label>
  ))}
</div>
```

### Confirmation Action Buttons

```tsx
{staffIdsToAdd.size > 0 && (
  <div className="flex items-center gap-2 mt-3">
    <Button
      size="sm"
      className="h-8 px-3 text-xs font-medium bg-[#0f172a] text-white rounded-md hover:bg-[#1e293b] transition-all"
      onClick={async () => {
        await handleAddStaffToProgram(programName, Array.from(staffIdsToAdd))
        setStaffIdsToAdd(new Set())
      }}
    >
      Add Selected ({staffIdsToAdd.size})
    </Button>
    <Button
      size="sm"
      variant="ghost"
      className="h-8 px-3 text-xs font-medium text-slate-500 hover:bg-white hover:text-slate-900 border border-transparent hover:border-slate-200 rounded-md transition-all"
      onClick={() => setStaffIdsToAdd(new Set())}
    >
      Clear
    </Button>
  </div>
)}
```

---

## Inline Delete Confirmation

Show "Confirm?" button after clicking delete/trash icon:

```tsx
const [staffIdPendingDelete, setStaffIdPendingDelete] = useState<string | null>(null)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alvin19921008-arch/RBIP-web-app](https://github.com/alvin19921008-arch/RBIP-web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
