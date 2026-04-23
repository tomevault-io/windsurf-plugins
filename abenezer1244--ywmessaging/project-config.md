---
trigger: always_on
description: ALERT: THIS IS NOT A MOCK OR TEST OR DUMMY PROJECT. IT IS A REAL WORLD ENTERPRISE LEVEL SAAS SO NEVER ADD A MOCK OR TEST OR DUMMY CODE.
---

ALERT: THIS IS NOT A MOCK OR TEST OR DUMMY PROJECT. IT IS A REAL WORLD ENTERPRISE LEVEL SAAS SO NEVER ADD A MOCK OR TEST OR DUMMY CODE.

When read a files and make sure to read the whole lines chunk by chunk so you wouldn't not miss anything because of any reason.

1. First think through the problem, read the codebase for relevant files, and write a plan to tasks/todo.md.
2. The plan should have a list of todo items that you can check off as you complete them
3. Before you begin working, check in with me and I will verify the plan.
4. Then, begin working on the todo items, marking them as complete as you go.
5. Please every step of the way just give me a high level explanation of what changes you made
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
7. Finally, add a review section to the todo.md file with a summary of the changes you made and any other relevant information.
8. DO NOT BE LAZY. NEVER BE LAZY. IF THERE IS A BUG FIND THE ROOT CAUSE AND FIX IT. NO TEMPORARY FIXES. YOU ARE A SENIOR DEVELOPER. NEVER BE LAZY
9. MAKE ALL FIXES AND CODE CHANGES AS SIMPLE AS HUMANLY POSSIBLE. THEY SHOULD ONLY IMPACT NECESSARY CODE RELEVANT TO THE TASK AND NOTHING ELSE. ASK IS THE WORK IS ENTERPRISE OR A HOBBY SO EVERY FIX WILL HAVE LEVEL OF FIX. YOUR GOAL IS TO NOT INTRODUCE ANY BUGS. IT'S ALL ABOUT SIMPLICITY

CRITICAL: When debugging, you MUST trace through the ENTIRE code flow step by step. No assumptions. No shortcuts.

## ⚠️ DATABASE SAFETY RULES (CRITICAL)

**NEVER** run these commands on production:
- `npx prisma db push --accept-data-loss` ❌ - CAN DROP TABLES AND DELETE ALL DATA
- `npx prisma migrate reset` ❌ - DELETES ALL DATA

**ALWAYS** do this before schema changes:
1. Ask user to confirm they have backups enabled on Render
2. Use `npx prisma db push` WITHOUT `--accept-data-loss`
3. If Prisma warns about data loss, STOP and ask user

**For tenant schema:** NEVER run `prisma db push` with tenant-schema.prisma on the main DATABASE_URL - it will drop registry tables (Church, Tenant, AdminEmailIndex).



```

**ErrorBoundary Features**:

- Automatic fallback UI with retry/refresh options
- Development mode error details
- Custom error handling callbacks
- Integration with NextUI design system

**ErrorFallback Variants**:

- `default` - Full error display with actions
- `minimal` - Compact inline error message  
- `minimal` - Compact inline error message
- `detailed` - Includes error stack trace in development

## Visual Development & Testing

### Design System

The project follows S-Tier SaaS design standards inspired by Stripe, Airbnb, and Linear. All UI development must adhere to:

- **Design Principles**: `/context/design-principles.md` - Comprehensive checklist for world-class UI
- **Component Library**: NextUI with custom Tailwind configuration

### Quick Visual Check

**IMMEDIATELY after implementing any front-end change:**

1. **Identify what changed** - Review the modified components/pages
2. **Navigate to affected pages** - Use `mcp__playwright__browser_navigate` to visit each changed view
3. **Verify design compliance** - Compare against `/context/design-principles.md`
4. **Validate feature implementation** - Ensure the change fulfills the user's specific request
5. **Check acceptance criteria** - Review any provided context files or requirements
6. **Capture evidence** - Take full page screenshot at desktop viewport (1440px) of each changed view
7. **Check for errors** - Run `mcp__playwright__browser_console_messages` ⚠️

This verification ensures changes meet design standards and user requirements.

### Comprehensive Design Review

For significant UI changes or before merging PRs, use the design review agent:

```bash
# Option 1: Use the slash command
/design-review

# Option 2: Invoke the agent directly
@agent-design-review
```

The design review agent will:

- Test all interactive states and user flows
- Verify responsiveness (desktop/tablet/mobile)
- Check accessibility (WCAG 2.1 AA compliance)
- Validate visual polish and consistency
- Test edge cases and error states
- Provide categorized feedback (Blockers/High/Medium/Nitpicks)

### Playwright MCP Integration

#### Essential Commands for UI Testing

```javascript
// Navigation & Screenshots
mcp__playwright__browser_navigate(url); // Navigate to page
mcp__playwright__browser_take_screenshot(); // Capture visual evidence
mcp__playwright__browser_resize(
  width,
  height
); // Test responsiveness

// Interaction Testing
mcp__playwright__browser_click(element); // Test clicks
mcp__playwright__browser_type(
  element,
  text
); // Test input
mcp__playwright__browser_hover(element); // Test hover states

// Validation
mcp__playwright__browser_console_messages(); // Check for errors
mcp__playwright__browser_snapshot(); // Accessibility check
mcp__playwright__browser_wait_for(
  text / element
); // Ensure loading
```

### Design Compliance Checklist

When implementing UI features, verify:

- [ ] **Visual Hierarchy**: Clear focus flow, appropriate spacing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abenezer1244) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
