# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2025-01-30

### Added (Android Support - Phase 2)

#### Espresso Templates
- `BasePage.kt.template` - Base class for View-based Page Objects
- `BaseUITest.kt.template` - Base class for Espresso tests
- `PageObject.kt.template` - Page Object template
- `Flow.kt.template` - User journey flow template
- `TestCase.kt.template` - Test case template

#### Compose Templates
- `ComposeBasePage.kt.template` - Base class for Compose Page Objects
- `ComposeBaseUITest.kt.template` - Base class for Compose tests
- `ComposePageObject.kt.template` - Compose Page Object template
- `ComposeTestUtils.kt.template` - Compose testing utilities
- `ComposeTestCase.kt.template` - Compose test case template

#### Support Utilities
- `ViewWait.kt.template` - Espresso wait utilities
- `ScrollUtils.kt.template` - Scroll helpers
- `SystemHelper.kt.template` - UI Automator helpers for system interactions
- `TestDataManager.kt.template` - Test data management
- `ScreenshotHelper.kt.template` - Screenshot capture utility

#### CI Integration
- `github-actions.yml.template` - GitHub Actions workflow for Android
- `build.gradle.kts.template` - Gradle configuration for Android tests

#### Agents
- `android-pom-generator.md` - Android Page Object generation logic
- `android-test-writer.md` - Android test writing logic
- `android-ci-builder.md` - Android CI configuration builder

#### Documentation
- `android-guide.md` - Comprehensive Android E2E testing guide
- `espresso-vs-uiautomator.md` - When to use each framework
- `compose-testing.md` - Jetpack Compose testing guide

### Changed
- Updated `/e2e-init` command with Android support
- Updated `/e2e-pom` command with Android/Compose detection
- Updated `/e2e-test` command for Android test generation
- Updated `/e2e-ci` command for Android CI workflows
- Updated `/e2e-guide` command with Android guidance
- Updated README with Android documentation
- Unified API naming across platforms (safeTap, scrollIntoView)

## [0.1.0] - 2025-01-30

### Added (iOS Support - Phase 1)
- `/e2e-init` command for project initialization
- `/e2e-pom` command for Page Object generation
- `/e2e-test` command for test case creation
- `/e2e-ci` command for CI workflow generation
- `/e2e-guide` command for interactive guidance
- iOS XCUITest templates
- BaseUITest and UITestConfiguration templates
- XCUIElement extensions (Wait, Scroll)
- SmartWait utility
- Getting Started documentation
- iOS Guide documentation
- POM Patterns documentation
- Sample iOS project
- Initial project structure
- README documentation
- MIT License
- Claude Code Extension settings
