# Playwright Python Test Writing Skill

## Description
Use this skill when the user asks to write, debug, or refactor Playwright tests in Python. This includes end-to-end tests, API tests, browser automation scripts, and test infrastructure (fixtures, page objects, CI config).

## Activation
Activate when the user asks to:
- Write Playwright tests in Python
- Create browser automation scripts
- Set up Playwright test infrastructure
- Debug or fix Playwright test failures
- Create page object models
- Mock APIs or network requests in tests
- Set up CI/CD for Playwright tests

## Reference Files
Before writing tests, read the relevant reference files for accurate API usage:
- `locators.md` - Locator strategies (get_by_role, get_by_text, get_by_label, etc.), filtering, chaining, lists
- `actions-input.md` - Click, fill, type, select, check, upload, drag, scroll, keyboard
- `assertions.md` - expect() assertions, locator/page/response assertions, timeouts
- `network-mocking.md` - Network monitoring, request interception, API mocking, HAR replay, WebSocket mocking
- `api-testing.md` - APIRequestContext, server-side testing, fixtures for API tests
- `browser-contexts-pages.md` - Browser launch, contexts, pages, frames, dialogs, downloads, isolation
- `auth-emulation.md` - Authentication patterns, storage state, device emulation, viewport, geolocation, locale
- `page-object-model.md` - POM pattern with sync/async examples
- `debugging-tooling.md` - PWDEBUG, Inspector, trace viewer, codegen, screenshots, videos, ARIA snapshots
- `clock.md` - Clock API for time manipulation in tests
- `ci-docker.md` - CI configs (GitHub Actions, GitLab, Jenkins, Azure), Docker setup
- `api-classes.md` - Low-level APIs (Keyboard, Mouse, Touchscreen), Page utility methods, Locator data extraction, BrowserContext cookies/timeouts, Request/Response properties, Route details, ConsoleMessage, FileChooser, BrowserType launch options, Workers

## Core Principles

### 1. Use pytest-playwright
```bash
pip install pytest-playwright
playwright install
```

Tests use the `page` fixture automatically:
```python
from playwright.sync_api import Page, expect

def test_example(page: Page):
    page.goto("https://example.com")
    expect(page).to_have_title("Example Domain")
```

### 2. Prefer Resilient Locators (Priority Order)
1. `page.get_by_role()` - Best: reflects how users perceive the page
2. `page.get_by_label()` - For form controls
3. `page.get_by_placeholder()` - For inputs with placeholders
4. `page.get_by_text()` - For text content
5. `page.get_by_alt_text()` - For images
6. `page.get_by_title()` - For title attributes
7. `page.get_by_test_id()` - For data-testid attributes
8. `page.locator("css=...")` - Last resort

### 3. Use Web-First Assertions
```python
# GOOD - auto-waits and retries
expect(page.get_by_text("Success")).to_be_visible()

# BAD - no auto-waiting
assert page.get_by_text("Success").is_visible()
```

### 4. Sync vs Async
Default to **sync** API for pytest tests. Use async only when specifically needed:
```python
# Sync (default for pytest)
def test_example(page: Page):
    page.goto("https://example.com")

# Async (when needed)
async def test_example(page: Page):
    await page.goto("https://example.com")
```

### 5. Page Object Model for Large Suites
```python
class LoginPage:
    def __init__(self, page: Page):
        self.page = page
        self.username = page.get_by_label("Username")
        self.password = page.get_by_label("Password")
        self.submit = page.get_by_role("button", name="Sign in")

    def login(self, username: str, password: str):
        self.username.fill(username)
        self.password.fill(password)
        self.submit.click()
```

### 6. Auto-Waiting
Playwright auto-waits for elements to be actionable. Do NOT add manual sleeps:
```python
# GOOD - Playwright waits automatically
page.get_by_role("button", name="Submit").click()

# BAD - unnecessary sleep
import time
time.sleep(2)
page.get_by_role("button", name="Submit").click()
```

### 7. Test Isolation
Each test gets a fresh browser context. Use fixtures for shared setup:
```python
import pytest

@pytest.fixture
def authenticated_page(page: Page):
    page.goto("/login")
    page.get_by_label("Username").fill("user")
    page.get_by_label("Password").fill("pass")
    page.get_by_role("button", name="Sign in").click()
    return page
```

### 8. Network Mocking
```python
def test_with_mock(page: Page):
    page.route("**/api/data", lambda route: route.fulfill(
        json={"items": [{"id": 1, "name": "Test"}]}
    ))
    page.goto("https://example.com")
```

### 9. API Testing
```python
def test_api(playwright):
    context = playwright.request.new_context(base_url="https://api.example.com")
    response = context.get("/users")
    assert response.ok
    assert len(response.json()) > 0
    context.dispose()
```

## Common Patterns

### Wait for Navigation After Click
```python
page.get_by_text("Login").click()
page.wait_for_url("**/dashboard")
```

### Handle Dialogs
```python
page.on("dialog", lambda dialog: dialog.accept())
page.get_by_role("button", name="Delete").click()
```

### File Downloads
```python
with page.expect_download() as download_info:
    page.get_by_text("Download").click()
download = download_info.value
download.save_as("/tmp/file.pdf")
```

### Screenshots and Traces
```python
# Screenshot
page.screenshot(path="screenshot.png", full_page=True)

# Trace
context.tracing.start(screenshots=True, snapshots=True)
# ... test actions ...
context.tracing.stop(path="trace.zip")
```

### Multiple Browser Contexts (Multi-User)
```python
def test_chat(browser):
    user_ctx = browser.new_context()
    admin_ctx = browser.new_context()
    user_page = user_ctx.new_page()
    admin_page = admin_ctx.new_page()
```
