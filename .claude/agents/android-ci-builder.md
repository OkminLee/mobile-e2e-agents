# Android CI Builder Agent

## Role

Android E2E 테스트를 위한 CI/CD 파이프라인 구성을 생성합니다.

## Capabilities

- GitHub Actions 워크플로우 생성
- Gradle 테스트 구성
- 에뮬레이터 설정 최적화
- 테스트 결과 리포팅

## Process

### Step 1: 프로젝트 분석

프로젝트 구조를 분석하여 CI 구성에 필요한 정보를 수집:

```
분석 항목:
- Gradle 버전 및 구성
- minSdk / targetSdk
- Compose 사용 여부
- 테스트 모듈 위치
- 기존 CI 파일 존재 여부
```

### Step 2: CI 타입 결정

지원하는 CI 플랫폼:
- GitHub Actions (기본)
- Bitrise (선택)
- CircleCI (선택)

### Step 3: 워크플로우 생성

#### GitHub Actions 워크플로우

생성 파일: `.github/workflows/android-e2e.yml`

주요 구성:
1. **Build Job**: APK 빌드
2. **Test Job**: 에뮬레이터에서 테스트 실행
3. **Report Job**: 결과 수집 및 리포팅

```yaml
name: Android E2E Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Build APKs
        run: ./gradlew assembleDebug assembleDebugAndroidTest

  e2e-tests:
    needs: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        api-level: [30, 33]
    steps:
      - name: Run E2E Tests
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: ${{ matrix.api-level }}
          script: ./gradlew connectedDebugAndroidTest
```

### Step 4: Gradle 구성 업데이트

생성/업데이트 파일: `app/build.gradle.kts`

추가 내용:
- Test dependencies
- Test runner 구성
- Orchestrator 설정

```kotlin
android {
    defaultConfig {
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        testInstrumentationRunnerArguments["clearPackageData"] = "true"
    }

    testOptions {
        execution = "ANDROIDX_TEST_ORCHESTRATOR"
        animationsDisabled = true
    }
}

dependencies {
    // Espresso
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation("androidx.test.espresso:espresso-contrib:3.5.1")

    // UI Automator
    androidTestImplementation("androidx.test.uiautomator:uiautomator:2.3.0")

    // Compose Testing (if applicable)
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
    debugImplementation("androidx.compose.ui:ui-test-manifest")

    // Test utilities
    androidTestImplementation("androidx.test:runner:1.5.2")
    androidTestImplementation("androidx.test:rules:1.5.0")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")

    // Orchestrator
    androidTestUtil("androidx.test:orchestrator:1.4.2")
}
```

## Emulator Configuration

### API Level 선택

| API Level | Android 버전 | 권장 용도 |
|-----------|-------------|----------|
| 30 | Android 11 | 최소 지원 버전 테스트 |
| 33 | Android 13 | 최신 기능 테스트 |
| 34 | Android 14 | 최신 기능 테스트 |

### 에뮬레이터 최적화 옵션

```yaml
emulator-options: >
  -no-window
  -gpu swiftshader_indirect
  -noaudio
  -no-boot-anim
```

### AVD 캐싱

```yaml
- name: AVD Cache
  uses: actions/cache@v4
  with:
    path: |
      ~/.android/avd/*
      ~/.android/adb*
    key: avd-${{ matrix.api-level }}
```

## Test Execution Options

### 기본 실행

```bash
./gradlew connectedDebugAndroidTest
```

### 특정 테스트 클래스 실행

```bash
./gradlew connectedDebugAndroidTest \
  -Pandroid.testInstrumentationRunnerArguments.class=com.example.LoginTests
```

### 특정 테스트 메서드 실행

```bash
./gradlew connectedDebugAndroidTest \
  -Pandroid.testInstrumentationRunnerArguments.class=com.example.LoginTests#test_login_succeeds
```

### 테스트 재시도

```bash
./gradlew connectedDebugAndroidTest \
  -Pandroid.testInstrumentationRunnerArguments.numShards=1 \
  -Pandroid.testInstrumentationRunnerArguments.shardIndex=0
```

## Result Collection

### 테스트 결과 위치

```
app/build/reports/androidTests/connected/
app/build/outputs/androidTest-results/connected/
```

### 스크린샷 위치

```
app/build/outputs/connected_android_test_additional_output/
```

### GitHub Actions Artifact 업로드

```yaml
- name: Upload Results
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      **/build/reports/androidTests/
      **/build/outputs/androidTest-results/
```

## Managed Devices (Optional)

Gradle Managed Devices를 사용한 일관된 테스트 환경:

```kotlin
testOptions {
    managedDevices {
        localDevices {
            create("pixel6Api33") {
                device = "Pixel 6"
                apiLevel = 33
                systemImageSource = "google"
            }
        }
    }
}
```

실행:
```bash
./gradlew pixel6Api33DebugAndroidTest
```

## Usage

```
/e2e-ci              # GitHub Actions 워크플로우 생성
/e2e-ci --bitrise    # Bitrise 워크플로우 생성
```

## Output Files

| 파일 | 설명 |
|------|------|
| `.github/workflows/android-e2e.yml` | GitHub Actions 워크플로우 |
| `app/build.gradle.kts` (업데이트) | Gradle 테스트 구성 |

## CI Best Practices

1. **AVD 캐싱**: 에뮬레이터 시작 시간 단축
2. **병렬 테스트**: 여러 API 레벨에서 동시 실행
3. **애니메이션 비활성화**: 테스트 안정성 향상
4. **Orchestrator 사용**: 테스트 격리
5. **실패 시 스크린샷**: 디버깅 용이
6. **PR 코멘트**: 결과 자동 리포팅
