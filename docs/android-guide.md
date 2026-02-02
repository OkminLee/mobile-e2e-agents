# Android E2E Testing Guide

Comprehensive guide for E2E testing on Android using Espresso and Jetpack Compose Testing.

## Overview

Android UI testing can be done with two main frameworks:

| Framework | Best For |
|-----------|----------|
| Espresso | Traditional View-based UI |
| Compose Testing | Jetpack Compose UI |
| UI Automator | System dialogs, cross-app testing |

This guide covers best practices and patterns for writing maintainable E2E tests on Android.

## Architecture

### Page Object Model (POM)

We use the Page Object Model pattern to organize test code:

```
┌─────────────────┐
│    Tests        │  ← Test scenarios (what to test)
├─────────────────┤
│    Flows        │  ← User journeys (optional)
├─────────────────┤
│    Pages        │  ← Screen representations (how to interact)
├─────────────────┤
│   Support       │  ← Utilities and base classes
└─────────────────┘
```

### File Organization

```
app/src/androidTest/java/{package}/
├── support/
│   ├── ViewWait.kt           # Espresso wait utilities
│   ├── ScrollUtils.kt        # Scroll helpers
│   ├── SystemHelper.kt       # UI Automator helpers
│   ├── TestDataManager.kt    # Test data management
│   └── ScreenshotHelper.kt   # Screenshot capture
│
├── pages/
│   ├── BasePage.kt           # Espresso base
│   ├── ComposeBasePage.kt    # Compose base
│   ├── LoginPage.kt
│   └── HomePage.kt
│
├── flows/
│   └── LoginFlow.kt
│
└── tests/
    ├── BaseUITest.kt         # Espresso test base
    ├── ComposeBaseUITest.kt  # Compose test base
    ├── LoginTests.kt
    └── HomeTests.kt
```

## Writing Page Objects

### Espresso Page Object (View-based)

```kotlin
class LoginPage : BasePage() {

    // MARK: - Screen Identifier
    override val screenIdentifier: Matcher<View>
        get() = withId(R.id.login_screen)

    // MARK: - Elements
    private val usernameInput
        get() = viewById(R.id.login_input_username)

    private val passwordInput
        get() = viewById(R.id.login_input_password)

    private val submitButton
        get() = viewById(R.id.login_button_submit)

    // MARK: - Verification
    fun verifyLoginDisplayed(): Boolean {
        return verifyScreenDisplayed()
    }

    // MARK: - Actions
    fun enterUsername(text: String) = apply {
        usernameInput.safeClearAndTypeText(text)
    }

    fun enterPassword(text: String) = apply {
        passwordInput.safeClearAndTypeText(text)
    }

    fun tapSubmit(): HomePage {
        submitButton.safeTap()
        return HomePage()
    }

    // MARK: - Compound Actions
    fun login(username: String, password: String): HomePage {
        enterUsername(username)
        enterPassword(password)
        return tapSubmit()
    }
}
```

### Compose Page Object

```kotlin
class LoginPage(
    composeRule: ComposeTestRule
) : ComposeBasePage(composeRule) {

    // MARK: - Screen Identifier
    override val screenTestTag = "login_screen"

    // MARK: - Elements
    private val usernameInput
        get() = nodeByTag("login_input_username")

    private val passwordInput
        get() = nodeByTag("login_input_password")

    private val submitButton
        get() = nodeByTag("login_button_submit")

    // MARK: - Verification
    fun verifyLoginDisplayed(): Boolean {
        return verifyScreenDisplayed()
    }

    // MARK: - Actions
    fun enterUsername(text: String) = apply {
        usernameInput.safeClearAndTypeText(text)
    }

    fun enterPassword(text: String) = apply {
        passwordInput.safeClearAndTypeText(text)
    }

    fun tapSubmit(): HomePage {
        submitButton.safeTap()
        return HomePage(composeRule)
    }

    // MARK: - Compound Actions
    fun login(username: String, password: String): HomePage {
        enterUsername(username)
        enterPassword(password)
        return tapSubmit()
    }
}
```

## Writing Tests

### Espresso Test

```kotlin
class LoginTests : BaseUITest() {

    private val loginPage by lazy { LoginPage() }

    override val shouldResetState = true

    @Test
    fun test_login_succeeds_withValidCredentials() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        val homePage = loginPage.login(
            username = "testuser@example.com",
            password = "password123"
        )

        // Then
        assertTrue(homePage.verifyScreenDisplayed())
    }

    @Test
    fun test_login_fails_withInvalidPassword() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        loginPage
            .enterUsername("testuser@example.com")
            .enterPassword("wrongpassword")
            .tapSubmit()

        // Then
        assertTrue(loginPage.verifyErrorDisplayed())
    }
}
```

### Compose Test

```kotlin
class LoginTests : ComposeBaseUITest() {

    private val loginPage by lazy { LoginPage(composeRule) }

    override val shouldResetState = true

    @Test
    fun test_login_succeeds_withValidCredentials() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        val homePage = loginPage.login(
            username = "testuser@example.com",
            password = "password123"
        )

        // Then
        assertTrue(homePage.verifyScreenDisplayed())
    }
}
```

## Element Identifiers

### View-based (Resource IDs)

In your layout XML:
```xml
<EditText
    android:id="@+id/login_input_username"
    android:hint="Email"
    android:inputType="textEmailAddress"/>
```

### Compose (Test Tags)

In your Composable:
```kotlin
TextField(
    modifier = Modifier.testTag("login_input_username"),
    value = username,
    onValueChange = { username = it }
)
```

### Naming Convention

```
{screenName}_{elementType}_{elementName}

Element Types:
- input     → TextField, EditText
- button    → Button, IconButton
- text      → Text, TextView
- toggle    → Switch, Checkbox
- list      → LazyColumn, RecyclerView
- cell      → List item
```

Examples:
```
login_input_username
login_input_password
login_button_submit
home_list_items
home_cell_item_0
settings_toggle_notifications
```

## Waiting Strategies

### Espresso

```kotlin
// Wait for view to appear
ViewWait.waitForView(withId(R.id.element), 5000)

// Wait for view to disappear
ViewWait.waitForViewToDisappear(withId(R.id.loading), 10000)

// Wait with custom condition
ViewWait.waitUntil { someCondition() }
```

### Compose

```kotlin
// Wait for node to appear
composeRule.waitUntil(5000) {
    composeRule.onAllNodesWithTag("element")
        .fetchSemanticsNodes()
        .isNotEmpty()
}

// Built-in waiting
composeRule.waitForIdle()
```

## Test Isolation

### Configuration

```kotlin
class MyTests : BaseUITest() {
    override val shouldResetState = true
}
```

### App-side Implementation

```kotlin
// In Application or MainActivity
if (ProcessInfo.testArguments.contains("-ResetState")) {
    clearSharedPreferences()
    clearDatabase()
}
```

### Using Test Orchestrator

In `build.gradle.kts`:
```kotlin
android {
    testOptions {
        execution = "ANDROIDX_TEST_ORCHESTRATOR"
    }
}

dependencies {
    androidTestUtil("androidx.test:orchestrator:1.4.2")
}
```

## System Interactions (UI Automator)

For system dialogs and cross-app testing:

```kotlin
// Handle permission dialog
SystemHelper.allowPermission()

// Open notifications
SystemHelper.openNotifications()

// Press back
SystemHelper.pressBack()

// See espresso-vs-uiautomator.md for when to use each
```

## Debugging

### Print View Hierarchy (Espresso)

```kotlin
// Hierarchy is printed automatically on failure
// Or use ADB:
// adb shell uiautomator dump
```

### Print Semantics Tree (Compose)

```kotlin
composeRule.onRoot().printToLog("DEBUG")
```

### Layout Inspector

1. Run app in debug mode
2. Android Studio > Tools > Layout Inspector
3. Select your process

### Capture Screenshots

```kotlin
captureScreenshot("debug_state")
```

## CI Integration

### GitHub Actions

```yaml
- name: Run E2E Tests
  uses: reactivecircus/android-emulator-runner@v2
  with:
    api-level: 33
    script: ./gradlew connectedDebugAndroidTest
```

### Disable Animations

```bash
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0
```

## Best Practices

### Do

- ✅ Use explicit waits
- ✅ Follow naming conventions
- ✅ Keep tests independent
- ✅ Use Page Objects
- ✅ Capture screenshots on failure
- ✅ Write descriptive test names
- ✅ Use Test Orchestrator for isolation

### Don't

- ❌ Use `Thread.sleep()` or fixed delays
- ❌ Share state between tests
- ❌ Hard-code element queries
- ❌ Write tests without assertions
- ❌ Ignore flaky tests
- ❌ Skip accessibility identifiers

## Further Reading

- [Espresso vs UI Automator](./espresso-vs-uiautomator.md)
- [Compose Testing Guide](./compose-testing.md)
- [POM Patterns](./pom-patterns.md)
- [CI Integration](./ci-integration.md)
