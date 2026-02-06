# Playwright Python Authentication & Emulation Reference

## Authentication Patterns

### Manual Login
```python
page = context.new_page()
page.goto('https://github.com/login')
page.get_by_label("Username or email address").fill("username")
page.get_by_label("Password").fill("password")
page.get_by_role("button", name="Sign in").click()
```

### Reusing Authenticated State (Recommended)
Save and restore cookies/localStorage across test runs:

```python
# Save state
storage = context.storage_state(path="state.json")

# Restore state in new context
context = browser.new_context(storage_state="state.json")
```

### Setup Authentication Directory
```bash
mkdir -p playwright/.auth
echo 'playwright/.auth' >> .gitignore
```
**Warning:** Never commit auth state files - they contain sensitive cookies/tokens.

### Session Storage Handling
```python
# Save session storage
session_storage = page.evaluate("() => JSON.stringify(sessionStorage)")
os.environ["SESSION_STORAGE"] = session_storage

# Restore session storage
session_storage = os.environ["SESSION_STORAGE"]
context.add_init_script("""(storage => {
  if (window.location.hostname === 'example.com') {
    const entries = JSON.parse(storage)
    for (const [key, value] of Object.entries(entries)) {
      window.sessionStorage.setItem(key, value)
    }
  }
})('""" + session_storage + "')")
```

## Device Emulation

```python
# Use device preset
iphone_13 = playwright.devices['iPhone 13']
context = browser.new_context(**iphone_13)
```

### Command Line
```bash
pytest test_login.py --device="iPhone 13"
```

## Viewport

```python
# Set via context
context = browser.new_context(viewport={'width': 1280, 'height': 1024})

# Resize page
page.set_viewport_size({"width": 1600, "height": 1200})

# High DPI
context = browser.new_context(
    viewport={'width': 2560, 'height': 1440},
    device_scale_factor=2
)
```

## Locale & Timezone

```python
context = browser.new_context(
    locale='de-DE',
    timezone_id='Europe/Berlin'
)
```

## Geolocation

```python
context = browser.new_context(
    geolocation={"longitude": 41.890221, "latitude": 12.492348},
    permissions=["geolocation"]
)

# Update geolocation mid-test
context.set_geolocation({"longitude": 48.858455, "latitude": 2.294474})
```

## Permissions

```python
context = browser.new_context(permissions=['notifications'])
context.grant_permissions(['notifications'], origin='https://skype.com')
context.clear_permissions()
```

## Color Scheme

```python
# Via context
context = browser.new_context(color_scheme='dark')

# Via page
page = browser.new_page(color_scheme='dark')
page.emulate_media(color_scheme='dark')

# Print media
page.emulate_media(media='print')
```

## User Agent

```python
context = browser.new_context(user_agent='My user agent')
```

## Offline Mode

```python
context = browser.new_context(offline=True)
```

## JavaScript Disabled

```python
context = browser.new_context(java_script_enabled=False)
```

## Mobile Mode

```python
context = browser.new_context(is_mobile=True)
```
