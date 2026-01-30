# iOS Sample App - E2E Testing Example

This sample demonstrates how to use `mobile-e2e-agents` in a real iOS project.

## Structure

```
ios-sample-app/
├── SampleApp/                    # Main app target
│   ├── Features/
│   │   ├── Login/
│   │   │   ├── LoginView.swift
│   │   │   └── LoginViewModel.swift
│   │   └── Home/
│   │       ├── HomeView.swift
│   │       └── HomeViewModel.swift
│   └── App.swift
│
└── SampleAppUITests/             # UI test target
    └── Sources/
        ├── Support/
        │   ├── BaseUITest.swift
        │   └── UITestConfiguration.swift
        ├── Pages/
        │   ├── LoginPage.swift
        │   └── HomePage.swift
        └── Tests/
            └── LoginTests.swift
```

## Running the Example

### Prerequisites

1. Xcode 15.0+
2. iOS 17.0+ Simulator

### Steps

1. Open `SampleApp.xcodeproj` in Xcode
2. Select `SampleAppUITests` scheme
3. Press `Cmd + U` to run tests

## Test Cases

### LoginTests

| Test | Description |
|------|-------------|
| `test_login_succeeds_withValidCredentials` | Valid login navigates to home |
| `test_login_fails_withInvalidPassword` | Invalid password shows error |
| `test_login_showsValidation_withEmptyFields` | Empty fields show validation |

## How This Was Created

1. **Initialize E2E structure**
   ```
   /e2e-init
   ```

2. **Generate Login page object**
   ```
   /e2e-pom Login --view-file SampleApp/Features/Login/LoginView.swift
   ```

3. **Generate Home page object**
   ```
   /e2e-pom Home
   ```

4. **Generate test cases**
   ```
   /e2e-test "User can login with valid credentials"
   /e2e-test "Login fails with invalid password"
   ```

## Accessibility Identifiers

The sample app includes proper accessibility identifiers:

```swift
// LoginView.swift
TextField("Username", text: $username)
    .accessibilityIdentifier("login_textField_username")

SecureField("Password", text: $password)
    .accessibilityIdentifier("login_textField_password")

Button("Login") { ... }
    .accessibilityIdentifier("login_button_submit")
```

## Learn More

- [Getting Started Guide](../../docs/getting-started.md)
- [iOS Testing Guide](../../docs/ios-guide.md)
- [POM Patterns](../../docs/pom-patterns.md)
