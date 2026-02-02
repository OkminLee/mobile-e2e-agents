# /e2e-pom - Generate Page Object Model

Generate a Page Object for a specific screen/view in your app.

## Trigger

```
/e2e-pom <ScreenName> [options]
```

## Parameters

- `ScreenName`: (required) Name of the screen (e.g., `Login`, `Home`, `Settings`)
- `--view-file <path>`: (optional) Path to the view file for analysis
- `--platform <ios|android>`: (optional) Target platform (auto-detected if not specified)
- `--ui-type <view|compose>`: (optional, Android only) UI framework type

## Usage Examples

```
/e2e-pom Login
/e2e-pom HomeScreen --view-file Sources/Features/Home/HomeView.swift
/e2e-pom Settings --platform ios
/e2e-pom Login --platform android --view-file app/src/main/java/.../LoginScreen.kt
/e2e-pom Login --platform android --ui-type compose
```

## Workflow

### Phase 1: Context Analysis

1. Check if E2E structure exists (run `/e2e-init` if not)
2. Determine target platform (iOS or Android)
3. For Android: Detect UI type (View/XML or Compose)
4. Find existing Page Objects for reference

### Phase 2: View Analysis (if --view-file provided)

#### iOS (SwiftUI)
```swift
// Look for these patterns:
Button("Label") { }                    // → button element
TextField("Placeholder", text: $var)   // → text field element
Text("Label")                          // → static text element
Toggle(isOn: $var) { }                 // → switch element
List { }                               // → scroll view / collection
```

#### iOS (UIKit)
```swift
// Look for these patterns:
UIButton                               // → button element
UITextField                            // → text field element
UILabel                                // → static text element
UISwitch                               // → switch element
UITableView / UICollectionView         // → scroll view / collection
```

#### Android (Jetpack Compose)
```kotlin
// Look for these patterns:
Modifier.testTag("tag")                // → element identifier
TextField(modifier = ...)              // → text input
Button(onClick = ...) { }              // → button element
Text("Label")                          // → text element
Switch(checked = ...) { }              // → toggle element
LazyColumn { }                         // → scrollable list
```

#### Android (View/XML)
```xml
<!-- Look for these patterns: -->
android:id="@+id/element_id"           <!-- element identifier -->
<EditText />                           <!-- text input -->
<Button />                             <!-- button element -->
<TextView />                           <!-- text element -->
<Switch />                             <!-- toggle element -->
<RecyclerView />                       <!-- scrollable list -->
```

### Phase 3: Identifier Detection

#### iOS - Accessibility Identifiers
```swift
.accessibilityIdentifier("login_button_submit")
```

#### Android - Test Tags (Compose)
```kotlin
Modifier.testTag("login_button_submit")
```

#### Android - Resource IDs (View)
```xml
android:id="@+id/login_button_submit"
```

### Phase 4: Code Generation

Generate Page Object using appropriate template based on platform and UI type.

### Phase 5: Identifier Suggestions

If view file was analyzed:
1. List elements missing identifiers
2. Provide code snippets to add identifiers
3. Offer to automatically add them (with user confirmation)

## Identifier Naming Convention (Cross-Platform)

```
{screenName}_{elementType}_{elementName}

Element Types (consistent across platforms):
- button     → Button (iOS/Android)
- input      → TextField (iOS), EditText/TextField (Android)
- text       → Text (iOS/Android)
- toggle     → Toggle/Switch (iOS/Android)
- cell       → List item, RecyclerView item
- list       → List, LazyColumn, RecyclerView
- image      → Image, ImageView
```

Examples:
```
login_input_username
login_input_password
login_button_submit
home_cell_alarmItem
settings_toggle_notifications
```

## Output Structure

### iOS Output
File: `{ProjectName}UITests/Sources/Pages/{ScreenName}Page.swift`

```swift
final class LoginPage: BasePage {
    // MARK: - Elements
    var usernameField: XCUIElement {
        app.textFields["login_input_username"]
    }

    // MARK: - Actions
    @discardableResult
    func enterUsername(_ text: String) -> Self {
        usernameField.safeClearAndTypeText(text)
        return self
    }
}
```

### Android Output (Compose)
File: `app/src/androidTest/java/{package}/pages/{ScreenName}Page.kt`

```kotlin
class LoginPage(composeRule: ComposeTestRule) : ComposeBasePage(composeRule) {
    override val screenTestTag = "login_screen"

    private val usernameInput get() = nodeByTag("login_input_username")

    fun enterUsername(text: String) = apply {
        usernameInput.safeClearAndTypeText(text)
    }
}
```

### Android Output (Espresso)
File: `app/src/androidTest/java/{package}/pages/{ScreenName}Page.kt`

```kotlin
class LoginPage : BasePage() {
    override val screenIdentifier = withId(R.id.login_screen)

    private val usernameInput get() = viewById(R.id.login_input_username)

    fun enterUsername(text: String) = apply {
        usernameInput.safeClearAndTypeText(text)
    }
}
```

## Interactive Mode

If no --view-file is provided, enter interactive mode:

1. Ask about screen purpose
2. Ask about main UI elements
3. For Android: Ask about UI framework (Compose or View)
4. Generate based on user input

## Error Handling

- Screen name conflict: Ask to overwrite or rename
- View file not found: Suggest alternatives or use interactive mode
- Invalid screen name: Suggest valid naming conventions
- UI type mismatch: Warn and ask for confirmation

---

## Agent Reference

This command uses platform-specific agents:

| Platform | UI Type | Agent |
|----------|---------|-------|
| iOS | SwiftUI/UIKit | `ios-pom-generator` |
| Android | Compose | `android-pom-generator` (Compose mode) |
| Android | View/XML | `android-pom-generator` (Espresso mode) |

See:
- `.claude/agents/ios-pom-generator.md`
- `.claude/agents/android-pom-generator.md`
