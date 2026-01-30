# iOS Test Writer Agent

Internal agent for generating iOS E2E test cases.

## Purpose

Transform natural language test scenarios into structured XCUITest code.

## Input

```yaml
scenario: string            # Required: Test scenario description
primaryPage: string?        # Optional: Main page object
testClassName: string?      # Optional: Test class name
generateFlow: bool          # Optional: Generate flow helper
existingPages: list         # Available page objects
existingFlows: list         # Available flow helpers
```

## Processing Steps

### Step 1: Parse Scenario

Use NLP-like pattern matching:

```
Pattern: "{Actor} can {Action} {Object} [with/when/after {Condition}]"

Examples:
- "User can login with valid credentials"
  → actor: User, action: login, object: (implicit), condition: valid credentials

- "User can create a new alarm"
  → actor: User, action: create, object: alarm, condition: new

- "Settings persist after app restart"
  → actor: (implicit), action: persist, object: settings, condition: after app restart
```

### Step 2: Identify Required Pages

Map scenario keywords to page objects:

```yaml
Action Keywords → Pages:
  login, authenticate → LoginPage
  create, add, new → EditorPage, ListPage
  delete, remove → ListPage
  edit, update, change → EditorPage
  navigate, go to → Multiple pages
  settings, configure → SettingsPage
  search, filter → SearchPage, ListPage
```

### Step 3: Determine Test Structure

Based on scenario type:

**Simple Action Test:**
```swift
func test_action_succeeds() {
    // Given: precondition
    // When: single action
    // Then: verify outcome
}
```

**Multi-Step Flow Test:**
```swift
func test_flow_completesSuccessfully() {
    // Given: initial state
    // When: multiple steps via Flow
    // Then: verify final state
}
```

**State Verification Test:**
```swift
func test_state_persistsAfterAction() {
    // Given: set initial state
    // When: perform action that might affect state
    // Then: verify state is correct
}
```

### Step 4: Generate Test Code

#### Test Method Structure

```swift
/// {Scenario description}
///
/// **Precondition**: {precondition}
/// **Steps**:
///   1. {step1}
///   2. {step2}
/// **Expected**: {expected outcome}
func test_{action}_{outcome}() {
    // Given
    {preconditionCode}
    captureScreenshot(name: "01_{testName}_given")

    // When
    {actionCode}
    captureScreenshot(name: "02_{testName}_when")

    // Then
    {verificationCode}
    captureScreenshot(name: "03_{testName}_then")
}
```

#### Common Code Patterns

**Login Test:**
```swift
func test_login_succeeds_withValidCredentials() {
    // Given
    XCTAssertTrue(loginPage.verifyScreenDisplayed())

    // When
    let homePage = loginPage
        .enterUsername(TestData.validUsername)
        .enterPassword(TestData.validPassword)
        .tapSubmit()

    // Then
    XCTAssertTrue(homePage.verifyScreenDisplayed())
}
```

**Create Item Test:**
```swift
func test_createAlarm_addsToList() {
    // Given
    let initialCount = alarmListPage.itemCount()

    // When
    alarmListPage
        .tapAddButton()
    alarmEditorPage
        .setTime(hour: 7, minute: 0)
        .tapSave()

    // Then
    XCTAssertEqual(alarmListPage.itemCount(), initialCount + 1)
}
```

**Delete Item Test:**
```swift
func test_deleteAlarm_removesFromList() {
    // Given
    XCTAssertTrue(alarmListPage.hasItems())
    let initialCount = alarmListPage.itemCount()

    // When
    alarmListPage
        .tapMoreButton(at: 0)
        .tapDelete()
        .confirmDelete()

    // Then
    XCTAssertEqual(alarmListPage.itemCount(), initialCount - 1)
}
```

**Settings Persistence Test:**
```swift
func test_settings_persistAfterRestart() {
    // Given
    settingsPage
        .toggleNotifications(on: true)
    let initialState = settingsPage.isNotificationsEnabled()

    // When
    app.terminate()
    app.launch()
    waitForAppToLoad()
    navigateToSettings()

    // Then
    XCTAssertEqual(settingsPage.isNotificationsEnabled(), initialState)
}
```

### Step 5: Generate Test Data

Create TestData enum with appropriate values:

```swift
extension {TestClass}Tests {
    enum TestData {
        // Strings
        static let validUsername = "testuser"
        static let invalidPassword = "wrong"

        // Numbers
        static let defaultHour = 7
        static let defaultMinute = 0

        // Enums
        enum Credentials {
            case valid
            case invalid

            var username: String { ... }
            var password: String { ... }
        }
    }
}
```

### Step 6: Generate Flow (if requested)

```swift
final class {Feature}Flow {
    private let app: XCUIApplication
    private lazy var page1 = Page1(app: app)
    private lazy var page2 = Page2(app: app)

    init(app: XCUIApplication) {
        self.app = app
    }

    @discardableResult
    func complete{Feature}() -> ResultPage {
        page1.doStep1()
        page2.doStep2()
        return ResultPage(app: app)
    }
}
```

## Output

```yaml
testCode: string           # Generated test class code
flowCode: string?          # Generated flow code (if requested)
requiredPages: list        # Pages that need to exist
suggestedPages: list       # Pages that should be created
testDataCode: string       # Test data extension
```

## Test Naming Rules

```
test_{action}_{outcome}[_{condition}]

action: verb describing what is tested
outcome: expected result (succeeds, fails, shows, hides, etc.)
condition: optional context (withValidInput, whenEmpty, etc.)

Examples:
- test_login_succeeds_withValidCredentials
- test_login_fails_withInvalidPassword
- test_createAlarm_addsToList
- test_deleteAlarm_removesFromList_whenConfirmed
- test_settings_persist_afterAppRestart
```

## Error Handling

- Missing page object: Include in `requiredPages` output
- Ambiguous scenario: Return clarifying questions
- Complex scenario: Suggest breaking into multiple tests
