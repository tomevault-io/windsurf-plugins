---
trigger: always_on
description: > Copia este prompt completo y pégalo en el agente que vaya a construir el proyecto (Claude Code, Cursor, etc.).
---

# SÚPER PROMPT — "Bissap Maison": web para vender productos naturales de hibisco (Reims / Châlons)

> Copia este prompt completo y pégalo en el agente que vaya a construir el proyecto (Claude Code, Cursor, etc.).
> Está escrito para ejecutarse en fases y es auto-contenido. Objetivo: **algo simple pero profesional**, listo para vender por WhatsApp.

---

## 0. Contexto del negocio

- Producto artesanal y **fait maison** a base de **hibisco** (*Hibiscus sabdariffa*, la flor con la que se hace el **bissap**).
- Venta **local**: zona de **Reims** y **Châlons-en-Champagne** (Marne, Francia).
- Canal de pedidos: **WhatsApp** (sin carrito, sin pasarela de pago en la v1).
- Mercado principal: **francófono** → la web se redacta en **francés**. (Opcional futuro: versión ES/EN.)
- Tono: natural, cercano, honesto, hecho a mano. Nada de promesas médicas.

---

## 1. ROL

Actúas como un **equipo combinado**:

1. **Diseñador/desarrollador web senior** especializado en *landing pages* de producto artesanal: rápidas, limpias, mobile-first, alta conversión.
2. **Redactor de marca y SEO local** con experiencia en pequeño comercio / alimentación artesanal en Francia, que conoce las reglas de **alegaciones de salud (Reglamento UE 1924/2006)** y redacta copy evocador sin infringirlas.

Trabajas de forma modular, documentas cada decisión y **no inventas propiedades del producto**: todo beneficio se expresa en lenguaje sensorial/tradicional, no como afirmación de salud verificable.

---

## 2. OBJETIVO

Construir **"Bissap Maison"** (nombre provisional, confirmar con el cliente): una web de **una sola página** (one-page, con secciones ancladas) que:

- Presenta la **gama de productos de hibisco** y sus **sabores**.
- Explica que el producto es **versátil**: se puede transformar de líquido a otros formatos.
- Recoge los **pedidos por WhatsApp** con un botón que abre una conversación con mensaje pre-rellenado.
- Muestra la **zona de reparto / punto de recogida** con **Google Maps** integrado (Reims y Châlons).
- Incluye la información **legal mínima** (mentions légales, RGPD, alérgenos, conservación).
- Carga rápido, se ve perfecta en móvil y transmite confianza.

---

## 3. GAMA DE PRODUCTO (confirmar precios y formatos con el cliente)

**Base:** infusión concentrada de flor de hibisco (bissap), elaborada a mano.

**Sabores / variantes:**

| Sabor | FR | Nota de cata sugerida |
|---|---|---|
| Menta | Menthe | fresco, herbáceo, final limpio |
| Jengibre | Gingembre | picante suave, cálido, tónico |
| Vainilla | Vanille | dulce, redondo, envolvente |
| Natural | Nature | ácido, floral, afrutado (base sin aromatizar) |

**Formatos / usos del mismo producto (versatilidad):**

- **Jus / boisson** — listo para beber (diluir al gusto).
- **Sirop / concentré** — para diluir en agua fría o caliente.
- **Infusion chaude** — servir caliente como tisana.
- **Confiture de bissap** — versión mermelada para tostadas, quesos, repostería.
- **En pâtisserie** — sirope o confitura para bizcochos, glaseados, postres ("de líquido a pastel").

> El texto debe dejar claro que **es un solo producto natural** que el cliente puede usar de varias formas, no cinco productos distintos (salvo la confitura, que se vende como tal).

---

## 4. PROPUESTA DE VALOR Y BENEFICIOS — REGLAS DE REDACCIÓN

El cliente quiere comunicar ideas como: *sentirse bien, boost de energía, concentración, fuerza, momento zen*.

**Cómo se redacta (permitido, lenguaje sensorial / de experiencia):**

- « Un moment pour souffler. » / « La pause hibiscus. »
- « Une boisson vive et réconfortante, à partager. »
- « Le rituel bissap : on se pose, on savoure. »
- « Fait main, à Reims, avec des fleurs d'hibiscus et rien d'inutile. »
- « Sans colorant, sans arôme artificiel, sans conservateur ajouté. » *(solo si es cierto — confirmar con el cliente)*

**Cómo NO se redacta (alegación de salud no autorizada — prohibido):**

- ❌ "aumenta la concentración", "da energía", "refuerza el sistema inmunitario", "reduce la tensión", "quema grasa", "desintoxica", "cura / previene …".
- ❌ Cualquier referencia a enfermedades o a efectos fisiológicos concretos.

**Si el cliente insiste en el ángulo "bienestar":** usar la fórmula de **uso tradicional** con matiz claro, p. ej.:
« En Afrique de l'Ouest, le bissap est la boisson conviviale par excellence, servie lors des fêtes et des retrouvailles. » — describe cultura, no promete efectos.

Añadir siempre, discreto, un aviso: « Produit alimentaire artisanal. Ne se substitue pas à une alimentation variée et équilibrée. »

---

## 5. STACK TÉCNICO (simple y profesional)

- **Astro** (recomendado) o **HTML + Tailwind** plano si se quiere aún más simple. Sin framework pesado, sin base de datos.
- **Tailwind CSS** para estilos.
- **Sin backend en la v1**: los pedidos van por WhatsApp (`https://wa.me/…` con `?text=` pre-rellenado).
- **Imágenes**: `<picture>` / formato WebP, `loading="lazy"`, `alt` descriptivo en francés. Comprimir a < 200 KB cada una.
- **Fuentes**: 1 tipografía con carácter para títulos + 1 sans legible para texto (self-hosted o `font-display: swap`).
- **Despliegue**: **Netlify** o **Vercel** (deploy desde Git, HTTPS y dominio gratis para empezar).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergiopo1369-spec/oussap-maison](https://github.com/sergiopo1369-spec/oussap-maison) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
