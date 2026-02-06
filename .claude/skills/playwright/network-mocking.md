# Playwright Python Network & Mocking Reference

## Network Event Monitoring

```python
# Log all requests and responses
page.on("request", lambda request: print(">>", request.method, request.url))
page.on("response", lambda response: print("<<", response.status, response.url))
page.goto("https://example.com")
```

## Wait for Specific Response

```python
# Glob pattern
with page.expect_response("**/api/fetch_data") as response_info:
    page.get_by_text("Update").click()
response = response_info.value

# Regex
with page.expect_response(re.compile(r"\.jpeg$")) as response_info:
    page.get_by_text("Update").click()

# Predicate function
with page.expect_response(lambda r: "token" in r.url) as response_info:
    page.get_by_text("Update").click()
```

## Wait for Request

```python
with page.expect_request("**/*logo*.png") as first:
    page.goto("https://wikipedia.org")
print(first.value.url)
```

## Mock API Requests

```python
def test_mock_api(page: Page):
    def handle(route: Route):
        json = [{"name": "Strawberry", "id": 21}]
        route.fulfill(json=json)

    page.route("*/**/api/v1/fruits", handle)
    page.goto("https://demo.playwright.dev/api-mocking")
    expect(page.get_by_text("Strawberry")).to_be_visible()
```

## Modify API Responses

```python
def test_modify_response(page: Page):
    def handle(route: Route):
        response = route.fetch()  # Call real API
        json = response.json()
        json.append({"name": "Loquat", "id": 100})
        route.fulfill(response=response, json=json)

    page.route("**/api/v1/fruits", handle)
    page.goto("https://example.com")
```

## Modify Requests

```python
# Delete a header
def handle_route(route):
    headers = route.request.headers
    del headers["x-secret"]
    route.continue_(headers=headers)
page.route("**/*", handle_route)

# Change method
page.route("**/*", lambda route: route.continue_(method="POST"))
```

## Modify Responses (Intercept and Alter)

```python
def handle_route(route: Route) -> None:
    response = route.fetch()
    body = response.text()
    body = body.replace("<title>", "<title>My prefix:")
    route.fulfill(
        response=response,
        body=body,
        headers={**response.headers, "content-type": "text/html"})
page.route("**/title.html", handle_route)
```

## Abort Requests

```python
# Block images by extension
page.route("**/*.{png,jpg,jpeg}", lambda route: route.abort())

# Block by resource type
page.route("**/*", lambda route: route.abort()
    if route.request.resource_type == "image" else route.continue_())
```

## Context-Level Routing (Applies to All Pages)

```python
context.route("**/api/login",
    lambda route: route.fulfill(status=200, body="accept"))
```

## HAR File Mocking

### Record HAR
```python
def test_record_har(page: Page):
    page.route_from_har("./hars/fruit.har", url="*/**/api/v1/fruits", update=True)
    page.goto("https://demo.playwright.dev/api-mocking")
```

### Replay from HAR
```python
def test_replay_har(page: Page):
    page.route_from_har("./hars/fruit.har", url="*/**/api/v1/fruits", update=False)
    page.goto("https://demo.playwright.dev/api-mocking")
    expect(page.get_by_text("Playwright", exact=True)).to_be_visible()
```

### Record HAR via CLI
```bash
playwright open --save-har=example.har --save-har-glob="**/api/**" https://example.com
```

## WebSocket Mocking

```python
from typing import Union

# Mock entire WebSocket
def message_handler(ws: WebSocketRoute, message: Union[str, bytes]):
    if message == "request":
        ws.send("response")

page.route_web_socket("wss://example.com/ws", lambda ws: ws.on_message(
    lambda message: message_handler(ws, message)))

# Intercept and modify WebSocket messages (connect to real server)
def handler(ws: WebSocketRoute):
    server = ws.connect_to_server()
    ws.on_message(lambda message: server.send(
        "modified" if message == "request" else message))

page.route_web_socket("wss://example.com/ws", handler)
```

## WebSocket Monitoring

```python
def on_web_socket(ws):
    print(f"WebSocket opened: {ws.url}")
    ws.on("framesent", lambda payload: print(payload))
    ws.on("framereceived", lambda payload: print(payload))
    ws.on("close", lambda payload: print("WebSocket closed"))
page.on("websocket", on_web_socket)
```

## HTTP Authentication

```python
context = browser.new_context(
    http_credentials={"username": "bill", "password": "pa55w0rd"})
page = context.new_page()
page.goto("https://example.com")
```

## HTTP Proxy

```python
# Global proxy
browser = chromium.launch(proxy={
    "server": "http://myproxy.com:3128",
    "username": "usr",
    "password": "pwd"})

# Per-context proxy
context = browser.new_context(proxy={"server": "http://myproxy.com:3128"})
```

## Glob URL Pattern Syntax

- `*` matches any characters except `/`
- `**` matches any characters including `/`
- `?` matches literal question mark only
- `{}` matches comma-separated options: `**/*.{png,jpg,jpeg}`
- Glob must match entire URL, not partial

## Service Workers Note
If network events are missing with `page.route()`, disable service workers:
```python
context = browser.new_context(service_workers='block')
```
