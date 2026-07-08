---
trigger: always_on
description: GesBike es una aplicación web para la gestión integral de mantenimiento de bicicletas. Permite gestionar vehículos (bicicletas), mantenimientos, recambios, compras, rutas, grupos y operaciones de mantenimiento. El sistema está diseñado para que usuarios particulares o pequeños talleres puedan llevar un control exhaustivo de sus bicicletas y sus mantenimientos.
---

# GesBike - Sistema de Gestión de Mantenimiento de Bicicletas

## 1. Descripción General

GesBike es una aplicación web para la gestión integral de mantenimiento de bicicletas. Permite gestionar vehículos (bicicletas), mantenimientos, recambios, compras, rutas, grupos y operaciones de mantenimiento. El sistema está diseñado para que usuarios particulares o pequeños talleres puedan llevar un control exhaustivo de sus bicicletas y sus mantenimientos.

## 2. Arquitectura

El proyecto sigue el patrón **MVC (Model-View-Controller)** con una capa adicional de **Repositories** para el acceso a datos. También cuenta con una capa de **API** para comunicación asíncrona y una capa de **Servicios JavaScript** para el frontend.

### Capas de la Arquitectura

```
┌─────────────────────────────────────────────┐
│         Views (Vistas PHP + HTML)           │
│     Listado, formulario, dashboard           │
├─────────────────────────────────────────────┤
│        Services (JS Frontend)               │
│   Axios calls a API, render dinámico        │
├─────────────────────────────────────────────┤
│              API (Endpoints)                │
│    api/{recurso}/{recurso}.php?{action}     │
├─────────────────────────────────────────────┤
│            Controllers (PHP)                │
│    Lógica de control, validación, flujo     │
├─────────────────────────────────────────────┤
│             Models (PHP)                    │
│    Wrappers que llaman a Repositories       │
├─────────────────────────────────────────────┤
│           Repositories (PHP)                │
│    Queries SQL, acceso a datos (PDO)        │
├─────────────────────────────────────────────┤
│        DatabaseConnection (SQLite PDO)      │
└─────────────────────────────────────────────┘
```

### Flujo de una operación típica (ej: crear vehículo)

```
Usuario → View (form.php) → JS (vehiculo.js: guardarVehiculo())
  → POST a API (api/vehiculos/vehiculo.php?nuevoVehiculo)
    → Controller (controllers/vehiculo.php: handle_nuevo_vehiculo())
      → Model (models/vehiculo.php: nuevoVehiculo())
        → Repository (repositories/vehiculo.php: nuevo_vehiculo_repo())
          → SQL INSERT → SQLite DB
```

## 3. Estructura de Directorios

```
gesBike/
├── api/                          # Endpoints API REST
│   ├── compras/compra.php
│   ├── grupos/grupo.php
│   ├── helpers/helper.php        # Endpoints genéricos (uploadFile, getGrupos, getKilometros...)
│   ├── login/login.php
│   ├── log/log.php
│   ├── mantenimientos/mantenimiento.php
│   ├── recambios/recambio.php
│   ├── rutas/ruta.php
│   ├── stocks/stock.php
│   └── vehiculos/vehiculo.php
├── assets/                       # Recursos estáticos
│   ├── css/
│   │   ├── bootstrap/
│   │   ├── compras/
│   │   ├── detalles/
│   │   ├── grupos/
│   │   ├── login/
│   │   ├── main/
│   │   ├── mantenimientos/
│   │   ├── recambios/
│   │   ├── rutas/
│   │   ├── stocks/
│   │   ├── vehiculos/
│   │   ├── theme.css             # 70+ variables CSS para modo claro/oscuro
│   │   └── style.css             # Estilos globales compartidos
│   ├── js/
│   │   ├── axios/
│   │   └── bootstrap/
│   └── images/
│       ├── icons/
│       │   ├── Grupos/           # 22 iconos predefinidos para grupos
│       │   └── Localizaciones/   # Iconos de localizaciones (delante, detras, etc.)
│       ├── Recambios/            # Imágenes subidas de recambios (UUID)
│       └── Vehiculos/            # Imágenes subidas de bicicletas (UUID)
├── attachments/                  # Archivos adjuntos a mantenimientos
├── controllers/                  # Controladores PHP
│   ├── attach.php                # Subida de archivos e imágenes + compressImage()
│   ├── compra.php
│   ├── grupo.php                 # CRUD completo (getList, getById, nuevo, editar, eliminar)
│   ├── helper.php
│   ├── login.php                 # Autenticación y persistencia de tema
│   ├── log.php
│   ├── mantenimiento.php         # CRUD + getKmsByGrupo + getHistorico
│   ├── recambio.php
│   ├── ruta.php
│   ├── selector.php
│   ├── stock.php
│   ├── translate.php
│   └── vehiculo.php              # CRUD + subida imagen
├── database/
│   ├── app.db                    # Archivo SQLite principal
│   ├── gesbike.db
│   ├── DatabaseConnection.php    # Conexión PDO
│   └── backups/                  # Copias de seguridad
├── helpers/
│   ├── backup.php
│   ├── config.php                # Carga de variables de entorno (.env)
│   └── helper.php                # Funciones auxiliares PHP (random_file_enumerator, new_guui_generator, etc.)
├── jobs/
│   └── cron_email.php            # Tareas programadas (SMTP Gmail)
├── models/                       # Modelos PHP (wrappers)
│   ├── attach.php
│   ├── compra.php
│   ├── grupo.php
│   ├── helper.php
│   ├── login.php
│   ├── log.php
│   ├── mantenimiento.php
│   ├── recambio.php
│   ├── ruta.php
│   ├── selector.php
│   ├── stock.php
│   ├── translate.php
│   └── vehiculo.php

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calcena/gesbike](https://github.com/calcena/gesbike) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
