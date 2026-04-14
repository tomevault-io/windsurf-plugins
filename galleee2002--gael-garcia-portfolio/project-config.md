---
trigger: always_on
description: Estas reglas DEBEN ser leídas y seguidas ANTES de ejecutar cualquier prompt o tarea en este proyecto.
---

# 🎯 Reglas de Cursor AI para Gael García Portfolio

## 📋 INSTRUCCIONES GENERALES

Estas reglas DEBEN ser leídas y seguidas ANTES de ejecutar cualquier prompt o tarea en este proyecto.

---

## 🏗️ ARQUITECTURA DEL PROYECTO

### Patrón Atomic Design

Este proyecto sigue ESTRICTAMENTE el patrón Atomic Design:

```
Atoms → Molecules → Organisms → Templates → Pages
```

**NUNCA:**

- Mezclar niveles jerárquicos incorrectamente
- Crear un componente en el nivel incorrecto
- Importar un nivel superior en un nivel inferior (ej: Organism en Atom)

**SIEMPRE:**

- Atoms solo contienen componentes UI básicos sin dependencias
- Molecules combinan Atoms
- Organisms combinan Molecules y Atoms
- Templates definen layouts sin contenido específico
- Pages son instancias de Templates con contenido real

---

## 📁 ESTRUCTURA DE ARCHIVOS

### Organización de Componentes

Cada componente DEBE seguir esta estructura EXACTA:

```
ComponentName/
├── ComponentName.tsx    (PascalCase, mismo nombre que la carpeta)
└── index.ts            (export default únicamente)
```

**PROHIBIDO:**

- Crear múltiples componentes en un solo archivo
- Usar nombres diferentes entre carpeta y archivo
- Exportar múltiples componentes desde index.ts
- Crear archivos de estilos CSS separados (usar Tailwind)

---

## 💻 CONVENCIONES DE CÓDIGO

### 1. Nomenclatura

**Componentes (Archivos .tsx):**

- SIEMPRE PascalCase: `Button.tsx`, `FormField.tsx`, `ProjectCard.tsx`
- El nombre del archivo DEBE coincidir con el nombre del componente
- El nombre de la carpeta DEBE coincidir con el nombre del componente

**Hooks:**

- SIEMPRE prefijo `use` + camelCase: `useWindowSize.ts`, `useAuth.ts`
- Un hook por archivo

**Utilidades:**

- SIEMPRE camelCase: `formatters.ts`, `validators.ts`
- Agrupar funciones relacionadas

**Tipos:**

- SIEMPRE PascalCase con sufijo `.types.ts`: `common.types.ts`, `user.types.ts`

### 2. Imports

**OBLIGATORIO usar alias de importación:**

```typescript
// ✅ CORRECTO
import Button from "@atoms/Button";
import FormField from "@molecules/FormField";
import Navbar from "@organisms/Navbar";
import MainLayout from "@templates/MainLayout";
import Home from "@pages/Home";
import { useWindowSize } from "@hooks/useWindowSize";
import { formatDate } from "@utils/formatters";
import type { Project } from "@types/common.types";

// ❌ INCORRECTO
import Button from "../../components/atoms/Button";
import FormField from "../molecules/FormField";
```

**Orden de imports (SIEMPRE en este orden):**

1. React y librerías externas
2. Componentes (atoms → molecules → organisms → templates → pages)
3. Hooks
4. Utils
5. Types
6. Estilos/Assets

### 3. Sintaxis TypeScript

**SIEMPRE:**

- Usar TypeScript strict mode
- Definir interfaces para props: `ComponentNameProps`
- Tipar todas las funciones, variables y parámetros
- Usar `React.FC<PropsType>` para componentes funcionales
- Preferir `interface` sobre `type` para objetos
- Usar `type` para unions, intersections y primitivas

**Ejemplo de componente correcto:**

```typescript
import React from "react";

interface ButtonProps {
  children: React.ReactNode;
  variant?: "primary" | "secondary" | "outline";
  size?: "sm" | "md" | "lg";
  onClick?: () => void;
  disabled?: boolean;
  className?: string;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = "primary",
  size = "md",
  onClick,
  disabled = false,
  className = "",
}) => {
  // lógica del componente

  return <button>{children}</button>;
};

export default Button;
```

### 4. Estilos

**SIEMPRE usar Tailwind CSS:**

- NO crear archivos CSS separados para componentes
- Usar clases de Tailwind directamente en JSX
- Para estilos dinámicos, usar template literals
- Mantener consistencia con las variables CSS en `src/style.css`

**PROHIBIDO:**

- Inline styles con el atributo `style`
- Importar archivos CSS en componentes (excepto main.tsx)
- Usar CSS modules

### 5. Formato de Código

**Prettier/Formatter configuración:**

- Comillas dobles `"` para strings (NO comillas simples)
- Punto y coma `;` al final de statements
- Trailing commas en objetos y arrays
- 2 espacios de indentación
- Max line length: 80-100 caracteres
- Arrow functions con paréntesis: `(x) => x`

---

## 🚫 PROHIBICIONES ESTRICTAS

### NUNCA hacer lo siguiente:

1. **Duplicar código:**

   - Si un código se repite 2+ veces, crear un componente/función reutilizable
   - Revisar si ya existe un componente similar antes de crear uno nuevo

2. **Modificar archivos de configuración sin razón:**
   - `tsconfig.json`
   - `vite.config.ts`
   - `package.json` (scripts)
3. **Crear componentes fuera de la estructura Atomic Design:**

   - TODO componente debe estar en atoms/molecules/organisms/templates/pages
   - NO crear carpeta "components/common" o similar

4. **Ignorar TypeScript errors:**

   - TODO código debe compilar sin errores de TypeScript
   - NO usar `@ts-ignore` o `any` sin justificación

5. **Desestructurar incorrectamente:**

   - Siempre desestructurar props en la firma de la función
   - NO acceder a props con `props.nombreProp` dentro del componente

6. **Crear archivos innecesarios:**
   - NO crear archivos de test (a menos que se solicite explícitamente)
   - NO crear archivos de configuración adicionales

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galleee2002) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
