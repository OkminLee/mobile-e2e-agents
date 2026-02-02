# mobile-e2e-agents

> 크로스 플랫폼 모바일 E2E 테스트를 위한 Claude Code 확장

iOS 및 Android 모바일 앱의 E2E 테스트 인프라 설정, Page Object Model 생성, CI/CD 통합을 자동화하는 종합 툴킷입니다.

## E2E 테스트란?

**E2E(End-to-End) 테스트**는 사용자 관점에서 앱의 전체 흐름을 검증하는 테스트입니다. 실제 사용자가 앱을 사용하는 것처럼 버튼을 누르고, 텍스트를 입력하고, 화면 전환을 확인합니다.

| 구분 | 단위 테스트 | E2E 테스트 |
|------|-------------|------------|
| 검증 범위 | 함수/클래스 단위 | 전체 사용자 흐름 |
| 실행 환경 | 메모리 내 | 시뮬레이터/실제 기기 |
| 속도 | 빠름 (ms) | 느림 (초~분) |
| 발견하는 버그 | 로직 오류 | UI 깨짐, 네비게이션 오류, 통합 문제 |

### Page Object Model(POM)이란?

**POM**은 화면별로 UI 요소와 동작을 캡슐화하는 디자인 패턴입니다. 테스트 코드의 중복을 줄이고 유지보수를 쉽게 만듭니다.

```swift
// POM 없이 - 요소 변경 시 모든 테스트 수정 필요
app.buttons["login_button"].tap()
app.buttons["login_button"].tap()  // 다른 테스트에서도 반복

// POM 사용 - 한 곳만 수정하면 됨
LoginPage(app).tapLoginButton()
```

> 자세한 내용: [docs/pom-patterns.md](docs/pom-patterns.md)

## 주요 기능

- **E2E 스캐폴딩**: 단일 명령어로 E2E 테스트 구조 초기화
- **POM 생성**: 뷰 분석을 통한 Page Object Model 파일 자동 생성
- **테스트 케이스 템플릿**: 적절한 구조와 패턴으로 테스트 케이스 생성
- **CI 통합**: GitHub Actions 워크플로우 및 Fastlane 설정 생성
- **크로스 플랫폼**: iOS (XCUITest) 및 Android (Espresso/UI Automator) 지원

## 설치

### 방법 1: Claude Code 플러그인으로 설치 (권장)

**1단계: Marketplace 추가**
```bash
/plugin marketplace add https://github.com/user/mobile-e2e-agents.git
```

**2단계: 플러그인 설치**
```bash
# 현재 사용자에게 설치 (모든 프로젝트에서 사용)
/plugin install mobile-e2e-agents@mobile-e2e-marketplace

# 또는 프로젝트 전용으로 설치 (팀과 공유)
/plugin install mobile-e2e-agents@mobile-e2e-marketplace --scope project
```

**3단계: 명령어 사용**
```bash
/mobile-e2e-agents:e2e-init
/mobile-e2e-agents:e2e-pom LoginScreen
```

### 방법 2: 로컬 플러그인으로 테스트

```bash
# 저장소 클론
git clone https://github.com/user/mobile-e2e-agents.git

# 플러그인 디렉토리 지정하여 Claude Code 실행
claude --plugin-dir ./mobile-e2e-agents
```

### 방법 3: 수동 설치

1. 저장소 클론:
```bash
git clone https://github.com/user/mobile-e2e-agents.git
```

2. 프로젝트의 `.claude/` 디렉토리에 명령어 복사:
```bash
cp -r mobile-e2e-agents/.claude/commands/* your-project/.claude/commands/
cp -r mobile-e2e-agents/.claude/agents/* your-project/.claude/agents/
```

## 빠른 시작

### 전제 조건

| 플랫폼 | 요구사항 |
|--------|----------|
| **공통** | [Claude Code CLI](https://docs.anthropic.com/claude-code) 설치 완료 |
| **iOS** | Xcode 15+, iOS 17+ 시뮬레이터 |
| **Android** | Android Studio, API 24+ 에뮬레이터 |

### 1단계: E2E 테스트 구조 초기화

```bash
/e2e-init
```

```
✓ 생성됨: MyAppUITests/Sources/Support/BaseUITest.swift
✓ 생성됨: MyAppUITests/Sources/Support/UITestConfiguration.swift
✓ 생성됨: MyAppUITests/Sources/Support/XCUIElement+Wait.swift
✓ 생성됨: MyAppUITests/Sources/Pages/.gitkeep
✓ 생성됨: MyAppUITests/Sources/Tests/.gitkeep
```

### 2단계: Page Object 생성

```bash
/e2e-pom LoginScreen
```

뷰 파일을 지정하면 자동으로 요소를 분석합니다:
```bash
/e2e-pom LoginScreen --view-file Sources/Features/Login/LoginView.swift
```

```
✓ 분석됨: LoginView.swift (3개 요소 발견)
✓ 생성됨: MyAppUITests/Sources/Pages/LoginScreenPage.swift
```

### 3단계: 테스트 케이스 생성

```bash
/e2e-test "유효한 자격 증명으로 로그인할 수 있다"
```

```
✓ 생성됨: MyAppUITests/Sources/Tests/LoginTests.swift
```

### 4단계: CI 워크플로우 생성 (선택)

```bash
/e2e-ci
```

```
✓ 생성됨: .github/workflows/ui_tests.yml
✓ 생성됨: fastlane/Fastfile
```

## 실전 예시: 로그인 테스트 만들기

### 명령어 흐름

```
/e2e-init → /e2e-pom → /e2e-test → /e2e-ci
    ↓           ↓           ↓          ↓
  구조 생성   페이지 생성  테스트 작성  CI 연동
```

### 시나리오: 로그인 화면 E2E 테스트

**1. View 파일 확인** - 테스트할 화면의 요소에 식별자가 있는지 확인합니다.

```swift
// Sources/Features/Login/LoginView.swift
TextField("이메일", text: $email)
    .accessibilityIdentifier("login_input_email")  // ✓ 식별자 있음

SecureField("비밀번호", text: $password)
    .accessibilityIdentifier("login_input_password")

Button("로그인") { ... }
    .accessibilityIdentifier("login_button_submit")
```

**2. POM 생성**

```bash
/e2e-pom LoginScreen --view-file Sources/Features/Login/LoginView.swift
```

생성된 Page Object:
```swift
// MyAppUITests/Sources/Pages/LoginScreenPage.swift
final class LoginScreenPage: BasePage {
    private var emailInput: XCUIElement { app.textFields["login_input_email"] }
    private var passwordInput: XCUIElement { app.secureTextFields["login_input_password"] }
    private var submitButton: XCUIElement { app.buttons["login_button_submit"] }

    func enterEmail(_ email: String) -> Self { ... }
    func enterPassword(_ password: String) -> Self { ... }
    func tapSubmit() -> HomeScreenPage { ... }
}
```

**3. 테스트 케이스 생성**

```bash
/e2e-test "유효한 자격 증명으로 로그인하면 홈 화면으로 이동한다"
```

생성된 테스트:
```swift
// MyAppUITests/Sources/Tests/LoginTests.swift
func test_login_withValidCredentials_navigatesToHome() throws {
    LoginScreenPage(app)
        .enterEmail("test@example.com")
        .enterPassword("password123")
        .tapSubmit()
        .verifyOnHomeScreen()
}
```

**4. 테스트 실행**

```bash
# Xcode에서 실행
Cmd + U

# 또는 커맨드라인
xcodebuild test -scheme MyAppUITests -destination 'platform=iOS Simulator,name=iPhone 15'
```

## 명령어 참조

| 명령어 | 설명 |
|--------|------|
| `/e2e-init` | 프로젝트에 E2E 테스트 구조 초기화 |
| `/e2e-pom <ScreenName>` | 화면에 대한 Page Object 생성 |
| `/e2e-test "<시나리오>"` | 시나리오 설명으로 테스트 케이스 생성 |
| `/e2e-ci` | CI/CD 워크플로우 파일 생성 |
| `/e2e-guide` | 테스트 시나리오 설계를 위한 대화형 가이드 |

## 트러블슈팅

### 자주 발생하는 문제

| 문제 | 원인 | 해결책 |
|------|------|--------|
| POM에 요소가 없음 | View에 `accessibilityIdentifier` 누락 | View 파일에 식별자 추가 후 재생성 |
| 테스트가 요소를 못 찾음 | 요소 로딩 전 접근 시도 | `SmartWait` 또는 `waitForExistence` 사용 |
| CI에서만 테스트 실패 | 시뮬레이터 상태 불일치 | 테스트 전 앱 리셋 (`launchArguments` 설정) |
| Android에서 Compose 요소 못 찾음 | `testTag` 누락 | `Modifier.testTag("id")` 추가 |

### 요소를 찾지 못할 때 디버깅

```swift
// 현재 화면의 모든 요소 출력
print(app.debugDescription)

// 특정 요소 존재 여부 확인
let button = app.buttons["login_button_submit"]
print("Exists: \(button.exists), Hittable: \(button.isHittable)")
```

### CI 환경에서 안정성 높이기

```swift
// BaseUITest.swift에서 앱 리셋 설정
override func setUpWithError() throws {
    app.launchArguments = ["--uitesting", "--reset-state"]
    app.launch()
}
```

> 더 많은 트러블슈팅: [docs/getting-started.md](docs/getting-started.md)

## 디렉토리 구조

```
mobile-e2e-agents/
├── .claude-plugin/         # Claude Code 플러그인 메타데이터
│   ├── plugin.json         # 플러그인 manifest
│   └── marketplace.json    # Marketplace 설정
│
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
