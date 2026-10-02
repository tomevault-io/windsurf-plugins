---
trigger: always_on
description: - La navegación inferior vigente es `home`, `library`, `search`, `playlists` y `settings`.
---

# Polentita Music — reglas de contribución

## Contexto rápido para futuras sesiones

- La navegación inferior vigente es `home`, `library`, `search`, `playlists` y `settings`.
- `SearchScreen` muestra solo `Explore`; `LibraryScreen` concentra la búsqueda de canciones, álbumes y artistas.
- Las favoritas se sincronizan en la playlist reservada `Tus me gusta`; el acceso Favoritas de Inicio abre esa playlist.
- `Recientes` y `Más reproducidas` son colecciones dinámicas. Las playlists de usuario permiten reordenar canciones con pulsación larga y arrastre.
- Las descargas yt-dlp desde Buscar dejan el álbum vacío por defecto; no restablecer el valor `YouTube` automáticamente.
- `NetworkAccessPolicy` centraliza modo offline, conectividad y descargas solo con Wi‑Fi; no reemplazarlo por comprobaciones aisladas en Compose.
- `SquareArtworkProcessor` conserva la portada original y genera la representación cuadrada adaptativa usada por Compose y Media3.
- Playlists incluye la acción `Importar playlist`; el flujo funcional actual es la importación desde JSON, CSV o TXT estructurado.
- Para entregar: preservar datos del usuario, actualizar pruebas de lógica, compilar `app-debug.apk` e instalarlo con `adb install -r` si hay un dispositivo conectado.

### Control de versiones y punto de restauración

- El repositorio remoto privado del proyecto es `https://github.com/polen-tita/Polentita-Music.git`.
- La rama principal es `main` y contiene un punto de restauración funcional anterior al rediseño de interfaz.
- El commit base de restauración es `1b0101a` (`Snapshot funcional antes del rediseño de interfaz`).
- Antes de cambios visuales mayores, conservar este commit y crear nuevos commits descriptivos; no reescribir ni eliminar el historial publicado.
- El repositorio remoto no autoriza incluir credenciales: mantener fuera del control de versiones `local.properties`, tokens, claves API, logs del dispositivo, datos personales, cachés y artefactos generados no necesarios.
- Para recuperar el estado base, verificar primero los cambios locales y usar el commit `1b0101a` de forma explícita, preservando cualquier trabajo posterior del usuario.

## Alcance

- Aplicación Android local-first y offline-first escrita en Kotlin y Jetpack Compose.
- El paquete raíz es `com.polentita.music`.
- La biblioteca del usuario se accede exclusivamente mediante `content://` y Storage Access Framework.
- No solicitar `MANAGE_EXTERNAL_STORAGE`, no habilitar tráfico HTTP en claro y no incluir analytics.
- Nunca incorporar claves, tokens, canciones comerciales ni datos personales al repositorio.

## Arquitectura

- Un módulo `app` con MVVM, Repository, Room, Coroutines/Flow, Hilt y Media3.
- Mantener IO fuera del hilo principal.
- Las consultas observables de Room deben devolver `Flow`.
- Los ViewModels publican estados inmutables.
- Toda operación destructiva sobre archivos requiere una decisión explícita en UI.
- La reproducción vive en `MediaSessionService`, no en una Activity.

## Estado funcional y visual vigente

### Biblioteca, búsqueda y descargas

- `LibraryViewModel` y `SearchViewModel` persisten el criterio de orden y la dirección mediante `PreferencesStore`.
- `LibraryScreen` muestra una barra de búsqueda compacta en el encabezado, una fila secundaria con cantidad y orden entre el buscador y las pestañas, y las pestañas `Canciones`, `Álbumes` y `Artistas` reutilizan ese filtro.
- Al cambiar el orden en `LibraryScreen`, primero se captura el índice y desplazamiento visibles y luego `LazyListState`/`LazyGridState` los conserva mediante `requestScrollToItem`; no se debe seguir la posición de la canción activa ni usar `animateScrollToItem` para este cambio.
- Los filtros de búsqueda son independientes del orden y la consulta visible se actualiza inmediatamente; el flujo hacia Room/proveedores usa debounce.
- `SearchScreen` contiene únicamente `Explore`: conserva el buscador superior y muestra recomendaciones, referencias guardadas y resultados relacionados con la biblioteca; no reintroducir una pestaña o sección `Mi biblioteca`.
- Los selectores de artista y álbum de `DownloadsScreen` muestran sugerencias por prefijo, mantienen el teclado al escribir y limitan el menú a una altura desplazable.
- El flujo de descarga conserva la confirmación de metadatos, permite elegir artistas/álbumes existentes o escribir nuevos y no modifica Room, SAF, yt-dlp ni Media3 fuera de sus callbacks actuales.
- En el flujo de descarga yt-dlp iniciado desde `SearchScreen`, el álbum queda vacío por defecto aunque el proveedor devuelva `YouTube`/`YouTube Music`; la selección manual de un álbum existente o la escritura de uno nuevo sí se conserva. `YtDlpDownloadWorker` debe mantener `useMetadataAlbumWhenBlank = false`.
- `Explore` usa `AuthorizedMusicProvider`; cuando la consulta está vacía intenta relacionarse con la biblioteca local y respeta estado offline, configuración, licencia y permisos de descarga del proveedor.
- Los proveedores que implementan `PaginatedAuthorizedMusicProvider` deben devolver `RemoteSearchPage` con su `nextPageToken`; `SearchViewModel` conserva las páginas ya cargadas, las agrega con claves estables y expone `Cargar más resultados` sin repetir pistas.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polen-tita/Polentita-Music](https://github.com/polen-tita/Polentita-Music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
