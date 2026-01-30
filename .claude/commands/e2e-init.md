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

### Phase 2: Structure Generation (iOS)

Create the following structure:

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

### Phase 3: Template Processing

For each template file:
1. Read from `templates/ios/xcuitest/` or `templates/ios/support/`
2. Replace template variables:
   - `{{PROJECT_NAME}}` → Detected project name
   - `{{BUNDLE_IDENTIFIER}}` → App bundle identifier + ".UITests"
3. Write to target location

### Phase 4: Xcode Integration (iOS)

Provide instructions for:
1. Adding UITest target in Xcode (if not exists)
2. Adding generated files to target
3. Setting up scheme for testing

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | Project name | `MyApp` |
| `{{BUNDLE_IDENTIFIER}}` | App bundle ID | `com.example.myapp` |
| `{{MIN_IOS_VERSION}}` | Minimum iOS version | `17.0` |

## Output

After running `/e2e-init`:

1. **Files created** in `{ProjectName}UITests/`
2. **README** with setup instructions
3. **Guidance** on next steps

## Next Steps After Init

1. Add UITest target in Xcode (if needed)
2. Run `/e2e-pom <ScreenName>` to generate page objects
3. Run `/e2e-test "<scenario>"` to generate test cases

## Error Handling

- If project type cannot be detected: Ask user to specify
- If UITest target exists: Ask to overwrite or skip
- If templates missing: Show error with recovery steps

---

## Implementation Notes

When implementing this command:

1. Use `Glob` to find project files
2. Use `Read` to analyze existing structure
3. Use `Write` to create files from templates
4. Provide clear progress feedback
5. Handle errors gracefully with recovery suggestions
