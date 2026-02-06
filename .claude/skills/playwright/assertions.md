# Playwright Python Assertions Reference

## Overview
Playwright assertions auto-wait and retry until conditions pass. Default timeout: 5 seconds.

```python
from playwright.sync_api import Page, expect
```

## Custom Error Messages
```python
expect(page.get_by_text("Name"), "should be logged in").to_be_visible()
```

## Timeout Configuration

```python
# Global timeout
expect.set_options(timeout=10_000)

# Per-assertion
expect(page.get_by_text("Name")).to_be_visible(timeout=10_000)
```

## Negation
All assertions support `.not_to_*()`:
```python
expect(page.get_by_role("button")).not_to_be_visible()
expect(page.get_by_role("button")).not_to_be_disabled()
```

## Locator Assertions

### Visibility & State
```python
expect(locator).to_be_visible()
expect(locator).to_be_hidden()
expect(locator).to_be_attached()        # In DOM
expect(locator).to_be_enabled()
expect(locator).to_be_disabled()
expect(locator).to_be_editable()
expect(locator).to_be_focused()
expect(locator).to_be_empty()
expect(locator).to_be_in_viewport()
```

### Checkbox
```python
expect(locator).to_be_checked()
expect(locator).not_to_be_checked()
```

### Text Content
```python
expect(locator).to_have_text("exact text")
expect(locator).to_have_text(re.compile(r"pattern"))
expect(locator).to_contain_text("substring")

# Multiple elements
expect(locator).to_have_text(["text1", "text2", "text3"])
expect(locator).to_contain_text(["sub1", "sub2"])
```

### Attributes & Properties
```python
expect(locator).to_have_attribute("href", "/docs")
expect(locator).to_have_attribute("href", re.compile(r"/docs.*"))
expect(locator).to_have_class("active")
expect(locator).to_have_class(["item active", "item"])
expect(locator).to_contain_class("active")
expect(locator).to_have_id("main-content")
expect(locator).to_have_css("color", "rgb(0, 0, 0)")
expect(locator).to_have_js_property("loaded", True)
expect(locator).to_have_role("button")
```

### Input Values
```python
expect(locator).to_have_value("text value")
expect(locator).to_have_value(re.compile(r"pattern"))
expect(locator).to_have_values(["opt1", "opt2"])  # Multi-select
```

### Count
```python
expect(locator).to_have_count(3)
```

### Accessibility
```python
expect(locator).to_have_accessible_name("Submit form")
expect(locator).to_have_accessible_description("Click to submit")
expect(locator).to_match_aria_snapshot("""
  - heading "Title" [level=1]
  - button "Submit"
""")
```

## Page Assertions

```python
expect(page).to_have_title("Page Title")
expect(page).to_have_title(re.compile(r".*Title.*"))
expect(page).to_have_url("https://example.com/path")
expect(page).to_have_url(re.compile(r".*/path.*"))
```

## API Response Assertions

```python
expect(response).to_be_ok()  # Status 200-299
expect(response).not_to_be_ok()
```

## ARIA Snapshot Assertions

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

Roles: `heading`, `button`, `link`, `list`, `listitem`, `textbox`, `checkbox`, `radio`, `img`, `navigation`, `banner`, `main`, `contentinfo`, `complementary`, `dialog`, `alert`, `status`, `tab`, `tabpanel`, `table`, `row`, `cell`, `group`, `separator`, `paragraph`, `text`

Attributes: `[level=N]`, `[checked]`, `[disabled]`, `[expanded]`, `[pressed=true]`, `[selected]`

Matching modes via `/children`:
- `contain` (default): specified children present in order
- `equal`: exact child match
- `deep-equal`: exact match including nested

### Generate Snapshots
```python
snapshot = page.locator("body").aria_snapshot()
print(snapshot)
```
