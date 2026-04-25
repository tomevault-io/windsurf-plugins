---
trigger: always_on
description: This document outlines mandatory rules for efficient mathematical calculations, vector operations, matrix transformations, and collection iteration in Unity projects using C#.
---


# C# Calculation and Collection Iteration Rules for Unity

This document outlines mandatory rules for efficient mathematical calculations, vector operations, matrix transformations, and collection iteration in Unity projects using C#.

## 1. Mathematical Function Optimization

### Prefer Lightweight Mathematical Operations
- **Rule**: Always use the most efficient mathematical functions available
- **Guidelines**:
  - Use `Mathf` over `System.Math` for Unity-optimized calculations
  - Prefer simple arithmetic over complex mathematical functions
  - Cache expensive calculations when possible
  - Use Unity's built-in mathematical utilities
- **Examples**:
  ```csharp
  // ❌ Expensive mathematical operations
  public float CalculateDistance(Vector3 a, Vector3 b)
  {
      return (float)System.Math.Sqrt(
          System.Math.Pow(a.x - b.x, 2) + 
          System.Math.Pow(a.y - b.y, 2) + 
          System.Math.Pow(a.z - b.z, 2)
      );
  }
  
  // ✅ Lightweight Unity-optimized calculation for exact distance
  public float CalculateDistance(Vector3 a, Vector3 b)
  {
      return Vector3.Distance(a, b); // Only use when exact distance is needed
  }
  
  // ✅ Most efficient for distance comparisons
  public float CalculateSqrDistance(Vector3 a, Vector3 b)
  {
      return (a - b).sqrMagnitude; // Fastest for comparisons - no square root
  }
  
  // ❌ Expensive trigonometric calculations
  public float CalculateAngle(Vector3 direction)
  {
      return (float)System.Math.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;
  }
  
  // ✅ Cached calculation with Unity utilities
  private float cachedAngle;
  private Vector3 lastDirection;
  
  public float CalculateAngle(Vector3 direction)
  {
      if (lastDirection != direction)
      {
          cachedAngle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;
          lastDirection = direction;
      }
      return cachedAngle;
  }
  
  // ❌ Repeated expensive calculations
  void Update()
  {
      var normalizedDirection = direction.normalized; // Expensive every frame
      var magnitude = direction.magnitude; // Expensive every frame
  }
  
  // ✅ Cache expensive calculations
  private Vector3 cachedNormalizedDirection;
  private float cachedMagnitude;
  private Vector3 lastDirection;
  
  void Update()
  {
      if (lastDirection != direction)
      {
          cachedMagnitude = direction.magnitude;
          cachedNormalizedDirection = direction / cachedMagnitude; // Avoid normalization
          lastDirection = direction;
      }
      
      // Use cached values
      ProcessDirection(cachedNormalizedDirection, cachedMagnitude);
  }
  ```

### Avoid Expensive Mathematical Functions
- **Rule**: Minimize use of computationally expensive mathematical operations
- **Guidelines**:
  - **CRITICAL**: NEVER use Vector3.Distance or Vector2.Distance for comparisons
  - **ALWAYS** use sqrMagnitude for distance comparisons (10-50x faster)
  - Only use Distance when you need the exact distance value (not for comparisons)
  - Avoid trigonometric functions in Update loops when possible
  - Prefer integer arithmetic over floating-point when precision allows
  - Use lookup tables for repeated trigonometric calculations

### Distance vs sqrMagnitude Optimization
- **Rule**: Use sqrMagnitude for ALL distance comparisons, Distance only for exact values
- **Guidelines**:
  - **For comparisons**: Always use `(vector1 - vector2).sqrMagnitude < threshold * threshold`
  - **For exact distance**: Use `Vector3.Distance(vector1, vector2)` only when needed
  - Cache squared threshold values to avoid repeated multiplication
  - Use Vector2.sqrMagnitude for 2D distance comparisons
  - Use Vector3.sqrMagnitude for 3D distance comparisons

### Performance Comparison: Distance vs sqrMagnitude
- **Performance Impact**: sqrMagnitude is **10-50x faster** than Distance for comparisons
- **Reason**: Distance requires expensive square root calculation, sqrMagnitude does not
- **When to use Distance**: Only when you need the exact distance value for display, logging, or calculations
- **When to use sqrMagnitude**: For ALL comparisons (collision detection, range checks, sorting by distance)
- **Examples**:
  ```csharp
  // ❌ Expensive square root in Update loop - NEVER USE FOR COMPARISONS
  void Update()
  {
      float distance = Vector3.Distance(player.position, enemy.position); // Expensive!
      if (distance < detectionRange)
      {
          // Process detection
      }
  }
  
  // ✅ ALWAYS USE - Squared distance comparison (no square root)
  void Update()
  {
      float sqrDistance = (player.position - enemy.position).sqrMagnitude;
      float sqrDetectionRange = detectionRange * detectionRange; // Cache squared value
      if (sqrDistance < sqrDetectionRange)
      {
          // Process detection - same logic, 10-50x faster
      }
  }
  
  // ❌ NEVER USE - Vector3.Distance for comparisons
  void Update()
  {
      // This is expensive even for comparisons!
      float distance = Vector3.Distance(player.position, enemy.position);
      if (distance < detectionRange) // Expensive square root calculation
      {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PhuNguyen182) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
