# Playwright Python Debugging & Tooling Reference

## Playwright Inspector (PWDEBUG)

```bash
# Launch tests in debug mode
PWDEBUG=1 pytest -s

# PowerShell
$env:PWDEBUG=1; pytest -s
```

Debug mode defaults:
- Browsers open in headed mode
- Default timeout disabled (infinite)

### Breakpoints
```python
page.pause()  # Pauses execution, opens Inspector
```

### Browser Console Access
```bash
PWDEBUG=console pytest -s
```

Console methods:
- `playwright.$('selector')` - Query single element
- `playwright.$$('selector')` - Query all matches
- `playwright.inspect('selector')` - Reveal in Elements panel
- `playwright.locator('selector')` - Create locator
- `playwright.selector(element)` - Generate selector from element

## Verbose API Logs

```bash
DEBUG=pw:api pytest -s
```

## Headed Mode

```python
chromium.launch(headless=False, slow_mo=100)  # slow_mo in ms
```

## Code Generator (codegen)

```bash
# Basic recording
playwright codegen demo.playwright.dev/todomvc

# With viewport
playwright codegen --viewport-size="800,600" playwright.dev

# With device
playwright codegen --device="iPhone 13" playwright.dev

# With color scheme
playwright codegen --color-scheme=dark playwright.dev

# With geolocation/locale
playwright codegen --timezone="Europe/Rome" --geolocation="41.890221,12.492348" --lang="it-IT" bing.com/maps

# Save auth state
playwright codegen github.com --save-storage=auth.json

# Load auth state
playwright codegen --load-storage=auth.json github.com
```

### Programmatic codegen (page.pause())
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    context = browser.new_context()
    page = context.new_page()
    page.pause()  # Opens codegen controls
```

## Trace Viewer

### Record Traces via pytest
```bash
pytest --tracing on                    # All tests
pytest --tracing retain-on-failure     # Only failed tests
```

### Record Traces Programmatically
```python
context.tracing.start(screenshots=True, snapshots=True, sources=True)
page = context.new_page()
page.goto("https://playwright.dev")
context.tracing.stop(path="trace.zip")
```

### View Traces
```bash
playwright show-trace trace.zip
playwright show-trace https://example.com/trace.zip
```

Or use https://trace.playwright.dev (drag and drop, runs locally in browser).

### Trace Features
- Actions timeline with DOM snapshots
- Before/During/After snapshots per action
- Source code highlighting
- Console logs
- Network requests
- Error details
- Test metadata

## Screenshots

```python
# Basic
page.screenshot(path="screenshot.png")

# Full page (scrollable)
page.screenshot(path="screenshot.png", full_page=True)

# To buffer
screenshot_bytes = page.screenshot()
print(base64.b64encode(screenshot_bytes).decode())

# Element screenshot
page.locator(".header").screenshot(path="screenshot.png")
```

## Video Recording

```python
# Enable recording
context = browser.new_context(record_video_dir="videos/")

# Custom size
context = browser.new_context(
    record_video_dir="videos/",
    record_video_size={"width": 640, "height": 480}
)

# Get video path (only after page/context close)
path = page.video.path()

# MUST close context to save videos
context.close()
```

## ARIA Snapshots

### Generate Snapshot
```python
snapshot = page.locator("body").aria_snapshot()
print(snapshot)
```

### Assert Snapshot
```python
expect(page.locator("body")).to_match_aria_snapshot("""
  - heading "Title" [level=1]
  - list "Features":
    - listitem: Feature 1
    - listitem: Feature 2
  - button "Submit"
""")
```

### Snapshot Format
```yaml
- role "name" [attribute=value]
```

Supported roles: `heading`, `button`, `link`, `list`, `listitem`, `textbox`, `checkbox`, `radio`, `img`, `navigation`, `banner`, `main`, `dialog`, `alert`, `tab`, `tabpanel`, `table`, `row`, `cell`, `group`, `paragraph`, `text`

Attributes: `[level=N]`, `[checked]`, `[disabled]`, `[expanded]`, `[pressed=true]`, `[selected]`

### Matching Modes
```yaml
# Default: contain - specified children present in order
- list:
  - listitem: Feature B

# Exact children match
- list:
  - /children: equal
  - listitem: Feature A
  - listitem: Feature B

# Exact including nested
- list:
  - /children: deep-equal
  - listitem: Feature A
```

### Regex in Snapshots
```yaml
- heading /Issues \d+/
```
