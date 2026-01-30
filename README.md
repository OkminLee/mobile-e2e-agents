# mobile-e2e-agents

> Claude Code Extension for Cross-Platform Mobile E2E Testing

A comprehensive toolkit for automating E2E test infrastructure setup, Page Object Model generation, and CI/CD integration for iOS and Android mobile applications.

## Features

- **E2E Scaffolding**: Initialize E2E test structure with a single command
- **POM Generation**: Auto-generate Page Object Model files from view analysis
- **Test Case Templates**: Create test cases with proper structure and patterns
- **CI Integration**: Generate GitHub Actions workflows and Fastlane configurations
- **Cross-Platform**: Support for iOS (XCUITest) and Android (Espresso/UI Automator)

## Installation

### As Claude Code Extension

1. Clone this repository:
```bash
git clone https://github.com/user/mobile-e2e-agents.git
```

2. Add to your project's `.claude/` directory or reference in settings:
```json
{
  "extensions": [
    "/path/to/mobile-e2e-agents"
  ]
}
```

## Quick Start

### Initialize E2E Test Structure

```bash
/e2e-init
```

This creates:
- `{Project}UITests/` directory
- `BaseUITest.swift`
- `UITestConfiguration.swift`
- Support utilities

### Generate Page Object

```bash
/e2e-pom LoginScreen
```

Or with view file analysis:
```bash
/e2e-pom LoginScreen --view-file Sources/Features/Login/LoginView.swift
```

### Create Test Case

```bash
/e2e-test "User can login with valid credentials"
```

### Generate CI Workflow

```bash
/e2e-ci
```

## Commands Reference

| Command | Description |
|---------|-------------|
| `/e2e-init` | Initialize E2E test structure in your project |
| `/e2e-pom <ScreenName>` | Generate Page Object for a screen |
| `/e2e-test "<scenario>"` | Create test case from scenario description |
| `/e2e-ci` | Generate CI/CD workflow files |
| `/e2e-guide` | Interactive guide for test scenario design |

## Directory Structure

```
mobile-e2e-agents/
├── .claude/
│   ├── commands/           # User-facing slash commands
│   │   ├── e2e-init.md
│   │   ├── e2e-pom.md
│   │   ├── e2e-test.md
│   │   ├── e2e-ci.md
│   │   └── e2e-guide.md
│   │
│   └── agents/             # Internal processing agents
│       ├── ios-pom-generator.md
│       ├── ios-test-writer.md
│       └── ci-workflow-builder.md
│
├── templates/
│   ├── ios/
│   │   ├── xcuitest/       # XCUITest templates
│   │   ├── support/        # Utility extensions
│   │   └── ci/             # CI configuration templates
│   │
│   └── android/            # Android templates (Phase 2)
│
├── examples/               # Sample projects
│   └── ios-sample-app/
│
└── docs/                   # Documentation
    ├── getting-started.md
    ├── ios-guide.md
    └── pom-patterns.md
```

## Generated Code Structure

When you run `/e2e-init`, the following structure is created:

```
{Project}UITests/
├── Sources/
│   ├── Support/
│   │   ├── BaseUITest.swift
│   │   ├── UITestConfiguration.swift
│   │   ├── XCUIElement+Wait.swift
│   │   ├── XCUIElement+Scroll.swift
│   │   └── SmartWait.swift
│   │
│   ├── Pages/
│   │   └── {ScreenName}Page.swift
│   │
│   ├── Flows/
│   │   └── {Feature}Flow.swift
│   │
│   └── Tests/
│       └── {Feature}Tests.swift
│
└── README.md
```

## Platform Support

### iOS (Phase 1 - Available)
- XCUITest framework
- Swift 5.9+
- Xcode 15.0+
- iOS 17.0+

### Android (Phase 2 - Coming Soon)
- Espresso + UI Automator
- Kotlin
- Android Studio
- API 26+

## Accessibility Identifier Convention

For reliable element identification, use this naming convention:

```swift
"{screenName}_{elementType}_{elementName}"

// Examples:
"login_textField_username"
"login_button_submit"
"home_cell_alarmItem"
"editor_picker_time"
```

## CI/CD Integration

The `/e2e-ci` command generates:

- **GitHub Actions workflow** (`.github/workflows/ui_tests.yml`)
- **Fastlane configuration** (`fastlane/Fastfile`)
- **Test report generator** (optional)
- **Slack notification setup** (optional)

## Contributing

Contributions are welcome! Please read our contributing guidelines before submitting PRs.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Acknowledgments

- Inspired by E2E testing patterns from production mobile applications
- Built with Claude Code Extension framework
