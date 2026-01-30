# Getting Started

This guide will help you set up E2E testing in your mobile project.

## Prerequisites

### iOS
- Xcode 15.0+
- iOS 17.0+ deployment target
- Swift 5.9+

### Android (Coming Soon)
- Android Studio
- API 26+
- Kotlin

## Quick Start

### Step 1: Clone the Extension

```bash
git clone https://github.com/user/mobile-e2e-agents.git
```

### Step 2: Configure Claude Code

Add to your project's Claude Code settings or reference the extension:

```json
{
  "extensions": ["/path/to/mobile-e2e-agents"]
}
```

### Step 3: Initialize E2E Structure

In your project directory:

```bash
/e2e-init
```

This creates:
- UITest target structure
- Base test classes
- Support utilities

### Step 4: Generate Your First Page Object

```bash
/e2e-pom Login
```

Or analyze an existing view:

```bash
/e2e-pom Login --view-file Sources/Features/Login/LoginView.swift
```

### Step 5: Create a Test

```bash
/e2e-test "User can login with valid credentials"
```

### Step 6: Run Tests

#### From Xcode
Press `Cmd + U` or use Test Navigator.

#### From Command Line
```bash
xcodebuild test \
  -workspace YourApp.xcworkspace \
  -scheme YourAppUITests \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro'
```

## Project Structure

After initialization, your project will have:

```
YourAppUITests/
├── Sources/
│   ├── Support/
│   │   ├── BaseUITest.swift
│   │   ├── UITestConfiguration.swift
│   │   └── ... (utilities)
│   ├── Pages/
│   │   └── LoginPage.swift
│   ├── Flows/
│   │   └── LoginFlow.swift
│   └── Tests/
│       └── LoginTests.swift
└── README.md
```

## Next Steps

1. **Add Accessibility Identifiers**
   - Add identifiers to your app's UI elements
   - Follow the naming convention: `{screen}_{type}_{name}`

2. **Generate More Page Objects**
   - Run `/e2e-pom` for each screen you want to test

3. **Write Test Scenarios**
   - Use `/e2e-test` to generate test cases
   - Follow Given-When-Then pattern

4. **Set Up CI**
   - Run `/e2e-ci` to generate GitHub Actions workflow

## Common Commands

| Command | Description |
|---------|-------------|
| `/e2e-init` | Initialize E2E test structure |
| `/e2e-pom <Screen>` | Generate Page Object |
| `/e2e-test "<scenario>"` | Generate test case |
| `/e2e-ci` | Generate CI workflow |
| `/e2e-guide` | Interactive help |

## Troubleshooting

### Tests can't find elements

1. Verify accessibility identifiers are set in your app
2. Use Accessibility Inspector to check identifiers
3. Print view hierarchy: `print(app.debugDescription)`

### Tests are flaky

1. Use explicit waits instead of `sleep()`
2. Enable test isolation (reset state between tests)
3. Mock network responses for consistency

### Build errors after initialization

1. Ensure UITest target is properly linked to app
2. Check scheme configuration
3. Verify deployment target matches

## Resources

- [iOS Guide](./ios-guide.md) - Detailed iOS testing guide
- [POM Patterns](./pom-patterns.md) - Page Object Model best practices
- [Test Isolation](./test-isolation.md) - Strategies for test isolation
- [CI Integration](./ci-integration.md) - CI/CD setup guide
