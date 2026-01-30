# Page Object Model Patterns

Best practices and patterns for implementing Page Objects in E2E tests.

## What is POM?

The Page Object Model (POM) is a design pattern that creates an object repository for UI elements. It provides a clean separation between test code and page-specific code.

## Core Principles

### 1. One Page = One Screen

Each Page class represents one screen in your app:

```swift
// Good - separate pages
class LoginPage: BasePage { }
class HomePage: BasePage { }
class SettingsPage: BasePage { }

// Avoid - multiple screens in one page
class AllScreensPage: BasePage { } // ❌
```

### 2. Encapsulate Elements

Elements should be private or computed properties:

```swift
final class LoginPage: BasePage {

    // Good - elements are encapsulated
    private var usernameField: XCUIElement {
        app.textFields["login_textField_username"]
    }

    // Actions expose functionality, not elements
    func enterUsername(_ text: String) -> Self {
        usernameField.safeClearAndTypeText(text)
        return self
    }
}
```

### 3. Fluent Interface

Methods return `Self` or the next page for chaining:

```swift
// Fluent interface enables this:
loginPage
    .enterUsername("user")
    .enterPassword("pass")
    .tapSubmit()
    .verifyWelcomeMessage()
```

### 4. Return Type Indicates Navigation

- Return `Self` when staying on the same page
- Return `NextPage` when navigating to a different page

```swift
// Same page - return Self
func enterUsername(_ text: String) -> Self {
    // ...
    return self
}

// Navigation - return next page
func tapSubmit() -> HomePage {
    submitButton.safeTap()
    return HomePage(app: app)
}
```

## Common Patterns

### Verification Methods

```swift
// Boolean return for flexible usage
@discardableResult
func verifyScreenDisplayed(timeout: TimeInterval = Timeout.medium) -> Bool {
    mainElement.waitForExistence(timeout: timeout)
}

// Self return for chaining
@discardableResult
func verifyTitle(_ expected: String) -> Self {
    XCTAssertEqual(titleLabel.label, expected)
    return self
}
```

### Element Queries

```swift
// Static identifier
var submitButton: XCUIElement {
    app.buttons["login_button_submit"]
}

// Dynamic identifier
func alarmCell(at index: Int) -> XCUIElement {
    app.cells["alarm_cell_\(index)"]
}

// Predicate-based
func cell(withTitle title: String) -> XCUIElement {
    let predicate = NSPredicate(format: "label CONTAINS %@", title)
    return app.cells.matching(predicate).firstMatch
}

// Descendant search
func element(withIdentifier id: String) -> XCUIElement {
    app.descendants(matching: .any)[id]
}
```

### Handling Lists

```swift
final class AlarmListPage: BasePage {

    // Count items
    func itemCount() -> Int {
        app.cells.matching(
            NSPredicate(format: "identifier BEGINSWITH %@", "alarm_cell_")
        ).count
    }

    // Check if has items
    func hasItems() -> Bool {
        itemCount() > 0
    }

    // Get specific item
    func cell(at index: Int) -> XCUIElement {
        app.cells["alarm_cell_\(index)"]
    }

    // Scroll to item
    @discardableResult
    func scrollToItem(at index: Int) -> Self {
        cell(at: index).scrollIntoView()
        return self
    }
}
```

### Handling Modals/Alerts

```swift
final class ConfirmationDialog: BasePage {

    var confirmButton: XCUIElement {
        app.buttons["dialog_button_confirm"]
    }

    var cancelButton: XCUIElement {
        app.buttons["dialog_button_cancel"]
    }

    @discardableResult
    func tapConfirm() -> PreviousPage {
        confirmButton.safeTap()
        return PreviousPage(app: app)
    }

    @discardableResult
    func tapCancel() -> PreviousPage {
        cancelButton.safeTap()
        return PreviousPage(app: app)
    }
}

// Usage in parent page
final class ListPage: BasePage {

    func tapDelete(at index: Int) -> ConfirmationDialog {
        cell(at: index).swipeLeft()
        deleteButton.safeTap()
        return ConfirmationDialog(app: app)
    }
}

// In test
listPage
    .tapDelete(at: 0)
    .tapConfirm()
```

### Handling Tab Bars

```swift
final class TabBarPage: BasePage {

    enum Tab: String {
        case home = "tab_home"
        case settings = "tab_settings"
        case profile = "tab_profile"
    }

    @discardableResult
    func tapTab(_ tab: Tab) -> Self {
        app.buttons[tab.rawValue].safeTap()
        return self
    }

    func currentTab() -> Tab? {
        for tab in [Tab.home, .settings, .profile] {
            if app.buttons[tab.rawValue].isSelected {
                return tab
            }
        }
        return nil
    }
}
```

### Page with Sections

For complex pages, split into sections:

```swift
final class ProfilePage: BasePage {

    // Sections
    lazy var headerSection = HeaderSection(app: app)
    lazy var settingsSection = SettingsSection(app: app)
    lazy var actionsSection = ActionsSection(app: app)

    // Sub-section classes
    class HeaderSection: BasePage {
        var avatarImage: XCUIElement { ... }
        var nameLabel: XCUIElement { ... }
    }

    class SettingsSection: BasePage {
        var notificationsToggle: XCUIElement { ... }
        var darkModeToggle: XCUIElement { ... }
    }
}

// Usage
profilePage.headerSection.nameLabel.label
profilePage.settingsSection.notificationsToggle.tap()
```

## Anti-Patterns

### Don't Expose Raw Elements

```swift
// Bad
var submitButton: XCUIElement {
    app.buttons["submit"]  // Exposed to tests
}

// Good
func tapSubmit() -> NextPage {
    submitButton.safeTap()
    return NextPage(app: app)
}
```

### Don't Put Assertions in Page Objects

```swift
// Bad - assertion in page object
func enterUsername(_ text: String) -> Self {
    XCTAssertTrue(usernameField.exists)  // ❌
    usernameField.typeText(text)
    return self
}

// Good - verification method returns bool
func verifyUsernameFieldExists() -> Bool {
    usernameField.exists
}
```

### Don't Create "God" Page Objects

```swift
// Bad - too many responsibilities
class AppPage: BasePage {
    func login() { }
    func createAlarm() { }
    func changeSettings() { }
    // ... 100 more methods
}

// Good - separate by screen
class LoginPage: BasePage { }
class AlarmPage: BasePage { }
class SettingsPage: BasePage { }
```

## Testing Page Objects

You can unit test page objects to ensure they work correctly:

```swift
class LoginPageTests: XCTestCase {

    func test_loginPage_findsElements() {
        let app = XCUIApplication()
        app.launch()

        let page = LoginPage(app: app)

        XCTAssertTrue(page.verifyScreenDisplayed())
    }
}
```
