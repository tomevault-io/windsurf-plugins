---
trigger: always_on
description: cat package.json 2>/dev/null || echo "Project not initialized yet"
---

# AIPulse — Website Brand Guide & Build Instructions
# For Claude Code — Read this file at the start of every session

---

## 0. FIRST STEPS (every session)

```bash
# 1. Read this file completely before writing any code
# 2. Check existing project state:
ls -la
cat package.json 2>/dev/null || echo "Project not initialized yet"
# 3. Continue from where the last session ended — never restart from scratch
```

---

## 1. PROJECT OVERVIEW

| Key | Value |
|-----|-------|
| Company | AIPulse |
| Domain | aipulse.ca |
| Type | AI Consulting & Automation — Canada |
| Languages | English (default, LTR) + Farsi/Persian (RTL) |
| Feel | Luxury tech consultancy — McKinsey meets OpenAI |
| Primary tagline | *"Your vision, our intelligence, one pulse."* |
| Farsi tagline | *«چشم‌انداز شما، هوش ما، یک نبض مشترک»* |

---

## 2. TECH STACK

```
Framework:      Next.js 14 (App Router, TypeScript)
Styling:        Tailwind CSS + CSS custom properties
Animations:     Framer Motion
i18n:           next-intl (locale routing: /en/... and /fa/...)
Forms:          React Hook Form + Zod
Booking:        Cal.com embed (@calcom/embed-react)
Images:         next/image (WebP, lazy load)
Fonts:          next/font/google
Icons:          lucide-react
Deployment:     Vercel
```

### Initialize (only if project doesn't exist yet)
```bash
npx create-next-app@latest aipulse \
  --typescript --tailwind --eslint --app --src-dir=false
cd aipulse
npm install framer-motion next-intl react-hook-form zod \
  @hookform/resolvers @calcom/embed-react lucide-react
```

---

## 3. TAILWIND CONFIG

Replace `tailwind.config.ts` with exactly this:

```ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        navy: {
          DEFAULT: '#0B1622',
          mid:     '#111F2E',
          soft:    '#1A2E42',
        },
        gold: {
          DEFAULT: '#C9A84C',
          light:   '#E8D08A',
          pale:    '#F5EDD0',
        },
        cream:  '#F8F7F3',
        silver: '#A8B4C0',
        ink:    '#1A2535',
      },
      fontFamily: {
        display: ['var(--font-cormorant)', 'Georgia', 'serif'],
        sans:    ['var(--font-inter)', 'Arial', 'sans-serif'],
        farsi:   ['var(--font-vazirmatn)', 'Arial', 'sans-serif'],
      },
      letterSpacing: {
        widest2: '0.3em',
        widest3: '0.4em',
      },
    },
  },
  plugins: [],
}
export default config
```

---

## 4. CSS VARIABLES

`app/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --navy:        #0B1622;
  --navy-mid:    #111F2E;
  --navy-soft:   #1A2E42;
  --gold:        #C9A84C;
  --gold-light:  #E8D08A;
  --gold-pale:   #F5EDD0;
  --cream:       #F8F7F3;
  --silver:      #A8B4C0;
  --ink:         #1A2535;
  --white:       #FFFFFF;
}

/* Gold rule utility */
.gold-rule {
  height: 1px;
  background: var(--gold);
  opacity: 0.4;
}

/* Section label style */
.section-label {
  font-size: 10px;
  letter-spacing: 0.35em;
  text-transform: uppercase;
  color: var(--gold);
  font-weight: 500;
}

/* RTL support */
[dir="rtl"] {
  font-family: var(--font-vazirmatn), Arial, sans-serif;
}
[dir="rtl"] .gold-rule-left {
  margin-right: 0;
  margin-left: auto;
}
```

---

## 5. FONTS

`lib/fonts.ts`:

```ts
import { Cormorant_Garamond, Inter, Vazirmatn } from 'next/font/google'

export const cormorant = Cormorant_Garamond({
  subsets: ['latin'],
  weight: ['300', '400', '600'],
  style: ['normal', 'italic'],
  variable: '--font-cormorant',
  display: 'swap',
})

export const inter = Inter({
  subsets: ['latin'],
  weight: ['300', '400', '500'],
  variable: '--font-inter',
  display: 'swap',
})

export const vazirmatn = Vazirmatn({
  subsets: ['arabic'],
  weight: ['300', '400', '700'],
  variable: '--font-vazirmatn',
  display: 'swap',
})
```

`app/layout.tsx`:

```tsx
import { cormorant, inter, vazirmatn } from '@/lib/fonts'
import { NextIntlClientProvider } from 'next-intl'
import { getMessages, getLocale } from 'next-intl/server'

export default async function RootLayout({ children }: { children: React.ReactNode }) {
  const locale = await getLocale()
  const messages = await getMessages()
  const dir = locale === 'fa' ? 'rtl' : 'ltr'

  return (
    <html lang={locale} dir={dir}
      className={`${cormorant.variable} ${inter.variable} ${vazirmatn.variable}`}>
      <body className="bg-cream text-ink antialiased">
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
```

---

## 6. PULSE CREST LOGO COMPONENT

`components/PulseCrest.tsx`:

```tsx
'use client'
import { motion } from 'framer-motion'

interface Props {
  size?: number
  variant?: 'light' | 'dark' | 'gold'
  animate?: boolean
}

export function PulseCrest({ size = 80, variant = 'dark', animate = true }: Props) {
  const strokeColor = variant === 'gold' ? '#0B1622' : '#C9A84C'
  const fillColor   = variant === 'light' ? '#F0EFE8' : '#111F2E'
  const gemColor    = variant === 'gold' ? '#0B1622' : '#C9A84C'

  return (
    <motion.svg
      width={size}
      height={size * 1.15}
      viewBox="0 0 130 150"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [instagram-a11y/aipulse](https://github.com/instagram-a11y/aipulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
