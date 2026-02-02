# Espresso vs UI Automator: Usage Guide

When to use Espresso and when to use UI Automator for Android E2E testing.

## Overview

| Aspect | Espresso | UI Automator |
|--------|----------|--------------|
| Scope | App internal only | System-wide |
| Synchronization | Automatic | Manual |
| Speed | Fast | Slower |
| Stability | High | Medium |
| API Level | API 18+ | API 18+ |

## When to Use Espresso

✅ **Use Espresso for:**

- All in-app UI interactions
- Form input and button clicks
- List scrolling and item selection
- Screen transitions within app
- View state verification
- Most of your test code

```kotlin
// Good: In-app interactions with Espresso
class LoginPage : BasePage() {
    fun enterEmail(text: String) = apply {
        viewById(R.id.email_input).safeClearAndTypeText(text)
    }

    fun tapSubmit() = apply {
        viewById(R.id.submit_button).safeTap()
    }
}
```

❌ **Don't use Espresso for:**

- System permission dialogs
- System notifications
- Other apps
- Device settings

## When to Use UI Automator

✅ **Use UI Automator for:**

- Permission request dialogs
- System alerts and dialogs
- Notification interactions
- Cross-app navigation
- Device controls (back, home)
- System settings changes

```kotlin
// Good: System interactions with UI Automator
class LoginTests : BaseUITest() {
    @Test
    fun test_enableNotifications() {
        // In-app: Espresso
        settingsPage.tapEnableNotifications()

        // System dialog: UI Automator
        SystemHelper.allowPermission()

        // Back to app: Espresso
        assertTrue(settingsPage.verifyNotificationsEnabled())
    }
}
```

❌ **Don't use UI Automator for:**

- Regular in-app interactions (Espresso is better)
- View hierarchy navigation (slower than Espresso)

## Hybrid Pattern

Most tests combine both frameworks:

```kotlin
class NotificationTests : BaseUITest() {

    @Test
    fun test_notificationPermission_granted() {
        // Given: Start on home screen (Espresso)
        val settingsPage = homePage.tapSettings()

        // When: Request notification permission
        settingsPage.tapRequestNotifications()

        // Then: Grant via system dialog (UI Automator)
        SystemHelper.allowPermission()

        // And: Verify in app (Espresso)
        assertTrue(settingsPage.verifyNotificationsEnabled())
    }

    @Test
    fun test_pushNotification_opensApp() {
        // Given: App is backgrounded (UI Automator)
        SystemHelper.pressHome()

        // When: Notification arrives and is tapped (UI Automator)
        assertTrue(SystemHelper.tapNotification("New message"))

        // Then: App opens to correct screen (Espresso)
        val chatPage = ChatPage()
        assertTrue(chatPage.verifyScreenDisplayed())
    }
}
```

## Code Organization

Keep UI Automator code isolated in `SystemHelper`:

```kotlin
// support/SystemHelper.kt
object SystemHelper {
    private val device: UiDevice by lazy {
        UiDevice.getInstance(InstrumentationRegistry.getInstrumentation())
    }

    fun allowPermission(timeoutMs: Long = 5000): Boolean {
        val button = device.wait(
            Until.findObject(By.text("Allow")),
            timeoutMs
        )
        return button?.let { it.click(); true } ?: false
    }

    fun denyPermission() { ... }
    fun pressBack() { ... }
    fun pressHome() { ... }
    fun openNotifications() { ... }
    fun tapNotification(text: String) { ... }
}
```

## Common System Interactions

### Permission Dialogs

```kotlin
// Grant permission
SystemHelper.allowPermission()

// Deny permission
SystemHelper.denyPermission()

// Allow always (location)
SystemHelper.allowPermissionAlways()

// Allow once
SystemHelper.allowPermissionOnce()
```

### Notifications

```kotlin
// Open notification shade
SystemHelper.openNotifications()

// Tap specific notification
SystemHelper.tapNotification("Order shipped")

// Clear all notifications
SystemHelper.clearAllNotifications()

// Check if notification exists
val exists = SystemHelper.notificationExists("New message")
```

### Navigation

```kotlin
// Press device back
SystemHelper.pressBack()

// Press home
SystemHelper.pressHome()

// Open recent apps
SystemHelper.openRecentApps()
```

### App Management

```kotlin
// Force stop app
SystemHelper.forceStopApp("com.example.app")

// Clear app data
SystemHelper.clearAppData("com.example.app")

// Launch app
SystemHelper.launchApp("com.example.app")
```

## Synchronization

### Espresso

Espresso automatically synchronizes with:
- Main thread idle
- AsyncTask
- Scheduled messages

Custom synchronization with IdlingResource:
```kotlin
val idlingResource = CountingIdlingResource("Network")
IdlingRegistry.getInstance().register(idlingResource)
```

### UI Automator

Manual synchronization required:
```kotlin
// Wait for specific element
val element = device.wait(
    Until.findObject(By.text("Allow")),
    5000
)

// Wait for window update
device.waitForWindowUpdate(packageName, 5000)

// Wait for idle
device.waitForIdle(5000)
```

## Best Practices

### 1. Default to Espresso

Start with Espresso. Only use UI Automator when necessary.

```kotlin
// Prefer this (Espresso)
viewById(R.id.button).safeTap()

// Over this (UI Automator) for in-app elements
device.findObject(By.res("com.example:id/button")).click()
```

### 2. Encapsulate UI Automator

Keep UI Automator code in dedicated helper classes:

```kotlin
// Bad: UI Automator scattered throughout tests
@Test
fun test_something() {
    val device = UiDevice.getInstance(...)
    device.wait(Until.findObject(By.text("Allow")), 5000)?.click()
}

// Good: Encapsulated in SystemHelper
@Test
fun test_something() {
    SystemHelper.allowPermission()
}
```

### 3. Clear Responsibility

- **Page Objects**: Espresso only
- **SystemHelper**: UI Automator only
- **Tests**: Combine both as needed

### 4. Handle Timing

UI Automator needs explicit waits:

```kotlin
// Bad: May fail if dialog not yet shown
device.findObject(By.text("Allow")).click()

// Good: Wait for element
device.wait(Until.findObject(By.text("Allow")), 5000)?.click()
```

### 5. Handle Localization

System dialogs may have localized text:

```kotlin
fun allowPermission(): Boolean {
    val texts = listOf("Allow", "허용", "ALLOW", "Erlauben")
    for (text in texts) {
        val button = device.wait(Until.findObject(By.text(text)), 1000)
        if (button != null) {
            button.click()
            return true
        }
    }
    return false
}
```

## Troubleshooting

### UI Automator element not found

1. Check text/resource ID is correct
2. Increase timeout
3. Verify dialog is actually displayed
4. Check for localized strings

### Espresso element not found

1. Verify accessibility ID / resource ID
2. Check view is visible
3. Ensure view is not covered
4. Print view hierarchy

### Timing issues

1. Add explicit waits
2. Disable animations in CI
3. Use IdlingResources for async operations

## Summary

| Use Case | Framework |
|----------|-----------|
| Button tap in app | Espresso |
| Text input in app | Espresso |
| List scrolling | Espresso |
| Screen verification | Espresso |
| Permission dialog | UI Automator |
| Notification tap | UI Automator |
| Press back/home | UI Automator |
| Cross-app testing | UI Automator |

**Rule of thumb**: Use Espresso for everything inside your app, UI Automator for everything outside.
