---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an embedded systems project for the SAME54 microcontroller featuring:
- LwIP TCP/IP stack implementation with socket API
- FreeRTOS real-time operating system
- Ethernet connectivity with GMAC controller
- Basic web server functionality
- Hardware abstraction layer (HAL) for peripherals

## Build System

The project has been modernized with a root-level Makefile for easier builds:

### Root Directory Build (Recommended)
```bash
make clean
make all
make size     # Check memory usage
make rebuild  # Clean and build
make help     # Show available targets
```

### Legacy Build (Still Available)
```bash
# GCC toolchain in subdirectory
cd gcc && make all

# ARM Compiler (ARMCC)
cd armcc && make all
```

Build outputs (in `build/` directory):
- `build/AtmelStart.elf` - Main executable
- `build/AtmelStart.bin` - Binary firmware image
- `build/AtmelStart.hex` - Intel HEX format
- `build/AtmelStart.map` - Memory map
- `build/AtmelStart.lss` - Assembly listing

## Architecture

### Core Components

**Hardware Layer (`hal/`, `hpl/`):**
- Hardware abstraction and peripheral drivers
- Low-level hardware configuration for SAME54

**Network Stack (`lwip/`):**
- LwIP 1.4.0 TCP/IP stack
- Socket API implementation in `lwip_socket_api.c`
- Ethernet MAC interface (`lwip/lwip-1.4.0/port/ethif_mac.c`)

**RTOS (`thirdparty/RTOS/freertos/`):**
- FreeRTOS v8.2.3 for task scheduling
- Configuration in `config/FreeRTOSConfig.h`

**Application Layer:**
- `main.c` - Entry point and basic socket demo
- `webserver_tasks.c` - Web server and LED tasks
- `ethernet_phy_main.c` - Ethernet PHY management

### Key Files

- `atmel_start.c:6-11` - System initialization sequence
- `main.c:64-77` - Main application entry with socket demo
- `config/FreeRTOSConfig.h` - RTOS configuration parameters
- `config/lwipopts.h` - Network stack configuration

### Memory Layout

The project uses heap_2.c memory management with 42KB heap (`configTOTAL_HEAP_SIZE`). Stack overflow checking is disabled in the current configuration.

## Development Notes

### Network Configuration
- The system demonstrates basic socket API usage with a web server
- DHCP support is available (check `config/lwipopts.h`)
- Ethernet PHY auto-negotiation is handled in `ethernet_phy/`

### RTOS Tasks
- LED blink task for status indication
- Web server task for HTTP requests
- Network stack runs in dedicated RTOS tasks

### Hardware Platform
- Target: SAME54P20A Cortex-M4F microcontroller
- Ethernet: GMAC with external PHY
- Debug: UART stdio redirection available

The codebase follows Microchip's ASF (Advanced Software Framework) conventions and is generated from Atmel Start configuration tool.

## Build Optimization

- When build project use as many cores as possible

## Driver Development Guide

This project implements a universal driver pattern for hardware abstraction. Follow this guide when creating new drivers.

### Universal Driver Architecture

The project uses a three-layer driver architecture:

1. **Universal Driver Interface** (`drivers/driver_*.h`) - Hardware-agnostic API
2. **Universal Driver Implementation** (`drivers/driver_*.c`) - Common logic and validation
3. **BSP Driver Implementation** (`hw/same54/drivers/bsp_*.c`) - Hardware-specific code

### Creating a New Driver

Follow these steps to add a new peripheral driver (using "SPI" as an example):

#### 1. Define Universal Driver Interface

Create `drivers/driver_spi.h`:

```c
#ifndef _DRIVER_SPI_H_
#define _DRIVER_SPI_H_

#include <stdbool.h>
#include <stdint.h>

// Status enumeration
typedef enum {
    DRV_SPI_STATUS_OK = 0,
    DRV_SPI_STATUS_ERROR = 1,
    DRV_SPI_STATUS_BUSY = 2,
    DRV_SPI_STATUS_TIMEOUT = 3,
} drv_spi_status_t;

// Callback types
typedef enum {
    DRV_SPI_CB_TRANSFER_COMPLETE = 0,
    DRV_SPI_CB_ERROR = 1,
} drv_spi_cb_type_t;

typedef void (*drv_spi_callback_t)(void);

// Driver structure with function pointers
typedef struct {
    bool is_init;
    bool is_enabled;
    const void *hw_context;
    
    // Core operations
    drv_spi_status_t (*init)(const void *hw_context);
    drv_spi_status_t (*deinit)(const void *hw_context);
    drv_spi_status_t (*enable)(const void *hw_context);
    drv_spi_status_t (*disable)(const void *hw_context);
    
    // Data operations
    drv_spi_status_t (*transfer)(const void *hw_context, const uint8_t *tx_data, 
                                uint8_t *rx_data, uint32_t length);
    
    // Configuration
    drv_spi_status_t (*set_baudrate)(const void *hw_context, uint32_t baudrate);
    
    // Callback management
    drv_spi_status_t (*register_callback)(const void *hw_context, 
                                         drv_spi_cb_type_t type, 
                                         drv_spi_callback_t callback);
} drv_spi_t;

#ifdef __cplusplus
extern "C" {
#endif

// Universal API functions
drv_spi_status_t hw_spi_init(drv_spi_t *handle);
drv_spi_status_t hw_spi_deinit(drv_spi_t *handle);
drv_spi_status_t hw_spi_enable(drv_spi_t *handle);
drv_spi_status_t hw_spi_disable(drv_spi_t *handle);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chipsoft/doip](https://github.com/chipsoft/doip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
