<!-- Powered by BMAD™ Core -->

# Game Development Guidelines (Unity & C#)

## Overview

This document establishes coding standards, architectural patterns, and development practices for 3D game development using Unity and C#. These guidelines ensure consistency, performance, and maintainability across all game development stories.

## C# Standards

### Naming Conventions

**Classes, Structs, Enums, and Interfaces:**

- PascalCase for types: `PlayerController`, `GameData`, `IInteractable`
- Prefix interfaces with 'I': `IDamageable`, `IControllable`
- Descriptive names that indicate purpose: `GameStateManager` not `GSM`

**Methods and Properties:**

- PascalCase for methods and properties: `CalculateScore()`, `CurrentHealth`
- Descriptive verb phrases for methods: `ActivateShield()` not `shield()`

**Fields and Variables:**

- `private` or `protected` fields: camelCase with an underscore prefix: `_playerHealth`, `_movementSpeed`
- `public` fields (use sparingly, prefer properties): PascalCase: `PlayerName`
- `static` fields: PascalCase: `Instance`, `GameVersion`
- `const` fields: PascalCase: `MaxHitPoints`
- `local` variables: camelCase: `damageAmount`, `isJumping`
- Boolean variables with is/has/can prefix: `_isAlive`, `_hasKey`, `_canJump`

**Files and Directories:**

- PascalCase for C# script files, matching the primary class name: `PlayerController.cs`
- PascalCase for Scene files: `MainMenu.unity`, `Level01.unity`

### Style and Formatting

- **Braces**: Use Allman style (braces on a new line).
- **Spacing**: Use 4 spaces for indentation (no tabs).
- **`using` directives**: Place all `using` directives at the top of the file, outside the namespace.
- **`this` keyword**: Only use `this` when necessary to distinguish between a field and a local variable/parameter.

## Unity Architecture Patterns

### Scene Lifecycle Management

**Loading and Transitioning Between Scenes:**

```csharp
// SceneLoader.cs - A singleton for managing scene transitions.
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections;

public class SceneLoader : MonoBehaviour
{
    public static SceneLoader Instance { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    public void LoadGameScene()
    {
        // Example of loading the main game scene, perhaps with a loading screen first.
        StartCoroutine(LoadSceneAsync("Level01"));
    }

    private IEnumerator LoadSceneAsync(string sceneName)
    {
        // Load a loading screen first (optional)
        SceneManager.LoadScene("LoadingScreen");

        // Wait a frame for the loading screen to appear
        yield return null;

        // Begin loading the target scene in the background
        AsyncOperation asyncLoad = SceneManager.LoadSceneAsync(sceneName);

        // Don't activate the scene until it's fully loaded
        asyncLoad.allowSceneActivation = false;

        // Wait until the asynchronous scene fully loads
        while (!asyncLoad.isDone)
        {
            // Here you could update a progress bar with asyncLoad.progress
            if (asyncLoad.progress >= 0.9f)
            {
                // Scene is loaded, allow activation
                asyncLoad.allowSceneActivation = true;
            }
            yield return null;
        }
    }
}
```

### MonoBehaviour Lifecycle

**Understanding Core MonoBehaviour Events:**

```csharp
// Example of a standard MonoBehaviour lifecycle
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    // AWAKE: Called when the script instance is being loaded.
    // Use for initialization before the game starts. Good for caching component references.
    private void Awake()
    {
        Debug.Log("PlayerController Awake!");
    }

    // ONENABLE: Called when the object becomes enabled and active.
    // Good for subscribing to events.
    private void OnEnable()
    {
        // Example: UIManager.OnGamePaused += HandleGamePaused;
    }

    // START: Called on the frame when a script is enabled just before any of the Update methods are called the first time.
    // Good for logic that depends on other objects being initialized.
    private void Start()
    {
        Debug.Log("PlayerController Start!");
    }

    // FIXEDUPDATE: Called every fixed framerate frame.
    // Use for physics calculations (e.g., applying forces to a Rigidbody).
    private void FixedUpdate()
    {
        // Handle Rigidbody movement here.
    }

    // UPDATE: Called every frame.
    // Use for most game logic, like handling input and non-physics movement.
    private void Update()
    {
        // Handle input and non-physics movement here.
    }

    // LATEUPDATE: Called every frame, after all Update functions have been called.
    // Good for camera logic that needs to track a target that moves in Update.
    private void LateUpdate()
    {
        // Camera follow logic here.
    }

    // ONDISABLE: Called when the behaviour becomes disabled or inactive.
    // Good for unsubscribing from events to prevent memory leaks.
    private void OnDisable()
    {
        // Example: UIManager.OnGamePaused -= HandleGamePaused;
    }

    // ONDESTROY: Called when the MonoBehaviour will be destroyed.
    // Good for any final cleanup.
    private void OnDestroy()
    {
        Debug.Log("PlayerController Destroyed!");
    }
}
```

### Game Object Patterns

**Component-Based Architecture:**

```csharp
// Player.cs - The main GameObject class, acts as a container for components.
using UnityEngine;

[RequireComponent(typeof(PlayerMovement), typeof(PlayerHealth))]
public class Player : MonoBehaviour
{
    public PlayerMovement Movement { get; private set; }
    public PlayerHealth Health { get; private set; }

    private void Awake()
    {
        Movement = GetComponent<PlayerMovement>();
        Health = GetComponent<PlayerHealth>();
    }
}

// PlayerHealth.cs - A component responsible only for health logic.
public class PlayerHealth : MonoBehaviour
{
    [SerializeField] private int _maxHealth = 100;
    private int _currentHealth;

    private void Awake()
    {
        _currentHealth = _maxHealth;
    }

    public void TakeDamage(int amount)
    {
        _currentHealth -= amount;
        if (_currentHealth <= 0)
        {
            Die();
        }
    }

    private void Die()
    {
        // Death logic
        Debug.Log("Player has died.");
        gameObject.SetActive(false);
    }
}
```

### Data-Driven Design with ScriptableObjects

**Define Data Containers:**

```csharp
// EnemyData.cs - A ScriptableObject to hold data for an enemy type.
using UnityEngine;

[CreateAssetMenu(fileName = "NewEnemyData", menuName = "Game/Enemy Data")]
public class EnemyData : ScriptableObject
{
    public string enemyName;
    public int maxHealth;
    public float moveSpeed;
    public int damage;
    public Material material;
    public Mesh mesh;
}

// Enemy.cs - A MonoBehaviour that uses the EnemyData.
public class Enemy : MonoBehaviour
{
    [SerializeField] private EnemyData _enemyData;
    private int _currentHealth;

    private void Start()
    {
        _currentHealth = _enemyData.maxHealth;
        var meshRenderer = GetComponent<MeshRenderer>();
        var meshFilter = GetComponent<MeshFilter>();
        meshRenderer.material = _enemyData.material;
        meshFilter.mesh = _enemyData.mesh;
    }

    // ... other enemy logic
}
```

## 3D Game Development Specifics

### Camera Systems

**3D Camera Controller:**

```csharp
// ThirdPersonCamera.cs - A flexible third-person camera controller.
using UnityEngine;

public class ThirdPersonCamera : MonoBehaviour
{
    [SerializeField] private Transform _target;
    [SerializeField] private float _distance = 5.0f;
    [SerializeField] private float _height = 2.0f;
    [SerializeField] private float _rotationSpeed = 2.0f;
    [SerializeField] private float _followSpeed = 5.0f;

    private float _currentRotationAngle;

    private void LateUpdate()
    {
        if (_target == null) return;

        // Handle camera rotation around target
        _currentRotationAngle += Input.GetAxis("Mouse X") * _rotationSpeed;

        // Calculate desired position
        Vector3 direction = new Vector3(0, _height, -_distance);
        Quaternion rotation = Quaternion.Euler(0, _currentRotationAngle, 0);
        Vector3 desiredPosition = _target.position + rotation * direction;

        // Smooth camera movement
        transform.position = Vector3.Lerp(transform.position, desiredPosition, _followSpeed * Time.deltaTime);
        transform.LookAt(_target.position + Vector3.up * _height);
    }
}
```

### 3D Movement and Controls

**Character Controller Based Movement:**

```csharp
// PlayerMovement3D.cs - Robust 3D character movement using CharacterController.
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class PlayerMovement3D : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField] private float _walkSpeed = 5.0f;
    [SerializeField] private float _runSpeed = 10.0f;
    [SerializeField] private float _jumpHeight = 2.0f;
    [SerializeField] private float _gravity = -20.0f;
    [SerializeField] private float _rotationSpeed = 720.0f;

    private CharacterController _controller;
    private Vector3 _velocity;
    private bool _isGrounded;
    private Transform _cameraTransform;

    private void Awake()
    {
        _controller = GetComponent<CharacterController>();
        _cameraTransform = Camera.main.transform;
    }

    private void Update()
    {
        _isGrounded = _controller.isGrounded;

        HandleMovement();
        HandleJumping();
        ApplyGravity();

        _controller.Move(_velocity * Time.deltaTime);
    }

    private void HandleMovement()
    {
        Vector2 input = new Vector2(Input.GetAxis("Horizontal"), Input.GetAxis("Vertical"));

        if (input.magnitude < 0.1f) return;

        // Calculate movement direction relative to camera
        Vector3 forward = _cameraTransform.forward;
        Vector3 right = _cameraTransform.right;
        forward.y = 0f;
        right.y = 0f;
        forward.Normalize();
        right.Normalize();

        Vector3 desiredMoveDirection = forward * input.y + right * input.x;
        float speed = Input.GetKey(KeyCode.LeftShift) ? _runSpeed : _walkSpeed;

        // Apply movement
        _velocity.x = desiredMoveDirection.x * speed;
        _velocity.z = desiredMoveDirection.z * speed;

        // Rotate character to face movement direction
        if (desiredMoveDirection.magnitude > 0.1f)
        {
            Quaternion targetRotation = Quaternion.LookRotation(desiredMoveDirection);
            transform.rotation = Quaternion.RotateTowards(transform.rotation, targetRotation, _rotationSpeed * Time.deltaTime);
        }
    }

    private void HandleJumping()
    {
        if (_isGrounded && Input.GetKeyDown(KeyCode.Space))
        {
            _velocity.y = Mathf.Sqrt(_jumpHeight * -2f * _gravity);
        }
    }

    private void ApplyGravity()
    {
        if (!_isGrounded)
        {
            _velocity.y += _gravity * Time.deltaTime;
        }
        else if (_velocity.y < 0)
        {
            _velocity.y = -2f; // Keep character grounded
        }
    }
}
```

### Lighting and Materials

**Dynamic Lighting System:**

```csharp
// LightingManager.cs - Manages dynamic lighting conditions.
using UnityEngine;

public class LightingManager : MonoBehaviour
{
    [Header("Day/Night Cycle")]
    [SerializeField] private Light _sunLight;
    [SerializeField] private Gradient _sunColor;
    [SerializeField] private AnimationCurve _sunIntensity;
    [SerializeField] private float _dayDurationSeconds = 120f;

    [Header("Environment")]
    [SerializeField] private Material _skyboxMaterial;
    [SerializeField] private string _skyboxTintProperty = "_Tint";

    private float _timeOfDay = 0.5f; // 0 = midnight, 0.5 = noon, 1 = midnight

    private void Update()
    {
        UpdateTimeOfDay();
        UpdateSunLighting();
        UpdateSkybox();
    }

    private void UpdateTimeOfDay()
    {
        _timeOfDay += Time.deltaTime / _dayDurationSeconds;
        if (_timeOfDay >= 1f) _timeOfDay = 0f;
    }

    private void UpdateSunLighting()
    {
        if (_sunLight == null) return;

        // Rotate sun based on time of day
        float sunAngle = _timeOfDay * 360f - 90f; // -90 to start at horizon
        _sunLight.transform.rotation = Quaternion.Euler(sunAngle, 30f, 0f);

        // Update sun color and intensity
        _sunLight.color = _sunColor.Evaluate(_timeOfDay);
        _sunLight.intensity = _sunIntensity.Evaluate(_timeOfDay);
    }

    private void UpdateSkybox()
    {
        if (_skyboxMaterial != null)
        {
            Color skyTint = Color.Lerp(Color.black, Color.white, _sunIntensity.Evaluate(_timeOfDay));
            _skyboxMaterial.SetColor(_skyboxTintProperty, skyTint);
        }
    }
}
```

### System Management

**Singleton Managers:**

```csharp
// GameManager.cs - A singleton to manage the overall game state.
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    public int Score { get; private set; }

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        DontDestroyOnLoad(gameObject); // Persist across scenes
    }

    public void AddScore(int amount)
    {
        Score += amount;
    }
}
```

## Performance Optimization

### Object Pooling

**Required for High-Frequency Objects (e.g., bullets, effects):**

```csharp
// ObjectPool.cs - A generic object pooling system.
using UnityEngine;
using System.Collections.Generic;

public class ObjectPool : MonoBehaviour
{
    [SerializeField] private GameObject _prefabToPool;
    [SerializeField] private int _initialPoolSize = 20;

    private Queue<GameObject> _pool = new Queue<GameObject>();

    private void Start()
    {
        for (int i = 0; i < _initialPoolSize; i++)
        {
            GameObject obj = Instantiate(_prefabToPool);
            obj.SetActive(false);
            _pool.Enqueue(obj);
        }
    }

    public GameObject GetObjectFromPool()
    {
        if (_pool.Count > 0)
        {
            GameObject obj = _pool.Dequeue();
            obj.SetActive(true);
            return obj;
        }
        // Optionally, expand the pool if it's empty.
        return Instantiate(_prefabToPool);
    }

    public void ReturnObjectToPool(GameObject obj)
    {
        obj.SetActive(false);
        _pool.Enqueue(obj);
    }
}
```

### Frame Rate Optimization

**Update Loop Optimization:**

- Avoid expensive calls like `GetComponent`, `FindObjectOfType`, or `Instantiate` inside `Update()` or `FixedUpdate()`. Cache references in `Awake()` or `Start()`.
- Use Coroutines or simple timers for logic that doesn't need to run every single frame.

**Physics Optimization:**

- Adjust the "Physics Settings" in Project Settings, especially the "Layer Collision Matrix", to prevent unnecessary collision checks.
- Use `Rigidbody.Sleep()` for objects that are not moving to save CPU cycles.
- Consider using continuous collision detection for fast-moving objects to prevent tunneling.
- Use appropriate collider types: BoxCollider for simple shapes, MeshCollider for complex geometry, but be mindful of performance.
- Implement occlusion culling for complex scenes to improve rendering performance.
- Use Level of Detail (LOD) systems for models that appear at various distances.

## Input Handling

### Cross-Platform Input (New Input System)

**Input Action Asset:** Create an Input Action Asset (`.inputactions`) to define controls.

**PlayerInput Component:**

- Add the `PlayerInput` component to the player GameObject.
- Set its "Actions" to the created Input Action Asset.
- Set "Behavior" to "Invoke Unity Events" to easily hook up methods in the Inspector, or "Send Messages" to use methods like `OnMove`, `OnFire`.

```csharp
// PlayerInputHandler.cs - Example of handling input via messages.
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerInputHandler : MonoBehaviour
{
    private Vector2 _moveInput;

    // This method is called by the PlayerInput component via "Send Messages".
    // The action must be named "Move" in the Input Action Asset.
    public void OnMove(InputValue value)
    {
        _moveInput = value.Get<Vector2>();
    }

    private void Update()
    {
        // Use _moveInput to control the player
        transform.Translate(new Vector3(_moveInput.x, _moveInput.y, 0) * Time.deltaTime * 5f);
    }
}
```

## Error Handling

### Graceful Degradation

**Asset Loading Error Handling:**

- When using Addressables or `Resources.Load`, always check if the loaded asset is null before using it.

```csharp
// Load a sprite and use a fallback if it fails
Sprite playerSprite = Resources.Load<Sprite>("Sprites/Player");
if (playerSprite == null)
{
    Debug.LogError("Player sprite not found! Using default.");
    playerSprite = Resources.Load<Sprite>("Sprites/Default");
}
```

### Runtime Error Recovery

**Assertions and Logging:**

- Use `Debug.Assert(condition, "Message")` to check for critical conditions that must be true.
- Use `Debug.LogError("Message")` for fatal errors and `Debug.LogWarning("Message")` for non-critical issues.

```csharp
// Example of using an assertion to ensure a component exists.
private Rigidbody _rb;

void Awake()
{
    _rb = GetComponent<Rigidbody>();
    Debug.Assert(_rb != null, "Rigidbody component not found on player!");
}
```

## Testing Standards

### Unit Testing (Edit Mode)

**Game Logic Testing:**

```csharp
// HealthSystemTests.cs - Example test for a simple health system.
using NUnit.Framework;
using UnityEngine;

public class HealthSystemTests
{
    [Test]
    public void TakeDamage_ReducesHealth()
    {
        // Arrange
        var gameObject = new GameObject();
        var healthSystem = gameObject.AddComponent<PlayerHealth>();
        // Note: This is a simplified example. You might need to mock dependencies.

        // Act
        healthSystem.TakeDamage(20);

        // Assert
        // This requires making health accessible for testing, e.g., via a public property or method.
        // Assert.AreEqual(80, healthSystem.CurrentHealth);
    }
}
```

### Integration Testing (Play Mode)

**Scene Testing:**

- Play Mode tests run in a live scene, allowing you to test interactions between multiple components and systems.
- Use `yield return null;` to wait for the next frame.

```csharp
// PlayerJumpTest.cs
using System.Collections;
using NUnit.Framework;
using UnityEngine;
using UnityEngine.TestTools;

public class PlayerJumpTest
{
    [UnityTest]
    public IEnumerator PlayerJumps_WhenSpaceIsPressed()
    {
        // Arrange
        var player = new GameObject().AddComponent<PlayerController>();
        var initialY = player.transform.position.y;

        // Act
        // Simulate pressing the jump button (requires setting up the input system for tests)
        // For simplicity, we'll call a public method here.
        // player.Jump();

        // Wait for a few physics frames
        yield return new WaitForSeconds(0.5f);

        // Assert
        Assert.Greater(player.transform.position.y, initialY);
    }
}
```

## File Organization

### Project Structure

```
Assets/
├── Scenes/
│   ├── MainMenu.unity
│   └── Level01.unity
├── Scripts/
│   ├── Core/
│   │   ├── GameManager.cs
│   │   └── AudioManager.cs
│   ├── Player/
│   │   ├── PlayerController.cs
│   │   └── PlayerHealth.cs
│   ├── Editor/
│   │   └── CustomInspectors.cs
│   └── Data/
│       └── EnemyData.cs
├── Prefabs/
│   ├── Player.prefab
│   └── Enemies/
│       └── Slime.prefab
├── Art/
│   ├── Sprites/
│   └── Animations/
├── Audio/
│   ├── Music/
│   └── SFX/
├── Data/
│   └── ScriptableObjects/
│       └── EnemyData/
└── Tests/
    ├── EditMode/
    │   └── HealthSystemTests.cs
    └── PlayMode/
        └── PlayerJumpTest.cs
```

## Development Workflow

### Story Implementation Process

1. **Read Story Requirements:**
   - Understand acceptance criteria
   - Identify technical requirements
   - Review performance constraints

2. **Plan Implementation:**
   - Identify files to create/modify
   - Consider Unity's component-based architecture
   - Plan testing approach

3. **Implement Feature:**
   - Write clean C# code following all guidelines
   - Use established patterns
   - Maintain stable FPS performance

4. **Test Implementation:**
   - Write edit mode tests for game logic
   - Write play mode tests for integration testing
   - Test cross-platform functionality
   - Validate performance targets

5. **Update Documentation:**
   - Mark story checkboxes complete
   - Document any deviations
   - Update architecture if needed

### Code Review Checklist

- [ ] C# code compiles without errors or warnings.
- [ ] All automated tests pass.
- [ ] Code follows naming conventions and architectural patterns.
- [ ] No expensive operations in `Update()` loops.
- [ ] Public fields/methods are documented with comments.
- [ ] New assets are organized into the correct folders.

## Performance Targets

### Frame Rate Requirements

- **PC/Console**: Maintain a stable 60+ FPS.
- **Mobile**: Maintain 60 FPS on mid-range devices, minimum 30 FPS on low-end.
- **Optimization**: Use the Unity Profiler to identify and fix performance drops.

### Memory Management

- **Total Memory**: Keep builds under platform-specific limits (e.g., 200MB for a simple mobile game).
- **Garbage Collection**: Minimize GC spikes by avoiding string concatenation, `new` keyword usage in loops, and by pooling objects.

### Loading Performance

- **Initial Load**: Under 5 seconds for game start.
- **Scene Transitions**: Under 2 seconds between scenes. Use asynchronous scene loading.

These guidelines ensure consistent, high-quality game development that meets performance targets and maintains code quality across all implementation stories.
