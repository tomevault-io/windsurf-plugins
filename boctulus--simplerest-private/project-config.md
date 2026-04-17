---
trigger: always_on
description: - **Bootstrap 5** (generalmente utilizado)
---

# Proyecto sobre framework SimpleRest

## Stack Tecnológico

### Backend
- PHP

### Frontend
- **Bootstrap 5** (generalmente utilizado)
- **jQuery** (generalmente utilizado)

### Bases de Datos
- **MySQL** (soporta otras) 

### Testing & Automation
- **PHPUnit** (unit testing)
- **Playwright 1.54+** (disponible desde NodeJS)
- **Puppeteer** (alternativa para Chrome)
- **Selenium** (disponible desde Python)

--

## Antes de Empezar

### ⚠️ Paso Obligatorio
**SIEMPRE** verifica si existe la carpeta `docs/` antes de iniciar cualquier tarea:

```bash
ls docs/
# o
dir docs
```

Esta carpeta contiene o podria contentener documentación crítica del proyecto.

Ademas leer el archivo `README.md` a nivel de root del proyecto. 

Debes leer la documentacion pertinente interna al module / package / component que vayas a utilizar que generalmente son archivos .md internos.


### Archivos Clave en `docs/`

| Archivo | Descripción 
|---------|-------------
| `login-credentials.md` | **Usuarios y passwords** para testing con diferentes roles  (vacio de momento)
| `docs/issues/` 		 | Problemas comunes y soluciones (puede requerir actualización) 

---

## Sistema de Comandos CLI

Crear nuevos comandos personalizados para realizar tareas de caracter repetitivo y de caracter general.

Ver `docs/CommandLine.md`

**💡 Tip:** Siempre revisa los comandos disponibles antes de empezar una tarea, pueden ahorrarte mucho trabajo.

---

## Testing y Debugging

### Scripting de automatizacion de tareas

Scripts creados para mover archivos, renombrar carpetas, modificar archivos, etc deben alojarse en `scripts/` y si son de caracter transitorio dentro de `scripts/tmp/`

Los archivos .js dentro de `scripts/tmp/` deben borrarse al terminar las tareas y contar esta con validacion del usuario.

Estos scripts no son para debugging ni realizan Web Scraping con la ayuda de herramientas como Playwright que utilicen web drivers.

Ningun archivo temporal a ser generado (Ej: `debug_*.png`, `debug_*.txt`, `debug_*.log`, etc) debe hacerse en el root del proyecto.


### Donde alojar los scripts

Los scripts para debugging y testing deben ir segun corresponda en `tests/` o en `tests/unit-tests/` segun sean tests normales o del tipo prueba unitaria.

Los scripts que hacen uso de Web Drivers como Playwright deben ir dentro de `web-automation/`

Debe mantenerse organizacion y bajo ninguna circunstancia tener archivos del tipo `test_*.j`s o `test-*.js` en el root del proyecto.

Al iniciar revisa si encuentras archivos del tipo `test_*.js` o `test-*.js` en el root del proyecto y si asi muevelos a donde corresponda antes de proseguir.

Si el script consideras ya no tiene utilidad lo puedes borrar sin problemas.

Si necesitas generar un reportes sobre "pruebas unitarias" crea los reportes en `reports/unit-tests/` pero nunca en el root del proyecto.


### Testing E2E y Debugging de UI

Las credenciales para diferentes roles se deberian ubicar en:
```
docs/login-credentials.md
```

Nota:

Ningun archivo temporal a ser generado incluyendo screenshots debe hacerse en el root del proyecto. Coloca las imagenes dentro de `automation/screenshoots/` o en carpetas dentro de `automation/screenshoots`

Si necesitas generar un reporte con Web Scraping crea los reportes en `reports/automation/` pero nunca en el root del proyecto.

---

## Workflow de Trabajo

### 1. Investigación Inicial
```bash
# Listar documentación disponible
ls docs/

# Revisar comandos CLI disponibles
php com help
```

### 2. Desarrollo

- Enfocarse en una arquitectura modular.
- Consulta la documentación específica del módulo / componente / paquete dentro del modulo / componente / paquete suele haber un archivo .md y tambien verifica dentro de `docs\`
- Antes de emprender el desarrollo de una libreria compleja que por su caracter general podria existir ya desarrollada (ya sea como libreria, pacakge, etc) debes hacer una busqueda para asegurarte no exista ya una solucion "open source" que se pueda utilizar. En caso de existir pero tener una API distinta a la solicitada evaluar construir un adapter o wrapper sobre la misma. 

#### Routing

Es recomendable busques la documentacion de "routing" (sera algun archivo con extension .md) del framework.

No es necesario que ocupes el routes.php principal para el caso de packages/ Cada package puede tener su propio routes.php si hay un ServiceProvider. Revisa la documentacion y otros packages del framework. Podrias estar duplicando codigo de rutas.

Si las rutas no funcionan en un package es posible el package no este correctamente registrado. Importante ejecutar `composer dumpautoload` luego de cualquier registro / actualizacion en el "composer.json" para regenerar el autoload.


#### Consumo de API

Cuando debas consumir APIs utiliza la clase ApiClient en vez usar funciones CURL directamente.

Documentacion:
```
docs/ApiClient.md
```


## Prácticas Obligatorias

-   Tener en cuenta los patrones existentes y seguirlos con la salvedad de que si fuera necesario cambiar o implementar uno nuevo debe exponerse el problema, justificar el nuevo patron y consultar antes de implementar.  
-   Cambios incrementales. Analizar si fuera necesario un refactoring importante o re-escribir una solucion / libreria / vista de forma significativa, justificar y consultar antes de implementar.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boctulus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
