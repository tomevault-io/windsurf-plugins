---
trigger: always_on
description: Este proyecto tiene como objetivo analizar los datos emitidos por el termostato **Carrier CRC2-NTC** (un modelo obsoleto) mediante comunicación RS485, con el fin de entender su funcionamiento interno y eventualmente reemplazarlo con un dispositivo compatible que permita programación avanzada, acceso remoto y una interfaz moderna.
---

# TermostatoAC-Carrier - Guía para Agentes de Código

## Visión General del Proyecto

Este proyecto tiene como objetivo analizar los datos emitidos por el termostato **Carrier CRC2-NTC** (un modelo obsoleto) mediante comunicación RS485, con el fin de entender su funcionamiento interno y eventualmente reemplazarlo con un dispositivo compatible que permita programación avanzada, acceso remoto y una interfaz moderna.

El proyecto combina:
- **Comunicación Serial**: Lectura y escritura de datos del termostato vía USB-RS485
- **Machine Learning**: Entrenamiento de modelos neuronales para predecir el estado del termostato
- **Análisis de Protocolos**: Ingeniería inversa del protocolo CCN (Carrier Comfort Network)

---

## Stack Tecnológico

- **Lenguaje**: Python 3
- **Librerías Principales** (ver `requirements.txt`):
  - `serial` (pyserial) - Comunicación con el puerto serie
  - `tensorflow` / `keras` - Entrenamiento de modelos de machine learning
  - `numpy` - Procesamiento numérico
  - `matplotlib` - Visualización de gráficos
  - `optparse` - Interfaz de línea de comandos

---

## Estructura del Proyecto

```
TermostatoAC-Carrier/
├── curryrtool.py              # Entry point principal de la aplicación (Python)
├── curryrtool                 # Wrapper bash para ejecutar curryrtool.py
├── curryrtoolslib/            # Paquete Python con la lógica completa
│   ├── interfaces/            # Interfaces de usuario (CLI)
│   │   ├── common_interface.py      # Clase base para interfaces
│   │   ├── devices_interface.py     # Comunicación serial (read/write/repeat)
│   │   └── trainning_interface.py   # Entrenamiento y predicción ML
│   ├── utils/                 # Utilidades
│   │   ├── ia.py              # Funciones TensorFlow/Keras
│   │   ├── trama.py           # Gestión de tramas (85/88/91/94 bytes)
│   │   ├── graphics.py        # Visualización de datos
│   │   └── config.py          # Gestión de configuración
│   ├── loader/                # Bootstrap y entry points
│   │   ├── main.py            # Inicialización de la aplicación
│   │   └── entry_points.py    # Router de comandos
│   └── __init__.py            # Configuración global
├── dumps/                     # Datos capturados del termostato (no en git)
│   ├── *.bin                  # Tramas en formato binario
│   ├── *.info                 # Metadatos JSON de las tramas
│   └── *.json                 # Listas de capturas
├── models/                    # Modelos entrenados (Keras .keras) (no en git)
├── images/                    # Documentación visual (fotos, diagramas)
├── docs/                      # Documentación adicional de análisis
├── CH341SER/                  # Drivers para el adaptador USB-RS485 (no en git)
├── requirements.txt           # Dependencias del proyecto
├── README.md                  # Documentación en inglés
├── README_es.md               # Documentación en español (principal)
├── CCN_PROTOCOL_ANALYSIS.md   # Análisis técnico del protocolo CCN
├── CHECKLIST.md               # Guía de ingeniería inversa
└── .gitignore                 # Excluye: *.zip, CH341SER, dumps, models, __pycache__
```

---

## Instalación y Configuración

### Instalación de Dependencias

```bash
pip install -r requirements.txt
```

Contenido actual de `requirements.txt`:
- `serial` (pyserial)
- `tensorflow`
- `types-tensorflow`

### Permisos de Dispositivo (Linux)

Para acceder al puerto USB sin sudo:
```bash
sudo usermod -a -G dialout $USER
# Requiere cerrar sesión y volver a iniciar
```

### Configuración

El proyecto utiliza un archivo de configuración en:
```
~/.config/curryrtools/config.ini
```

Gestión vía `curryrtoolslib/utils/config.py` (clase `ConfigurationManager`).

Valores por defecto:
```ini
[CONFIG]
dumpsfolder = dumps
modelofolder = models
port = /dev/ttyUSB0
baudrate = 2400
timeout = 0.1
posiciones = 000000
savefile = s
repeticionesenvio = 30
portsalida = /dev/ttyUSB1
neuronasfirstlayer = 79
```

---

## Arquitectura del Código

### Flujo de Ejecución

```
curryrtool.py
    └── startup() [loader/main.py]
        └── entry_points.main_devices() / entry_points.main_trainning()
            ├── DeviceInterface [interfaces/devices_interface.py]
            │   ├── read (captura datos)
            │   ├── write (envía tramas)
            │   ├── repeat (modo repetidor)
            │   └── menu (interactivo)
            └── TrainningInterface [interfaces/trainning_interface.py]
                ├── entrenar (entrena modelo)
                ├── predecir (predice estado)
                └── pesos (muestra pesos del modelo)
```

### Módulos Principales

#### 1. curryrtool.py (Entry Point)
**Propósito**: Punto de entrada principal.

```bash
python3 curryrtool.py <comando> <acción> [opciones]
```

Comandos disponibles:
- `device` - Operaciones de comunicación serial
- `trainning` - Entrenamiento y predicción ML

#### 2. curryrtool (Wrapper Bash)
Script bash ejecutable que facilita la ejecución:
```bash
./curryrtool device read -p /dev/ttyUSB0
```

#### 3. devices_interface.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darioaxel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
