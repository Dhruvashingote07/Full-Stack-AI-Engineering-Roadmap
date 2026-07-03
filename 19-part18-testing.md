# Part 18: Testing

## The Testing Trophy (Pyramid Reimagined)

Modern testing follows the "Testing Trophy" (coined by Kent C. Dodds), which prioritizes integration tests over unit tests, unlike the traditional pyramid:

```
        /\
       /E2E\
      /─────\
     /  INT   \
    /───────────\
   /   UNIT      \
  /───────────────\
 /   STATIC        \
/  (TypeScript, ESLint, type checkers)
```

- **Static Analysis** (base): Catches type errors, unused variables, lint issues BEFORE tests run
- **Unit Tests**: Test individual functions/classes in isolation (fast, but low confidence)
- **Integration Tests**: Test how components work together — the sweet spot for confidence per dollar spent
- **E2E Tests**: Test critical user flows through the entire system (slow, brittle, but high fidelity)

> 💡 **Pro Tip:** Follow the **80/20 rule**: spend ~20% of your testing effort on unit tests and ~80% on integration/E2E tests. Unit tests are fast but test implementation details. Integration tests test behavior and give you the highest confidence that your app actually works. Prefer testing behavior over implementation.

> ✅ **Best Practice:** A test that doesn't fail when you break the code is worse than no test — it creates false confidence. When writing a test, first make it fail (Red), then make it pass (Green), then refactor (Refactor). This is the Red-Green-Refactor cycle of TDD.

## Chapter 1: JUnit 5

### Architecture

JUnit 5 consists of three modules:
- **JUnit Platform**: Launches testing frameworks on the JVM
- **JUnit Jupiter**: Programming model and extension model
- **JUnit Vintage**: Backward compatibility with JUnit 4

### Core Annotations

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @BeforeAll
    static void setupAll() {
        // Runs once before all tests
    }

    @BeforeEach
    void setup() {
        // Runs before each test
    }

    @Test
    void testAddition() {
        assertEquals(4, calculator.add(2, 2));
        assertTrue(calculator.add(1, 1) == 2);
    }

    @Test
    void testDivisionByZero() {
        assertThrows(ArithmeticException.class, () -> calculator.divide(1, 0));
    }

    @AfterEach
    void tearDown() { }

    @AfterAll
    static void tearDownAll() { }
}
```

### Parameterized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

@ParameterizedTest
@ValueSource(strings = {"racecar", "radar", "level"})
void testPalindromes(String word) {
    assertTrue(isPalindrome(word));
}

@ParameterizedTest
@CsvSource({"1, 1, 2", "2, 3, 5", "5, 5, 10"})
void testAddition(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}

@ParameterizedTest
@MethodSource("provideTestData")
void testWithMethodSource(int input, int expected) {
    assertEquals(expected, process(input));
}

static Stream<Arguments> provideTestData() {
    return Stream.of(
        Arguments.of(1, 2),
        Arguments.of(2, 4),
        Arguments.of(3, 6)
    );
}
```

### @Nested Tests

```java
class UserServiceTest {

    @Nested
    class Registration {
        @Test
        void shouldRegisterValidUser() { }
        @Test
        void shouldRejectDuplicateEmail() { }
    }

    @Nested
    class Authentication {
        @Test
        void shouldAuthenticateValidCredentials() { }
        @Test
        void shouldRejectInvalidPassword() { }
    }
}
```

### Extensions

```java
import org.junit.jupiter.api.extension.*;

public class LoggingExtension implements TestWatcher {
    @Override
    public void testSuccessful(ExtensionContext context) {
        System.out.println("Passed: " + context.getDisplayName());
    }
    @Override
    public void testFailed(ExtensionContext context, Throwable cause) {
        System.err.println("Failed: " + context.getDisplayName());
    }
}

@ExtendWith(LoggingExtension.class)
class MyTests { }
```

### @Timeout and @RepeatedTest

```java
import org.junit.jupiter.api.Timeout;
import java.time.Duration;

@Timeout(5) // Fails if test takes longer than 5 seconds
@Test
void testPerformance() {
    // Test that should complete quickly
}

@RepeatedTest(value = 10, name = "Attempt {currentRepetition}/{totalRepetitions}")
void testRandomizedInput(RepetitionInfo info) {
    int attempt = info.getCurrentRepetition();
    String input = generateTestInput(attempt);
    assertNotNull(process(input));
}
```

### Assumptions (Conditional Tests)

```java
import static org.junit.jupiter.api.Assumptions.*;

@Test
void testOnlyOnWindows() {
    assumeTrue(System.getProperty("os.name").contains("Windows"));
    // Test Windows-specific behavior
}

@Test
void testWithEnvironment() {
    assumeNotNull(System.getenv("CI"));
    // Only runs in CI environment
}
```

### Real World Analogy: The Quality Inspector

JUnit 5 is like an automated factory quality control system:
- **@Test** = Each inspection station checks one specific thing
- **@BeforeEach/@AfterEach** = The conveyor belt resets between inspections
- **@BeforeAll/@AfterAll** = The factory is set up at the start of the shift and torn down at the end
- **@Nested** = Different inspection bays for different product lines (but same factory floor)
- **@ParameterizedTest** = One station inspects 100 different widgets using the same checklist
- **@Timeout** = If an inspection takes too long, flag it — the conveyor stops
- **@RepeatedTest** = The same widget is inspected 10 times to check for intermittent defects
- **Extensions** = Cameras, sensors, and meters that log every inspection

### Practical Exercises

1. **TDD Kata**: Implement a FizzBuzz function using TDD — write tests first, then implement, then refactor
2. **Parameterized Testing**: Write a parameterized test for a `validateEmail` function with 10+ test cases including edge cases
3. **Custom Extension**: Create a `@RetryOnFailure` extension that retries a flaky test up to 3 times before reporting failure
4. **Timeout Hunt**: Find an existing slow test in your project, analyze why it's slow, and either optimize it or add a timeout

### Revision Notes
- JUnit Platform runs tests, Jupiter is the API you write, Vintage provides backward compat
- Key annotations: @Test, @BeforeAll/Each, @AfterAll/Each, @Nested, @ParameterizedTest, @RepeatedTest, @Timeout
- Assertions: assertEquals, assertTrue, assertThrows, assertAll (grouped assertions), assertTimeout
- Assumptions (assumeTrue/assumeFalse) let tests conditionally run based on environment
- Extensions replace JUnit 4's @Rule and @RunWith — more powerful and composable

---



## Chapter 2: Mockito

### Core Concepts

```java
import static org.mockito.Mockito.*;
import static org.mockito.ArgumentMatchers.*;

// Create mock
List<String> mockedList = mock(List.class);

// Stubbing
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(anyInt())).thenReturn("default");
when(mockedList.get(-1)).thenThrow(IndexOutOfBoundsException.class);

// Verification
mockedList.add("one");
verify(mockedList).add("one");
verify(mockedList, never()).clear();
verify(mockedList, timeout(100)).add("one");
```

### @InjectMocks and @Mock

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldCreateUser() {
        User user = new User("john@example.com", "John");
        when(userRepository.save(any(User.class))).thenReturn(user);
        User result = userService.createUser(user);
        assertEquals("john@example.com", result.getEmail());
        verify(userRepository).save(any(User.class));
    }
}
```

### BDDMockito (Given/When/Then)

```java
import static org.mockito.BDDMockito.*;

@Test
void bddStyleTest() {
    // Given
    given(userRepository.findByEmail("john@example.com"))
        .willReturn(Optional.of(new User("John")));
    // When
    User result = userService.findUser("john@example.com");
    // Then
    then(userRepository).should().findByEmail("john@example.com");
}
```

### ArgumentCaptor

```java
import org.mockito.ArgumentCaptor;

@Test
void shouldSendWelcomeEmail() {
    // Given
    User user = new User("john@example.com", "John");
    given(userRepository.save(any())).willReturn(user);

    // When
    userService.registerUser(user);

    // Then
    ArgumentCaptor<Email> emailCaptor = ArgumentCaptor.forClass(Email.class);
    verify(emailService).send(emailCaptor.capture());
    Email sentEmail = emailCaptor.getValue();
    assertEquals("john@example.com", sentEmail.getTo());
    assertEquals("Welcome!", sentEmail.getSubject());
}
```

### Spies (Partial Mocking)

```java
import static org.mockito.Mockito.spy;

@Test
void shouldTrackCallsOnRealObject() {
    List<String> realList = new ArrayList<>();
    List<String> spyList = spy(realList);

    spyList.add("one");
    spyList.add("two");

    verify(spyList).add("one");
    verify(spyList).add("two");

    // You can stub specific calls on a spy
    doReturn(100).when(spyList).size();
    assertEquals(100, spyList.size()); // Overridden
    assertEquals(2, spyList.size()); // Actual — oops, the stub persists!
}
```

> ⚠️ **Warning:** Over-mocking is a common anti-pattern. If you find yourself mocking 5+ dependencies in a single test, your class likely has too many responsibilities. Also, don't mock types you don't own (third-party libraries) — wrap them in your own abstraction and mock that instead.

> ❓ **Interview Question:** What's the difference between `@Mock` and `@InjectMocks`? How does Mockito handle cases where there are multiple constructors with the same parameter types?

> ✅ **Best Practice:** Use `BDDMockito.given()` (Given/When/Then style) over `Mockito.when()` for test readability. This aligns with the BDD pattern that behavior-driven tests follow. The test becomes a readable specification: "Given this input, when I call this method, then expect this output."

### Real World Analogy: The Flight Simulator

Mockito is like a flight simulator for pilot training:
- **@Mock** = A fake control tower — the pilot thinks they're talking to real air traffic control, but it's all simulated
- **@InjectMocks** = The simulated cockpit into which all the fake instruments are wired
- **Spy** = A real airplane instrument that's been modified to report fake readings for specific inputs (altitude = 10,000ft regardless of actual altitude)
- **ArgumentCaptor** = The flight recorder that captures what the pilot said to the tower for later analysis
- **BDDMockito** = The checklist: "Given wind at 20kts, When the pilot requests landing clearance, Then the tower grants permission"

### Practical Exercises

1. **Mock a REST Client**: Write a service that calls an external weather API. Mock the HTTP client (RestTemplate/WebClient) and test the service with various API responses (success, 404, timeout)
2. **Argument Captor**: Test an email notification service — verify that the correct email object with the right subject, recipient, and body was sent
3. **Spy for Testing**: Use a spy on a `CacheManager` to verify that cache entries are correctly updated without resetting the actual cache
4. **Over-mocking Detection**: Review an existing test that mocks 5+ dependencies and refactor the production code to be more testable with fewer mocks

### Revision Notes
- `@Mock` creates a mock, `@InjectMocks` injects mocks into the object under test
- `verify()` checks that a method was called with specific arguments
- `ArgumentCaptor` captures arguments for assertions — essential for testing command/event objects
- `doThrow()`, `doReturn()`, `doAnswer()` are needed for void methods and spies
- `@Spy` creates a partial mock — real methods run unless stubbed
- Never mock value objects (POJOs/DTOs) — use real instances

---

## Chapter 3: Selenium WebDriver

### Setup and Basic Usage

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

WebDriverManager.chromedriver().setup();
WebDriver driver = new ChromeDriver();
driver.get("https://example.com");

// Finding elements
WebElement element = driver.findElement(By.id("username"));
WebElement btn = driver.findElement(By.cssSelector(".btn-primary"));
List<WebElement> items = driver.findElements(By.className("item"));

// Waits
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));

// Page Object Model
public class LoginPage {
    @FindBy(id = "username")
    private WebElement usernameField;

    public LoginPage(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }

    public void loginAs(String username, String password) {
        usernameField.sendKeys(username);
        passwordField.sendKeys(password);
        loginButton.click();
    }
}
```

### Taking Screenshots and Recording

```java
// Take screenshot
File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(screenshot, new File("screenshots/login_page.png"));

// Execute JavaScript
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");
String title = (String) js.executeScript("return document.title");
```

> ⚠️ **Warning:** Selenium tests are notoriously flaky because they rely on timing. Implicit waits (`driver.manage().timeouts().implicitlyWait()`) apply globally and can hide real issues. Prefer **explicit waits** (`WebDriverWait` with `ExpectedConditions`) for specific elements, and avoid `Thread.sleep()` entirely — it makes tests slow AND flaky.

> ✅ **Best Practice:** The Page Object Model (POM) is the gold standard for Selenium test organization. Each page gets a class that encapsulates its elements and actions. Tests call business-level methods like `loginPage.loginAs("user", "pass")` instead of interacting with raw WebElements. This makes tests readable and resilient to UI changes.

### Real World Analogy: The Robot Tester

Selenium is like a robotic arm programmed to test a physical product:
- **WebDriver** = The robotic arm that can press buttons and read displays
- **findElement** = The arm's sensors locating a specific button on the control panel
- **WebDriverWait** = Waiting for a status light to turn green before pressing the next button
- **Page Object Model** = The instruction manual for each machine: "To start the machine: 1. Check power is on, 2. Press green button, 3. Wait for humming sound"
- **Screenshots** = The robot taking a photo of the display at each step for the quality log

### Practical Exercises

1. **Page Object Model**: Create a Page Object for a login page and a dashboard page on a test site (saucedemo.com), then write a test that logs in and verifies the dashboard
2. **Multi-Browser Test**: Write a parameterized Selenium test that runs the same test case on Chrome, Firefox, and Edge using WebDriverManager
3. **Screenshot on Failure**: Implement a `TestWatcher` (JUnit 5 extension) that takes a screenshot whenever a Selenium test fails

### Revision Notes
- Use WebDriverManager for automatic driver binary management — no manual geckodriver/chromedriver downloads
- Prefer explicit waits over implicit waits and Thread.sleep
- POM is the standard pattern for test maintainability
- Selenium Grid allows parallel test execution across multiple machines/browsers
- Use `FluentWait` for polling with custom ignore conditions (stale elements, timeouts)

---

## Chapter 4: Playwright

```javascript
const { chromium } = require("playwright");

(async () => {
    const browser = await chromium.launch();
    const page = await browser.newPage();
    await page.goto("https://example.com");

    // Locators
    await page.locator("#username").fill("testuser");
    await page.getByRole("button", { name: "Submit" }).click();
    await page.getByText("Welcome").click();

    // Auto-waiting
    await expect(page.locator(".title")).toHaveText("Dashboard");
    await expect(page.locator("#status")).toBeVisible();

    // Network interception
    await page.route("**/api/users", route => {
        route.fulfill({
            status: 200,
            body: JSON.stringify([{ id: 1, name: "Mocked" }])
        });
    });

    await browser.close();
})();
```

### Playwright with Python

```python
from playwright.sync_api import sync_playwright

def test_user_login():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto("https://example.com/login")

        # Auto-waiting locators
        page.fill("#username", "testuser")
        page.fill("#password", "secret123")
        page.click("button[type='submit']")

        # Assertions with auto-retry
        expect(page.locator(".welcome")).to_have_text("Welcome, testuser!")
        expect(page.locator(".avatar")).to_be_visible()

        # Network interception
        def handle_request(route):
            route.fulfill(status=200, body='{"users": []}')
        page.route("**/api/users", handle_request)

        browser.close()
```

### Playwright Codegen (Test Generator)

```bash
# Record tests by interacting with the browser
npx playwright codegen https://example.com

# This opens a browser — every click/type generates test code
# Select Java, Python, JS, or C# as the output language
```

> 💡 **Pro Tip:** Playwright's **auto-waiting** is its killer feature. When you call `page.click()`, Playwright automatically waits for the element to be visible, enabled, and stable (not animating) before clicking. It also waits for navigation to complete after clicking links. This eliminates 90% of the flaky test issues that plague Selenium.

> ❓ **Interview Question:** Compare Playwright and Selenium for a modern SPA application. What advantages does Playwright's architecture (single browser context, CDP-based) have over Selenium WebDriver's WebDriver wire protocol?

### Real World Analogy: The Smart QA Engineer

Playwright is like a QA engineer who:
- **Auto-waiting** = Knows to wait for a page to fully load before checking anything
- **Locators** = Uses precise, semantic descriptions: "the submit button in the login form" vs "the third button on the page"
- **Network interception** = Can replace the server's response with prepared test data without talking to the real server
- **Codegen** = Follows you around, watches what you test manually, and writes the test automation code for you
- **Trace Viewer** = Records a full video replay of the test — watch what the browser saw when the test failed

### Practical Exercises

1. **E2E Login Flow**: Write a Playwright test that navigates to a login page, fills credentials, submits, and verifies the dashboard URL
2. **API Mocking**: Mock a slow API endpoint (add 3-second delay) and verify that a loading spinner is shown while waiting
3. **Cross-Browser Test**: Run the same Playwright test on Chromium, Firefox, and WebKit — note any rendering differences
4. **Trace Viewer**: Generate a Playwright trace on test failure and inspect the DOM snapshot, network log, and console output

### Revision Notes
- Playwright supports Chromium, Firefox, WebKit — one API for all browsers
- Auto-waiting eliminates flaky timing issues (no more Thread.sleep!)
- Network interception allows full API mocking without external dependencies
- Playwright Codegen generates test code from manual browser interactions
- Trace Viewer provides a full replay of failed tests (DOM snapshots at every step)
- Playwright is generally faster than Selenium because it uses CDP (Chrome DevTools Protocol) directly

---

## Chapter 5: Cypress

```javascript
describe("User Login Flow", () => {
    beforeEach(() => {
        cy.visit("https://example.com/login");
    });

    it("logs in with valid credentials", () => {
        cy.intercept("POST", "/api/auth/login", {
            statusCode: 200,
            body: { token: "abc123", user: { name: "John" } }
        }).as("loginRequest");

        cy.get("#email").type("john@example.com");
        cy.get("#password").type("password123");
        cy.get("button[type='submit']").click();
        cy.wait("@loginRequest");
        cy.contains("Welcome, John");
    });
});
```

### Cypress Component Testing

```javascript
// Cypress Component Testing (for React/Vue/Angular)
import { mount } from "cypress/react18";
import Button from "./Button";

describe("Button Component", () => {
    it("renders with correct text", () => {
        mount(<Button label="Click me" variant="primary" />);
        cy.contains("Click me").should("be.visible");
    });

    it("handles click events", () => {
        const onClick = cy.stub();
        mount(<Button onClick={onClick} label="Submit" />);
        cy.contains("Submit").click();
        expect(onClick).to.have.been.calledOnce;
    });
});
```

> ⚠️ **Warning:** Cypress does NOT support cross-browser testing natively (it only supports Chromium-based browsers and Electron). For Firefox and WebKit testing, you'll need a separate tool like Playwright or Selenium. Also, Cypress runs inside the browser, which means you can't test across multiple browser tabs simultaneously.

> ✅ **Best Practice:** Use `cy.intercept()` to stub network requests at the earliest opportunity — before the request is made. This makes your tests deterministic (no dependency on real API availability) and fast (no network latency). Use fixture files for complex response bodies.

### Practical Exercises

1. **Form Validation**: Write a Cypress test that submits a form with invalid data, verifies each validation error message appears, then corrects each field and successfully submits
2. **API Stubbing**: Use `cy.intercept()` to stub a search endpoint with a 2-second delay and verify that a loading indicator appears during the wait
3. **Custom Command**: Create a custom Cypress command `cy.login(email, password)` that abstracts the authentication flow for reuse across tests

### Revision Notes
- Cypress runs in-browser, giving it access to the DOM, network, and localStorage directly
- `cy.intercept()` replaces the deprecated `cy.route()` — supports all HTTP methods
- Cypress retries assertions automatically (like Playwright)
- Component testing (cypress/react18, @cypress/vue) allows testing UI components in isolation
- Screenshots and videos are automatically captured on failure in CI

---

## Chapter 6: REST Assured

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

@Test
void testGetUsers() {
    given()
        .baseUri("https://api.example.com")
        .header("Authorization", "Bearer token123")
        .queryParam("page", 1)
    .when()
        .get("/users")
    .then()
        .statusCode(200)
        .body("total", equalTo(100))
        .body("data.size()", greaterThan(0));
}

@Test
void testCreateUser() {
    given()
        .contentType("application/json")
        .body(new User("john@example.com", "John"))
    .when()
        .post("/users")
    .then()
        .statusCode(201)
        .body("id", notNullValue());
}

### Extracting Response for Further Tests

```java
@Test
void testExtractResponse() {
    int userId =
        given()
            .contentType("application/json")
            .body(new User("john@example.com", "John"))
        .when()
            .post("/users")
        .then()
            .extract()
            .path("id");

    // Use extracted ID in subsequent requests
    given()
        .pathParam("id", userId)
    .when()
        .get("/users/{id}")
    .then()
        .statusCode(200)
        .body("email", equalTo("john@example.com"));
}

@Test
void testAuthenticationFlow() {
    // Login, extract token, and use it
    String token =
        given()
            .body("{\"username\":\"admin\",\"password\":\"pass\"}")
        .when()
            .post("/auth/login")
        .then()
            .extract()
            .path("access_token");

    given()
        .header("Authorization", "Bearer " + token)
    .when()
        .get("/api/protected-resource")
    .then()
        .statusCode(200);
}
```

> ❓ **Interview Question:** Your API returns a list of users. How would you use REST Assured to: (a) verify the response matches a JSON schema, (b) assert that every user in the list has a non-null email, and (c) ensure the response time is under 500ms?

> ✅ **Best Practice:** Structure your REST Assured tests following the Arrange-Act-Assert pattern. Use the `given()` block for setup (headers, query params, body), `when()` for the actual request, and `then()` for assertions. This makes tests readable and consistent across your codebase.

### Practical Exercises

1. **CRUD Testing**: Write REST Assured tests for a full CRUD API — create a resource, read it, update it, and delete it, verifying the correct status codes and response bodies at each step
2. **Authentication Flow**: Test a JWT-based auth API: register, login (extract token), access protected endpoint, and verify expired/invalid tokens are rejected with 401
3. **Schema Validation**: Add response schema validation using Hamcrest matchers and `matchesJsonSchemaInClasspath()`

### Revision Notes
- REST Assured follows the Given/When/Then pattern for API testing
- Response extraction allows chaining requests (e.g., create → get → update → delete)
- Use `responseTime()` matcher for performance assertions
- Log requests/responses with `log().all()` for debugging failing tests
- REST Assured integrates with Spring MockMvc for testing controllers without a running server

---

## Chapter 7: PyTest (Python)

### Fixtures

```python
import pytest

@pytest.fixture
def temp_db():
    db = Database(":memory:")
    yield db
    db.close()

@pytest.fixture(scope="module")
def module_data():
    return {"data": "test"}

def test_database_query(temp_db):
    temp_db.insert("user", {"name": "John"})
    result = temp_db.query("user", name="John")
    assert result["name"] == "John"
```

### Parametrize

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (2, 3, 5),
    (5, 5, 10),
])
def test_addition(a, b, expected):
    assert add(a, b) == expected
```

### Markers and Monkeypatch

```python
@pytest.mark.smoke
def test_basic():
    assert True

@pytest.mark.skip(reason="Not implemented")
def test_future():
    pass

def test_mock_env(monkeypatch):
    monkeypatch.setenv("API_KEY", "test_key")
    assert get_api_key() == "test_key"

def test_mock_function(monkeypatch):
    def mock_get_data():
        return {"mock": "data"}
    monkeypatch.setattr("module.get_data", mock_get_data)
    result = get_data_from_api()
    assert result == {"mock": "data"}
```

### conftest.py (Shared Fixtures)

```python
# conftest.py — automatically discovered by pytest
import pytest
import tempfile
import os

@pytest.fixture(scope="session")
def test_db_url():
    """Shared database URL for all tests in the session."""
    return "sqlite:///test.db"

@pytest.fixture
def temp_upload_dir():
    """Temporary directory that's cleaned up after each test."""
    with tempfile.TemporaryDirectory() as tmpdir:
        yield tmpdir

@pytest.fixture(autouse=True)
def setup_test_env(monkeypatch):
    """Automatically applied to every test in the directory."""
    monkeypatch.setenv("ENVIRONMENT", "test")
    monkeypatch.setenv("LOG_LEVEL", "ERROR")
```

### Async Tests (pytest-asyncio)

```python
import pytest

@pytest.mark.asyncio
async def test_async_api_call():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
        assert response.status_code == 200
        data = response.json()
        assert "results" in data
```

> ✅ **Best Practice:** Use `conftest.py` for shared fixtures and configuration. Pytest automatically discovers conftest.py files in each test directory. Put session-scoped fixtures (database connections, API clients) in conftest.py at the root of your tests, and module-specific fixtures in local conftest.py files.

> 💡 **Pro Tip:** Use `pytest -k "pattern"` to run a subset of tests matching a keyword expression. For example, `pytest -k "api or integration and not slow"` runs all tests with "api" or "integration" in their name that don't have "slow". Combined with `-x` (stop on first failure) and `--pdb` (drop into debugger on failure), this is incredibly powerful for debugging.

> ❓ **Interview Question:** You have a test suite with 500 tests that takes 30 minutes to run. How would you structure your tests, fixtures, and CI pipeline so that developers get feedback in under 5 minutes for most changes?

### Real World Analogy: The Kitchen Inspector

PyTest is like a health inspector evaluating a restaurant kitchen:
- **Fixtures** = The inspector's checklist template that gets filled out for each kitchen section
- **conftest.py** = The master checklist book that applies to every kitchen in the franchise
- **Parametrize** = The inspector uses the same checklist for all 10 refrigerators, not a unique form for each
- **Monkeypatch** = The inspector replaces the head chef's thermometer with a calibrated one for the test
- **Async Tests** = The inspector watches the kitchen operate during a busy dinner service (real-time, async)
- **Markers** = Flags on the checklist: `@pytest.mark.smoke` = quick check, `@pytest.mark.slow` = deep inspection

### Practical Exercises

1. **API Test Suite**: Write a PyTest suite for a FastAPI application — use pytest-asyncio for async endpoints and httpx for the test client
2. **Fixture Factory**: Create a fixture that generates fake user data using Faker, and use it in 5 different tests
3. **CI Integration**: Configure pytest with JUnit XML output, coverage reporting, and GitHub Actions annotations for test failures
4. **Plugin Authoring**: Write a simple pytest plugin that times each test and prints the 10 slowest tests at the end of the run

### Revision Notes
- Fixtures can request other fixtures — dependency injection built-in
- `scope` parameter: function (default), class, module, session — choose carefully to balance isolation vs speed
- `conftest.py` files are auto-discovered; put shared fixtures there
- Use `@pytest.mark.asyncio` with `pytest-asyncio` plugin for async test support
- `monkeypatch` is the built-in way to mock without external dependencies
- `capsys` fixture captures stdout/stderr for testing console output
- `tmp_path` and `tmp_path_factory` provide temporary directories with cleanup

---

## Chapter 8: Integration Testing

### Spring Boot Test

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() throws Exception {
        given(userService.findById(1L)).willReturn(new User("John"));
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("John"));
    }
}

@DataJpaTest
class UserRepositoryTest {
    @Autowired
    private TestEntityManager entityManager;
    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindByEmail() {
        entityManager.persist(new User("john@example.com", "John"));
        User found = userRepository.findByEmail("john@example.com");
        assertThat(found.getName()).isEqualTo("John");
    }
}
```

### TestContainers

```java
@Testcontainers
class DatabaseIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
    }

    @Test
    void shouldUsePostgresContainer() {
        userRepository.save(new User("test@example.com", "Test"));
        assertThat(userRepository.count()).isEqualTo(1);
    }
}
```

### Testcontainers with Redis

```java
@Testcontainers
class RedisCacheTest {
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);

    private RedisTemplate<String, String> redisTemplate;

    @BeforeEach
    void setup() {
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration(
            redis.getHost(), redis.getFirstMappedPort()
        );
        redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(new JedisConnectionFactory(config));
        redisTemplate.afterPropertiesSet();
    }

    @Test
    void shouldStoreAndRetrieveValue() {
        redisTemplate.opsForValue().set("key", "value");
        assertEquals("value", redisTemplate.opsForValue().get("key"));
    }
}
```

> ⚠️ **Warning:** Integration tests with Testcontainers are slower than unit tests. A PostgreSQL container takes 5-10 seconds to start. Mitigate this by: (a) using `@Container` (static) so the container starts once for all tests, (b) reusing containers across test classes with `@Testcontainers(parallel = true)`, and (c) setting Ryuk resource reaper timeout appropriately.

> ❓ **Interview Question:** You're testing a microservice that depends on PostgreSQL, Redis, and a third-party REST API. Design an integration testing strategy that covers: database interactions (with Testcontainers), caching behavior, and API fault tolerance. How do you handle the third-party dependency?

### Real World Analogy: The Flight Simulator (Full Motion)

Integration testing with Testcontainers is like a full-motion flight simulator:
- **Unit tests** = Testing the altimeter by itself on the bench (fast, but doesn't tell you if the plane flies)
- **Testcontainers (PostgreSQL)** = A simulated air traffic control system that uses real radio frequencies and protocols — the pilot can't tell it's not real
- **Testcontainers (Redis)** = A real fuel gauge simulated in the cockpit — all the wiring is real, only the fuel tank is simulated
- **@DynamicPropertySource** = The flight simulator re-plumbing the fuel lines to connect to the simulated tank instead of the real one
- **@SpringBootTest** = The full simulator with all instruments, controls, and visual systems working together

### Practical Exercises

1. **Full-Stack Integration**: Write an integration test that starts PostgreSQL (Testcontainers), creates a user, calls the REST API to retrieve it, and verifies the full HTTP → service → repository → database stack
2. **Integration Test Refactor**: Take an existing unit test with 8 mocks and refactor it into an integration test that uses real (containerized) dependencies — notice the difference in confidence and speed
3. **Graceful Degradation**: Use Testcontainers + a service shutdown test to verify that your application gracefully handles the database being temporarily unavailable (circuit breaker pattern)

### Revision Notes
- @WebMvcTest: controller layer only, mocks services, use MockMvc
- @DataJpaTest: JPA repository layer, in-memory database by default
- @SpringBootTest: full application context, use with Testcontainers for real databases
- @Testcontainers manages Docker container lifecycle via JUnit 5 extensions
- Testcontainers supports PostgreSQL, MySQL, Redis, Kafka, Elasticsearch, and 80+ modules
- Use `@DynamicPropertySource` to override `application.properties` with container connection details

---

## Chapter 9: Performance Testing

### k6

```javascript
import http from "k6/http";
import { check, sleep } from "k6";

export const options = {
    stages: [
        { duration: "1m", target: 50 },
        { duration: "3m", target: 50 },
        { duration: "1m", target: 0 },
    ],
    thresholds: {
        http_req_duration: ["p(95)<500"],
        http_req_failed: ["rate<0.01"],
    },
};

export default function () {
    const response = http.get("https://api.example.com/users");
    check(response, {
        "status is 200": (r) => r.status === 200,
        "response time < 300ms": (r) => r.timings.duration < 300,
    });
    sleep(1);
}
```

### Locust (Python)

```python
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 5)

    @task(3)
    def view_users(self):
        self.client.get("/api/users")

    @task(1)
    def create_user(self):
        self.client.post("/api/users", json={"name": "Load Test"})
```

| Type | Purpose | Example Metric |
|------|---------|----------------|
| Load Testing | Normal expected traffic | 1000 users, think time 3s |
| Stress Testing | Find breaking point | Ramp up until 50% error rate |
| Endurance Testing | Long-running stability | 8 hours at 80% max load |
| Spike Testing | Sudden traffic surge | 10x load in 30 seconds |
| Soak Testing | Memory leak detection | Steady load for 24+ hours |

> ✅ **Best Practice:** Define performance SLOs (Service Level Objectives) before running tests. Common SLOs: p95 latency < 500ms, error rate < 0.1%, throughput > 1000 req/s. Run performance tests in a staging environment that mirrors production (same instance sizes, same database configuration, same network topology).

> ❓ **Interview Question:** A service has p95 latency of 200ms but p99 latency of 5000ms. What does this distribution tell you? How would you find and fix the root cause of these outliers?

### Real World Analogy: The Bridge Stress Test

Performance testing is like testing a new bridge before it opens:
- **Load Testing** = Drive 1000 cars across at normal speed — does the bridge handle it?
- **Stress Testing** = Fill the bridge with 5000 cars — at what point does it start to crack?
- **Endurance Testing** = Drive cars across continuously for 7 days — do any bolts loosen over time?
- **Spike Testing** = A football match just ended, and 5000 people cross in 5 minutes — does it hold?
- **p95 latency** = "95 out of 100 cars cross in under 5 seconds" — ensures a good experience for most
- **p99 latency** = "The slowest 1 in 100" — these are the ones that ruin the user experience

### Practical Exercises

1. **k6 Script**: Write a k6 script that simulates an e-commerce Black Friday scenario: 500 concurrent users browsing products, adding to cart, and checking out
2. **Performance Baseline**: Run a load test against your local development server, record the baseline, then add a 200ms `Thread.sleep()` in a controller and observe the degradation
3. **Locust with Custom Shapes**: Use Locust's `LoadTestShape` to create a custom traffic pattern: steady ramp-up to target, hold for 3 minutes, then a sudden 2x spike for 30 seconds
4. **Grafana Dashboard**: Set up k6 + InfluxDB + Grafana to visualize real-time test metrics (request rate, latency percentiles, error rate)

### Revision Notes
- k6 is the most popular modern load testing tool — scripts in JavaScript, supports HTTP/2, gRPC, WebSocket
- Locust is Python-native and easier for teams already in the Python ecosystem
- Always test under controlled conditions: isolate the system under test, warm up caches, and run multiple iterations
- Monitor server-side metrics (CPU, memory, database connection pool) alongside client-side metrics (latency, errors)
- The "tail at scale" problem means p99 can be 10-100x worse than p50 due to garbage collection, network congestion, and resource contention
- Performance test in CI on every significant change — regressions are much cheaper to fix immediately than after deployment

---

## Chapter 10: BDD with Cucumber

### Feature Files (Gherkin)

```gherkin
Feature: User Login
  As a registered user
  I want to log into the system
  So that I can access my dashboard

  Scenario: Successful login with valid credentials
    Given I am on the login page
    When I enter "john@example.com" in the email field
    And I enter "password123" in the password field
    And I click the "Sign In" button
    Then I should see my dashboard
    And I should see "Welcome, John!"

  Scenario: Login with invalid email
    Given I am on the login page
    When I enter "invalid@email" in the email field
    And I click the "Sign In" button
    Then I should see "Invalid email format"
```

### Step Definitions (Java)

```java
import io.cucumber.java.en.*;
import static org.junit.jupiter.api.Assertions.*;

public class LoginSteps {
    private LoginPage loginPage;
    private DashboardPage dashboardPage;

    @Given("I am on the login page")
    public void i_am_on_login_page() {
        loginPage = new LoginPage(driver);
        loginPage.navigateTo();
    }

    @When("I enter {string} in the email field")
    public void enter_email(String email) {
        loginPage.enterEmail(email);
    }

    @Then("I should see {string}")
    public void i_should_see(String expectedText) {
        assertTrue(driver.getPageSource().contains(expectedText));
    }
}
```

### Cucumber Runner

```java
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
public class CucumberTestRunner {
    // Discovers and runs all .feature files in the features directory
}
```

> ✅ **Best Practice:** Write feature files in collaboration with product managers and QA — Gherkin is designed to be readable by non-technical stakeholders. If a feature file uses technical terms or implementation details, rewrite it. The goal is a shared understanding of behavior, not test automation.

> ⚠️ **Warning:** Cucumber tests are slow and expensive to maintain. Only use BDD for critical business workflows that multiple stakeholders need to understand. Don't write a Gherkin scenario for every unit test — that defeats the purpose and creates a maintenance nightmare.

### Practical Exercises

1. **Feature First**: Before writing code, write Gherkin scenarios for a shopping cart checkout flow — include happy path, payment failure, and empty cart scenarios
2. **Step Definition Refactor**: Refactor step definitions to use the Page Object Model — steps should call high-level page methods, not interact with WebElements directly
3. **Scenario Outline**: Write a data-driven scenario outline that tests checkout with different payment methods (credit card, PayPal, Apple Pay) and shipping options

### Revision Notes
- Gherkin uses Given/When/Then/And/But — one scenario per behavior
- Feature files are the source of truth for acceptance criteria
- Cucumber supports Java, JavaScript, Python (behave), Ruby, and many other languages
- Scenario Outlines with Examples tables provide data-driven testing
- Tags (@smoke, @regression, @wip) allow selective test execution

---

## Chapter 11: Contract Testing (Pact)

```javascript
// Consumer-side Pact test (Node.js example)
const { PactV3, MatchersV3 } = require("@pact-foundation/pact");

const provider = new PactV3({
    consumer: "UserServiceClient",
    provider: "UserServiceAPI",
});

describe("UserService API Contract", () => {
    it("should return user by ID", async () => {
        provider
            .given("user with ID 1 exists")
            .uponReceiving("a request for user 1")
            .withRequest({
                method: "GET",
                path: "/users/1",
                headers: { Accept: "application/json" },
            })
            .willRespondWith({
                status: 200,
                headers: { "Content-Type": "application/json" },
                body: {
                    id: MatchersV3.integer(1),
                    name: MatchersV3.string("John"),
                    email: MatchersV3.string("john@example.com"),
                },
            });

        return provider.executeTest(async (mockServer) => {
            const response = await fetch(`${mockServer.url}/users/1`);
            const user = await response.json();
            expect(response.status).toBe(200);
            expect(user.id).toBeDefined();
        });
    });
});
```

> ❓ **Interview Question:** Your team owns 12 microservices that communicate via REST APIs. How would contract testing (Pact) help you catch breaking changes before deployment? How does this differ from integration testing with Testcontainers?

> ✅ **Best Practice:** Implement contract testing in your CI pipeline using a **Pact Broker**. When the consumer's contract tests pass, the generated contract is published to the broker. The provider CI then verifies it against the real provider. This creates a "can I deploy?" check — if the provider can't verify the consumer's contracts, deployment is blocked.

### Practical Exercises

1. **Consumer Contract**: Write a Pact consumer test for a "getOrder" API endpoint, publish the contract, and verify the provider can fulfill it
2. **Provider Verification**: On the provider side, add Pact verification to an existing Spring Boot controller — run it against contracts published by consumers
3. **Breaking Change Detection**: Make a breaking change to the provider API (remove a field), run the contract verification, and observe the failure — then roll back and add the field as optional

### Revision Notes
- Pact is the most popular consumer-driven contract testing framework
- Consumer tests generate contracts → provider tests verify them
- Pact Broker enables the "can I deploy?" workflow across microservices
- Contract testing catches breaking API changes before production deployment
- Works with REST and GraphQL; Pactflow provides a managed broker

---

## Chapter 12: Mutation Testing (PiTest)

```xml
<!-- pom.xml configuration for Pitest -->
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.15.0</version>
    <configuration>
        <targetClasses>
            <param>com.myapp.service.*</param>
        </targetClasses>
        <targetTests>
            <param>com.myapp.service.*</param>
        </targetTests>
        <mutationThreshold>80</mutationThreshold>
        <coverageThreshold>90</coverageThreshold>
    </configuration>
</plugin>
```

```bash
# Run mutation testing
mvn org.pitest:pitest-maven:mutationCoverage

# Open results: target/pit-reports/index.html
```

> 💡 **Pro Tip:** Mutation testing is the ultimate test quality metric. If you have 90% line coverage but only 40% mutation score, your tests are asserting the wrong things or not asserting anything meaningful. Run PiTest on critical business logic modules — aim for 80%+ mutation coverage. Green tests but low mutation score = false confidence.

### Practical Exercises

1. **Mutation Score Analysis**: Run PiTest on a module and analyze the surviving mutants — for each, determine if you need a new test or if the mutant is functionally equivalent
2. **Test Improvement**: Find a class with 100% line coverage but < 60% mutation coverage — identify the gaps (missing edge cases, weak assertions) and add tests to close them
3. **CI Integration**: Add PiTest to your CI pipeline as a non-blocking stage — let it run on every commit but only fail the build if mutation coverage drops below a threshold

### Revision Notes
- Mutation testing introduces small changes (mutations) to your code and checks if tests detect them
- Common mutations: flip conditionals, remove method calls, change return values, swap arithmetic operators
- Line coverage ≠ test quality — mutation testing gives you real confidence
- PiTest is the leading mutation testing framework for Java/JVM languages
- Run on targeted modules (core business logic) rather than the whole codebase — it's computationally expensive

---

## Part 18 Revision Notes

### Key Concepts
- **Testing Trophy**: Static analysis > Unit > Integration > E2E — prioritize integration tests
- **JUnit 5**: Platform, Jupiter, Vintage; assertions, parameterized, @Nested, @Timeout, extensions
- **Mockito**: mocking, stubbing, verification, spies, ArgumentCaptor, BDDMockito
- **Selenium**: WebDriver, explicit/fluent waits, Page Object Model, screenshots
- **Playwright**: auto-waiting, locators, network interception, codegen, trace viewer, cross-browser
- **Cypress**: component/E2E tests, fixtures, stubs, real-time reloading
- **REST Assured**: given/when/then, response/body extraction, schema validation
- **PyTest**: fixtures (scope), parametrize, conftest, markers, monkeypatch, pytest-asyncio
- **Spring Boot Test**: @WebMvcTest, @DataJpaTest, @SpringBootTest, TestContainers
- **Performance**: k6, Locust — load, stress, endurance, spike, soak testing
- **BDD**: Cucumber, Gherkin (Given/When/Then), feature files, step definitions
- **Contract Testing**: Pact, consumer-driven contracts, Pact Broker, can-i-deploy
- **Mutation Testing**: PiTest, mutation score, surviving mutants

### Common Mistakes
- Testing implementation details instead of behavior (refactoring breaks tests)
- Not using parameterized tests for repeated patterns (copy-paste test code)
- Flaky tests due to timing issues (using Thread.sleep instead of explicit waits)
- Over-mocking (testing mocks, not real behavior — anti-pattern)
- Writing E2E tests for everything (slow, brittle, expensive to maintain)
- Ignoring flaky tests (they erode trust in the test suite)
- Not running tests in CI on every commit
- Writing tests after the fact that assert the current behavior, not the desired behavior

### Key Interview Questions
1. Explain the difference between @Mock and @InjectMocks — how does Mockito resolve ambiguous injections?
2. How does Playwright auto-waiting work? How does it differ from Selenium's wait mechanism?
3. What is the Page Object Model in Selenium? What are its benefits?
4. How would you test a method that calls an external API? (3 approaches: mock, Testcontainers, WireMock)
5. What metrics do you monitor in a load test, and what's the difference between p50, p95, and p99 latency?
6. How do you decide between unit, integration, and E2E tests for a given feature?
7. What is contract testing, and how does it prevent breaking API changes in microservice architectures?
8. What is mutation testing, and how does it measure test quality differently than code coverage?
9. How would you handle flaky tests in CI? (quarantine, retry, root cause analysis)
10. Describe the Given/When/Then pattern in BDD — how does it differ from Arrange/Act/Assert?

---
