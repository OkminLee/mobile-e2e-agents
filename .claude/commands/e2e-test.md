# /e2e-test - Generate Test Case

Generate E2E test cases from scenario descriptions.

## Trigger

```
/e2e-test "<scenario>" [options]
```

## Parameters

- `scenario`: (required) Test scenario description in natural language
- `--page <PageName>`: (optional) Primary page object to use
- `--class <ClassName>`: (optional) Test class name (default: derived from scenario)
- `--flow`: (optional) Also generate a Flow helper

## Usage Examples

```
/e2e-test "User can login with valid credentials"
/e2e-test "User can create a new alarm" --page AlarmList --flow
/e2e-test "Settings are persisted after app restart" --class SettingsPersistenceTests
```

## Workflow

### Phase 1: Scenario Analysis

Parse the scenario description to extract:

1. **Actor**: Who is performing the action (usually "User")
2. **Action**: What action is being performed
3. **Object**: What is being acted upon
4. **Condition**: Any preconditions or context
5. **Expected Outcome**: What should happen

Example parsing:
```
"User can login with valid credentials"
→ Actor: User
→ Action: login
→ Object: (app)
→ Condition: with valid credentials
→ Expected: successful login
```

### Phase 2: Page Object Discovery

1. List existing Page Objects in `Pages/` directory
2. Match scenario keywords to Page Objects
3. If `--page` specified, use that as primary
4. Suggest additional pages that might be needed

### Phase 3: Test Structure Generation

Generate test following Given-When-Then pattern:

```swift
func test_scenario_expectedOutcome() {
    // Given: Setup preconditions

    // When: Execute action

    // Then: Verify outcome
}
```

### Phase 4: Flow Generation (if --flow)

If scenario involves multiple screens/steps, generate a Flow helper:

```swift
final class LoginFlow {
    func completeLogin(username: String, password: String) -> HomePage {
        // Multi-step flow logic
    }
}
```

## Test Naming Convention

```
test_{action}_{outcome}

Examples:
- test_login_succeeds_withValidCredentials
- test_login_fails_withInvalidPassword
- test_createAlarm_addsToList
- test_deleteAlarm_removesFromList
```

## Output Structure

### Test File

Location: `{Project}UITests/Sources/Tests/{Feature}Tests.swift`

```swift
final class LoginTests: BaseUITest {

    // MARK: - Properties
    private lazy var loginPage = LoginPage(app: app)

    // MARK: - Configuration
    override var shouldSkipOnboarding: Bool { false }

    // MARK: - Tests

    /// User can login with valid credentials
    ///
    /// **Precondition**: User is on login screen
    /// **Steps**:
    ///   1. Enter valid username
    ///   2. Enter valid password
    ///   3. Tap login button
    /// **Expected**: User is navigated to home screen
    func test_login_succeeds_withValidCredentials() {
        // Given
        XCTAssertTrue(loginPage.verifyScreenDisplayed())
        captureScreenshot(name: "01_login_screen")

        // When
        loginPage
            .enterUsername(TestData.validUsername)
            .enterPassword(TestData.validPassword)
            .tapSubmit()

        // Then
        let homePage = HomePage(app: app)
        XCTAssertTrue(homePage.verifyScreenDisplayed())
        captureScreenshot(name: "02_home_screen")
    }
}

extension LoginTests {
    enum TestData {
        static let validUsername = "testuser"
        static let validPassword = "password123"
    }
}
```

### Flow File (if --flow)

Location: `{Project}UITests/Sources/Flows/{Feature}Flow.swift`

## Interactive Mode

If scenario is ambiguous, ask clarifying questions:

1. "What screen does this test start on?"
2. "What elements need to be interacted with?"
3. "What indicates success/failure?"
4. "Are there any preconditions?"

## Scenario Templates

### CRUD Operations

```
"User can create a {item}"
"User can read/view {item}"
"User can update/edit {item}"
"User can delete {item}"
```

### Authentication

```
"User can login with {credentials}"
"User can logout"
"User can register with {details}"
"User can reset password"
```

### Navigation

```
"User can navigate to {screen}"
"User can go back from {screen}"
"User can switch between {tabs}"
```

### Settings

```
"User can change {setting}"
"Settings persist after {action}"
"User can toggle {feature}"
```

## Error Handling

- Missing Page Object: Suggest running `/e2e-pom` first
- Ambiguous scenario: Enter interactive mode
- Existing test file: Ask to add to existing or create new

---

## Agent Reference

This command uses the `ios-test-writer` agent.

See: `.claude/agents/ios-test-writer.md`
