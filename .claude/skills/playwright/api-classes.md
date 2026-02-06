# Playwright Python API Classes Reference

Supplementary reference covering low-level APIs, utility methods, and class details
not fully covered in the topic-based reference files.

## Keyboard (`page.keyboard`)

```python
# Type text (sends keydown/keypress/keyup per character)
page.keyboard.type("Hello World!", delay=100)  # delay in ms between keys

# Insert text (only input event, no keydown/keyup/keypress)
page.keyboard.insert_text("嗨")  # Useful for non-Latin characters

# Press key combo (shortcut for down + up)
page.keyboard.press("Enter")
page.keyboard.press("Control+A")
page.keyboard.press("ControlOrMeta+C")  # Ctrl on Win/Linux, Cmd on Mac
page.keyboard.press("Shift+ArrowLeft")

# Hold keys (down/up separately)
page.keyboard.down("Shift")
page.keyboard.press("ArrowLeft")
page.keyboard.press("ArrowLeft")
page.keyboard.up("Shift")
page.keyboard.press("Backspace")

# Select all and delete
page.keyboard.press("ControlOrMeta+A")
page.keyboard.press("Backspace")
```

**Note:** Modifier keys affect `keyboard.down()` but NOT `keyboard.type()` or `keyboard.insert_text()`.

## Mouse (`page.mouse`)

All coordinates are CSS pixels relative to viewport top-left.

```python
# Click at coordinates
page.mouse.click(100, 200)
page.mouse.click(100, 200, button="right")
page.mouse.click(100, 200, click_count=2)  # double-click
page.mouse.dblclick(100, 200)

# Move mouse
page.mouse.move(100, 200)
page.mouse.move(200, 300, steps=10)  # interpolated movement

# Manual drag
page.mouse.move(0, 0)
page.mouse.down()
page.mouse.move(100, 100)
page.mouse.up()

# Scroll
page.mouse.wheel(0, 100)   # scroll down 100px
page.mouse.wheel(0, -100)  # scroll up 100px
page.mouse.wheel(100, 0)   # scroll right
```

## Touchscreen (`page.touchscreen`)

Requires `has_touch=True` on browser context.

```python
context = browser.new_context(has_touch=True)
page = context.new_page()
page.touchscreen.tap(100, 200)  # tap at coordinates

# Locator tap (preferred)
page.get_by_role("button").tap()
```

## Page Utility Methods

### Get/Set HTML Content
```python
html = page.content()                          # Get full HTML
page.set_content("<h1>Hello</h1>")             # Set page HTML
page.set_content(html, wait_until="domcontentloaded")
```

### Generate PDF (Chromium only)
```python
page.pdf(path="page.pdf")
page.pdf(path="page.pdf", format="A4", print_background=True)
pdf_bytes = page.pdf()  # Returns bytes
```

### Wait Methods
```python
page.wait_for_load_state("load")               # "load", "domcontentloaded", "networkidle"
page.wait_for_url("**/dashboard")              # Wait for URL pattern
page.wait_for_url(re.compile(r".*/login"))

page.wait_for_function("() => document.title === 'Ready'")
page.wait_for_function("selector => !!document.querySelector(selector)", arg=".loaded")

page.wait_for_timeout(1000)                    # Wait N ms (use sparingly, prefer assertions)
page.wait_for_selector("#element")             # DEPRECATED: use locator.wait_for()
```

### Expose Functions to Browser
```python
# Expose Python function callable from browser JS
def compute(a, b):
    return a * b

page.expose_function("compute", compute)
result = page.evaluate("async () => await window.compute(3, 7)")
# result == 21

# Expose binding (receives source info)
def log_click(source, x, y):
    print(f"Clicked at {x},{y} on {source['url']}")
page.expose_binding("logClick", log_click)
```

### Add Script/Style Tags
```python
page.add_script_tag(content="window.myVar = 42")
page.add_script_tag(url="https://cdn.example.com/lib.js")
page.add_script_tag(path="./local-script.js")

page.add_style_tag(content="body { background: red; }")
page.add_style_tag(url="https://cdn.example.com/style.css")
```

### Locator Handler (Overlay Detection)
```python
# Dismiss overlays automatically when they appear
page.add_locator_handler(
    page.get_by_role("dialog"),
    lambda overlay: overlay.get_by_role("button", name="Close").click()
)
```

### Page Navigation
```python
page.go_back()
page.go_back(wait_until="domcontentloaded")
page.go_forward()
page.reload()
page.bring_to_front()  # Activate this tab
```

### Drag and Drop (Selector-Based)
```python
page.drag_and_drop("#source", "#target")
page.drag_and_drop("#source", "#target", source_position={"x": 0, "y": 0},
                    target_position={"x": 50, "y": 50})
```

## Locator Data Extraction Methods

```python
locator = page.locator(".my-element")

# Text content
locator.text_content()        # textContent (includes hidden text)
locator.inner_text()          # innerText (visible text only)
locator.inner_html()          # innerHTML
locator.all_inner_texts()     # List of innerText for all matches
locator.all_text_contents()   # List of textContent for all matches

# Input value
locator.input_value()         # Value of <input>, <textarea>, <select>

# Attributes
locator.get_attribute("href")
locator.get_attribute("data-id")

# Bounding box
box = locator.bounding_box()  # {"x": 0, "y": 0, "width": 100, "height": 50} or None

# State checks (non-retrying, use expect() for assertions)
locator.is_visible()
locator.is_hidden()
locator.is_enabled()
locator.is_disabled()
locator.is_editable()
locator.is_checked()
```

### Locator Wait and Utility

```python
# Wait for element state
locator.wait_for()                          # Default: wait for "visible"
locator.wait_for(state="attached")          # "attached", "detached", "visible", "hidden"
locator.wait_for(state="hidden", timeout=5000)

# Clear input field
locator.clear()

# Mobile tap
locator.tap()  # Requires has_touch=True context

# Highlight for debugging
locator.highlight()

# Add description for tracing
locator.describe("Login button")
```

## BrowserContext Cookie & Timeout Management

```python
# Cookies
context.add_cookies([{
    "name": "session",
    "value": "abc123",
    "domain": "example.com",
    "path": "/",
}])
cookies = context.cookies()                         # All cookies
cookies = context.cookies("https://example.com")    # Filtered by URL
context.clear_cookies()                             # Clear all
context.clear_cookies(name="session")               # Clear specific
context.clear_cookies(domain="example.com")         # Clear by domain

# Timeouts
context.set_default_timeout(10_000)                 # 10s for all actions
context.set_default_navigation_timeout(30_000)      # 30s for navigations

# Extra HTTP headers (applied to all requests)
context.set_extra_http_headers({"X-Custom": "value"})

# Expose function to all pages in context
context.expose_function("sha256", lambda text: hashlib.sha256(text.encode()).hexdigest())
context.expose_binding("pageURL", lambda source: source["url"])

# Remove routes
context.unroute("**/api/**")
context.unroute_all()                               # Remove all routes

# CDP session (Chromium only)
cdp = context.new_cdp_session(page)
cdp.send("Runtime.evaluate", {"expression": "1+1"})

# API requests sharing context cookies
response = context.request.get("https://api.example.com/data")
assert response.ok
```

## Request Object Properties

```python
request = response.request  # or from event handler

request.url                    # Full URL
request.method                 # "GET", "POST", etc.
request.headers                # Dict of headers (lowercase keys)
request.post_data              # Body as string (or None)
request.post_data_buffer       # Body as bytes (or None)
request.post_data_json         # Parsed JSON body (for form-urlencoded or JSON)
request.resource_type          # "document", "stylesheet", "image", "script", "xhr", "fetch", etc.
request.is_navigation_request() # True if driving frame navigation
request.frame                  # Frame that initiated request
request.timing                 # Performance timing data
request.failure                # Error text if failed, else None
request.redirected_from        # Original request if redirected
request.redirected_to          # New request if this was redirected

# Full headers (including security headers)
all_headers = request.all_headers()
value = request.header_value("content-type")

# Size info
sizes = request.sizes()  # {"requestBodySize": N, "requestHeadersSize": N, ...}
```

## Response Object Properties

```python
response.url                   # Response URL
response.status                # HTTP status code (200, 404, etc.)
response.status_text           # "OK", "Not Found", etc.
response.ok                    # True if status 200-299
response.headers               # Dict of headers (lowercase keys)
response.from_service_worker   # True if from Service Worker

response.text()                # Body as text
response.json()                # Body parsed as JSON
response.body()                # Body as bytes

response.finished()            # Wait for response to finish (returns None)
response.request               # Matching Request object
response.frame                 # Frame that initiated

# Full headers
all_headers = response.all_headers()
value = response.header_value("content-type")
values = response.header_values("set-cookie")

# Security/Server info
security = response.security_details()  # {"issuer": ..., "protocol": ..., "validFrom": ...}
server = response.server_addr()         # {"ipAddress": "...", "port": N}
```

## Route Methods (Complete)

```python
def handle(route: Route):
    # Fulfill with custom response
    route.fulfill(status=200, json={"ok": True})
    route.fulfill(body="hello", content_type="text/plain")
    route.fulfill(path="./mock-data.json")
    route.fulfill(response=api_response)  # From route.fetch()

    # Continue to network (with optional overrides)
    route.continue_()
    route.continue_(headers={**route.request.headers, "X-New": "value"})
    route.continue_(method="POST", post_data='{"key": "value"}')
    route.continue_(url="https://other-api.com/data")

    # Fallback - like continue_ but allows other matching route handlers to run first
    route.fallback()
    route.fallback(headers={**route.request.headers, "X-New": "value"})

    # Abort the request
    route.abort()
    route.abort("failed")         # Error codes: "aborted", "accessdenied", "addressunreachable",
    route.abort("timedout")       # "blockedbyclient", "blockedbyresponse", "connectionaborted",
                                  # "connectionclosed", "connectionfailed", "connectionrefused",
                                  # "connectionreset", "internetdisconnected", "namenotresolved",
                                  # "timedout", "failed"

    # Fetch actual response for modification
    response = route.fetch()
    body = response.json()
    body["injected"] = True
    route.fulfill(response=response, json=body)
```

### Route.fallback() vs Route.continue_()
`continue_()` immediately sends to network. `fallback()` passes to the next matching route handler first, enabling layered middleware-style interception:

```python
# First handler: add auth header
def add_auth(route):
    route.fallback(headers={**route.request.headers, "Authorization": "Bearer token"})

# Second handler: log requests
def log_request(route):
    print(f"Request: {route.request.url}")
    route.fallback()

# Handlers run in reverse registration order
page.route("**/*", log_request)
page.route("**/*", add_auth)
```

## ConsoleMessage

```python
def handle_console(msg):
    print(f"[{msg.type}] {msg.text}")
    # msg.type: "log", "debug", "info", "error", "warning", "dir",
    #           "dirxml", "table", "trace", "clear", "assert", etc.
    print(f"Location: {msg.location}")  # {"url": "...", "lineNumber": N, "columnNumber": N}

    for arg in msg.args:
        print(arg.json_value())

page.on("console", handle_console)

# Wait for specific console message
with page.expect_console_message(lambda msg: "error" in msg.text) as msg_info:
    page.evaluate("console.error('test error')")
console_msg = msg_info.value
```

## Assertion Parameters (Advanced)

```python
# ignore_case - case-insensitive text matching
expect(locator).to_have_text("hello world", ignore_case=True)
expect(locator).to_contain_text("hello", ignore_case=True)
expect(locator).to_have_attribute("title", "hello", ignore_case=True)
expect(locator).to_have_accessible_name("submit", ignore_case=True)

# use_inner_text - use innerText instead of textContent
expect(locator).to_have_text("visible only", use_inner_text=True)
expect(locator).to_contain_text("visible", use_inner_text=True)

# to_be_checked with indeterminate
expect(locator).to_be_checked(indeterminate=True)

# to_be_in_viewport with ratio
expect(locator).to_be_in_viewport(ratio=0.5)  # At least 50% visible

# to_have_accessible_error_message (new)
expect(locator).to_have_accessible_error_message("This field is required")
expect(locator).to_have_accessible_error_message(re.compile(r"required"))
```

## FileChooser

```python
with page.expect_file_chooser() as fc_info:
    page.get_by_text("Upload").click()
file_chooser = fc_info.value

file_chooser.is_multiple()           # True if accepts multiple files
file_chooser.page                    # The Page object
file_chooser.element                 # The input ElementHandle

file_chooser.set_files("file.pdf")
file_chooser.set_files(["a.pdf", "b.pdf"])
file_chooser.set_files({"name": "test.txt", "mimeType": "text/plain", "buffer": b"content"})
```

## BrowserType Launch Options

```python
# Standard launch
browser = playwright.chromium.launch(
    headless=True,          # Default True
    slow_mo=100,            # Slow down operations by N ms
    channel="chrome",       # "chrome", "msedge", "chrome-beta", etc.
    args=["--disable-gpu"], # Additional browser args
    timeout=30_000,         # Launch timeout
    downloads_path="/tmp",  # Default download location
    proxy={"server": "http://proxy:8080", "username": "u", "password": "p"},
)

# Persistent context (extensions, user data)
context = playwright.chromium.launch_persistent_context(
    "/path/to/user-data",
    headless=False,
    channel="chromium",
    args=["--load-extension=/path/to/ext"],
)

# Connect to remote browser
browser = playwright.chromium.connect("ws://localhost:3000/")
browser = playwright.chromium.connect_over_cdp("http://localhost:9222")
```

## Workers

```python
# Listen for web workers
page.on("worker", lambda worker: print(f"Worker: {worker.url}"))

with page.expect_worker(lambda w: "worker.js" in w.url) as worker_info:
    page.goto("https://example.com")
worker = worker_info.value
result = worker.evaluate("() => self.data")
```
