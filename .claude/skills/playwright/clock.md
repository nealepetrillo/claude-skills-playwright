# Playwright Python Clock API Reference

## Overview
The Clock API controls time in tests, allowing simulation of time-dependent behavior without real delays.

## Overridden Functions
When activated, the clock overrides: `Date`, `setTimeout`, `clearTimeout`, `setInterval`, `clearInterval`, `requestAnimationFrame`, `cancelAnimationFrame`, `requestIdleCallback`, `cancelIdleCallback`, `performance`, `Event.timeStamp`

## Fixed Time (Simplest)

Sets a constant value for `Date.now()` and `new Date()`:

```python
import datetime

page.clock.set_fixed_time(datetime.datetime(2024, 2, 2, 10, 0, 0))
page.goto("http://localhost:3333")
expect(page.get_by_test_id("current-time")).to_have_text("2/2/2024, 10:00:00 AM")

# Update fixed time
page.clock.set_fixed_time(datetime.datetime(2024, 2, 2, 10, 30, 0))
expect(page.get_by_test_id("current-time")).to_have_text("2/2/2024, 10:30:00 AM")
```

## Install + Pause + Fast-Forward

For more control over time progression:

```python
# Install clock BEFORE any other clock calls
page.clock.install(time=datetime.datetime(2024, 2, 2, 8, 0, 0))
page.goto("http://localhost:3333")

# Pause at specific time
page.clock.pause_at(datetime.datetime(2024, 2, 2, 10, 0, 0))
expect(page.get_by_test_id("current-time")).to_have_text("2/2/2024, 10:00:00 AM")

# Fast-forward (string format "HH:MM" or milliseconds)
page.clock.fast_forward("30:00")
expect(page.get_by_test_id("current-time")).to_have_text("2/2/2024, 10:30:00 AM")
```

## Inactivity/Timeout Testing

```python
page.clock.install()
page.goto("http://localhost:3333")
page.get_by_role("button").click()
page.clock.fast_forward("05:00")
expect(page.get_by_text("You have been logged out due to inactivity.")).to_be_visible()
```

## Manual Ticking (Precise Control)

```python
page.clock.install(time=datetime.datetime(2024, 2, 2, 8, 0, 0))
page.goto("http://localhost:3333")

page.clock.pause_at(datetime.datetime(2024, 2, 2, 10, 0, 0))
expect(locator).to_have_text("2/2/2024, 10:00:00 AM")

# run_for takes milliseconds
page.clock.run_for(2000)
expect(locator).to_have_text("2/2/2024, 10:00:02 AM")
```

## Resume Normal Time

```python
page.clock.install(time=datetime.datetime(2024, 2, 2, 8, 0, 0))
page.clock.pause_at(datetime.datetime(2024, 2, 2, 10, 0, 0))
page.clock.resume()  # Time flows normally from paused point
```

## Methods Summary

| Method | Description |
|--------|-------------|
| `set_fixed_time(time)` | Freeze time at constant value |
| `install(time=...)` | Initialize clock system (MUST be first clock call) |
| `pause_at(time)` | Halt time at specific moment |
| `fast_forward(ticks)` | Jump forward ("HH:MM" string or ms) |
| `run_for(ms)` | Advance by specific milliseconds |
| `resume()` | Resume normal time flow |
| `set_system_time(time)` | Update system time (advanced) |

**Critical:** If using `install()`, it MUST be called before any other clock methods.
