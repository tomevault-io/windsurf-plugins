---
trigger: always_on
description: Guidelines for working with the New Input System in Unity 6.2
---


# Unity Input System Rules

> Note: All private fields in the examples use camelCase names with a leading underscore, following common Microsoft C# naming conventions for fields.[web:17]

## REQUIRED: New Input System

⚠️ **Legacy Input System is FORBIDDEN for new projects**

### Package installation

```
Package Manager → Input System → Install
```

In Project Settings → Player → Active Input Handling:
- Select **"Input System Package (New)"**

## Input Actions Asset

### Creating Input Actions

```
// ✅ DO: Create an Input Actions Asset
// Assets → Create → Input Actions
```

### Input Actions structure

```
PlayerInputActions
├── Gameplay
│   ├── Move (Value, Vector2)
│   ├── Look (Value, Vector2)
│   ├── Jump (Button)
│   ├── Fire (Button)
│   └── Interact (Button)
├── UI
│   ├── Navigate (Value, Vector2)
│   ├── Submit (Button)
│   └── Cancel (Button)
└── Menu
    ├── Pause (Button)
    └── OpenInventory (Button)
```

## PlayerInput Component Approach

### Automatic wiring via PlayerInput

```
using UnityEngine;
using UnityEngine.InputSystem;

/// <summary>
/// Handles player input via the PlayerInput component.
/// </summary>
public class PlayerInputHandler : MonoBehaviour
{
    [Header("References")]
    [SerializeField] private PlayerInput _playerInput;

    private Vector2 _moveInput;
    private Vector2 _lookInput;
    private bool _jumpPressed;

    private void Awake()
    {
        if (_playerInput == null)
        {
            _playerInput = GetComponent<PlayerInput>();
            if (_playerInput == null)
            {
                Debug.LogError("PlayerInput component is missing on PlayerInputHandler.");
                enabled = false;
            }
        }
    }

    // ✅ DO: Use message methods (automatically called by PlayerInput)
    // NOTE: Prefer InputAction.CallbackContext over InputValue
    public void OnMove(InputAction.CallbackContext context)
    {
        _moveInput = context.ReadValue<Vector2>();
    }

    public void OnLook(InputAction.CallbackContext context)
    {
        _lookInput = context.ReadValue<Vector2>();
    }

    public void OnJump(InputAction.CallbackContext context)
    {
        _jumpPressed = context.ReadValueAsButton();

        if (_jumpPressed)
        {
            PerformJump();
        }
    }

    public void OnFire(InputAction.CallbackContext context)
    {
        if (context.performed)
        {
            PerformFire();
        }
    }

    private void Update()
    {
        // Use cached values for continuous input
        ProcessMovement(_moveInput);
        ProcessLook(_lookInput);

        // Example: reset one-frame jump flag if it is meant to be transient
        _jumpPressed = false;
    }

    private void ProcessMovement(Vector2 input)
    {
        // Movement logic
    }

    private void ProcessLook(Vector2 input)
    {
        // Look logic
    }

    private void PerformJump()
    {
        // Jump logic
    }

    private void PerformFire()
    {
        // Fire logic
    }
}
```

## Manual Input Actions Approach

### Working directly with InputActionAsset

```
using UnityEngine;
using UnityEngine.InputSystem;
using UnityEngine.InputSystem.InputActionRebindingExtensions;

/// <summary>
/// Controls input via direct references to Input Actions.
/// </summary>
public class ManualInputController : MonoBehaviour
{
    [Header("Input Actions")]
    [SerializeField] private InputActionAsset _inputActions;

    private InputActionMap _gameplayActionMap;
    private InputAction _moveAction;
    private InputAction _jumpAction;
    private InputAction _fireAction;

    private void Awake()
    {
        if (_inputActions == null)
        {
            Debug.LogError("InputActionAsset reference is missing on ManualInputController.");
            enabled = false;
            return;
        }

        // ✅ DO: Retrieve actions from the asset
        _gameplayActionMap = _inputActions.FindActionMap("Gameplay", throwIfNotFound: true);
        _moveAction = _gameplayActionMap.FindAction("Move", throwIfNotFound: true);
        _jumpAction = _gameplayActionMap.FindAction("Jump", throwIfNotFound: true);
        _fireAction = _gameplayActionMap.FindAction("Fire", throwIfNotFound: true);
    }

    private void OnEnable()
    {
        // ✅ DO: Subscribe to events
        _jumpAction.performed += OnJumpPerformed;
        _jumpAction.canceled += OnJumpCanceled;
        _fireAction.performed += OnFirePerformed;

        // ✅ DO: Prefer enabling the entire action map
        _gameplayActionMap.Enable();
    }

    private void OnDisable()
    {
        // ✅ DO: ALWAYS unsubscribe and disable
        _jumpAction.performed -= OnJumpPerformed;
        _jumpAction.canceled -= OnJumpCanceled;
        _fireAction.performed -= OnFirePerformed;

        _gameplayActionMap.Disable();
    }

    private void Update()
    {
        // ✅ DO: Read values in Update for continuous input
        Vector2 moveInput = _moveAction.ReadValue<Vector2>();
        ProcessMovement(moveInput);
    }

    private void OnJumpPerformed(InputAction.CallbackContext context)
    {
        Debug.Log("Jump performed!");
        PerformJump();
    }

    private void OnJumpCanceled(InputAction.CallbackContext context)
    {
        Debug.Log("Jump released!");
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Common-ka/ai-agent-unity-rules](https://github.com/Common-ka/ai-agent-unity-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
