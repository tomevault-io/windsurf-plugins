---
trigger: always_on
description: The Kotlin demo app showcases best practices for integrating the Snowplow Android Tracker in a Kotlin application. It demonstrates event tracking, media tracking, configuration management, and various tracking scenarios using idiomatic Kotlin patterns.
---

# Snowplow Kotlin Demo App - CLAUDE.md

## Demo App Overview

The Kotlin demo app showcases best practices for integrating the Snowplow Android Tracker in a Kotlin application. It demonstrates event tracking, media tracking, configuration management, and various tracking scenarios using idiomatic Kotlin patterns.

## Key Implementation Patterns

### Tracker Initialization
```kotlin
// ✅ Initialize tracker with Kotlin DSL style
private fun setupTracker() {
    val networkConfig = NetworkConfiguration(collectorEndpoint, HttpMethod.POST)
    val trackerConfig = TrackerConfiguration(appId).apply {
        base64Encoding = false
        sessionContext = true
        platformContext = true
    }
    tracker = Snowplow.createTracker(this, namespace, networkConfig, trackerConfig)
}
```

### Event Tracking Patterns
```kotlin
// ✅ Use Kotlin's apply for fluent configuration
fun trackScreenView(name: String) {
    tracker.track(ScreenView(name).apply {
        entities = mutableListOf(createCustomContext())
        trueTimestamp = System.currentTimeMillis()
    })
}
```

### Extension Functions
```kotlin
// ✅ Create extension functions for common patterns
fun TrackerController.trackUserAction(action: String, category: String) {
    track(Structured(category, action).apply {
        label = "user-interaction"
        value = 1.0
    })
}
```

### Coroutine Integration
```kotlin
// ✅ Use coroutines for async operations
lifecycleScope.launch {
    withContext(Dispatchers.IO) {
        tracker.track(event)
    }
    withContext(Dispatchers.Main) {
        updateUI()
    }
}
```

## Media Tracking Implementation

### Video Player Integration
```kotlin
// ✅ Implement media tracking controller
class VideoViewController(private val videoView: VideoView) {
    private lateinit var mediaTracking: MediaTracking
    
    fun startTracking(tracker: TrackerController) {
        val config = MediaTrackingConfiguration(
            id = "video-${System.currentTimeMillis()}",
            player = MediaPlayerEntity("android-videoplayer")
        ).apply {
            boundaries = listOf(10, 25, 50, 75)
            captureEvents = listOf(MediaEvent.PLAY, MediaEvent.PAUSE)
        }
        mediaTracking = tracker.media.startMediaTracking(config)
    }
}
```

### Media Event Handling
```kotlin
// ✅ Track media events with player state
videoView.setOnPreparedListener { player ->
    mediaTracking.track(MediaReadyEvent())
    mediaTracking.update(player = MediaPlayerEntity(
        duration = player.duration.toDouble() / 1000
    ))
}
```

## Activity Patterns

### Base Activity Pattern
```kotlin
// ✅ Create base activity for common tracking
abstract class BaseTrackingActivity : AppCompatActivity() {
    protected val tracker: TrackerController by lazy {
        Snowplow.defaultTracker ?: throw IllegalStateException("Tracker not initialized")
    }
    
    override fun onResume() {
        super.onResume()
        trackScreenView()
    }
    
    abstract fun trackScreenView()
}
```

### Lifecycle-aware Tracking
```kotlin
// ✅ Use lifecycle observers
class ScreenTrackingObserver(private val tracker: TrackerController) : 
    DefaultLifecycleObserver {
    override fun onStart(owner: LifecycleOwner) {
        tracker.track(Foreground())
    }
    override fun onStop(owner: LifecycleOwner) {
        tracker.track(Background())
    }
}
```

## Configuration Management

### BuildConfig Usage
```kotlin
// ✅ Use BuildConfig for environment-specific values
object TrackerConfig {
    val COLLECTOR_URL = if (BuildConfig.DEBUG) {
        "https://staging-collector.example.com"
    } else {
        "https://collector.example.com"
    }
}
```

### Shared Preferences Integration
```kotlin
// ✅ Store user preferences
class TrackingPreferences(context: Context) {
    private val prefs = context.getSharedPreferences("tracking", Context.MODE_PRIVATE)
    
    var isTrackingEnabled: Boolean
        get() = prefs.getBoolean("enabled", true)
        set(value) = prefs.edit().putBoolean("enabled", value).apply()
}
```

## UI Integration Patterns

### Click Tracking
```kotlin
// ✅ Track UI interactions
binding.purchaseButton.setOnClickListener {
    tracker.track(Structured("ui", "click").apply {
        label = "purchase_button"
        property = productId
        value = productPrice
    })
    processPurchase()
}
```

### Form Tracking
```kotlin
// ✅ Track form submissions
fun trackFormSubmission(formData: Map<String, String>) {
    val entity = SelfDescribingJson(
        "iglu:com.example/form_submission/jsonschema/1-0-0",
        formData
    )
    tracker.track(SelfDescribing(entity))
}
```

## Testing Patterns

### Mock Tracker for Testing
```kotlin
// ✅ Create test doubles
class MockTrackerController : TrackerController {
    val trackedEvents = mutableListOf<Event>()
    
    override fun track(event: Event): UUID? {
        trackedEvents.add(event)
        return UUID.randomUUID()
    }
}
```

### UI Testing with Tracking
```kotlin
// ✅ Verify tracking in UI tests
@Test
fun testButtonClickTracking() {
    onView(withId(R.id.button)).perform(click())

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowplow/snowplow-android-tracker](https://github.com/snowplow/snowplow-android-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
