---
trigger: always_on
description: public class PlayerController : MonoBehaviour { }
---



# Unity Core Rules - C# & MonoBehaviour

## Naming Conventions for Unity 6.2

### Classes and Structs
```
// ✅ DO: PascalCase
public class PlayerController : MonoBehaviour { }

// ✅ DO: Readonly structs for data integrity
public readonly struct GameConfig { }

public interface IHealthSystem { }

// ❌ DON'T
public class player_controller { } // snake_case
public class playerController { } // camelCase
public struct MutableConfig { } // Avoid mutable structs
```

### Fields and Properties
```
public class Example : MonoBehaviour
{
    // ✅ DO: Private fields with _ prefix (.NET Style)
    [SerializeField] private float _moveSpeed = 5f;
    private Transform _targetTransform;
    
    // ✅ DO: Public properties in PascalCase
    public float MoveSpeed => _moveSpeed;
    public bool IsMoving { get; private set; }
    
    // ✅ DO: Constants in PascalCase (Microsoft Standard)
    private const float MaxHealth = 100f;
    private const string PlayerTag = "Player";
    
    // ✅ DO: Static fields with _ prefix
    private static int _instanceCount = 0;
    
    // ✅ DO: Booleans with is/has/can prefix
    private bool _isGrounded;
    private bool _hasWeapon;
    private bool _canJump;
    
    // ❌ DON'T: Public fields without [SerializeField]
    public float moveSpeed; // Use property or [SerializeField] private
    
    // ❌ DON'T: Hungarian notation
    private float m_Speed; 
    private float fSpeed;
}
```

### Methods and Events
```
public class EventExample : MonoBehaviour
{
    // ✅ DO: Methods in PascalCase
    public void ProcessInput() { }
    private void HandleCollision() { }
    
    // ✅ DO: Events with On prefix
    public event Action OnPlayerDeath;
    public event Action<int> OnScoreChanged;
    
    // ✅ DO: Async methods with Async suffix (Use Awaitable for Unity 6.2)
    private async Awaitable LoadDataAsync() { }
    public async Awaitable<bool> TryConnectAsync() { }
}
```

## MonoBehaviour Lifecycle

### Correct Method Order
```
public class LifecycleExample : MonoBehaviour
{
    // 1. Serialized Fields
    [SerializeField] private float _speed = 5f;
    
    // 2. Private Fields
    private Rigidbody _rigidbody;
    private bool _isInitialized;
    
    // 3. Properties
    public bool IsInitialized => _isInitialized;
    
    // 4. Unity Lifecycle Methods (in call order)
    private void Awake()
    {
        // Initialize components on this object
        // Use TryGetComponent to avoid implicit allocation if missing
        if (!TryGetComponent(out _rigidbody))
        {
            Debug.LogError("Rigidbody missing!");
        }
    }
    
    private void OnEnable()
    {
        // Subscribe to events
        GameEvents.OnLevelStart += HandleLevelStart;
    }
    
    private void Start()
    {
        // Initialization after all Awake calls
        _isInitialized = true;
    }
    
    private void FixedUpdate()
    {
        // Physics
        if (_isInitialized)
        {
            ApplyPhysics();
        }
    }
    
    private void Update()
    {
        // Game logic and input
        ProcessInput();
    }
    
    private void LateUpdate()
    {
        // Called after all Update calls (e.g., camera)
    }
    
    private void OnDisable()
    {
        // Unsubscribe from events
        GameEvents.OnLevelStart -= HandleLevelStart;
    }
    
    private void OnDestroy()
    {
        // Cleanup
        CleanupResources();
    }
    
    // 5. Custom Methods
    private void ProcessInput() { }
    private void ApplyPhysics() { }
    private void HandleLevelStart() { }
    private void CleanupResources() { }
}
```

## Serialization

### [SerializeField] Best Practices
```
public class SerializationExample : MonoBehaviour
{
    // ✅ DO: Private fields with [SerializeField] and _ prefix
    [SerializeField] private float _health = 100f;
    [SerializeField] private GameObject _weaponPrefab;
    
    // ✅ DO: Headers for grouping
    [Header("Movement Settings")]
    [SerializeField] private float _moveSpeed = 5f;
    [SerializeField] private float _jumpForce = 10f;
    
    [Header("Audio")]
    [SerializeField] private AudioClip _jumpSound;
    [SerializeField] private AudioClip _landSound;
    
    // ✅ DO: Tooltip for inspector documentation
    [Tooltip("Maximum speed the player can reach")]
    [SerializeField] private float _maxSpeed = 20f;
    
    // ✅ DO: Range to limit values
    [Range(0f, 1f)]
    [SerializeField] private float _volume = 0.5f;
    
    // ✅ DO: HideInInspector to hide runtime values
    [HideInInspector]
    public int RuntimeValue; // Used by systems but not editable
}
```

### ScriptableObject for Configuration
```
// ✅ DO: Configuration via ScriptableObject
[CreateAssetMenu(fileName = "WeaponConfig", menuName = "Game/Weapon Config")]
public class WeaponConfig : ScriptableObject
{
    [Header("Stats")]
    [SerializeField] private int _damage = 10;
    [SerializeField] private float _fireRate = 0.5f;
    
    [Header("Visuals")]
    [SerializeField] private GameObject _modelPrefab;
    [SerializeField] private ParticleSystem _muzzleFlash;
    
    public int Damage => _damage;
    public float FireRate => _fireRate;
    public GameObject ModelPrefab => _modelPrefab;
}

public class Weapon : MonoBehaviour
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Common-ka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
