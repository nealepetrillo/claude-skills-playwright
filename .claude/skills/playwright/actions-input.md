# Playwright Python Actions & Input Reference

## Text Input

```python
page.get_by_role("textbox").fill("Peter")
page.get_by_label("Birth date").fill("2020-02-02")
page.get_by_label("Appointment time").fill("13:15")
page.get_by_label("Local time").fill("2020-03-02T05:15")
```

## Checkboxes and Radio Buttons

```python
page.get_by_label('I agree to the terms above').check()
page.get_by_label('I agree to the terms above').uncheck()
page.get_by_label('XL').check()  # Radio button
expect(page.get_by_label('Subscribe to newsletter')).to_be_checked()
```

## Select Options

```python
# By value
page.get_by_label('Choose a color').select_option('blue')
# By label text
page.get_by_label('Choose a color').select_option(label='Blue')
# Multiple
page.get_by_label('Choose multiple colors').select_option(['red', 'green', 'blue'])
```

## Mouse Click

```python
page.get_by_role("button").click()
page.get_by_text("Item").dblclick()
page.get_by_text("Item").click(button="right")
page.get_by_text("Item").click(modifiers=["Shift"])
page.get_by_text("Item").click(modifiers=["ControlOrMeta"])
page.get_by_text("Item").hover()
page.get_by_text("Item").click(position={"x": 0, "y": 0})

# Force click (skip actionability checks)
page.get_by_role("button").click(force=True)

# Programmatic click via JS dispatch
page.get_by_role("button").dispatch_event('click')
```

### Click Actionability Checks
Before clicking, Playwright:
1. Waits for element in DOM
2. Checks element is visible
3. Waits for element to stop moving (stable)
4. Scrolls element into view
5. Waits for element to receive pointer events
6. Retries if element detaches during checks

## Type Characters (Key by Key)

```python
# For special character-by-character input (triggers keydown/keypress/keyup)
page.locator('#area').press_sequentially('Hello World!')
```
**Note:** Prefer `fill()` for most text input. Use `press_sequentially()` only when you need individual key events.

## Keys and Shortcuts

```python
page.get_by_text("Submit").press("Enter")
page.get_by_role("textbox").press("Control+ArrowRight")
page.get_by_role("textbox").press("$")
page.locator('#name').press('Shift+A')
page.locator('#name').press('Shift+ArrowLeft')
```

### Key Names
`Backquote`, `Minus`, `Equal`, `Backslash`, `Backspace`, `Tab`, `Delete`, `Escape`, `ArrowDown`, `ArrowUp`, `ArrowLeft`, `ArrowRight`, `End`, `Enter`, `Home`, `Insert`, `PageDown`, `PageUp`, `F1`-`F12`, `Digit0`-`Digit9`, `KeyA`-`KeyZ`

Modifiers: `Shift`, `Control`, `Alt`, `Meta`, `ControlOrMeta`

## Upload Files

```python
# Single file
page.get_by_label("Upload file").set_input_files('myfile.pdf')

# Multiple files
page.get_by_label("Upload files").set_input_files(['file1.txt', 'file2.txt'])

# Directory
page.get_by_label("Upload directory").set_input_files('mydir')

# Clear files
page.get_by_label("Upload file").set_input_files([])

# Buffer (in-memory)
page.get_by_label("Upload file").set_input_files(
    files=[{"name": "test.txt", "mimeType": "text/plain", "buffer": b"this is a test"}]
)

# Dynamic file chooser
with page.expect_file_chooser() as fc_info:
    page.get_by_label("Upload file").click()
file_chooser = fc_info.value
file_chooser.set_files("myfile.pdf")
```

## Focus

```python
page.get_by_label('password').focus()
```

## Drag and Drop

```python
# Simple drag
page.locator("#item-to-be-dragged").drag_to(page.locator("#item-to-drop-at"))

# Manual drag (for complex scenarios)
page.locator("#item-to-be-dragged").hover()
page.mouse.down()
page.locator("#item-to-drop-at").hover()
page.mouse.up()
```
**Note:** For `dragover` events, use at least two mouse moves.

## Scrolling

```python
# Automatic: Playwright scrolls before actions
page.get_by_role("button").click()

# Manual scroll into view
page.get_by_text("Footer text").scroll_into_view_if_needed()

# Mouse wheel
page.get_by_test_id("scrolling-container").hover()
page.mouse.wheel(0, 10)

# JS scroll
page.get_by_test_id("scrolling-container").evaluate("e => e.scrollTop += 100")
```

## Actionability Checks Table

| Action | Visible | Stable | Receives Events | Enabled | Editable |
|--------|---------|--------|-----------------|---------|----------|
| check() | Yes | Yes | Yes | Yes | - |
| click() | Yes | Yes | Yes | Yes | - |
| dblclick() | Yes | Yes | Yes | Yes | - |
| set_checked() | Yes | Yes | Yes | Yes | - |
| tap() | Yes | Yes | Yes | Yes | - |
| uncheck() | Yes | Yes | Yes | Yes | - |
| hover() | Yes | Yes | Yes | - | - |
| drag_to() | Yes | Yes | Yes | - | - |
| screenshot() | Yes | Yes | - | - | - |
| fill() | Yes | - | - | Yes | Yes |
| clear() | Yes | - | - | Yes | Yes |
| select_option() | Yes | - | - | Yes | - |
| select_text() | Yes | - | - | - | - |
| scroll_into_view_if_needed() | - | Yes | - | - | - |
| blur() | - | - | - | - | - |
| dispatch_event() | - | - | - | - | - |
| focus() | - | - | - | - | - |
| press() | - | - | - | - | - |
| press_sequentially() | - | - | - | - | - |
| set_input_files() | - | - | - | - | - |

### Definitions
- **Visible**: Non-empty bounding box, not `visibility:hidden`. `opacity:0` IS visible.
- **Stable**: Same bounding box across two animation frames.
- **Enabled**: Not `[disabled]`, not in disabled fieldset, not `[aria-disabled=true]`.
- **Editable**: Enabled and not `[readonly]` or `[aria-readonly=true]`.
- **Receives Events**: Element is the hit target at action coordinates.

### Force Option
`force=True` skips non-essential actionability checks:
```python
page.get_by_role("button").click(force=True)
```
