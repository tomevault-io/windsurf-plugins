---
trigger: always_on
description: Este archivo es la referencia principal para que cualquier sesion de Claude entienda el
---

# AGENTS.md — Guia operativa para Claude

Este archivo es la referencia principal para que cualquier sesion de Claude entienda el
estado actual del proyecto, las convenciones vigentes y como ayudar correctamente.
**Leer completo antes de proponer cambios.**

---

## 1. Que es este proyecto

Sistema de captura de aceleracion en tiempo real con sensores ADXL335 conectados a una
ESP32 via ESP-NOW inalambrico, con un receptor USB que retransmite por serial.
La aplicacion de escritorio (GUI Python/Tkinter) recibe el stream serial, hace un
precheck de integridad dual, captura datos y genera un paquete de archivos estructurado.

**Stack vigente:**
- Firmware ESP32: C++ / PlatformIO (`firmware/single_node_calibration/`)
- GUI de captura live: Python 3.11 / Tkinter (`gui/`)
- Distribucion: ejecutable standalone Windows (`dist/ADXL335_Captura.exe`)
- Analisis historico: MATLAB (en proceso de migracion, ya no es ruta operativa live)

---

## 2. Estado operativo de sensores

| Sensor   | Estado       | Rol                                          |
|----------|--------------|----------------------------------------------|
| sensor_B | **Primario** | Fuente operativa principal. sensor_id = 1.   |
| sensor_A | Backup       | Fallback si sensor_B falla. sensor_id = 2.   |
| sensor_C | Referencia   | Historico/provisional. No usar en produccion.|
| sensor_D | Descartado   | No usar.                                     |

---

## 3. Estructura de archivos del repositorio

```
Repo/
├── gui/
│   ├── adxl_live_gui.py        # Punto de entrada de la aplicacion (GUI Tkinter)
│   └── adxl_live_core.py       # Logica de captura, serial, procesamiento y guardado
│
├── firmware/
│   ├── single_node_calibration/    # Firmware operativo actual (PlatformIO)
│   └── dual_node_espnow/           # Arquitectura futura ESP-NOW (Fase 14B/14C)
│
├── data/
│   ├── raw/sensor_B_live/          # CSVs crudos de sesiones live (NO modificar)
│   └── processed/                  # CSVs procesados + JSONs de sesion
│
├── reports/
│   ├── analysis_outputs/           # _summary.txt y _precheck.txt por sesion
│   └── change_log.md               # Registro tecnico de todos los cambios
│
├── live_session_hub/
│   ├── sensorB_live_gui_session.ps1    # Launcher oficial de la GUI
│   └── sensorB_live_prompt_session.m  # Launcher MATLAB (legado, no usar)
│
├── scripts/
│   ├── install_gui_requirements.ps1   # Instala pyserial en el venv
│   ├── run_sensorB_live_session.ps1   # Lanza GUI con parametros
│   └── run_sensorB_operational_capture.ps1
│
├── config/
│   └── adxl335_module_template.json   # Plantilla de configuracion de modulo
│
├── matlab/                            # Analisis historico (referencia, no operativo)
├── docs/                              # Guia maestra LaTeX + PDF
├── hardware/                          # Pinout y registro de sensores
├── handoff/                           # Exports de handoff por rol (Fase 14C.1)
│
├── adxl_captura.spec          # Spec de PyInstaller para compilar el .exe
├── build_exe.ps1              # Script: compila .exe y genera ZIP de distribucion
├── build_manual_pdf.py        # Script: genera ADXL335_Captura_Manual.pdf
├── ADXL335_Captura_Manual.pdf # Manual de usuario (generado por build_manual_pdf.py)
├── requirements-gui.txt       # Dependencias Python de la GUI (solo pyserial==3.5)
├── AGENTS.md                  # Este archivo
└── README.md                  # Documentacion principal del proyecto
```

Carpetas generadas automaticamente (no commitear):
```
.venv/          # Entorno virtual Python local
dist/           # Ejecutable compilado y ZIP de distribucion
build_work/     # Artefactos intermedios de PyInstaller
```

---

## 4. La aplicacion GUI — como funciona

### Punto de entrada
```
gui/adxl_live_gui.py
```

### Dependencias
- `pyserial==3.5` (unica dependencia externa; ver `requirements-gui.txt`)
- Python stdlib: `tkinter`, `threading`, `csv`, `json`, `pathlib`, etc.

### Flujo de una sesion
1. **Autodeteccion de puerto**: sondea puertos COM buscando el stream del firmware.
2. **Precheck dual** (~10 s): verifica integridad de ambos sensores antes de capturar.
3. **Captura principal** (10-90 s): graba datos, actualiza graficas en tiempo real.
4. **Guardado**: genera los 5 archivos de salida.

### Deteccion del directorio de datos (IMPORTANTE para el exe)
```python
# gui/adxl_live_gui.py — funcion _resolve_repo_root()
if getattr(sys, "frozen", False):
    # Ejecutando como .exe compilado con PyInstaller
    return Path(sys.executable).resolve().parent
else:
    # Ejecutando en desarrollo desde el repo
    return Path(__file__).resolve().parents[1]
```
Esto garantiza que los datos se guarden junto al .exe en distribucion y junto a la
raiz del repo en desarrollo. No cambiar este patron.

---

## 5. Archivos de salida — formato y campos

Cada sesion exitosa genera estos archivos con nombre comun:
```
{prefijo}_{sesion}_{AAAAMMDD}_{HHMMSS}_{tipo}.{ext}
```

### _raw.csv
Lectura directa del firmware. Campos:
```
sensor_id, seq, t_us, wall_s, raw_x, raw_y, raw_z, mv_x, mv_y, mv_z
```
- `sensor_id`: 1=sensor_B (principal), 2=sensor_A (secundario)
- `seq`: numero de secuencia del firmware (saltos = paquetes perdidos)
- `t_us`: microsegundos del reloj interno de la ESP32

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hujjika1099-glitch/Repo](https://github.com/hujjika1099-glitch/Repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
