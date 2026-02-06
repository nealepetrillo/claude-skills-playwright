# Playwright Python Page Object Model Reference

## Purpose
- **Simplify authoring** with higher-level API for your app
- **Simplify maintenance** by centralizing selectors, avoiding repetition

## Sync Page Object

```python
class SearchPage:
    def __init__(self, page):
        self.page = page
        self.search_term_input = page.locator('[aria-label="Enter your search term"]')

    def navigate(self):
        self.page.goto("https://bing.com")

    def search(self, text):
        self.search_term_input.fill(text)
        self.search_term_input.press("Enter")
```

### Usage in Sync Tests
```python
from models.search import SearchPage

page = browser.new_page()
search_page = SearchPage(page)
search_page.navigate()
search_page.search("search query")
```

## Async Page Object

```python
class SearchPage:
    def __init__(self, page):
        self.page = page
        self.search_term_input = page.locator('[aria-label="Enter your search term"]')

    async def navigate(self):
        await self.page.goto("https://bing.com")

    async def search(self, text):
        await self.search_term_input.fill(text)
        await self.search_term_input.press("Enter")
```

### Usage in Async Tests
```python
from models.search import SearchPage

page = await browser.new_page()
search_page = SearchPage(page)
await search_page.navigate()
await search_page.search("search query")
```

## Recommended Pattern for pytest-playwright

```python
# pages/login_page.py
from playwright.sync_api import Page, expect

class LoginPage:
    URL = "/login"

    def __init__(self, page: Page):
        self.page = page
        self.username_input = page.get_by_label("Username")
        self.password_input = page.get_by_label("Password")
        self.submit_button = page.get_by_role("button", name="Sign in")
        self.error_message = page.get_by_role("alert")

    def navigate(self):
        self.page.goto(self.URL)

    def login(self, username: str, password: str):
        self.username_input.fill(username)
        self.password_input.fill(password)
        self.submit_button.click()

    def expect_error(self, message: str):
        expect(self.error_message).to_have_text(message)

    def expect_logged_in(self):
        expect(self.page).to_have_url("/dashboard")
```

```python
# tests/test_login.py
import pytest
from playwright.sync_api import Page
from pages.login_page import LoginPage

@pytest.fixture
def login_page(page: Page) -> LoginPage:
    login = LoginPage(page)
    login.navigate()
    return login

def test_successful_login(login_page: LoginPage):
    login_page.login("admin", "password123")
    login_page.expect_logged_in()

def test_invalid_credentials(login_page: LoginPage):
    login_page.login("admin", "wrong")
    login_page.expect_error("Invalid credentials")
```

## Key Characteristics
- Encapsulates page-specific selectors and interactions
- Provides reusable methods for common workflows
- Separates test logic from page implementation
- Locators are defined once in the constructor (lazy - resolved when used)
- Works with both sync and async APIs
