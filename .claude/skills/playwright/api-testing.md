# Playwright Python API Testing Reference

## Overview
Playwright enables direct REST API testing without loading web pages via `APIRequestContext`.

## Setup Fixture

```python
import os
from typing import Generator
import pytest
from playwright.sync_api import Playwright, APIRequestContext

GITHUB_API_TOKEN = os.getenv("GITHUB_API_TOKEN")
assert GITHUB_API_TOKEN, "GITHUB_API_TOKEN is not set"

GITHUB_USER = os.getenv("GITHUB_USER", "your-username")
GITHUB_REPO = os.getenv("GITHUB_REPO", "test-repo")

@pytest.fixture(scope="session")
def api_request_context(
    playwright: Playwright,
) -> Generator[APIRequestContext, None, None]:
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"token {GITHUB_API_TOKEN}",
    }
    request_context = playwright.request.new_context(
        base_url="https://api.github.com", extra_http_headers=headers
    )
    yield request_context
    request_context.dispose()
```

## Test API Endpoints

```python
def test_should_create_bug_report(api_request_context: APIRequestContext) -> None:
    data = {
        "title": "[Bug] report 1",
        "body": "Bug description",
    }
    new_issue = api_request_context.post(
        f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues", data=data
    )
    assert new_issue.ok

    issues = api_request_context.get(f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues")
    assert issues.ok
    issues_response = issues.json()
    issue = list(
        filter(lambda issue: issue["title"] == "[Bug] report 1", issues_response)
    )[0]
    assert issue["body"] == "Bug description"
```

## Setup and Teardown via API

```python
@pytest.fixture(scope="session", autouse=True)
def create_test_repository(
    api_request_context: APIRequestContext,
) -> Generator[None, None, None]:
    # Setup: create test repo
    new_repo = api_request_context.post("/user/repos", data={"name": GITHUB_REPO})
    assert new_repo.ok
    yield
    # Teardown: delete test repo
    deleted_repo = api_request_context.delete(f"/repos/{GITHUB_USER}/{GITHUB_REPO}")
    assert deleted_repo.ok
```

## Prepare Server State Before UI Tests

```python
def test_last_created_issue_should_be_first_in_the_list(
    api_request_context: APIRequestContext, page: Page
) -> None:
    def create_issue(title: str) -> None:
        data = {"title": title, "body": "Feature description"}
        new_issue = api_request_context.post(
            f"/repos/{GITHUB_USER}/{GITHUB_REPO}/issues", data=data
        )
        assert new_issue.ok

    create_issue("[Feature] request 1")
    create_issue("[Feature] request 2")
    page.goto(f"https://github.com/{GITHUB_USER}/{GITHUB_REPO}/issues")
    first_issue = page.locator("a[data-hovercard-type='issue']").first
    expect(first_issue).to_have_text("[Feature] request 2")
```

## Validate Server State After UI Actions

```python
def test_last_created_issue_should_be_on_the_server(
    api_request_context: APIRequestContext, page: Page
) -> None:
    page.goto(f"https://github.com/{GITHUB_USER}/{GITHUB_REPO}/issues")
    page.locator("text=New issue").click()
    page.locator("[aria-label='Title']").fill("Bug report 1")
    page.locator("[aria-label='Comment body']").fill("Bug description")
    page.locator("text=Submit new issue").click()
    issue_id = page.url.split("/")[-1]

    new_issue = api_request_context.get(
        f"https://github.com/{GITHUB_USER}/{GITHUB_REPO}/issues/{issue_id}"
    )
    assert new_issue.ok
    assert new_issue.json()["title"] == "[Bug] report 1"
```

## Reuse Authentication State Between API and Browser

```python
request_context = playwright.request.new_context(
    http_credentials={"username": "test", "password": "test"}
)
request_context.get("https://api.example.com/login")

# Transfer auth state from API context to browser context
state = request_context.storage_state()
context = browser.new_context(storage_state=state)
```

## Available HTTP Methods
- `api_request_context.get(url)`
- `api_request_context.post(url, data=...)`
- `api_request_context.put(url, data=...)`
- `api_request_context.patch(url, data=...)`
- `api_request_context.delete(url)`
- `api_request_context.head(url)`

## Response Methods
- `response.ok` - True if status 200-299
- `response.status` - HTTP status code
- `response.json()` - Parse JSON body
- `response.text()` - Get text body
- `response.body()` - Get raw bytes
- `response.headers` - Response headers
