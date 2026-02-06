# Playwright Python Locators Reference

## Recommended Locator Strategies (Priority Order)

### 1. get_by_role() - Best Practice
Reflects how users and assistive technology perceive the page.

```python
page.get_by_role("heading", name="Sign up").click()
page.get_by_role("checkbox", name="Subscribe").check()
page.get_by_role("button", name=re.compile("submit", re.IGNORECASE)).click()
expect(page.get_by_role("heading", name="Sign up")).to_be_visible()
```

Common roles: `button`, `checkbox`, `heading`, `link`, `listitem`, `list`, `textbox`, `radio`, `tab`, `tabpanel`, `dialog`, `navigation`, `main`, `banner`, `contentinfo`, `complementary`, `form`, `img`, `table`, `row`, `cell`, `columnheader`, `rowheader`, `alert`, `status`, `menu`, `menuitem`, `tree`, `treeitem`, `progressbar`, `slider`, `spinbutton`, `switch`, `combobox`, `option`, `group`, `separator`, `toolbar`

### 2. get_by_label() - Form Controls
```python
page.get_by_label("Password").fill("secret")
page.get_by_label("Username").fill("admin")
```

### 3. get_by_placeholder()
```python
page.get_by_placeholder("name@example.com").fill("test@test.com")
```

### 4. get_by_text() - Text Content
```python
# Substring match (default)
expect(page.get_by_text("Welcome, John")).to_be_visible()

# Exact match
expect(page.get_by_text("Welcome, John", exact=True)).to_be_visible()

# Regex match
expect(page.get_by_text(re.compile("welcome, john", re.IGNORECASE))).to_be_visible()
```

### 5. get_by_alt_text() - Images
```python
page.get_by_alt_text("playwright logo").click()
```

### 6. get_by_title()
```python
expect(page.get_by_title("Issues count")).to_have_text("25 issues")
```

### 7. get_by_test_id() - data-testid Attributes
```python
page.get_by_test_id("directions").click()

# Custom test ID attribute
playwright.selectors.set_test_id_attribute("data-pw")
```

### 8. CSS/XPath (Last Resort)
```python
page.locator("css=button").click()
page.locator("xpath=//button").click()

# Shorthand
page.locator("button").click()        # CSS
page.locator("//button").click()      # XPath (starts with //)
```

## Filtering Locators

### By Text
```python
page.get_by_role("listitem").filter(has_text="Product 2").click()
page.get_by_role("listitem").filter(has_text=re.compile("Product 2")).click()
page.get_by_role("listitem").filter(has_not_text="Out of stock")
```

### By Child/Descendant
```python
page.get_by_role("listitem").filter(
    has=page.get_by_role("heading", name="Product 2")
).get_by_role("button", name="Add to cart").click()

# Not having child
page.get_by_role("listitem").filter(
    has_not=page.get_by_role("heading", name="Product 2")
)
```

### Visible Only
```python
page.locator("button").filter(visible=True).click()
```

## Locator Operators

### Matching Inside a Locator
```python
product = page.get_by_role("listitem").filter(has_text="Product 2")
product.get_by_role("button", name="Add to cart").click()

# Scope a locator to another
dialog = page.get_by_test_id("settings-dialog")
dialog.locator(page.get_by_role("button", name="Save")).click()
```

### AND - Both Conditions
```python
button = page.get_by_role("button").and_(page.get_by_title("Subscribe"))
```

### OR - Either Condition
```python
new_email = page.get_by_role("button", name="New")
dialog = page.get_by_text("Confirm security settings")
expect(new_email.or_(dialog).first).to_be_visible()
if dialog.is_visible():
    page.get_by_role("button", name="Dismiss").click()
new_email.click()
```

## Working with Lists

```python
# Count items
expect(page.get_by_role("listitem")).to_have_count(3)

# Assert all text
expect(page.get_by_role("listitem")).to_have_text(["apple", "banana", "orange"])

# Get by index (0-based)
banana = page.get_by_role("listitem").nth(1)

# First and last
page.get_by_role("button").first
page.get_by_role("button").last

# Iterate
for row in page.get_by_role("listitem").all():
    print(row.text_content())

# Evaluate all
texts = page.get_by_role("listitem").evaluate_all(
    "list => list.map(el => el.textContent)"
)
```

### Chaining Filters
```python
row_locator = page.get_by_role("listitem")
row_locator.filter(has_text="Mary").filter(
    has=page.get_by_role("button", name="Say goodbye")
).screenshot(path="screenshot.png")
```

## Strictness
Locators are strict by default - operations throw if multiple elements match.
```python
# Throws if multiple buttons exist
page.get_by_role("button").click()

# These always work with multiple matches
page.get_by_role("button").count()
page.get_by_role("button").first.click()
page.get_by_role("button").nth(0).click()
```

## Frame Locators
```python
# Locate inside iframe
username = page.frame_locator('.frame-class').get_by_label('User Name')
username.fill('John')

# By frame name
frame = page.frame('frame-login')
frame.fill('#username-input', 'John')

# By URL pattern
frame = page.frame(url=r'.*domain.*')
```

## Shadow DOM
Most locators work with Shadow DOM. XPath does not pierce shadow roots.
```python
page.get_by_text("Details").click()
page.locator("x-details", has_text="Details").click()
```

## CSS Pseudo-Classes
```python
# Visible only
page.locator("button:visible").click()

# Has text (substring, case-insensitive)
page.locator('article:has-text("Playwright")').click()

# Text (smallest element)
page.locator("#nav-bar :text('Home')").click()

# Exact text
page.locator(":text-is('Log')").click()

# Regex text
page.locator(':text-matches("Log\\s*in", "i")').click()

# Has child element
page.locator("article:has(div.promo)").text_content()

# Nth match (1-based)
page.locator(":nth-match(:text('Buy'), 3)").click()
```

## Nth Element (0-based)
```python
page.locator("button").locator("nth=0").click()   # First
page.locator("button").locator("nth=-1").click()  # Last
```

## Parent Element
```python
child = page.get_by_text("Hello")
parent = page.get_by_role("listitem").filter(has=child)
```
