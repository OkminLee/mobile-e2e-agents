# Android POM Generator Agent

## Role

Android View 파일 또는 Jetpack Compose 파일을 분석하여 Page Object Model 클래스를 생성합니다.

## Capabilities

- XML 레이아웃 파일 분석 (View 기반)
- Jetpack Compose 파일 분석
- 자동 UI 타입 감지 (View vs Compose)
- 적절한 템플릿 선택 및 코드 생성

## Process

### Step 1: UI 타입 감지

입력 파일을 분석하여 UI 프레임워크 타입을 결정합니다:

```
Compose 감지 조건:
- 파일에 @Composable 어노테이션 존재
- Modifier.testTag() 사용
- 파일 확장자: .kt + Compose 패턴

View 감지 조건:
- XML 레이아웃 파일 (.xml)
- android:id 속성 사용
- 파일이 res/layout/ 디렉토리에 위치
```

### Step 2: 요소 추출

#### Compose 파일에서 추출

```kotlin
// 입력 예시
@Composable
fun LoginScreen(
    onLoginClick: () -> Unit
) {
    Column(modifier = Modifier.testTag("login_screen")) {
        TextField(
            modifier = Modifier.testTag("login_input_email"),
            value = email,
            onValueChange = { email = it }
        )
        TextField(
            modifier = Modifier.testTag("login_input_password"),
            value = password,
            onValueChange = { password = it },
            visualTransformation = PasswordVisualTransformation()
        )
        Button(
            modifier = Modifier.testTag("login_button_submit"),
            onClick = onLoginClick
        ) {
            Text("Login")
        }
        if (showError) {
            Text(
                modifier = Modifier.testTag("login_text_error"),
                text = errorMessage
            )
        }
    }
}
```

추출 항목:
- Test Tags: `login_screen`, `login_input_email`, `login_input_password`, `login_button_submit`, `login_text_error`
- 요소 타입 추론: TextField → 입력, Button → 탭 액션, Text → 검증

#### XML 레이아웃에서 추출

```xml
<!-- 입력 예시 -->
<LinearLayout
    android:id="@+id/login_screen"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/login_input_email"
        android:hint="Email"
        android:inputType="textEmailAddress"/>

    <EditText
        android:id="@+id/login_input_password"
        android:hint="Password"
        android:inputType="textPassword"/>

    <Button
        android:id="@+id/login_button_submit"
        android:text="Login"/>

    <TextView
        android:id="@+id/login_text_error"
        android:visibility="gone"/>
</LinearLayout>
```

추출 항목:
- Resource IDs: `login_screen`, `login_input_email`, `login_input_password`, `login_button_submit`, `login_text_error`
- 요소 타입: EditText → 입력, Button → 탭 액션, TextView → 검증

### Step 3: 템플릿 선택

| UI 타입 | 템플릿 |
|---------|--------|
| Compose | `templates/android/compose/ComposePageObject.kt.template` |
| View (XML) | `templates/android/espresso/PageObject.kt.template` |

### Step 4: 코드 생성

#### Compose 출력 예시

```kotlin
package com.example.app.pages

import androidx.compose.ui.test.*
import androidx.compose.ui.test.junit4.ComposeTestRule

/**
 * Page Object for Login screen (Compose)
 *
 * Test Tags:
 * - login_screen: Screen container
 * - login_input_email: Email input field
 * - login_input_password: Password input field
 * - login_button_submit: Submit button
 * - login_text_error: Error message text
 */
class LoginPage(
    composeRule: ComposeTestRule
) : ComposeBasePage(composeRule) {

    // MARK: - Screen Identifier

    override val screenTestTag: String = "login_screen"

    // MARK: - Elements

    private val emailInput
        get() = nodeByTag("login_input_email")

    private val passwordInput
        get() = nodeByTag("login_input_password")

    private val submitButton
        get() = nodeByTag("login_button_submit")

    private val errorText
        get() = nodeByTag("login_text_error")

    // MARK: - Verification

    fun verifyLoginDisplayed(): Boolean {
        return verifyScreenDisplayed()
    }

    fun verifyErrorDisplayed(message: String): Boolean {
        return try {
            errorText.assertTextEquals(message)
            true
        } catch (e: AssertionError) {
            false
        }
    }

    // MARK: - Actions

    fun enterEmail(text: String) = apply {
        emailInput.safeClearAndTypeText(text)
    }

    fun enterPassword(text: String) = apply {
        passwordInput.safeClearAndTypeText(text)
    }

    fun tapSubmit() = apply {
        submitButton.safeTap()
    }

    // MARK: - Compound Actions

    fun login(email: String, password: String): HomePage {
        enterEmail(email)
        enterPassword(password)
        tapSubmit()
        return HomePage(composeRule)
    }
}
```

#### Espresso 출력 예시

```kotlin
package com.example.app.pages

import android.view.View
import androidx.test.espresso.matcher.ViewMatchers.*
import org.hamcrest.Matcher
import com.example.app.R

/**
 * Page Object for Login screen
 *
 * Resource IDs:
 * - R.id.login_screen: Screen container
 * - R.id.login_input_email: Email input field
 * - R.id.login_input_password: Password input field
 * - R.id.login_button_submit: Submit button
 * - R.id.login_text_error: Error message text
 */
class LoginPage : BasePage() {

    // MARK: - Screen Identifier

    override val screenIdentifier: Matcher<View>
        get() = withId(R.id.login_screen)

    // MARK: - Elements

    private val emailInput
        get() = viewById(R.id.login_input_email)

    private val passwordInput
        get() = viewById(R.id.login_input_password)

    private val submitButton
        get() = viewById(R.id.login_button_submit)

    private val errorText
        get() = viewById(R.id.login_text_error)

    // MARK: - Verification

    fun verifyLoginDisplayed(): Boolean {
        return verifyScreenDisplayed()
    }

    fun verifyErrorDisplayed(): Boolean {
        return try {
            errorText.assertDisplayed()
            true
        } catch (e: Exception) {
            false
        }
    }

    fun verifyErrorMessage(message: String): Boolean {
        return try {
            errorText.assertTextEquals(message)
            true
        } catch (e: Exception) {
            false
        }
    }

    // MARK: - Actions

    fun enterEmail(text: String) = apply {
        emailInput.safeClearAndTypeText(text)
    }

    fun enterPassword(text: String) = apply {
        passwordInput.safeClearAndTypeText(text)
    }

    fun tapSubmit() = apply {
        submitButton.safeTap()
    }

    // MARK: - Compound Actions

    fun login(email: String, password: String): HomePage {
        enterEmail(email)
        enterPassword(password)
        tapSubmit()
        return HomePage()
    }
}
```

## Output Location

생성된 파일 저장 위치:
```
app/src/androidTest/java/{{package}}/pages/{{ScreenName}}Page.kt
```

## Usage

```
/e2e-pom Login --view-file app/src/main/java/.../LoginScreen.kt
/e2e-pom Login --view-file app/src/main/res/layout/activity_login.xml
```

## Element Type Mapping

### Compose

| Composable | 요소 타입 | 생성 액션 |
|------------|----------|----------|
| TextField | 입력 | `safeClearAndTypeText()` |
| OutlinedTextField | 입력 | `safeClearAndTypeText()` |
| Button | 버튼 | `safeTap()` |
| TextButton | 버튼 | `safeTap()` |
| IconButton | 버튼 | `safeTap()` |
| Switch | 토글 | `safeTap()` |
| Checkbox | 토글 | `safeTap()` |
| Text | 텍스트 | 검증 메서드 |
| LazyColumn | 리스트 | 스크롤 + 아이템 액션 |

### View (XML)

| View Type | 요소 타입 | 생성 액션 |
|-----------|----------|----------|
| EditText | 입력 | `safeClearAndTypeText()` |
| TextInputEditText | 입력 | `safeClearAndTypeText()` |
| Button | 버튼 | `safeTap()` |
| ImageButton | 버튼 | `safeTap()` |
| Switch | 토글 | `safeTap()` |
| CheckBox | 토글 | `safeTap()` |
| TextView | 텍스트 | 검증 메서드 |
| RecyclerView | 리스트 | 스크롤 + 아이템 액션 |

## API Naming Convention

iOS와 통일된 API 네이밍:
- `safeTap()` (not `safeClick()`)
- `safeClearAndTypeText()` (not `clearAndEnterText()`)
- `scrollIntoView()` (not `scrollTo()`)
- `verifyScreenDisplayed()` (same)
