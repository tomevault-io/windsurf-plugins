---
trigger: always_on
description: El branding completo está en `assets/branding.png`. Úsalo como referencia visual
---

# UNGUZY — E-commerce de Skincare

## Identidad de marca

El branding completo está en `assets/branding.png`. Úsalo como referencia visual
permanente antes de tomar cualquier decisión de diseño.

Marca: **UNGUZY** — skincare natural con estética orgánica, cálida y editorial.
Tagline: *"Balance Natural Para Tu Piel"*

---

## Estilo visual de la página

La página debe sentirse como una publicación editorial de lujo accesible —
no una tienda genérica de skincare. Las referencias visuales son una revista
de moda slow, un mercado orgánico de autor y una botica artesanal.

Características que deben percibirse en cada sección:

- **Calidez material**: fondos en beige cálido, sombras suaves, sin blancos fríos
- **Respiración**: mucho espacio en blanco, layouts que no saturan
- **Contraste editorial**: títulos grandes en Playfair Display contra cuerpos
  pequeños en Neue Haas, creando jerarquía sin necesidad de color
- **Naturaleza contenida**: el ícono floral lineal del logo puede usarse como
  elemento decorativo recurrente y sutil entre secciones
- **Fotografía integrada**: las imágenes de productos y modelos deben tratarse
  con consistencia tonal — cálidas, con luz natural, nunca con fondos de
  estudio fríos si se pueden evitar
- **Movimiento discreto**: las animaciones deben ser lentas y orgánicas,
  nunca abruptas — entradas con fade + ligero desplazamiento vertical,
  hover states suaves en productos

---

## Paleta de colores

Define estas variables CSS globales y úsalas de forma consistente en todo
el proyecto. No uses colores fuera de esta paleta.

```css
:root {
  --color-raiz:   #5B2A18; /* marrón oscuro — fondos hero, textos principales */
  --color-terra:  #A24B2A; /* terracota — acentos, CTAs, hover states */
  --color-papel:  #E7DED1; /* beige claro — fondos secundarios, cards */
  --color-ambar:  #9B7A45; /* dorado apagado — bordes, detalles, íconos */
  --color-blanco: #FDFAF6; /* blanco cálido — fondo principal de página */
}
```

---

## Tipografía

- **Fuente de marca** (display principal): archivo `.otf` ubicado en
  `public/fonts/` — declárala con `@font-face` en `globals.css` y úsala
  exclusivamente para el logotipo, hero headlines y nombre de la marca
- **Playfair Display** — títulos de sección, nombres de productos, headings
- **Neue Haas Grotesk** (fallback: `DM Sans`) — cuerpo, navegación,
  etiquetas, precios, descripciones

```css
/* En app/globals.css */
@font-face {
  font-family: 'UnguzyFont';
  src: url('/fonts/[nombre-exacto-del-archivo].otf') format('opentype');
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

:root {
  --font-brand:   'UnguzyFont', serif;
  --font-display: 'Playfair Display', serif;
  --font-body:    'Neue Haas Grotesk', 'DM Sans', sans-serif;
}
```

> Nota: si el rendimiento de carga es prioritario, convierte el `.otf` a
> `.woff2` con Transfonter (transfonter.org) y actualiza el `src`.

---

## Stack técnico

- **Framework**: Next.js (App Router) + React
- **Estilos**: CSS puro con variables CSS — sin Tailwind, sin CSS-in-JS
- **Imágenes**: `next/image` con optimización activada
- **Assets**: todas las imágenes en `assets/`

---

## Assets disponibles

```
assets/
  branding.png                  → referencia de identidad de marca (no usar en UI)
  productos.png                 → foto grupal de línea completa — usar como hero
  hero_image.png                → imagen alternativa para hero
  logo.png                      → logotipo oficial de la marca

  /* Productos — 6 en total */
  jabon_facial.png              → Jabón Facial de Carbón Activado, Aloe Vera y Ácido Salicílico
  hidratante.png                → imagen principal: Crema Hidratante
  crema_hidratante.png          → imagen secundaria: Crema Hidratante (carrusel)
  serum_colageno.png            → Sérum de Colágeno y Ácido Hialurónico
  serum_vitaminac.png           → Sérum de Vitamina C, Extracto de Arroz, Elastina y Colágeno
  tonico_manzanilla.png         → Tónico de Manzanilla y Ácido Hialurónico
  tonico_hamamelis.png          → Tónico de Hamamelis y Ácido Salicílico

  /* Modelos */
  modelo1.PNG                   → foto de modelo — usar en hero o sección Nosotros
  modelo2.PNG                   → foto de modelo — usar en hero o sección Nosotros
```

**Uso del logo**: usa `logo.png` en el header/navbar y footer.
No uses el texto "UNGUZY" en tipografía genérica donde debería ir el logo.

**Carrusel de detalle**: solo la Crema Hidratante tiene dos imágenes (`hidratante.PNG`
como principal, `Crema hidratante.PNG` como secundaria). Los demás productos muestran
imagen estática — no renderices carrusel si hay una sola imagen.

**Uso de fotos de modelos**: úsalas en el hero, en la sección "Nosotros" y
como contexto visual en páginas de producto. Prioriza imágenes que muestren
el producto en uso sobre las que solo muestren la persona.

**Edición de imágenes**: si alguna imagen de producto o modelo no es
consistente en tono o recorte con el resto, ajústala (crop, temperatura
de color, contraste) para mantener coherencia visual en todo el catálogo.

---

## Estructura de rutas

```
/               → Home: hero + productos destacados + fragmento de "Nosotros"
/productos      → Catálogo completo (6 productos)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdrianaMaguea/unguzy](https://github.com/AdrianaMaguea/unguzy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
