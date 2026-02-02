# mobile-e2e-agents

> 크로스 플랫폼 모바일 E2E 테스트를 위한 Claude Code 확장

iOS 및 Android 모바일 앱의 E2E 테스트 인프라 설정, Page Object Model 생성, CI/CD 통합을 자동화하는 종합 툴킷입니다.

## 주요 기능

- **E2E 스캐폴딩**: 단일 명령어로 E2E 테스트 구조 초기화
- **POM 생성**: 뷰 분석을 통한 Page Object Model 파일 자동 생성
- **테스트 케이스 템플릿**: 적절한 구조와 패턴으로 테스트 케이스 생성
- **CI 통합**: GitHub Actions 워크플로우 및 Fastlane 설정 생성
- **크로스 플랫폼**: iOS (XCUITest) 및 Android (Espresso/UI Automator) 지원

## 설치

### Claude Code 확장으로 설치

1. 저장소 클론:
```bash
git clone https://github.com/user/mobile-e2e-agents.git
```

2. 프로젝트의 `.claude/` 디렉토리에 추가하거나 설정에서 참조:
```json
{
  "extensions": [
    "/path/to/mobile-e2e-agents"
  ]
}
```

## 빠른 시작

### E2E 테스트 구조 초기화

```bash
/e2e-init
```

생성되는 항목:
- `{Project}UITests/` 디렉토리
- `BaseUITest.swift`
- `UITestConfiguration.swift`
- 지원 유틸리티

### Page Object 생성

```bash
/e2e-pom LoginScreen
```

뷰 파일 분석과 함께:
```bash
/e2e-pom LoginScreen --view-file Sources/Features/Login/LoginView.swift
```

### 테스트 케이스 생성

```bash
/e2e-test "유효한 자격 증명으로 로그인할 수 있다"
```

### CI 워크플로우 생성

```bash
/e2e-ci
```

## 명령어 참조

| 명령어 | 설명 |
|--------|------|
| `/e2e-init` | 프로젝트에 E2E 테스트 구조 초기화 |
| `/e2e-pom <ScreenName>` | 화면에 대한 Page Object 생성 |
| `/e2e-test "<시나리오>"` | 시나리오 설명으로 테스트 케이스 생성 |
| `/e2e-ci` | CI/CD 워크플로우 파일 생성 |
| `/e2e-guide` | 테스트 시나리오 설계를 위한 대화형 가이드 |

## 디렉토리 구조

```
mobile-e2e-agents/
├── .claude/
│   ├── commands/           # 사용자 대상 슬래시 명령어
│   │   ├── e2e-init.md
│   │   ├── e2e-pom.md
│   │   ├── e2e-test.md
│   │   ├── e2e-ci.md
│   │   └── e2e-guide.md
│   │
│   └── agents/             # 내부 처리 에이전트
│       ├── ios-pom-generator.md
│       ├── ios-test-writer.md
│       ├── android-pom-generator.md
│       ├── android-test-writer.md
│       ├── android-ci-builder.md
│       └── ci-workflow-builder.md
│
├── templates/
│   ├── ios/
│   │   ├── xcuitest/       # XCUITest 템플릿
│   │   ├── support/        # 유틸리티 익스텐션
│   │   └── ci/             # CI 설정 템플릿
│   │
│   └── android/
│       ├── espresso/       # Espresso 템플릿 (View 기반)
│       ├── compose/        # Compose Testing 템플릿
│       ├── support/        # Android 유틸리티
│       └── ci/             # Android CI 템플릿
│
├── examples/               # 샘플 프로젝트
│   └── ios-sample-app/
│
└── docs/                   # 문서
    ├── getting-started.md
    ├── ios-guide.md
    ├── android-guide.md
    ├── pom-patterns.md
    ├── espresso-vs-uiautomator.md
    └── compose-testing.md
```

## 생성되는 코드 구조

### iOS

`/e2e-init` 실행 시 생성되는 구조:

```
{Project}UITests/
├── Sources/
│   ├── Support/
│   │   ├── BaseUITest.swift
│   │   ├── UITestConfiguration.swift
│   │   ├── XCUIElement+Wait.swift
│   │   ├── XCUIElement+Scroll.swift
│   │   └── SmartWait.swift
│   ├── Pages/
│   │   └── {ScreenName}Page.swift
│   ├── Flows/
│   │   └── {Feature}Flow.swift
│   └── Tests/
│       └── {Feature}Tests.swift
└── README.md
```

### Android

`/e2e-init android` 실행 시 생성되는 구조:

```
app/src/androidTest/java/{package}/
├── support/
│   ├── ViewWait.kt
│   ├── ScrollUtils.kt
│   ├── SystemHelper.kt
│   ├── TestDataManager.kt
│   └── ScreenshotHelper.kt
├── pages/
│   ├── BasePage.kt            # Espresso 베이스
│   ├── ComposeBasePage.kt     # Compose 베이스 (감지 시)
│   └── {ScreenName}Page.kt
├── flows/
│   └── {Feature}Flow.kt
└── tests/
    ├── BaseUITest.kt          # Espresso 베이스 테스트
    ├── ComposeBaseUITest.kt   # Compose 베이스 테스트 (감지 시)
    └── {Feature}Tests.kt
```

## 플랫폼 지원

### iOS
- XCUITest 프레임워크
- Swift 5.9+
- Xcode 15.0+
- iOS 17.0+

### Android
- Espresso (View 기반 UI)
- Jetpack Compose Testing
- UI Automator (시스템 상호작용)
- Kotlin
- Android Studio
- API 24+

## 요소 식별자 명명 규칙

플랫폼 간 안정적인 요소 식별을 위해 다음 명명 규칙을 사용합니다:

```
{screenName}_{elementType}_{elementName}
```

### iOS (accessibilityIdentifier)
```swift
TextField("Email", text: $email)
    .accessibilityIdentifier("login_input_email")
```

### Android (Compose용 testTag)
```kotlin
TextField(
    modifier = Modifier.testTag("login_input_email"),
    value = email,
    onValueChange = { email = it }
)
```

### Android (View용 resource ID)
```xml
<EditText
    android:id="@+id/login_input_email"
    ... />
```

### 공통 요소 타입
- `input` - 텍스트 필드
- `button` - 버튼
- `text` - 레이블/텍스트
- `toggle` - 스위치, 체크박스
- `list` - 리스트, 컬렉션
- `cell` - 리스트 아이템

## CI/CD 통합

`/e2e-ci` 명령어로 생성되는 항목:

- **GitHub Actions 워크플로우** (`.github/workflows/ui_tests.yml`)
- **Fastlane 설정** (`fastlane/Fastfile`)
- **테스트 리포트 생성기** (선택)
- **Slack 알림 설정** (선택)

## 기여하기

기여를 환영합니다! PR 제출 전 기여 가이드라인을 읽어주세요.

## 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE)를 참조하세요.

## 감사의 말

- 프로덕션 모바일 애플리케이션의 E2E 테스트 패턴에서 영감을 받았습니다
- Claude Code Extension 프레임워크로 제작되었습니다
