# Jetpack Compose Testing Guide

Complete guide for E2E testing Jetpack Compose applications.

## Overview

Compose testing uses a fundamentally different approach from View-based testing:

| Aspect | View Testing (Espresso) | Compose Testing |
|--------|-------------------------|-----------------|
| Element ID | `onView(withId(...))` | `onNodeWithTag(...)` |
| Interaction | `ViewInteraction` | `SemanticsNodeInteraction` |
| Rule | `ActivityScenarioRule` | `ComposeTestRule` |
| Identification | Resource ID | Test Tag / Semantics |
| Synchronization | IdlingResource | Automatic |

## Setup

### Dependencies

In `app/build.gradle.kts`:

```kotlin
dependencies {
    // Compose BOM
    val composeBom = platform("androidx.compose:compose-bom:2024.02.00")
    androidTestImplementation(composeBom)

    // Compose testing
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
    debugImplementation("androidx.compose.ui:ui-test-manifest")

    // Optional: JUnit runner
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
}
```

## Setting Up Test Tags

### In Your Composables

```kotlin
@Composable
fun LoginScreen(
    onLoginClick: (email: String, password: String) -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .testTag("login_screen")
    ) {
        TextField(
            value = email,
            onValueChange = { email = it },
            modifier = Modifier.testTag("login_input_email"),
            placeholder = { Text("Email") }
        )

        TextField(
            value = password,
            onValueChange = { password = it },
            modifier = Modifier.testTag("login_input_password"),
            visualTransformation = PasswordVisualTransformation(),
            placeholder = { Text("Password") }
        )

        Button(
            onClick = { onLoginClick(email, password) },
            modifier = Modifier.testTag("login_button_submit")
        ) {
            Text("Login")
        }

        if (showError) {
            Text(
                text = errorMessage,
                color = Color.Red,
                modifier = Modifier.testTag("login_text_error")
            )
        }
    }
}
```

### Naming Convention

```
{screenName}_{elementType}_{elementName}

Types:
- screen    → Screen container
- input     → TextField
- button    → Button, IconButton
- text      → Text
- toggle    → Switch, Checkbox
- list      → LazyColumn, LazyRow
- item      → List item
- image     → Image
```

Examples:
```
login_screen
login_input_email
login_input_password
login_button_submit
login_text_error
home_list_items
home_item_0
settings_toggle_darkMode
```

## Page Object Pattern

### ComposeBasePage

```kotlin
abstract class ComposeBasePage(
    protected val composeRule: ComposeTestRule
) {
    abstract val screenTestTag: String

    fun verifyScreenDisplayed(timeoutMs: Long = 5000): Boolean {
        return try {
            composeRule.waitUntil(timeoutMs) {
                composeRule
                    .onAllNodesWithTag(screenTestTag)
                    .fetchSemanticsNodes()
                    .isNotEmpty()
            }
            true
        } catch (e: Exception) {
            false
        }
    }

    protected fun nodeByTag(tag: String) =
        composeRule.onNodeWithTag(tag)

    protected fun SemanticsNodeInteraction.safeTap() =
        this.assertExists().performClick()

    protected fun SemanticsNodeInteraction.safeClearAndTypeText(text: String) =
        this.assertExists().performTextClearance().performTextInput(text)
}
```

### LoginPage Example

```kotlin
class LoginPage(
    composeRule: ComposeTestRule
) : ComposeBasePage(composeRule) {

    override val screenTestTag = "login_screen"

    // Elements
    private val emailInput get() = nodeByTag("login_input_email")
    private val passwordInput get() = nodeByTag("login_input_password")
    private val submitButton get() = nodeByTag("login_button_submit")
    private val errorText get() = nodeByTag("login_text_error")

    // Actions
    fun enterEmail(text: String) = apply {
        emailInput.safeClearAndTypeText(text)
    }

    fun enterPassword(text: String) = apply {
        passwordInput.safeClearAndTypeText(text)
    }

    fun tapSubmit() = apply {
        submitButton.safeTap()
    }

    // Navigation
    fun login(email: String, password: String): HomePage {
        enterEmail(email)
        enterPassword(password)
        tapSubmit()
        return HomePage(composeRule)
    }

    // Verification
    fun verifyErrorDisplayed(message: String): Boolean {
        return try {
            errorText.assert(hasText(message))
            true
        } catch (e: AssertionError) {
            false
        }
    }
}
```

## Writing Tests

### Basic Test Structure

```kotlin
class LoginTests : ComposeBaseUITest() {

    private val loginPage by lazy { LoginPage(composeRule) }

    @Test
    fun test_login_succeeds_withValidCredentials() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        val homePage = loginPage.login(
            email = "test@example.com",
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
            .enterEmail("test@example.com")
            .enterPassword("wrong")
            .tapSubmit()

        // Then
        assertTrue(loginPage.verifyErrorDisplayed("Invalid credentials"))
    }
}
```

### Testing Animations

```kotlin
@Test
fun test_withAnimation() {
    // Disable auto-advance for manual animation control
    composeRule.mainClock.autoAdvance = false

    // Trigger animation
    loginPage.tapSubmit()

    // Advance time to complete animation
    composeRule.mainClock.advanceTimeBy(500)

    // Verify result
    assertTrue(homePage.verifyScreenDisplayed())
}
```

### Testing Lists

```kotlin
class ItemListPage(composeRule: ComposeTestRule) : ComposeBasePage(composeRule) {

    override val screenTestTag = "item_list_screen"

    private val itemList get() = nodeByTag("item_list")

    fun tapItemAt(index: Int) = apply {
        composeRule
            .onNodeWithTag(itemList)
            .performScrollToNode(hasTestTag("item_$index"))
        nodeByTag("item_$index").safeTap()
    }

    fun getItemCount(): Int {
        return composeRule
            .onAllNodes(hasTestTag("item_", substring = true))
            .fetchSemanticsNodes()
            .size
    }

    fun scrollToItem(index: Int) = apply {
        composeRule
            .onNodeWithTag("item_list")
            .performScrollToNode(hasTestTag("item_$index"))
    }
}
```

## Waiting Strategies

### Built-in Waiting

```kotlin
// Wait for composition to complete
composeRule.waitForIdle()

// Wait with custom condition
composeRule.waitUntil(5000) {
    composeRule
        .onAllNodesWithTag("login_screen")
        .fetchSemanticsNodes()
        .isNotEmpty()
}
```

### Custom Wait Utilities

```kotlin
object ComposeTestUtils {

    fun ComposeTestRule.waitUntilDisplayed(tag: String, timeoutMs: Long = 5000) {
        this.waitUntil(timeoutMs) {
            this.onAllNodesWithTag(tag)
                .fetchSemanticsNodes()
                .isNotEmpty()
        }
    }

    fun ComposeTestRule.waitUntilGone(tag: String, timeoutMs: Long = 5000) {
        this.waitUntil(timeoutMs) {
            this.onAllNodesWithTag(tag)
                .fetchSemanticsNodes()
                .isEmpty()
        }
    }

    fun ComposeTestRule.waitForLoadingToComplete(
        loadingTag: String = "loading_indicator",
        timeoutMs: Long = 10000
    ) {
        // Wait for loading to appear (might already be gone)
        try {
            this.waitUntil(1000) {
                this.onAllNodesWithTag(loadingTag)
                    .fetchSemanticsNodes()
                    .isNotEmpty()
            }
        } catch (e: Exception) {
            return
        }

        // Wait for it to disappear
        waitUntilGone(loadingTag, timeoutMs)
    }
}
```

## Gestures

### Basic Gestures

```kotlin
// Click
node.performClick()

// Long press
node.performTouchInput { longClick() }

// Double tap
node.performTouchInput { doubleClick() }
```

### Swipe Gestures

```kotlin
// Swipe directions
node.performTouchInput { swipeUp() }
node.performTouchInput { swipeDown() }
node.performTouchInput { swipeLeft() }
node.performTouchInput { swipeRight() }

// Swipe to refresh
node.performTouchInput {
    swipeDown(startY = centerY, endY = bottom)
}
```

### Scroll

```kotlin
// Scroll to node
composeRule
    .onNodeWithTag("list")
    .performScrollToNode(hasTestTag("item_10"))

// Scroll to index
composeRule
    .onNodeWithTag("list")
    .performScrollToIndex(10)
```

## Assertions

### Node Assertions

```kotlin
// Exists and displayed
node.assertExists()
node.assertIsDisplayed()

// Enabled/Disabled
node.assertIsEnabled()
node.assertIsNotEnabled()

// Text content
node.assert(hasText("Expected"))
node.assert(hasText("partial", substring = true))

// Selection state
node.assertIsSelected()
node.assertIsNotSelected()

// Toggle state
node.assertIsOn()
node.assertIsOff()
```

### Collection Assertions

```kotlin
// Count nodes
val count = composeRule
    .onAllNodesWithTag("item")
    .fetchSemanticsNodes()
    .size

assertEquals(5, count)

// Assert on each
composeRule
    .onAllNodesWithTag("item")
    .assertCountEquals(5)
```

## Debugging

### Print Semantics Tree

```kotlin
// Entire tree
composeRule.onRoot().printToLog("TREE")

// From specific node
composeRule.onNodeWithTag("login_screen").printToLog("LOGIN")
```

### Check Node Exists

```kotlin
fun nodeExists(tag: String): Boolean {
    return composeRule
        .onAllNodesWithTag(tag)
        .fetchSemanticsNodes()
        .isNotEmpty()
}
```

### Capture Screenshot

```kotlin
// Compose doesn't have built-in screenshot
// Use UI Automator or custom implementation
composeRule.onRoot().captureToImage()
```

## Hybrid Apps (Compose + Views)

For apps using both Compose and traditional Views:

```kotlin
class HybridTests : ComposeBaseUITest() {

    @Test
    fun test_navigateFromComposeToView() {
        // Compose screen
        val composePage = ComposePage(composeRule)
        assertTrue(composePage.verifyScreenDisplayed())
        composePage.tapNavigate()

        // View screen - use Espresso
        onView(withId(R.id.view_element))
            .check(matches(isDisplayed()))
    }
}
```

## Best Practices

### Do

- ✅ Use meaningful test tags
- ✅ Follow naming conventions
- ✅ Use `waitUntil` for async operations
- ✅ Use Page Object pattern
- ✅ Test navigation flows
- ✅ Test error states

### Don't

- ❌ Use Thread.sleep()
- ❌ Hard-code selectors
- ❌ Test implementation details
- ❌ Skip accessibility semantics
- ❌ Ignore loading states

## Further Reading

- [Android Guide](./android-guide.md)
- [Espresso vs UI Automator](./espresso-vs-uiautomator.md)
- [POM Patterns](./pom-patterns.md)
