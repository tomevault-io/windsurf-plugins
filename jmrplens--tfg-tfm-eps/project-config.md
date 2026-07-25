---
trigger: always_on
description: Plantilla LaTeX para TFG/TFM de la Escuela Politécnica Superior (EPS),
---

# Instrucciones para GitHub Copilot — Plantilla TFG/TFM EPS UA

Plantilla LaTeX para TFG/TFM de la Escuela Politécnica Superior (EPS),
Universidad de Alicante. Versión 2.2.1 (2026).

---

## Reglas críticas

1. **Motor:** SIEMPRE **LuaLaTeX**. Nunca sugieras `pdflatex` ni `xelatex`.
   LuaLaTeX maneja UTF-8 nativamente; no usar `\usepackage[utf8]{inputenc}`.
2. **Bibliografía:** **BibLaTeX + Biber**, estilo APA 7. Nunca BibTeX puro ni
   `\bibliographystyle{}`.
3. **Código fuente:** Paquete **minted 3.x** con `latexminted`. Usar siempre
   los entornos predefinidos en `sty/eps-codigo.sty`, nunca `verbatim` ni
   `lstlisting`.
4. **Tablas:** Siempre con `booktabs`. Nunca `\hline`.
5. **Portadas:** Generadas automáticamente con `\generarportada[ambas]`.
   Nunca crear portadas manualmente.
6. **Paquetes obsoletos prohibidos:** `utf8x`, `subfigure`, `subfig`, `t1enc`,
   `ae`, `times`, `mathptmx`.

---

## Estructura del proyecto

```text
main.tex              → Raíz. Solo estructura (inputs, usepackage, addbibresource).
configuracion.tex     → Datos del usuario (\EPSsetup{...}).
referencias.bib       → Bibliografía BibLaTeX.
cls/eps-tfg.cls       → Clase principal (no modificar).
cls/eps-metadata.tex  → Metadatos PDF/UA-2 (solo cambiar lang= al cambiar idioma).
sty/eps-codigo.sty    → Entornos de código (minted).
sty/eps-componentes.sty → Cargador modular de componentes.
sty/componentes/      → Módulos: software, telecom, arquitectura, quimica, etc.
contenido/capitulos/  → Un .tex por capítulo.
contenido/anexos/     → Anexos y acrónimos.
contenido/frontmatter/→ Resumen, agradecimientos.
```

---

## Configuración del documento

```latex
% En configuracion.tex
\EPSsetup{
  titulo      = {Título del trabajo},
  subtitulo   = {Subtítulo opcional},
  autor       = {Nombre Apellido1 Apellido2},
  genero      = m,              % m, f, n (neutro → "Autoría")
  email       = nombre@alu.ua.es,
  tutor       = {Dr. Nombre Apellido},
  tutor-genero = m,
  tutor-departamento = {Departamento de ...},
  titulacion  = informatica,    % ver lista completa abajo
  idioma      = espanol,        % espanol, valenciano, ingles
  fecha       = {Junio 2026},
  optimizar-tikz = true,
}
```

### Titulaciones disponibles

**Grados (TFG):** `arquitectura`, `arquitectura-tecnica`, `civil`,
`informatica`, `multimedia`, `quimica`, `robotica`, `teleco`

**Másteres (TFM):** `master-agua`, `master-caminos`, `master-ciberseguridad`,
`master-edificacion`, `master-geologica`, `master-informatica`,
`master-materiales`, `master-moviles`, `master-prevencion`, `master-quimica`,
`master-robotica`, `master-teleco`, `master-web`

---

## Regla de idioma

Al cambiar `idioma` en `configuracion.tex`, actualizar también `lang=` en
`cls/eps-metadata.tex`:

| `idioma` | `lang=` |
| --- | --- |
| `espanol` | `es-ES` |
| `valenciano` | `ca-ES` |
| `ingles` | `en-GB` |

---

## Componentes especializados

Activar en `main.tex`:

```latex
\usepackage[software]{eps-componentes}       % Informática, Multimedia, Robótica
\usepackage[telecom]{eps-componentes}        % Telecomunicaciones
\usepackage[arquitectura]{eps-componentes}   % Arquitectura, Civil
\usepackage[quimica]{eps-componentes}        % Química
\usepackage[geologia]{eps-componentes}       % Geología
\usepackage[prevencion]{eps-componentes}     % Prevención de Riesgos
\usepackage[all]{eps-componentes}            % Todos
```

### Cajas de aviso (siempre disponibles)

```latex
\begin{infobox}{Título}    Texto informativo.    \end{infobox}
\begin{warningbox}{Título} Texto de advertencia. \end{warningbox}
\begin{dangerbox}{Título}  Texto de peligro.     \end{dangerbox}
\begin{successbox}{Título} Operación correcta.   \end{successbox}
\begin{tipbox}{Título}     Consejo útil.         \end{tipbox}
\begin{notebox}{Título}    Nota adicional.       \end{notebox}
```

### Contenedores de contenido (siempre disponibles)

```latex
\begin{definitionbox}{Definición: término}
  Descripción del término.
\end{definitionbox}

\begin{examplebox}{Ejemplo práctico}
  Caso de uso ilustrativo.
\end{examplebox}
```

### Módulo `[software]`

```latex
% Terminal / consola
\begin{terminal}[title={bash}]
$ npm install && npm start
\end{terminal}

% Endpoint REST
\begin{apiendpoint}{POST}{/api/v1/login}{Autenticación de usuario}
  Body: { "email": "...", "password": "..." }
\end{apiendpoint}

% Árbol de directorios
\begin{dirtreebox}
  proyecto/
  ├── src/
  │   ├── main.py
  │   └── utils.py
  └── tests/
\end{dirtreebox}
```

### Módulo `[telecom]`

```latex
% Trama de protocolo
\begin{protocolframe}
  % Campos de la trama
\end{protocolframe}
```

---

## Entornos de código

Usar siempre los entornos de `sty/eps-codigo.sty`:

```latex
% Python
\begin{pythoncode}[title={clasificador.py}]
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier(n_estimators=100)
clf.fit(X_train, y_train)
\end{pythoncode}

% JavaScript
\begin{jscode}[title={index.js}]
const app = require('./app');
app.listen(process.env.PORT || 3000);
\end{jscode}

% C++
\begin{cppcode}
#include <vector>
std::vector<int> v = {1, 2, 3};
\end{cppcode}

% Java
\begin{javacode}
public class Main {
    public static void main(String[] args) {}
}
\end{javacode}

% SQL
\begin{sqlcode}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmrplens/TFG-TFM_EPS](https://github.com/jmrplens/TFG-TFM_EPS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
