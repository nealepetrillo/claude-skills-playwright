# Playwright Python Browser, Contexts & Pages Reference

## Installation & Browser Setup

```bash
pip install pytest-playwright
playwright install                    # Install all browsers
playwright install chromium           # Install specific browser
playwright install --with-deps        # Install with system dependencies
playwright install --with-deps chromium
```

### Running Tests on Different Browsers
```bash
pytest test_login.py --browser webkit
pytest test_login.py --browser webkit --browser firefox
pytest test_login.py --device="iPhone 13"
pytest test_login.py --browser-channel msedge
```

### Browser Channels
`chrome`, `msedge`, `chrome-beta`, `msedge-beta`, `chrome-dev`, `msedge-dev`, `chrome-canary`, `msedge-canary`

```python
browser = p.chromium.launch(channel="msedge")
```

## Browser Contexts (Isolation)

Each test gets a fresh browser context (like incognito). No state leaks between tests.

```python
# Create context manually
browser = playwright.chromium.launch()
context = browser.new_context()
page = context.new_page()
```

### Multiple Contexts (Multi-User Testing)
```python
def run(playwright: Playwright):
    browser = playwright.chromium.launch()
    user_context = browser.new_context()
    admin_context = browser.new_context()
    user_page = user_context.new_page()
    admin_page = admin_context.new_page()
    # Interact with both independently
```

## Pages

```python
# Create page
page = context.new_page()
page.goto('http://example.com')

# Multiple pages in one context
page_one = context.new_page()
page_two = context.new_page()
all_pages = context.pages
```

### Handle New Pages (target="_blank")
```python
with context.expect_page() as new_page_info:
    page.get_by_text("open new tab").click()
new_page = new_page_info.value
new_page.get_by_role("button").click()
print(new_page.title())
```

### Event Listener for New Pages
```python
def handle_page(page):
    page.wait_for_load_state()
    print(page.title())
context.on("page", handle_page)
```

### Handle Popups
```python
with page.expect_popup() as popup_info:
    page.get_by_text("open the popup").click()
popup = popup_info.value
popup.get_by_role("button").click()
```

## Frames (iframes)

### Frame Locator (Recommended)
```python
username = page.frame_locator('.frame-class').get_by_label('User Name')
username.fill('John')
```

### Frame Object
```python
# By name
frame = page.frame('frame-login')
frame.fill('#username-input', 'John')

# By URL pattern
frame = page.frame(url=r'.*domain.*')
```

## Dialogs

### Default Behavior
Dialogs are auto-dismissed. Register handlers to customize:

```python
page.on("dialog", lambda dialog: dialog.accept())
page.get_by_role("button").click()

# Accept with value (for prompt dialogs)
page.once("dialog", lambda dialog: dialog.accept("2021"))
page.evaluate("prompt('Enter a number:')")
```

**Critical:** Dialog handlers MUST call `accept()` or `dismiss()`. Not handling a dialog causes the action to hang.

### Beforeunload Dialog
```python
def handle_dialog(dialog):
    assert dialog.type == 'beforeunload'
    dialog.dismiss()

page.on('dialog', handle_dialog)
page.close(run_before_unload=True)
```

## Downloads

```python
# Wait for download
with page.expect_download() as download_info:
    page.get_by_text("Download file").click()
download = download_info.value
download.save_as("/path/to/save/" + download.suggested_filename)

# Event-based
page.on("download", lambda download: print(download.path()))
```

**Note:** Downloads are deleted when browser context closes. Use `downloads_path` in `browser_type.launch()` to persist.

## Events

### Wait for Events
```python
with page.expect_request("**/*logo*.png") as first:
    page.goto("https://wikipedia.org")
print(first.value.url)
```

### Add/Remove Listeners
```python
def print_request(request):
    print("Request sent: " + request.url)

page.on("request", print_request)
page.goto("https://example.com")
page.remove_listener("request", print_request)
```

### One-Off Listeners
```python
page.once("dialog", lambda dialog: dialog.accept("2021"))
```

## Chrome Extensions

Extensions require persistent contexts:

```python
path_to_extension = "./my-extension"
context = playwright.chromium.launch_persistent_context(
    "",  # user data dir
    channel="chromium",
    args=[
        f"--disable-extensions-except={path_to_extension}",
        f"--load-extension={path_to_extension}",
    ],
)

# Get extension service worker
if len(context.service_workers) == 0:
    service_worker = context.wait_for_event('serviceworker')
else:
    service_worker = context.service_workers[0]

# Get extension ID for popup testing
extension_id = service_worker.url.split("/")[2]
page.goto(f"chrome-extension://{extension_id}/popup.html")
```

## JavaScript Evaluation

```python
# Simple evaluation
href = page.evaluate('() => document.location.href')
status = page.evaluate("""async () => {
  response = await fetch(location.href)
  return response.status
}""")

# Pass arguments
data = "some data"
result = page.evaluate("data => { window.myApp.use(data) }", data)

# Multiple arguments via object
page.evaluate('({a, b}) => a + b', {'a': 1, 'b': 2})

# Init scripts (run before page load)
page.add_init_script(path="mocks/preload.js")
```

## Custom Selector Engines

```python
tag_selector = """
{
  query(root, selector) { return root.querySelector(selector); },
  queryAll(root, selector) { return Array.from(root.querySelectorAll(selector)); }
}
"""
playwright.selectors.register("tag", tag_selector)
button = page.locator("tag=button")
```

**Note:** Register selectors before page creation.

## WebView2 (Windows)

```python
browser = playwright.chromium.connect_over_cdp("http://localhost:9222")
context = browser.contexts[0]
page = context.pages[0]
```
