# iOS POM Generator Agent

Internal agent for generating iOS Page Object Model files.

## Purpose

Analyze iOS view files and generate corresponding Page Object files for XCUITest.

## Input

```yaml
screenName: string          # Required: Name of the screen
viewFilePath: string?       # Optional: Path to view file
existingIdentifiers: dict?  # Optional: Already detected identifiers
interactiveAnswers: dict?   # Optional: Answers from interactive mode
```

## Processing Steps

### Step 1: Determine View Type

```
If viewFilePath provided:
  - Read file content
  - Detect: SwiftUI vs UIKit
    - SwiftUI: Contains "import SwiftUI", "struct ... View", "var body:"
    - UIKit: Contains "import UIKit", "class ... UIViewController"
Else:
  - Use interactive mode
```

### Step 2: Extract UI Elements (SwiftUI)

Pattern matching for SwiftUI views:

```swift
// Buttons
Button("Label") { }           → type: button, label: "Label"
Button(action: {}) { Text() } → type: button, needs identifier
.buttonStyle()                → confirms button

// Text Fields
TextField("Placeholder", text:) → type: textField, placeholder: "Placeholder"
SecureField()                   → type: secureField

// Text
Text("Label")                   → type: label, text: "Label"
Text(variable)                  → type: label, dynamic

// Toggles
Toggle(isOn:) { }               → type: switch

// Lists
List { }                        → type: scrollView
ForEach { }                     → type: collection

// Navigation
NavigationLink { }              → type: button (navigation)

// Images
Image()                         → type: image
AsyncImage()                    → type: image
```

### Step 3: Extract UI Elements (UIKit)

Pattern matching for UIKit:

```swift
// IBOutlet declarations
@IBOutlet weak var nameLabel: UILabel!      → type: label, name: "nameLabel"
@IBOutlet weak var submitButton: UIButton!  → type: button, name: "submitButton"

// Programmatic creation
let button = UIButton()                     → type: button
lazy var tableView: UITableView             → type: scrollView
```

### Step 4: Detect Existing Identifiers

Search for accessibilityIdentifier assignments:

```swift
// Direct assignment
view.accessibilityIdentifier = "identifier"

// Modifier (SwiftUI)
.accessibilityIdentifier("identifier")

// Extension method
.testID(.screen.element)
```

### Step 5: Generate Identifier Suggestions

For elements without identifiers:

```swift
// Convention: {screenName}_{type}_{name}
screenName = "login"
element = Button("Submit")
suggested = "login_button_submit"
```

### Step 6: Generate Page Object

Template processing:

```swift
// Replace variables in Page.swift.template
{{PAGE_NAME}}    → screenName (PascalCase)
{{SCREEN_ID}}    → screenName_screen (snake_case)
{{ELEMENTS}}     → Generated element properties
{{ACTIONS}}      → Generated action methods
{{VERIFICATIONS}} → Generated verification methods
```

## Output

```yaml
pageObjectCode: string      # Generated Swift code
suggestedIdentifiers: list  # Identifiers to add to view
identifierDiffs: list       # Code snippets to add identifiers
warnings: list              # Any warnings or notes
```

## Element Generation Rules

### Element Property Template

```swift
var {elementName}: XCUIElement {
    app.{elementQuery}[Identifier.{identifierName}]
}
```

Element query mapping:
- button → `buttons`
- textField → `textFields`
- secureField → `secureTextFields`
- label → `staticTexts`
- switch → `switches`
- cell → `cells`
- scrollView → `scrollViews`
- image → `images`
- picker → `pickers`

### Action Method Template

```swift
@discardableResult
func {actionName}({parameters}) -> {ReturnType} {
    {elementName}.{action}
    return {returnValue}
}
```

Common actions:
- Button tap → `safeTap()`
- TextField input → `safeClearAndTypeText(text)`
- Switch toggle → `tap()`
- Cell tap → `safeTap()` returns next page

### Verification Method Template

```swift
@discardableResult
func verify{Condition}(timeout: TimeInterval = Timeout.medium) -> Bool {
    {element}.{waitMethod}(timeout: timeout)
}
```

## Error Handling

- File not found: Return error with suggestion
- Parse failure: Return partial result with warnings
- Naming conflict: Add suffix or ask user

## Example Output

Input:
```yaml
screenName: "Login"
viewFilePath: "Features/Login/LoginView.swift"
```

Output:
```swift
final class LoginPage: BasePage {

    private enum Identifier {
        static let screen = "login_screen"
        static let usernameField = "login_textField_username"
        static let passwordField = "login_textField_password"
        static let submitButton = "login_button_submit"
    }

    var usernameField: XCUIElement {
        app.textFields[Identifier.usernameField]
    }

    var passwordField: XCUIElement {
        app.secureTextFields[Identifier.passwordField]
    }

    var submitButton: XCUIElement {
        app.buttons[Identifier.submitButton]
    }

    @discardableResult
    func verifyScreenDisplayed(timeout: TimeInterval = Timeout.medium) -> Bool {
        usernameField.waitForExistence(timeout: timeout)
    }

    @discardableResult
    func enterUsername(_ text: String) -> Self {
        usernameField.safeClearAndTypeText(text)
        return self
    }

    @discardableResult
    func enterPassword(_ text: String) -> Self {
        passwordField.safeClearAndTypeText(text)
        return self
    }

    @discardableResult
    func tapSubmit() -> HomePage {
        submitButton.safeTap()
        return HomePage(app: app)
    }
}
```
