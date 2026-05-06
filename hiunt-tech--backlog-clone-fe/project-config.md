---
trigger: always_on
description: Hệ thống Credit Management hỗ trợ tùy chỉnh màu sắc giao diện theo từng người dùng. Tất cả màu sắc phải sử dụng theme variables thay vì hardcode để đảm bảo tính nhất quán và khả năng tùy chỉnh.
---


# Quy ước sử dụng Theme Colors

## 1. Tổng quan

Hệ thống Credit Management hỗ trợ tùy chỉnh màu sắc giao diện theo từng người dùng. Tất cả màu sắc phải sử dụng theme variables thay vì hardcode để đảm bảo tính nhất quán và khả năng tùy chỉnh.

## 2. CSS Variables có sẵn

```css
/* Theme colors được set trong :root */
--theme-main: #15bec6; /* Màu chính */
--theme-hover: #1d4ed8; /* Màu khi hover */
--theme-text: #1f2937; /* Màu chữ chính */
--theme-disabled: #9ca3af; /* Màu khi disabled */
--theme-border: #e5e7eb; /* Màu viền */
--theme-background: #ffffff; /* Màu nền */
```

## 3. Sử dụng trong CSS/Tailwind

### ✅ ĐÚNG - Sử dụng CSS variables

```css
/* Trong CSS files */
.my-button {
  background-color: var(--theme-main);
  color: var(--theme-text);
  border: 1px solid var(--theme-border);
}

.my-button:hover {
  background-color: var(--theme-hover);
}

.my-button:disabled {
  background-color: var(--theme-disabled);
  color: var(--theme-disabled);
}
```

```tsx
// Trong Tailwind CSS với arbitrary values
<button
  className="bg-[var(--theme-main)] text-[var(--theme-neutral-11)]
             border border-theme-main
             hover:bg-[var(--theme-hover)]
             disabled:bg-theme-neutral-6 disabled:text-theme-neutral-7"
>
  Button
</button>

// HOẶC sử dụng theme classes đã định nghĩa
<button
  className="bg-theme-main text-theme-neutral-11
             border border-theme-main
             hover:bg-theme-hover
             disabled:bg-theme-neutral-6 disabled:text-theme-neutral-7"
>
  Button
</button>
```

### ❌ SAI - Hardcode màu sắc

```css
/* KHÔNG làm thế này */
.my-button {
  background-color: #2563eb; /* Hardcode màu */
  color: #1f2937; /* Hardcode màu */
}

.my-button:hover {
  background-color: #1d4ed8; /* Hardcode màu */
}
```

```tsx
// KHÔNG làm thế này
<button
  className="bg-blue-600 text-gray-800 border border-gray-200
             hover:bg-blue-700 disabled:bg-gray-400 disabled:text-gray-400"
>
  Button
</button>
```

## 4. Sử dụng trong React Components

### ✅ ĐÚNG - Sử dụng CSS variables trong inline styles

```tsx
const MyButton = () => {
  return (
    <button
      className="px-4 py-2 rounded border"
      style={{
        backgroundColor: 'var(--theme-main)',
        color: 'var(--theme-text)',
        borderColor: 'var(--theme-border)',
      }}
      onMouseEnter={e => {
        e.currentTarget.style.backgroundColor = 'var(--theme-hover)';
      }}
      onMouseLeave={e => {
        e.currentTarget.style.backgroundColor = 'var(--theme-main)';
      }}
    >
      Button
    </button>
  );
};
```

### ❌ SAI - Hardcode màu trong React components

```tsx
const MyButton = () => {
  return (
    <button
      className="bg-blue-600 text-gray-800 border border-gray-200 px-4 py-2 rounded"
      style={{
        backgroundColor: '#2563eb', // Hardcode!
        color: '#1f2937', // Hardcode!
      }}
    >
      Button
    </button>
  );
};
```

## 5. Sử dụng với Shadcn/UI Components

### Button Component

```tsx
import { Button } from '@/components/ui/button';

// ✅ ĐÚNG - Sử dụng theme classes
const ThemedButton = () => {
  return (
    <Button
      className="bg-theme-main text-theme-neutral-1
                 hover:bg-theme-hover
                 border-theme-main"
    >
      Themed Button
    </Button>
  );
};

// HOẶC sử dụng arbitrary values
const ThemedButtonAlt = () => {
  return (
    <Button
      className="bg-[var(--theme-main)] text-[var(--theme-neutral-1)]
                 hover:bg-[var(--theme-hover)]
                 border-[var(--theme-main)]"
    >
      Themed Button
    </Button>
  );
};
```

### Input Component

```tsx
import { Input } from '@/components/ui/input';

const ThemedInput = () => {
  return (
    <Input
      className="border-theme-neutral-4
                 focus:border-theme-main
                 text-theme-neutral-11
                 placeholder:text-theme-neutral-6"
      placeholder="Enter text..."
    />
  );
};
```

### Card Component

```tsx
import { Card, CardContent, CardHeader } from '@/components/ui/card';

const ThemedCard = () => {
  return (
    <Card className="border-theme-neutral-4 bg-theme-neutral-1">
      <CardHeader className="text-theme-neutral-11">Card Title</CardHeader>
      <CardContent>
        <p className="text-theme-neutral-9">Card content with themed colors</p>
      </CardContent>
    </Card>
  );
};
```

## 6. Sử dụng với Data Tables

### Table Headers

```tsx
const TableHeader = () => {
  return (
    <thead className="bg-[var(--theme-background)] border-b border-[var(--theme-border)]">
      <tr>
        <th className="text-left text-[var(--theme-text)] font-medium p-4">
          Column 1
        </th>
        <th className="text-left text-[var(--theme-text)] font-medium p-4">
          Column 2
        </th>
      </tr>
    </thead>
  );
};
```

### Table Rows

```tsx
const TableRow = ({ isSelected }: { isSelected?: boolean }) => {
  return (
    <tr
      className={`border-b border-[var(--theme-border)]
                  hover:bg-[var(--theme-background)]`}
      style={{
        backgroundColor: isSelected ? 'var(--theme-main)' : 'transparent',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiuNT-Tech/backlog-clone-fe](https://github.com/HiuNT-Tech/backlog-clone-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
