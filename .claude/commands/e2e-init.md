# /e2e-init - Initialize E2E Test Structure

Initialize E2E test infrastructure in an iOS or Android project.

## Trigger

```
/e2e-init [platform]
```

## Parameters

- `platform`: (optional) Target platform - `ios` (default) or `android`

## Usage Examples

```
/e2e-init
/e2e-init ios
/e2e-init android
```

## Workflow

### Phase 1: Project Detection

1. Detect project type:
   - Look for `*.xcworkspace` or `*.xcodeproj` (iOS)
   - Look for `build.gradle` or `build.gradle.kts` (Android)

2. If iOS project detected:
   - Find main app target name
   - Check if UITest target already exists
   - Determine source file locations

3. If Android project detected:
   - Find app module
   - Check if androidTest directory exists
   - Determine package structure
   - Detect Compose usage (look for `@Composable` annotations)

### Phase 2: Structure Generation

#### iOS Structure

```
{ProjectName}UITests/
├── Sources/
│   ├── Support/
│   │   ├── BaseUITest.swift
│   │   ├── UITestConfiguration.swift
│   │   ├── XCUIElement+Wait.swift
│   │   ├── XCUIElement+Scroll.swift
│   │   └── SmartWait.swift
│   ├── Pages/
│   │   └── .gitkeep
│   ├── Flows/
│   │   └── .gitkeep
│   └── Tests/
│       └── .gitkeep
├── Info.plist
└── README.md
```

#### Android Structure

```
app/src/androidTest/java/{package}/
├── support/
│   ├── ViewWait.kt
│   ├── ScrollUtils.kt
│   ├── SystemHelper.kt
│   ├── TestDataManager.kt
│   └── ScreenshotHelper.kt
├── pages/
│   ├── BasePage.kt              # Espresso base
│   └── ComposeBasePage.kt       # Compose base (if Compose detected)
├── flows/
│   └── .gitkeep
├── tests/
│   ├── BaseUITest.kt            # Espresso base test
│   └── ComposeBaseUITest.kt     # Compose base test (if Compose detected)
└── README.md
```

### Phase 3: Template Processing

#### iOS Templates
Source: `templates/ios/xcuitest/` and `templates/ios/support/`

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | Project name | `MyApp` |
| `{{BUNDLE_IDENTIFIER}}` | App bundle ID | `com.example.myapp` |
| `{{MIN_IOS_VERSION}}` | Minimum iOS version | `17.0` |

#### Android Templates
Source: `templates/android/espresso/`, `templates/android/compose/`, and `templates/android/support/`

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PACKAGE}}` | Package name | `com.example.myapp` |
| `{{MAIN_ACTIVITY}}` | Main activity class | `MainActivity` |
| `{{COMPOSE_BOM_VERSION}}` | Compose BOM version | `2024.02.00` |

### Phase 4: Platform Integration

#### iOS - Xcode Integration
1. Adding UITest target in Xcode (if not exists)
2. Adding generated files to target
3. Setting up scheme for testing

#### Android - Gradle Integration
1. Update `app/build.gradle.kts` with test dependencies
2. Configure test runner and orchestrator
3. Add managed device configuration (optional)

## Output

After running `/e2e-init`:

1. **Files created** in appropriate test directory
2. **README** with setup instructions
3. **Guidance** on next steps

### iOS Output
- Files in `{ProjectName}UITests/`
- Instructions for Xcode target setup

### Android Output
- Files in `app/src/androidTest/java/{package}/`
- Updated `build.gradle.kts` with dependencies
- Instructions for running tests

## Next Steps After Init

### iOS
1. Add UITest target in Xcode (if needed)
2. Run `/e2e-pom <ScreenName>` to generate page objects
3. Run `/e2e-test "<scenario>"` to generate test cases

### Android
1. Sync Gradle to download dependencies
2. Run `/e2e-pom <ScreenName>` to generate page objects
3. Run `/e2e-test "<scenario>"` to generate test cases
4. Run `./gradlew connectedDebugAndroidTest` to execute

## Error Handling

- If project type cannot be detected: Ask user to specify
- If test directory exists: Ask to overwrite or skip
- If templates missing: Show error with recovery steps
- If Compose detected but not sure: Ask user to confirm

---

## Implementation Notes

When implementing this command:

1. Use `Glob` to find project files
2. Use `Read` to analyze existing structure
3. Use `Write` to create files from templates
4. Provide clear progress feedback
5. Handle errors gracefully with recovery suggestions

### Android-Specific Detection

```kotlin
// Compose detection patterns
- @Composable annotation in source files
- androidx.compose dependencies in build.gradle
- setContent { } in Activity files
```

### Template Selection Logic

```
if platform == android:
    if compose_detected:
        use espresso/* AND compose/* templates
    else:
        use espresso/* templates only
else:
    use ios/xcuitest/* templates
```
