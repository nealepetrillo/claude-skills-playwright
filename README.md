# claude-skills-playwright

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that gives Claude deep knowledge of the [Playwright](https://playwright.dev/) Python API, enabling it to write, debug, and refactor Playwright tests with accurate, up-to-date API usage.

## What is a Claude Code Skill?

Claude Code skills are structured reference documents that live in `.claude/skills/` and activate automatically when relevant. When you ask Claude to help with Playwright tests, this skill provides it with comprehensive API knowledge so it generates correct, idiomatic code instead of relying on potentially outdated training data.

## What This Skill Covers

The skill contains 13 reference documents (~2,500 lines) covering the full Playwright Python API:

| Reference File | Topics |
|---|---|
| **skill.md** | Core principles, activation triggers, common patterns |
| **locators.md** | `get_by_role`, `get_by_text`, `get_by_label`, filtering, chaining, lists, frames, shadow DOM |
| **actions-input.md** | Click, fill, type, select, check, upload, drag, scroll, keyboard shortcuts |
| **assertions.md** | `expect()` API, locator/page/response assertions, timeouts, negation, ARIA snapshots |
| **network-mocking.md** | Request interception, API mocking, HAR recording/replay, WebSocket mocking |
| **api-testing.md** | `APIRequestContext`, server-side REST testing, fixtures |
| **browser-contexts-pages.md** | Browser launch, contexts, pages, frames, dialogs, downloads, events |
| **auth-emulation.md** | Authentication state, device emulation, viewport, geolocation, locale, permissions |
| **page-object-model.md** | POM pattern with sync/async examples and pytest fixtures |
| **debugging-tooling.md** | Inspector, trace viewer, codegen, screenshots, videos, verbose logging |
| **clock.md** | Clock API for time manipulation (freeze, fast-forward, manual tick) |
| **ci-docker.md** | GitHub Actions, GitLab CI, Azure Pipelines, Jenkins, CircleCI, Docker |
| **api-classes.md** | Keyboard, Mouse, Touchscreen, Route, Request/Response, ConsoleMessage, launch options |

## Installation

Clone this repo and add it to your Claude Code project configuration:

```bash
git clone https://github.com/your-org/claude-skills-playwright.git
```

Copy the skill into your project's `.claude/skills/` directory:

```bash
cp -r claude-skills-playwright/.claude/skills/playwright your-project/.claude/skills/
```

Or add it as a git submodule:

```bash
git submodule add https://github.com/your-org/claude-skills-playwright.git .claude/skills-repo/playwright
```

## Usage

Once installed, the skill activates automatically when you ask Claude Code to:

- Write Playwright tests in Python
- Create browser automation scripts
- Set up Playwright test infrastructure
- Debug or fix Playwright test failures
- Create page object models
- Mock APIs or network requests in tests
- Set up CI/CD for Playwright tests

### Example Prompts

```
Write a Playwright test that logs into the app and verifies the dashboard loads.
```

```
Create a page object model for the checkout flow.
```

```
Mock the /api/users endpoint and test the user list component.
```

```
Set up GitHub Actions to run our Playwright tests in CI.
```

```
Debug why this test is flaky — it fails intermittently on the submit button click.
```

## Key Principles

The skill teaches Claude to follow these Playwright best practices:

1. **Resilient locators** — prefer `get_by_role()` and other semantic locators over CSS/XPath selectors
2. **Web-first assertions** — use `expect()` with auto-waiting instead of raw `assert` statements
3. **No manual sleeps** — rely on Playwright's built-in auto-waiting
4. **Test isolation** — each test gets a fresh browser context
5. **Sync API by default** — use async only when specifically needed
6. **Page Object Model** — for large test suites, encapsulate selectors and interactions

## Requirements

This skill targets projects using:

- **Python 3.12+**
- **pytest-playwright** (`pip install pytest-playwright`)
- **Playwright browsers** (`playwright install`)

## License

MIT
