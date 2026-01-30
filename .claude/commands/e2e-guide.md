# /e2e-guide - Interactive Test Scenario Guide

Interactive guide for designing and writing E2E test scenarios.

## Trigger

```
/e2e-guide [topic]
```

## Parameters

- `topic`: (optional) Specific topic to get guidance on
  - `pom` - Page Object Model patterns
  - `isolation` - Test isolation strategies
  - `wait` - Waiting and synchronization
  - `ci` - CI/CD integration
  - `debug` - Debugging test failures

## Usage Examples

```
/e2e-guide
/e2e-guide pom
/e2e-guide isolation
```

## Topics

### General Guide (no topic)

Interactive walkthrough:

1. **Project Assessment**
   - What platform are you testing? (iOS/Android)
   - Do you have existing E2E tests?
   - What's your main testing goal?

2. **Architecture Recommendation**
   - Page Object Model explanation
   - Flow pattern for complex scenarios
   - Test organization best practices

3. **Next Steps**
   - Suggest relevant commands (`/e2e-init`, `/e2e-pom`, etc.)

### POM Guide (`/e2e-guide pom`)

Page Object Model patterns and best practices:

```markdown
## Page Object Model (POM)

### What is POM?
A design pattern that creates an object repository for UI elements,
separating test logic from page structure.

### Benefits
- Maintainability: UI changes only affect Page classes
- Reusability: Same page object used across tests
- Readability: Tests read like user stories

### Structure
```
Pages/
├── LoginPage.swift      # One page = one screen
├── HomePage.swift
└── SettingsPage.swift
```

### Best Practices

1. **One Page = One Screen**
   - Each Page class represents one app screen
   - Complex screens can be split into sections

2. **Element Encapsulation**
   - Elements are private or computed properties
   - External access through action methods

3. **Fluent Interface**
   - Methods return `Self` or next page for chaining
   - Example: `page.enterText().tapSubmit()`

4. **Verification Methods**
   - Include `verify*` methods for assertions
   - Return Bool for flexible usage

### Example

```swift
final class LoginPage: BasePage {

    // MARK: - Elements (private)
    private var usernameField: XCUIElement {
        app.textFields["login_textField_username"]
    }

    // MARK: - Actions (public, fluent)
    @discardableResult
    func enterUsername(_ text: String) -> Self {
        usernameField.safeClearAndTypeText(text)
        return self
    }

    @discardableResult
    func tapSubmit() -> HomePage {
        submitButton.safeTap()
        return HomePage(app: app)
    }

    // MARK: - Verification
    func verifyScreenDisplayed() -> Bool {
        usernameField.waitForExistence(timeout: 5)
    }
}
```
```

### Isolation Guide (`/e2e-guide isolation`)

Test isolation strategies:

```markdown
## Test Isolation

### Why Isolation Matters
- Tests should not depend on each other
- Each test should start from a known state
- Parallel execution requires isolation

### Strategies

1. **App State Reset**
   - Use `-ResetState` launch argument
   - Clear UserDefaults, databases, caches

2. **In-Memory Database**
   - Use in-memory storage during tests
   - Each test gets fresh database

3. **Test Data Seeding**
   - Seed known data before each test
   - Use `-SetupTestData` argument

4. **Dependency Injection**
   - Mock network responses
   - Control time/date for deterministic tests

### Implementation

```swift
// UITestConfiguration
configuration
    .resetState()
    .setupTestData()
    .setMockNetworkDelay(100)
```

```swift
// App side detection
if ProcessInfo.processInfo.arguments.contains("-ResetState") {
    clearAllData()
}
```
```

### Wait Guide (`/e2e-guide wait`)

Waiting and synchronization:

```markdown
## Waiting Strategies

### Anti-Patterns
- ❌ `sleep(5)` - Fixed delays are flaky
- ❌ Arbitrary timeouts - Too short = flaky, too long = slow

### Best Practices

1. **Element-Based Waiting**
   ```swift
   element.waitForExistence(timeout: 5)
   element.waitToBeHittable(timeout: 5)
   ```

2. **Predicate-Based Waiting**
   ```swift
   let predicate = NSPredicate(format: "exists == true")
   let expectation = XCTNSPredicateExpectation(predicate: predicate, object: element)
   XCTWaiter.wait(for: [expectation], timeout: 5)
   ```

3. **Smart Wait Utilities**
   ```swift
   SmartWait.forLoadingToComplete(in: app)
   SmartWait.until { page.isDataLoaded }
   ```

### Timeout Guidelines
- Short (2s): UI response, button state
- Medium (5s): Screen transitions, data loading
- Long (10s): Network operations, complex flows
- Extra Long (30s): File operations, heavy processing
```

### CI Guide (`/e2e-guide ci`)

CI/CD integration:

```markdown
## CI/CD Integration

### GitHub Actions Setup
1. Run `/e2e-ci` to generate workflow
2. Configure secrets (if needed)
3. Set up Slack notifications (optional)

### Best Practices

1. **Parallel Testing**
   - Split tests across multiple simulators
   - Use test sharding for large suites

2. **Retry on Failure**
   - Use `-retry-tests-on-failure` flag
   - Quarantine flaky tests

3. **Artifact Collection**
   - Screenshots on failure
   - xcresult bundle for debugging
   - Test report generation

### Example Workflow
```yaml
- name: Run UI Tests
  run: |
    xcodebuild test \
      -workspace App.xcworkspace \
      -scheme AppUITests \
      -destination 'platform=iOS Simulator,name=iPhone 15' \
      -resultBundlePath results.xcresult
```
```

### Debug Guide (`/e2e-guide debug`)

Debugging test failures:

```markdown
## Debugging E2E Tests

### Common Issues

1. **Element Not Found**
   - Check accessibility identifier
   - Verify element is on screen
   - Print view hierarchy: `print(app.debugDescription)`

2. **Timing Issues**
   - Add appropriate waits
   - Check for animations
   - Use SmartWait utilities

3. **State Issues**
   - Verify test isolation
   - Check setup/teardown
   - Review launch arguments

### Debugging Tools

1. **View Hierarchy**
   ```swift
   print(app.debugDescription)
   ```

2. **Screenshot Capture**
   ```swift
   captureScreenshot(name: "debug_state")
   ```

3. **Accessibility Inspector**
   - Xcode > Open Developer Tool > Accessibility Inspector
   - Inspect live app for identifiers

4. **Test Recording**
   - Xcode test navigator > Record UI Test
   - Use as starting point, then refine

### Flaky Test Checklist
- [ ] Using explicit waits (not sleep)?
- [ ] Test isolated from other tests?
- [ ] Handling animations properly?
- [ ] Network calls mocked/stable?
- [ ] Date/time deterministic?
```

## Interactive Mode

When no topic specified, guide user through questions:

1. What are you trying to accomplish?
2. What challenges are you facing?
3. Provide targeted guidance and suggest commands

## Output

Provide:
- Relevant documentation
- Code examples
- Links to templates
- Suggested next commands
