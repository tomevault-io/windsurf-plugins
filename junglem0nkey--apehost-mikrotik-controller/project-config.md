---
trigger: always_on
description: **CRITICAL**: This document defines the design system and UI implementation rules for this project. All code must follow these guidelines.
---

# MikroTik Dashboard Design Guidelines

**CRITICAL**: This document defines the design system and UI implementation rules for this project. All code must follow these guidelines.

## Design System Architecture

This application uses a **custom design system** with **selective Ant Design integration**. It is NOT a pure Ant Design application.

### Component Hierarchy

```
Atoms (Basic Components)
├── Input
├── Button
├── Toggle
└── Textarea

Molecules (Composite Components)
├── FormField (wraps Input/Textarea with label, helpText, error)
├── ToggleField (wraps Toggle with label, description)
└── [other molecules as needed]

Organisms (Complex Components)
├── SettingsSection (groups related settings)
└── [other organisms as needed]

Ant Design (Selective Use Only)
├── Tabs (has token overrides)
├── Slider (has token overrides)
├── Card (has token overrides)
├── Alert (has token overrides)
├── Spin (has token overrides)
├── Progress (has token overrides)
├── Badge (has token overrides)
├── Tag (has token overrides)
├── Statistic (has token overrides)
└── Modal (has token overrides)
```

## Core Rules

### Rule 0: No Emojis

**NEVER** use emojis in any code, documentation, comments, commit messages, or UI text unless explicitly instructed by the user.

This includes but is not limited to:
- Code comments
- README files
- Documentation
- UI labels and text
- Error messages
- Log messages
- Commit messages
- Component names or descriptions

Use plain text equivalents instead:
- Instead of "✅ Success", use "Success" or "[OK]"
- Instead of "❌ Error", use "Error" or "[FAILED]"
- Instead of "🚀 Deploy", use "Deploy"
- Instead of "⚠️ Warning", use "Warning" or "[WARN]"

### Rule 1: Use Custom Components for Form Elements

**ALWAYS** use our custom components instead of Ant Design equivalents:

**Correct:**
```tsx
import { Input } from '../../components/atoms/Input/Input';
import { Button } from '../../components/atoms/Button/Button';
import { Toggle } from '../../components/atoms/Toggle/Toggle';
import { Textarea } from '../../components/atoms/Textarea/Textarea';

<Input value={value} onChange={setValue} />
<Button variant="primary">Save</Button>
<Toggle checked={enabled} onChange={setEnabled} />
<Textarea value={text} onChange={setText} />
```

**Wrong:**
```tsx
import { Input, Button, Switch, Form } from 'antd';

<Form.Item>
  <Input value={value} onChange={e => setValue(e.target.value)} />
</Form.Item>
<Button type="primary">Save</Button>
<Switch checked={enabled} onChange={setEnabled} />
```

### Rule 2: Use FormField and ToggleField for Form Layouts

**ALWAYS** wrap form inputs with FormField or ToggleField molecules:

**Correct:**
```tsx
import { FormField } from '../../components/molecules/FormField/FormField';
import { ToggleField } from '../../components/molecules/ToggleField/ToggleField';

<FormField
  label="Server Port"
  helpText="Port number for the backend server"
  error={errors.port}
>
  <Input
    type="number"
    value={port}
    onChange={setPort}
    placeholder="3000"
  />
</FormField>

<ToggleField
  label="Enable Feature"
  description="Toggle this feature on or off"
  checked={enabled}
  onChange={setEnabled}
/>
```

**Wrong:**
```tsx
<div>
  <label>Server Port</label>
  <Input value={port} onChange={setPort} />
  <span>Port number for the backend server</span>
</div>

<div>
  <Toggle checked={enabled} onChange={setEnabled} />
  <span>Enable Feature</span>
</div>
```

### Rule 3: Use SettingsSection for Grouping

**ALWAYS** group related settings using SettingsSection:

**Correct:**
```tsx
import { SettingsSection } from '../../components/organisms/SettingsSection/SettingsSection';

<SettingsSection
  title="Server Configuration"
  description="Configure backend server settings"
>
  <FormField label="Port">
    <Input value={port} onChange={setPort} />
  </FormField>
  {/* more fields */}
</SettingsSection>
```

**Wrong:**
```tsx
<div>
  <h3>Server Configuration</h3>
  <p>Configure backend server settings</p>
  <Input value={port} onChange={setPort} />
</div>
```

### Rule 4: Only Use Approved Ant Design Components

**ONLY** use Ant Design components that have CSS token overrides in `src/styles/tokens.css`:

**Approved for Use:**
- `Tabs` - Tab navigation
- `Slider` - Range inputs
- `Card` - Content containers
- `Alert` - Notifications
- `Spin` - Loading indicators
- `Progress` - Progress bars
- `Badge` - Status indicators
- `Tag` - Labels and tags
- `Statistic` - Numeric displays
- `Modal` - Dialog boxes and confirmations

**Never Use (Use Custom Components Instead):**
- `Form` → Use controlled components with useState
- `Input` → Use custom `Input` component
- `Button` → Use custom `Button` component
- `Switch` → Use custom `Toggle` component
- `Select` → Use custom `<select>` with CSS modules
- `InputNumber` → Use custom `Input` with type="number"
- `TextArea` → Use custom `Textarea` component
- `Checkbox` → Use custom checkbox implementation
- `Radio` → Use custom radio implementation

### Rule 5: All Styling Via CSS Modules with Design Tokens

**NEVER** use inline styles. **ALWAYS** use CSS modules with design tokens.

**Correct:**
```tsx
// Component.module.css
.container {
  padding: var(--space-xl);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JungleM0nkey/apehost-mikrotik-controller](https://github.com/JungleM0nkey/apehost-mikrotik-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
