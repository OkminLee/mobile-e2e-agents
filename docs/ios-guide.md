# iOS E2E Testing Guide

Comprehensive guide for E2E testing on iOS using XCUITest.

## Overview

XCUITest is Apple's UI testing framework that allows you to write tests that interact with your app's UI. This guide covers best practices and patterns for writing maintainable E2E tests.

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
YourAppUITests/
├── Sources/
│   ├── Support/
│   │   ├── BaseUITest.swift        # Base test class
│   │   ├── UITestConfiguration.swift # Launch configuration
│   │   ├── XCUIElement+Wait.swift  # Wait utilities
│   │   ├── XCUIElement+Scroll.swift # Scroll utilities
│   │   └── SmartWait.swift         # Intelligent waiting
│   │
│   ├── Pages/
│   │   ├── LoginPage.swift
│   │   ├── HomePage.swift
│   │   └── SettingsPage.swift
│   │
│   ├── Flows/
│   │   ├── LoginFlow.swift
│   │   └── OnboardingFlow.swift
│   │
│   └── Tests/
│       ├── LoginTests.swift
│       └── SettingsTests.swift
│
└── Info.plist
```

## Writing Page Objects

### Basic Structure

```swift
final class LoginPage: BasePage {

    // MARK: - Identifiers
    private enum Identifier {
        static let screen = "login_screen"
        static let usernameField = "login_textField_username"
        static let passwordField = "login_textField_password"
        static let submitButton = "login_button_submit"
    }

    // MARK: - Elements
    var usernameField: XCUIElement {
        app.textFields[Identifier.usernameField]
    }

    var passwordField: XCUIElement {
        app.secureTextFields[Identifier.passwordField]
    }

    var submitButton: XCUIElement {
        app.buttons[Identifier.submitButton]
    }

    // MARK: - Verification
    @discardableResult
    func verifyScreenDisplayed(timeout: TimeInterval = Timeout.medium) -> Bool {
        usernameField.waitForExistence(timeout: timeout)
    }

    // MARK: - Actions
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

    // MARK: - Compound Actions
    @discardableResult
    func login(username: String, password: String) -> HomePage {
        enterUsername(username)
            .enterPassword(password)
            .tapSubmit()
    }
}
```

### Fluent Interface

Methods return `Self` or the next page for chaining:

```swift
// Good - fluent interface
loginPage
    .enterUsername("user")
    .enterPassword("pass")
    .tapSubmit()

// Avoid - separate calls without chaining
loginPage.enterUsername("user")
loginPage.enterPassword("pass")
loginPage.tapSubmit()
```

## Writing Tests

### Test Structure

```swift
final class LoginTests: BaseUITest {

    // MARK: - Properties
    private lazy var loginPage = LoginPage(app: app)

    // MARK: - Configuration
    override var shouldSkipOnboarding: Bool { false }

    // MARK: - Tests

    func test_login_succeeds_withValidCredentials() {
        // Given
        XCTAssertTrue(loginPage.verifyScreenDisplayed())
        captureScreenshot(name: "01_login_screen")

        // When
        let homePage = loginPage.login(
            username: TestData.validUsername,
            password: TestData.validPassword
        )

        // Then
        XCTAssertTrue(homePage.verifyScreenDisplayed())
        captureScreenshot(name: "02_home_screen")
    }

    func test_login_fails_withInvalidPassword() {
        // Given
        XCTAssertTrue(loginPage.verifyScreenDisplayed())

        // When
        loginPage
            .enterUsername(TestData.validUsername)
            .enterPassword(TestData.invalidPassword)
            .tapSubmit()

        // Then
        XCTAssertTrue(loginPage.verifyErrorDisplayed())
    }
}

extension LoginTests {
    enum TestData {
        static let validUsername = "testuser"
        static let validPassword = "password123"
        static let invalidPassword = "wrong"
    }
}
```

### Test Naming Convention

```
test_{action}_{outcome}[_{condition}]

Examples:
- test_login_succeeds_withValidCredentials
- test_login_fails_withInvalidPassword
- test_createAlarm_addsToList
- test_settings_persist_afterRestart
```

## Accessibility Identifiers

### Naming Convention

```
{screenName}_{elementType}_{elementName}

Element Types:
- button
- textField
- secureField
- label
- switch
- cell
- scrollView
- picker
- image
```

### Adding to SwiftUI

```swift
Button("Submit") {
    action()
}
.accessibilityIdentifier("login_button_submit")

TextField("Username", text: $username)
    .accessibilityIdentifier("login_textField_username")
```

### Adding to UIKit

```swift
submitButton.accessibilityIdentifier = "login_button_submit"
usernameField.accessibilityIdentifier = "login_textField_username"
```

## Waiting Strategies

### Use Explicit Waits

```swift
// Good
element.waitForExistence(timeout: 5)
element.waitToBeHittable(timeout: 5)

// Avoid
sleep(5)
Thread.sleep(forTimeInterval: 5)
```

### Smart Wait Utilities

```swift
// Wait for loading to complete
SmartWait.forLoadingToComplete(in: app)

// Wait with custom condition
SmartWait.until(timeout: 10) {
    return page.isDataLoaded
}

// Wait for screen transition
SmartWait.forScreenTransition(to: "home_screen", in: app)
```

## Test Isolation

### Configuration

```swift
override var shouldSkipOnboarding: Bool { true }
override var shouldResetState: Bool { true }
override var shouldSetupTestData: Bool { true }
```

### App Side Implementation

```swift
// In AppDelegate or SceneDelegate
if ProcessInfo.processInfo.arguments.contains("-ResetState") {
    clearUserDefaults()
    clearDatabase()
    clearCache()
}

if ProcessInfo.processInfo.arguments.contains("-SetupTestData") {
    seedTestData()
}
```

## Debugging

### Print View Hierarchy

```swift
print(app.debugDescription)
```

### Capture Screenshots

```swift
captureScreenshot(name: "debug_current_state")
```

### Use Accessibility Inspector

1. Open Xcode
2. Xcode > Open Developer Tool > Accessibility Inspector
3. Point at simulator to inspect elements

## Best Practices

### Do

- ✅ Use explicit waits
- ✅ Follow naming conventions
- ✅ Keep tests independent
- ✅ Use Page Objects
- ✅ Capture screenshots on failure
- ✅ Write descriptive test names

### Don't

- ❌ Use `sleep()` or fixed delays
- ❌ Share state between tests
- ❌ Hard-code element queries
- ❌ Write tests without assertions
- ❌ Ignore flaky tests

## Further Reading

- [Apple XCTest Documentation](https://developer.apple.com/documentation/xctest)
- [POM Patterns](./pom-patterns.md)
- [Test Isolation](./test-isolation.md)
- [CI Integration](./ci-integration.md)
