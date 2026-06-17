---
trigger: always_on
description: This guide covers how to use the zen component library in your React applications.
---

# SKILL.md - Using @umami/react-zen

This guide covers how to use the zen component library in your React applications.

## Installation

```bash
pnpm add @umami/react-zen
```

Import the stylesheet in your app entry point:

```tsx
import '@umami/react-zen/styles.css';
```

## Setup

Wrap your app with `ZenProvider`:

```tsx
import { ZenProvider } from '@umami/react-zen';

function App() {
  return (
    <ZenProvider theme="light" palette="neutral">
      {children}
    </ZenProvider>
  );
}
```

### Provider Options

| Prop | Values | Description |
|------|--------|-------------|
| `theme` | `'light'` \| `'dark'` | Color theme |
| `colorScheme` | `'light'` \| `'dark'` \| `'system'` | System preference support |
| `palette` | `'neutral'` \| `'slate'` \| `'gray'` \| `'zinc'` \| `'stone'` | Color palette |
| `toast` | `{ duration: number }` | Toast defaults |

## Components

### Layout

#### Box

Universal container with styling props:

```tsx
<Box padding="4" margin="2" backgroundColor="surface-raised" borderRadius="lg">
  Content
</Box>
```

#### Flexbox, Row, Column

Flex layouts with gap, alignment, and justification:

```tsx
<Row gap="4" justifyContent="space-between" alignItems="center">
  <Text>Left</Text>
  <Text>Right</Text>
</Row>

<Column gap="2">
  <Text>First</Text>
  <Text>Second</Text>
</Column>

<Flexbox direction="row" wrap="wrap" gap="2">
  {items.map(item => <Item key={item.id} />)}
</Flexbox>
```

#### Grid

CSS Grid layout:

```tsx
<Grid columns="3" gap="4">
  <Box>1</Box>
  <Box>2</Box>
  <Box>3</Box>
</Grid>
```

### Responsive Props

All layout props support responsive values:

```tsx
<Box
  padding={{ base: '2', md: '4', lg: '6' }}
  display={{ base: 'none', md: 'block' }}
  width={{ base: 'full', md: '1/2', lg: '1/3' }}
>
  Responsive content
</Box>
```

Breakpoints: `base`, `sm`, `md`, `lg`, `xl`, `2xl`

### Buttons

```tsx
// Variants
<Button variant="default">Default</Button>
<Button variant="primary">Primary</Button>
<Button variant="outline">Outline</Button>
<Button variant="quiet">Quiet</Button>
<Button variant="danger">Danger</Button>

// Sizes
<Button size="xs">Extra Small</Button>
<Button size="sm">Small</Button>
<Button size="md">Medium</Button>
<Button size="lg">Large</Button>
<Button size="xl">Extra Large</Button>

// With loading state
<LoadingButton isLoading={loading} variant="primary">
  Submit
</LoadingButton>

// Copy to clipboard
<CopyButton value="text to copy" />

// Theme toggle
<ThemeButton />
```

### Typography

```tsx
// Headings
<Heading size="6xl">Hero</Heading>
<Heading size="4xl">Page Title</Heading>
<Heading size="2xl">Section</Heading>
<Heading size="lg">Subsection</Heading>

// Text
<Text>Default text</Text>
<Text size="sm" color="muted">Small muted text</Text>
<Text weight="bold" color="primary">Bold primary</Text>

// Code
<Code>inline code</Code>
<CodeBlock language="typescript">{code}</CodeBlock>
```

### Form System

The form system is built on react-hook-form:

```tsx
import { Form, FormField, FormSubmitButton, TextField } from '@umami/react-zen';

function LoginForm() {
  const handleSubmit = (data) => {
    console.log(data);
  };

  return (
    <Form onSubmit={handleSubmit} defaultValues={{ email: '', password: '' }}>
      <FormField
        name="email"
        label="Email"
        description="We'll never share your email"
        rules={{ required: 'Email is required' }}
      >
        <TextField type="email" />
      </FormField>

      <FormField
        name="password"
        label="Password"
        rules={{ required: 'Password is required', minLength: 8 }}
      >
        <PasswordField />
      </FormField>

      <FormButtons>
        <FormResetButton>Reset</FormResetButton>
        <FormSubmitButton variant="primary">Login</FormSubmitButton>
      </FormButtons>
    </Form>
  );
}
```

#### Form Fields

```tsx
// Text input
<TextField placeholder="Enter text" />

// Password with visibility toggle
<PasswordField />

// Search with debouncing
<SearchField onSearch={handleSearch} delay={300} />

// Checkbox
<Checkbox>Remember me</Checkbox>

// Radio group
<RadioGroup label="Size">
  <Radio value="sm">Small</Radio>
  <Radio value="md">Medium</Radio>
  <Radio value="lg">Large</Radio>
</RadioGroup>

// Switch
<Switch>Enable notifications</Switch>

// Toggle buttons
<ToggleGroup>
  <Toggle id="bold">Bold</Toggle>
  <Toggle id="italic">Italic</Toggle>
</ToggleGroup>
```

#### Dynamic Field Arrays

```tsx
<FormFieldArray name="items" label="Items">
  {({ fields, append, remove }) => (
    <>
      {fields.map((field, index) => (
        <Row key={field.id} gap="2">
          <FormField name={`items.${index}.name`}>
            <TextField />
          </FormField>
          <Button onClick={() => remove(index)}>Remove</Button>
        </Row>
      ))}
      <Button onClick={() => append({ name: '' })}>Add Item</Button>
    </>
  )}
</FormFieldArray>
```

### Select & ComboBox

```tsx
// Basic select
<Select label="Country">
  <ListItem id="us">United States</ListItem>
  <ListItem id="uk">United Kingdom</ListItem>
  <ListItem id="ca">Canada</ListItem>
</Select>

// Searchable select
<Select label="Country" allowSearch onSearch={handleSearch}>
  <ListItem id="us">United States</ListItem>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umami-software/react-zen](https://github.com/umami-software/react-zen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
