---
trigger: always_on
description: Este archivo es para ti, el modelo, cuando un usuario te pida montar
---

# AGENTS.md — guía para LLMs que usan packet-tracer-mcp

Este archivo es para ti, el modelo, cuando un usuario te pida montar
topologías en Cisco Packet Tracer 9 a través de este servidor MCP.
Léelo antes de empezar a llamar tools — te ahorra retrabajos.

## 1. Antes de tocar nada

1. Llama a `pt_bridge_status` para verificar que el bridge ve la
   extensión PT. Si no, pide al usuario el bootstrap.
2. Llama a `pt_query_topology` para ver qué hay ya en el canvas.
3. Si el usuario aporta un diagrama (PDF, imagen, sketch), descríbelo
   con tus palabras antes de crear nada. Identifica:
   - Cuántos routers, switches y endpoints.
   - Qué LANs son **de usuario** (con PCs/servidores).
   - Qué LANs son **de tránsito** entre routers (sólo switch, sin PCs).
   - Qué enlaces son **LAN interna** (Gigabit) y cuáles **WAN exterior**
     (Serial /30 público).
4. Si el diagrama es ambiguo, **pregunta al usuario antes de adivinar**.
   Una topología hecha sobre la suposición equivocada cuesta más tiempo
   de rehacer que la pregunta.

## 2. Convenciones de cableado

Esta es la fuente de errores más común. Lee con atención.

| Tipo de enlace                                  | Cable     | Interfaces                          |
| ----------------------------------------------- | --------- | ----------------------------------- |
| Router ↔ Switch (LAN de oficina)                | straight  | GigabitEthernet ↔ Gigabit/Fast      |
| Switch ↔ PC / Server                            | straight  | Fast/GigabitEthernet                |
| Router ↔ Router por LAN compartida vía switch   | straight  | GigabitEthernet (ambos vía switch)  |
| **Router ↔ Router por WAN exterior pública**    | **serial**| **Serial0/1/0** (requiere HWIC-2T)  |
| Switch ↔ Switch (trunk)                         | cross     | GigabitEthernet                     |

**Regla mnemotécnica:** si en el enunciado aparecen palabras como
"red pública", "ISP", "/30 público", "WAN", "punto a punto público", o
en el diagrama hay líneas rojas entre routers atravesando una nube de
Internet → es **serial**. Si es un segmento dentro de la misma oficina
→ **GigabitEthernet**.

**Importante:** los routers ISR (1941, 2901, 2911, ISR4321, ISR4331)
**no tienen puertos serie por defecto**. Antes de crear un enlace
serial llama a `pt_add_module` con `HWIC-2T` en cada router. Si te
saltas este paso, el `pt_create_link` con `cable="serial"` fallará.

**Slots por chasis (validados por `pt_add_module`):**

| Modelo  | Bays disponibles                | Familia            |
| ------- | ------------------------------- | ------------------ |
| 1841    | `0/0`, `0/1`                    | HWIC / WIC         |
| 2811    | `0/0`, `0/1`, `0/2`, `0/3`      | HWIC / WIC / NM    |
| 1941    | `0/0`, `0/1`                    | EHWIC / HWIC / WIC |
| 2901    | `0/0`, `0/1`, `0/2`, `0/3`      | EHWIC / HWIC / WIC |
| 2911    | `0/0`, `0/1`, `0/2`, `0/3`      | EHWIC / HWIC / WIC |
| ISR4321 | `0/1`, `1/0`                    | NIM / SM-X         |
| ISR4331 | `0/1`, `0/2`, `1/0`             | NIM / SM-X         |

**Aviso ISR4xxx:** la serie 4000 NO usa HWIC sino **NIM** (Network
Interface Module). En Packet Tracer 9 el módulo `HWIC-2T` se acepta
en NIM bays por compatibilidad, pero la práctica académica suele
pedir `NIM-2T`. Comprueba el enunciado: si menciona NIM-2T, instala
ese; si dice HWIC-2T, ambos funcionan en PT.

**Errores típicos al añadir módulos:**
- ❌ Slot `0/0` en un ISR4321/4331 → ese bay es BUILTIN (los GE de
  fábrica), no acepta módulos. Usa `0/1` (o `0/2` en el 4331).
- ❌ Slot `0/2` en un 1941 → sólo tiene 2 bays (`0/0`, `0/1`).
- ❌ `0/3` en un 1841 → mismo motivo, sólo 2 bays.

## 3. LANs de usuario vs LANs de tránsito

Un patrón muy frecuente en prácticas académicas:

```
LAN1, LAN2  → de oficina A, con PCs colgando del switch
LAN3        → segmento de tránsito entre R1 y R2 (mismo edificio,
               sólo el switch que une los dos routers, SIN PCs)
LAN4        → segmento de tránsito entre R3 y R4 (oficina B, SIN PCs)
LAN5, LAN6  → de oficina B, con PCs colgando del switch
```

La frase "todas las LANs con misma capacidad de direccionamiento"
**se refiere al prefijo** (todas /23, todas /24…), no a que todas
tengan endpoints de usuario. Si una LAN aparece sólo entre dos routers
de la misma oficina y no se mencionan ordenadores en ella, es de
tránsito: pon switch + cable a los dos routers, y nada más.

## 4. Flujo recomendado para topologías de varios routers

```
1. pt_clear_canvas             (si vas a empezar de cero)
2. pt_add_device  × N          (crea TODOS los devices SIN x/y)
3. pt_add_module  × M          (HWIC-2T donde haga falta serial)
4. pt_create_link × K          (cablea siguiendo convenciones de §2)
5. pt_auto_layout              (re-rejilla todo el canvas)
6. pt_run_cli_bulk             (configuración IP / hostname)
7. pt_apply_advanced_routing   (EIGRP/OSPF/BGP)
8. pt_save_pkt                 (persistir el resultado)
```

**No inventes coordenadas a mano.** El layout topology-aware de
`pt_auto_layout` produce un canvas ordenado: routers en una banda,
switches debajo de su router padre, endpoints debajo de su switch,
nubes/clouds compartidas centradas entre los routers que las
comparten. Inventar X/Y suele acabar en el típico canvas alargado y

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jcorderop02/packet-tracer-mcp](https://github.com/Jcorderop02/packet-tracer-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
