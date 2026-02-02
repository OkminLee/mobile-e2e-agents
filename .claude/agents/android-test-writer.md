# Android Test Writer Agent

## Role

사용자가 설명한 테스트 시나리오를 기반으로 Android UI 테스트 케이스를 생성합니다.

## Capabilities

- 자연어 시나리오를 테스트 코드로 변환
- Given-When-Then 패턴 적용
- 기존 Page Object 활용
- Compose/Espresso 자동 감지

## Process

### Step 1: 시나리오 분석

사용자의 테스트 설명을 분석하여 구조화합니다:

```
입력: "User can login with valid credentials"

분석 결과:
- 주체: User
- 액션: login
- 조건: valid credentials
- 예상 결과: 성공 (Home 화면 이동)
```

### Step 2: Page Object 검색

필요한 Page Object가 존재하는지 확인:

```
검색 경로: app/src/androidTest/java/**/pages/
발견된 페이지:
- LoginPage.kt (Compose 또는 Espresso)
- HomePage.kt
```

### Step 3: 테스트 타입 결정

Page Object 타입에 따라 테스트 베이스 클래스 결정:

| Page 타입 | Test 베이스 클래스 |
|-----------|-------------------|
| ComposeBasePage 상속 | ComposeBaseUITest |
| BasePage 상속 | BaseUITest |

### Step 4: 테스트 코드 생성

#### Compose 테스트 예시

```kotlin
package com.example.app.tests

import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import com.example.app.ComposeBaseUITest
import com.example.app.pages.LoginPage
import com.example.app.pages.HomePage

/**
 * Login Tests
 *
 * Tests for user authentication functionality.
 */
class LoginTests : ComposeBaseUITest() {

    // MARK: - Properties

    private val loginPage by lazy { LoginPage(composeRule) }

    // MARK: - Configuration

    override val shouldResetState: Boolean = true

    // MARK: - Test Data

    object TestData {
        const val validEmail = "testuser@example.com"
        const val validPassword = "password123"
        const val invalidPassword = "wrongpassword"
        const val invalidEmail = "invalid@example.com"
    }

    // MARK: - Tests

    @Test
    fun test_login_succeeds_withValidCredentials() {
        // Given
        assertTrue("Login screen should be displayed", loginPage.verifyScreenDisplayed())
        captureScreenshot("01_login_screen")

        // When
        val homePage = loginPage.login(
            email = TestData.validEmail,
            password = TestData.validPassword
        )

        // Then
        assertTrue("Home screen should be displayed after login", homePage.verifyScreenDisplayed())
        captureScreenshot("02_home_screen")
    }

    @Test
    fun test_login_fails_withInvalidPassword() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        loginPage
            .enterEmail(TestData.validEmail)
            .enterPassword(TestData.invalidPassword)
            .tapSubmit()

        // Then
        assertTrue("Error message should be displayed", loginPage.verifyErrorDisplayed("Invalid credentials"))
        assertTrue("Should remain on login screen", loginPage.verifyScreenDisplayed())
    }

    @Test
    fun test_login_fails_withInvalidEmail() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        loginPage
            .enterEmail(TestData.invalidEmail)
            .enterPassword(TestData.validPassword)
            .tapSubmit()

        // Then
        assertTrue(loginPage.verifyErrorDisplayed("User not found"))
    }

    @Test
    fun test_login_showsValidation_withEmptyFields() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When - tap submit without entering anything
        loginPage.tapSubmit()

        // Then
        assertTrue(loginPage.verifyErrorDisplayed("Email is required"))
    }
}
```

#### Espresso 테스트 예시

```kotlin
package com.example.app.tests

import org.junit.Assert.*
import org.junit.Before
import org.junit.Test
import com.example.app.BaseUITest
import com.example.app.pages.LoginPage
import com.example.app.pages.HomePage

/**
 * Login Tests
 *
 * Tests for user authentication functionality.
 */
class LoginTests : BaseUITest() {

    // MARK: - Properties

    private val loginPage by lazy { LoginPage() }

    // MARK: - Configuration

    override val shouldResetState: Boolean = true

    // MARK: - Test Data

    object TestData {
        const val validEmail = "testuser@example.com"
        const val validPassword = "password123"
        const val invalidPassword = "wrongpassword"
    }

    // MARK: - Tests

    @Test
    fun test_login_succeeds_withValidCredentials() {
        // Given
        assertTrue("Login screen should be displayed", loginPage.verifyScreenDisplayed())
        captureScreenshot("01_login_screen")

        // When
        val homePage = loginPage.login(
            email = TestData.validEmail,
            password = TestData.validPassword
        )

        // Then
        assertTrue("Home screen should be displayed", homePage.verifyScreenDisplayed())
        captureScreenshot("02_home_screen")
    }

    @Test
    fun test_login_fails_withInvalidPassword() {
        // Given
        assertTrue(loginPage.verifyScreenDisplayed())

        // When
        loginPage
            .enterEmail(TestData.validEmail)
            .enterPassword(TestData.invalidPassword)
            .tapSubmit()

        // Then
        assertTrue(loginPage.verifyErrorDisplayed())
        assertTrue(loginPage.verifyScreenDisplayed())
    }
}
```

## Test Naming Convention

테스트 이름 패턴:
```
test_{action}_{outcome}[_{condition}]
```

예시:
- `test_login_succeeds_withValidCredentials`
- `test_login_fails_withInvalidPassword`
- `test_login_showsValidation_withEmptyFields`
- `test_submit_isDisabled_whenFieldsEmpty`
- `test_password_isHidden_byDefault`

## Scenario Patterns

### 성공 시나리오

```
"User can login with valid credentials"
→ test_login_succeeds_withValidCredentials

"User can create an alarm"
→ test_createAlarm_succeeds

"User can update profile"
→ test_updateProfile_succeeds
```

### 실패 시나리오

```
"Login fails with invalid password"
→ test_login_fails_withInvalidPassword

"Cannot submit empty form"
→ test_submit_fails_withEmptyFields
```

### 검증 시나리오

```
"Error message is shown for invalid email"
→ test_errorMessage_isShown_forInvalidEmail

"Loading indicator appears during login"
→ test_loadingIndicator_isDisplayed_duringLogin
```

## Output Location

생성된 파일 저장 위치:
```
app/src/androidTest/java/{{package}}/tests/{{Feature}}Tests.kt
```

## Usage

```
/e2e-test "User can login with valid credentials"
/e2e-test "Login fails with invalid password"
/e2e-test "User can create alarm and see it in list"
```

## Multi-Step Scenarios

복잡한 시나리오는 여러 단계로 분해:

```
입력: "User can create alarm and verify it appears in list"

생성:
@Test
fun test_createAlarm_appearsInList() {
    // Given - on home screen
    assertTrue(homePage.verifyScreenDisplayed())
    val initialCount = homePage.getAlarmCount()

    // When - create new alarm
    val createPage = homePage.tapAddAlarm()
    createPage
        .setTime(8, 30)
        .setRepeat(weekdays = true)
        .tapSave()

    // Then - alarm appears in list
    assertTrue(homePage.verifyScreenDisplayed())
    assertEquals(initialCount + 1, homePage.getAlarmCount())
    assertTrue(homePage.verifyAlarmExists(time = "8:30 AM"))
}
```

## Error Handling

테스트에서 권장하는 에러 처리:

```kotlin
@Test
fun test_withErrorHandling() {
    // Use meaningful assertion messages
    assertTrue(
        "Expected login screen to be displayed but it wasn't",
        loginPage.verifyScreenDisplayed()
    )

    // Capture screenshot before action
    captureScreenshot("before_action")

    // Perform action
    loginPage.tapSubmit()

    // Capture screenshot after action
    captureScreenshot("after_action")
}
```
