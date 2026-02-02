# /e2e-ci - Generate CI Workflow

Generate CI/CD workflow configuration for E2E tests.

## Trigger

```
/e2e-ci [options]
```

## Parameters

- `--platform <ios|android>`: Target platform (auto-detected if not specified)
- `--provider <github|gitlab|bitrise>`: CI provider (default: github)
- `--fastlane`: Include Fastlane configuration (iOS only)
- `--slack`: Include Slack notifications
- `--parallel <n>`: Number of parallel workers
- `--api-level <level>`: Android API level (default: 33, Android only)

## Usage Examples

```
/e2e-ci
/e2e-ci --fastlane --slack
/e2e-ci --parallel 4
/e2e-ci --provider gitlab
/e2e-ci --platform android --api-level 30,33
```

## Workflow

### Phase 1: Project Analysis

1. Detect project structure
2. Find test target name
3. Identify existing CI configuration
4. Determine scheme and workspace names

### Phase 2: Workflow Generation

Generate appropriate CI configuration based on provider.

### Phase 3: Optional Components

Add based on flags:
- Fastlane integration
- Slack notifications
- Parallel testing
- Report generation

## Output Files

### GitHub Actions (default)

Location: `.github/workflows/ui_tests.yml`

### Fastlane (if --fastlane)

Location: `fastlane/Fastfile` (ui_tests lane added)

### Report Script

Location: `scripts/generate_e2e_report.py`

## Generated Workflow Features

### Triggers

```yaml
on:
  workflow_dispatch:
    inputs:
      test_target:
        description: 'Test target'
        default: '{ProjectName}UITests'
      device:
        description: 'Simulator device'
        default: 'iPhone 16 Pro'
```

### Jobs

1. **Setup**: Checkout, install dependencies
2. **Build**: Build for testing
3. **Test**: Run UI tests
4. **Report**: Generate report (if tests run)
5. **Notify**: Send Slack notification (if configured)

### Artifacts

- Test results (xcresult)
- Screenshots (on failure)
- Generated report (HTML/PDF)

## Templates

### Basic Workflow

```yaml
name: UI Tests

on:
  workflow_dispatch:
  pull_request:
    paths:
      - '**/*.swift'
      - '*.xcodeproj/**'

jobs:
  ui-tests:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run UI Tests
        run: |
          xcodebuild test \
            -workspace {{WORKSPACE}}.xcworkspace \
            -scheme {{SCHEME}} \
            -destination 'platform=iOS Simulator,name=iPhone 16 Pro' \
            -resultBundlePath test_results/results.xcresult

      - name: Upload Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test_results/
```

### With Fastlane

```yaml
- name: Run UI Tests
  run: bundle exec fastlane ui_tests
```

### With Parallel Testing

```yaml
- name: Run UI Tests (Parallel)
  run: |
    xcodebuild test \
      -parallel-testing-enabled YES \
      -parallel-testing-worker-count {{WORKERS}} \
      ...
```

### With Slack Notification

```yaml
- name: Notify Slack
  if: always()
  uses: slackapi/slack-github-action@v1.25.0
  with:
    channel-id: ${{ secrets.SLACK_CHANNEL }}
    payload: |
      {
        "text": "UI Tests ${{ job.status }}",
        ...
      }
```

## Fastlane Lane

```ruby
lane :ui_tests do |options|
  test_target = options[:test_target] || "{{PROJECT_NAME}}UITests"
  device = options[:device] || "iPhone 16 Pro"

  scan(
    workspace: "{{WORKSPACE}}.xcworkspace",
    scheme: "{{SCHEME}}",
    devices: [device],
    only_testing: [test_target],
    result_bundle: true,
    output_directory: "./test_results"
  )
end
```

## Report Script

Python script for generating HTML/PDF reports from xcresult:

- Extract test results
- Map screenshots to test cases
- Generate HTML report
- Convert to PDF (optional)

## Error Handling

- Missing workspace/scheme: Prompt user
- Existing workflow: Ask to overwrite or merge
- Missing dependencies: Provide installation instructions

---

## Android-Specific Configuration

### Emulator Setup

```yaml
- name: Run E2E Tests
  uses: reactivecircus/android-emulator-runner@v2
  with:
    api-level: 33
    target: google_apis
    arch: x86_64
    profile: pixel_6
    script: ./gradlew connectedDebugAndroidTest
```

### AVD Caching

```yaml
- name: AVD Cache
  uses: actions/cache@v4
  with:
    path: |
      ~/.android/avd/*
      ~/.android/adb*
    key: avd-${{ matrix.api-level }}
```

### Android Gradle Configuration

```kotlin
android {
    testOptions {
        execution = "ANDROIDX_TEST_ORCHESTRATOR"
        animationsDisabled = true
    }
}
```

### Matrix Testing (Multiple API Levels)

```yaml
strategy:
  matrix:
    api-level: [30, 33]
```

---

## Agent Reference

This command uses platform-specific agents:

| Platform | Agent |
|----------|-------|
| iOS | `ci-workflow-builder` |
| Android | `android-ci-builder` |

See:
- `.claude/agents/ci-workflow-builder.md`
- `.claude/agents/android-ci-builder.md`
