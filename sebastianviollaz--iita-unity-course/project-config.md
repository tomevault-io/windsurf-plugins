---
trigger: always_on
description: Este workspace contiene un **curso de Unity para estudiantes de 12 a 20 años**.
---

# Curso Unity para Adolescentes — Instrucciones del Workspace

## Contexto del Proyecto

Este workspace contiene un **curso de Unity para estudiantes de 12 a 20 años**.
El curso dura un año dividido en 2 mitades:
- **Primera mitad:** 5-6 proyectos guiados que enseñan Unity de forma progresiva (2-3 clases c/u, 1 clase/semana, ~4 clases/mes)
- **Segunda mitad:** Los alumnos crean su propio juego con lo aprendido

**Unity Version:** 2022 LTS (migración a Unity 6 planificada)

## REGLA FUNDAMENTAL: NO ES UN CURSO DE PROGRAMACIÓN

Los alumnos **NUNCA escriben código**. Todo se configura desde el **Inspector de Unity** usando:
- Arrastrar prefabs a la escena
- Configurar componentes desde el Inspector (`[SerializeField]` fields)
- Conectar scripts mediante **UnityEvents** (drag & drop en el Inspector)

**Si cualquier funcionalidad requiere que un alumno escriba C#, esa funcionalidad VIOLA el modelo del curso.**

## Scripts Compartidos (carpeta Scripts/)

Bloques modulares que los alumnos arrastran a GameObjects y configuran desde el Inspector. Se comunican entre sí mediante UnityEvents y referencias en el Inspector.

### PlayerInputs.cs — Capa 2 (Movimiento)
- **Qué hace:** Captura input del teclado/mouse (WASD, salto, agacharse, correr, cámara, zoom)
- **Campos visibles:** `AxisSettings` (ejes de movimiento), `ActionSettings` (teclas de acción), `CameraSettings` (mouse/cámara)
- **Cómo se usa:** Se agrega a un GameObject vacío. Otros scripts (como `BALLPlayer`) lo referencian en su campo `inputController`
- **NO tiene UnityEvents** — expone campos ocultos (`[HideInInspector]`) que otros scripts leen
- **Métodos invocables desde otros UnityEvents:** `ResetInputs()`

### BALLPlayer.cs — Capa 2 (Movimiento - tipo bola)
- **Qué hace:** Mueve un objeto con física (Rigidbody) tipo bola. Soporta salto, checkpoints, teletransporte
- **Campos visibles (secciones colapsables):** `BallCameraSettings` (cámara), `BallSpeedSettings` (maxSpeed, acceleration), `BallJumpSettings` (groundCheckDistance, jumpForce), `BallEvents` (eventos)
- **Requiere:** Referencia a `PlayerInputs` en el campo `inputController`. Rigidbody se agrega automáticamente con RequireComponent
- **Cámara:** El alumno arrastra una Camera al campo `Player Camera`. NO se auto-asigna. Soporta múltiples jugadores con cámaras distintas
- **UnityEvents:** `OnTeleport`, `OnSaveCheckpoint`, `OnResetToCheckpoint`
- **Métodos invocables desde otros UnityEvents:** `ResetToCheckPoint()`, `SaveCheckPoint()`, `TeleportTo(Transform)`, `SetAcceleration(float)`

### FPSPlayer.cs — Capa 2 (Movimiento - primera persona)
- **Qué hace:** Controlador FPS completo. El alumno camina, corre, salta, se agacha y mira con el mouse. Usa CharacterController (se agrega solo con RequireComponent)
- **Campos visibles (secciones colapsables):** `FPSMovementSettings` (walkSpeed, runSpeed, crouchSpeed), `FPSJumpSettings` (jumpForce, gravity), `FPSCameraSettings` (playerCamera, cameraOffset, maxLookUp/Down), `FPSCrouchSettings` (enableCrouch, smoothCrouch, crouchTransitionSpeed, crouchHeight, standingHeight), `FPSCursorSettings` (lockCursorOnStart), `FPSHeadBobSettings` (enableHeadBob, amount, frecuencias), `FPSLandingSettings` (enableLandingImpact, impactAmount), `FPSPlayerEvents`
- **Requiere:** Solo referencia a `PlayerInputs` en el campo `inputController`. CharacterController se agrega automáticamente. AudioListener se gestiona al configurar la cámara
- **Cámara:** El alumno arrastra una Camera al campo `Player Camera`. El script la hace hija del jugador y la posiciona a la altura de los ojos automáticamente. Soporta múltiples jugadores con cámaras distintas
- **UnityEvents:** `OnJump`, `OnTeleport`, `OnSaveCheckpoint`, `OnResetToCheckpoint`
- **Métodos invocables desde otros UnityEvents:** `ResetToCheckPoint()`, `SaveCheckPoint()`, `TeleportTo(Transform)`, `SetWalkSpeed(float)`, `SetRunSpeed(float)`

### OnTriggerEvent.cs — Capa 3 (Eventos/Triggers)
- **Qué hace:** Detecta cuando un objeto entra/sale de un trigger collider, filtrado por Tag
- **Campos visibles:** Array de `TagEvent` — cada uno tiene: `tag` (string), `OnEnter` (UnityEvent), `OnExit` (UnityEvent). Opción `deactivateOnEnter` para coleccionables
- **Requiere:** Un Collider con `Is Trigger = true` en el mismo GameObject
- **Cómo se conecta:** En `OnEnter`/`OnExit` el alumno arrastra un GameObject y selecciona qué método ejecutar (ej: `CountEvent.AddCount`, `GeneralGameManager.LoadScene`)

### CountEvent.cs — Capa 4 (Objetivos/Contadores)
- **Qué hace:** Cuenta ocurrencias. Al llegar al máximo, dispara un evento
- **Campos visibles:** `Text` (UI Text opcional), `Slider` (UI Slider opcional), `MaxCount` (int, mínimo 1), `OnMaxReached` (UnityEvent)
- **Métodos invocables:** `AddCount()`, `ResetCount()` — se conecta desde `OnTriggerEvent.OnEnter` u otros UnityEvents
- **Ejemplo:** 5 monedas → OnTriggerEvent llama `AddCount()` por cada moneda → al llegar a 5 → OnMaxReached dispara `GeneralGameManager.LoadScene("Victoria")`

### TimeEvent.cs — Capa 5 (Tiempo/Presión)
- **Qué hace:** Temporizador configurable (cuenta atrás o hacia adelante) con UI opcional
- **Campos visibles:** `Duration`, `CountDown`, `AutoStart`, `Loop`, `StartDelay`, UI (Text/Slider), `OnStartEvent`, `OnCompleteEvent`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SebastianViollaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
