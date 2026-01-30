# /e2e-pom - Generate Page Object Model

Generate a Page Object for a specific screen/view in your app.

## Trigger

```
/e2e-pom <ScreenName> [options]
```

## Parameters

- `ScreenName`: (required) Name of the screen (e.g., `Login`, `Home`, `Settings`)
- `--view-file <path>`: (optional) Path to the view file for analysis
- `--platform <ios|android>`: (optional) Target platform (default: ios)

## Usage Examples

```
/e2e-pom Login
/e2e-pom HomeScreen --view-file Sources/Features/Home/HomeView.swift
/e2e-pom Settings --platform ios
```

## Workflow

### Phase 1: Context Analysis

1. Check if E2E structure exists (run `/e2e-init` if not)
2. Determine target platform
3. Find existing Page Objects for reference

### Phase 2: View Analysis (if --view-file provided)

For iOS (SwiftUI):
```swift
// Look for these patterns:
Button("Label") { }                    // → button element
TextField("Placeholder", text: $var)   // → text field element
Text("Label")                          // → static text element
Toggle(isOn: $var) { }                 // → switch element
List { }                               // → scroll view / collection
```

For iOS (UIKit):
```swift
// Look for these patterns:
UIButton                               // → button element
UITextField                            // → text field element
UILabel                                // → static text element
UISwitch                               // → switch element
UITableView / UICollectionView         // → scroll view / collection
```

### Phase 3: Accessibility Identifier Detection

1. Find existing `accessibilityIdentifier` assignments
2. Identify elements WITHOUT identifiers
3. Generate suggested identifiers using convention:
   ```
   {screenName}_{elementType}_{elementName}
   ```

### Phase 4: Code Generation

Generate Page Object using template with:
- Screen identifier
- Element definitions
- Action methods
- Verification methods

### Phase 5: Identifier Suggestions

If view file was analyzed:
1. List elements missing accessibilityIdentifier
2. Provide code snippets to add identifiers
3. Offer to automatically add them (with user confirmation)

## Accessibility Identifier Convention

```
{screenName}_{elementType}_{elementName}

Element Types:
- button     → UIButton, Button
- textField  → UITextField, TextField
- label      → UILabel, Text
- switch     → UISwitch, Toggle
- cell       → TableViewCell, List item
- scrollView → UIScrollView, ScrollView
- picker     → UIPickerView, Picker
- image      → UIImageView, Image
```

Examples:
```
login_textField_username
login_textField_password
login_button_submit
home_cell_alarmItem
settings_switch_notifications
```

## Output Structure

Generated file: `{ProjectName}UITests/Sources/Pages/{ScreenName}Page.swift`

```swift
final class LoginPage: BasePage {

    // MARK: - Element Identifiers
    private enum Identifier {
        static let screen = "login_screen"
        static let usernameField = "login_textField_username"
        static let passwordField = "login_textField_password"
        static let submitButton = "login_button_submit"
        static let errorLabel = "login_label_error"
    }

    // MARK: - Elements
    var usernameField: XCUIElement {
        app.textFields[Identifier.usernameField]
    }
    // ... more elements

    // MARK: - Verification
    @discardableResult
    func verifyScreenDisplayed(timeout: TimeInterval = Timeout.medium) -> Bool {
        // ...
    }

    // MARK: - Actions
    @discardableResult
    func enterUsername(_ text: String) -> Self {
        usernameField.safeClearAndTypeText(text)
        return self
    }
    // ... more actions
}
```

## Interactive Mode

If no --view-file is provided, enter interactive mode:

1. Ask about screen purpose
2. Ask about main UI elements
3. Generate based on user input

## Error Handling

- Screen name conflict: Ask to overwrite or rename
- View file not found: Suggest alternatives or use interactive mode
- Invalid screen name: Suggest valid naming conventions

---

## Agent Reference

This command uses the `ios-pom-generator` agent for iOS or `android-pom-generator` for Android.

See: `.claude/agents/ios-pom-generator.md`
