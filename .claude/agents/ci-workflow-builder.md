# CI Workflow Builder Agent

Internal agent for generating CI/CD configurations.

## Purpose

Generate CI workflow files for E2E test automation.

## Input

```yaml
platform: string           # ios or android
provider: string           # github, gitlab, bitrise
projectName: string        # Project name
workspaceName: string      # Workspace file name
schemeName: string         # Test scheme name
testTargetName: string     # UI test target name
includeFastlane: bool      # Generate Fastlane config
includeSlack: bool         # Include Slack notifications
parallelWorkers: int       # Number of parallel workers (0 = disabled)
```

## Processing Steps

### Step 1: Validate Input

- Verify platform is supported
- Check provider is implemented
- Validate project structure exists

### Step 2: Generate Main Workflow

Select template based on provider:
- GitHub: `.github/workflows/ui_tests.yml`
- GitLab: `.gitlab-ci.yml`
- Bitrise: `bitrise.yml`

### Step 3: Add Optional Components

Based on flags, add:
- Fastlane integration
- Slack notifications
- Parallel testing configuration
- Report generation

### Step 4: Generate Support Files

- Fastlane `Fastfile` (if requested)
- Report generation script
- README with instructions

## Output Templates

### GitHub Actions - Basic

```yaml
name: UI Tests

on:
  workflow_dispatch:
    inputs:
      test_target:
        description: 'Test target to run'
        required: false
        default: '{{TEST_TARGET}}'
      device:
        description: 'Simulator device'
        required: false
        default: 'iPhone 16 Pro'

concurrency:
  group: ui-tests-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ui-tests:
    runs-on: macos-14
    timeout-minutes: 30

    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Select Xcode
      run: sudo xcode-select -s /Applications/Xcode_15.4.app

    - name: Run UI Tests
      run: |
        xcodebuild test \
          -workspace {{WORKSPACE}}.xcworkspace \
          -scheme {{SCHEME}} \
          -destination 'platform=iOS Simulator,name=${{ inputs.device }}' \
          -only-testing:${{ inputs.test_target }} \
          -resultBundlePath test_results/results.xcresult \
          | xcpretty

    - name: Upload Test Results
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: test-results
        path: test_results/
        retention-days: 7
```

### GitHub Actions - With Fastlane

```yaml
    - name: Bundle Install
      run: bundle install

    - name: Run UI Tests
      run: |
        bundle exec fastlane ui_tests \
          test_target:${{ inputs.test_target }} \
          device:"${{ inputs.device }}"
```

### GitHub Actions - With Parallel

```yaml
    - name: Run UI Tests (Parallel)
      run: |
        xcodebuild test \
          -workspace {{WORKSPACE}}.xcworkspace \
          -scheme {{SCHEME}} \
          -destination 'platform=iOS Simulator,name=${{ inputs.device }}' \
          -only-testing:${{ inputs.test_target }} \
          -parallel-testing-enabled YES \
          -parallel-testing-worker-count {{WORKERS}} \
          -retry-tests-on-failure \
          -resultBundlePath test_results/results.xcresult
```

### GitHub Actions - With Slack

```yaml
    - name: Notify Slack - Started
      if: ${{ inputs.slack_notify == 'true' }}
      uses: slackapi/slack-github-action@v1.25.0
      with:
        channel-id: ${{ secrets.SLACK_CHANNEL }}
        payload: |
          {
            "text": ":microscope: UI Tests Started",
            "blocks": [
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*Branch:* `${{ github.ref_name }}`\n*Target:* `${{ inputs.test_target }}`"
                }
              }
            ]
          }
      env:
        SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

    - name: Notify Slack - Completed
      if: always()
      uses: slackapi/slack-github-action@v1.25.0
      with:
        channel-id: ${{ secrets.SLACK_CHANNEL }}
        payload: |
          {
            "text": "UI Tests ${{ job.status == 'success' && ':white_check_mark: Passed' || ':x: Failed' }}",
            "blocks": [
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*Status:* ${{ job.status }}\n*Duration:* ${{ github.run_duration }}s"
                }
              }
            ]
          }
      env:
        SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Fastlane - ui_tests Lane

```ruby
desc "Run UI Tests"
lane :ui_tests do |options|
  test_target = options[:test_target] || "{{TEST_TARGET}}"
  device = options[:device] || "iPhone 16 Pro"
  parallel = options[:parallel] || false
  workers = options[:workers] || 2

  # Build xcargs
  xcargs = ""
  if parallel
    xcargs = "-parallel-testing-enabled YES -parallel-testing-worker-count #{workers}"
  end

  begin
    scan(
      workspace: "{{WORKSPACE}}.xcworkspace",
      scheme: "{{SCHEME}}",
      devices: [device],
      only_testing: [test_target],
      result_bundle: true,
      output_directory: "./test_results",
      xcargs: xcargs,
      fail_build: true
    )
  rescue => e
    UI.error("Tests failed: #{e.message}")
    raise e
  ensure
    # Always collect artifacts
    if File.exist?("./test_results")
      UI.message("Test results available at ./test_results")
    end
  end
end
```

## Output

```yaml
workflowFile: string       # Path to generated workflow file
fastlaneFile: string?      # Path to Fastfile (if requested)
reportScript: string?      # Path to report script
instructions: string       # Setup instructions
secretsRequired: list      # Required secrets to configure
```

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | Project name | `MyApp` |
| `{{WORKSPACE}}` | Workspace name | `MyApp` |
| `{{SCHEME}}` | Build scheme | `MyApp` |
| `{{TEST_TARGET}}` | UI test target | `MyAppUITests` |
| `{{WORKERS}}` | Parallel workers | `4` |

## Error Handling

- Missing project files: Return error with guidance
- Invalid provider: List supported providers
- Existing files: Ask before overwriting
