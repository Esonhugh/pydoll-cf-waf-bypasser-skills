---
name: pydoll-antibot-bypasser
description: >
  Stealth browser automation using pydoll library, specialized in bypassing
  Cloudflare WAF, Turnstile CAPTCHA, and other bot detection systems. **You MUST
  invoke this skill in ANY of these situations:** (1) The user explicitly asks
  to bypass WAF, anti-bot, CAPTCHA, or human verification (Cloudflare, DataDome,
  PerimeterX, Akamai, etc.). (2) You encounter Cloudflare protection while
  scraping or fetching a URL — signs include HTTP 403/503 responses, pages
  containing "Checking your browser", "Just a moment...", "Attention Required",
  "cf-browser-verification", "cf-chl-bypass", "_cf_chl_opt", Turnstile widget
  HTML, or any response body mentioning "cloudflare" with a challenge/block
  page. (3) A WebFetch or curl fails with what appears to be bot detection or
  WAF blocking. (4) The user wants to crawl, scrape, or automate interaction
  with a website known to use anti-bot protection. When in doubt about whether a
  site is WAF-protected, invoke this skill — it is better to invoke and not need
  it than to miss a bypass opportunity. Also suitable for: stealth browser
  operations, Shadow DOM handling, human-like behavior simulation, and
  concurrent web automation.
---

# Pydoll Antibot Bypasser Skill

Pydoll is an **async-native, zero WebDriver dependency** Chromium browser automation library designed for **stealth and human-like interaction**.

## Quick Reference

### Core Features

| Feature | Description |
|---------|-------------|
| Zero WebDriver | Direct WebSocket connection to CDP, no `navigator.webdriver` flag |
| Human-like Interaction | Bezier curve mouse + typing error simulation |
| Shadow DOM | Can access closed shadow roots |
| Cloudflare | Built-in Turnstile auto-handling |
| Async Performance | 100% async, supports concurrency |

### WAF Bypass Support

| WAF | Status | Notes |
|-----|--------|-------|
| Cloudflare Turnstile | ✅ Fully Supported | Works in headless mode |
| Cloudflare JS Challenge | ✅ Supported | Auto-executes JS |
| **Cloudflare Managed Challenge** | ✅ **Verified** | **Requires `headless=False` + xvfb** |
| DataDome | ⚠️ Partial Support | Needs high-quality proxy |
| PerimeterX | ⚠️ Partial Support | Needs randomized behavior |
| reCAPTCHA | ⚠️ Manual Handling | Via Shadow DOM |

### Running Methods

```bash
# Recommended: uv script (auto-installs dependencies)
uv run script.py

# Traditional method
pip install pydoll-python
```

### Core Code Template

```python
# /// script
# requires-python = ">=3.10"
# dependencies = ["pydoll-python"]
# ///

import asyncio
import time
from pydoll.browser import Chrome
from pydoll.browser.options import ChromiumOptions

async def main():
    options = ChromiumOptions()
    options.headless = True

    # Anti-detection config (CRITICAL!)
    fake_engagement_time = int(time.time()) - (7 * 24 * 60 * 60)
    options.browser_preferences = {
        'profile': {
            'last_engagement_time': fake_engagement_time,
            'exit_type': 'Normal',
            'exited_cleanly': True,
        },
    }
    options.webrtc_leak_protection = True

    # Docker environment
    options.add_argument('--no-sandbox')
    options.add_argument('--disable-dev-shm-usage')

    async with Chrome(options=options) as browser:
        tab = await browser.start()

        # Bypass Cloudflare
        async with tab.expect_and_bypass_cloudflare_captcha():
            await tab.go_to('https://protected-site.com')

        print(await tab.title)

if __name__ == '__main__':
    asyncio.run(main())
```

---

## Browser Configuration

### Basic Configuration

```python
from pydoll.browser import Chrome
from pydoll.browser.options import ChromiumOptions

options = ChromiumOptions()
options.headless = True
options.add_argument('--window-size=1920,1080')
options.start_timeout = 20  # Startup timeout (seconds)

# Specify Chrome path
options.binary_location = '/usr/bin/google-chrome-stable'
```

### Anti-Detection Configuration (Required)

```python
import time

fake_engagement_time = int(time.time()) - (7 * 24 * 60 * 60)

options.browser_preferences = {
    'profile': {
        'last_engagement_time': fake_engagement_time,  # Simulate months-old browser
        'exit_type': 'Normal',
        'exited_cleanly': True,
        'default_content_setting_values': {
            'notifications': 2,
            'geolocation': 2,
        },
        'password_manager_enabled': False,
    },
    'intl': {
        'accept_languages': 'en-US,en',
    },
}
options.webrtc_leak_protection = True
```

### Proxy Configuration

```python
# HTTP proxy
options.add_argument('--proxy-server=http://user:pass@proxy:8080')

# Isolated browser context
context_id = await browser.create_browser_context(
    proxy_server='http://proxy:8080'
)
```

---

## Cloudflare Bypass

### Method 1: Context Manager (Recommended)

```python
async with Chrome() as browser:
    tab = await browser.start()
    async with tab.expect_and_bypass_cloudflare_captcha():
        await tab.go_to('https://protected-site.com')
```

### Method 2: Enable/Disable

```python
await tab.enable_auto_solve_cloudflare_captcha()
await tab.go_to('https://protected-site.com')
await asyncio.sleep(5)
await tab.disable_auto_solve_cloudflare_captcha()
```

### Managed Challenge Bypass (Important)

**Key Finding: Managed Challenge detects headless mode, must use `headless=False`**

| Mode | Result |
|------|--------|
| `headless=True` | ❌ Infinite wait |
| `headless=False` | ✅ Successful bypass |

**Server Environment:**

```bash
# Install xvfb
apt-get install -y xvfb

# Use xvfb-run
xvfb-run -a --server-args="-screen 0 1920x1080x24" uv run script.py
```

---

## Element Operations

### Finding Elements

```python
# Find by attributes
button = await tab.find(tag_name='button', class_name='btn-primary')

# Find by ID
username = await tab.find(id='username')

# CSS selector
nav = await tab.query('nav.main-menu')

# Find multiple
links = await tab.find(tag_name='a', find_all=True)

# With timeout and error handling
element = await tab.find(class_name='dynamic', timeout=10, raise_exc=False)
```

### Interaction Operations

```python
# Click
await button.click()

# Human-like typing (key for bot detection bypass)
await input_element.type_text('Hello World', humanize=True)

# Direct value setting
await input_element.insert_text('value')

# Clear
await input_element.clear()

# File upload
async with tab.expect_file_chooser() as fc:
    await upload_btn.click()
await fc.upload_file('/path/to/file')
```

---

## Keyboard/Mouse Operations

```python
from pydoll.constants import Key

# Keyboard
await tab.keyboard.press(Key.ENTER)
await tab.keyboard.hotkey(Key.CONTROL, Key.A)  # Select all

# Mouse (human-like movement)
await tab.mouse.move(500, 300, humanize=True)
await tab.mouse.click(500, 300, humanize=True)

# Scroll
from pydoll.constants import ScrollPosition
await tab.scroll.by(ScrollPosition.DOWN, 500, smooth=True)
```

---

## Shadow DOM

```python
# Get shadow root
shadow = await element.get_shadow_root()
button = await shadow.query('.internal-btn')

# Find all shadow roots on page
shadow_roots = await tab.find_shadow_roots()
for sr in shadow_roots:
    checkbox = await sr.query('input[type="checkbox"]', raise_exc=False)
    if checkbox:
        await checkbox.click()

# Shadow roots in cross-origin iframes
shadow_roots = await tab.find_shadow_roots(deep=True, timeout=10)
```

---

## Network Control

### Hybrid Automation (UI + API)

```python
# After UI login, make API requests with browser session
response = await tab.request.get('https://example.com/api/profile')
user_data = response.json()
```

### Request Interception

```python
from pydoll.protocol.fetch.events import FetchEvent, RequestPausedEvent
from pydoll.protocol.network.types import ErrorReason

async def block_resources(event: RequestPausedEvent):
    rid = event['params']['requestId']
    rtype = event['params']['resourceType']
    if rtype in ['Image', 'Stylesheet', 'Font', 'Media']:
        await tab.fail_request(rid, ErrorReason.BLOCKED_BY_CLIENT)
    else:
        await tab.continue_request(rid)

await tab.enable_fetch_events()
await tab.on(FetchEvent.REQUEST_PAUSED, block_resources)
await tab.go_to('https://example.com')
await tab.disable_fetch_events()
```

---

## Tab Management

```python
# New tab
tab2 = await browser.new_tab(url='https://example.com')

# Isolated context (like incognito)
context_id = await browser.create_browser_context()
tab3 = await browser.new_tab(browser_context_id=context_id)

# Get all open tabs
tabs = await browser.get_opened_tabs()

# Close
await tab.close()
```

---

## Screenshot & Download

```python
# Screenshot
await tab.take_screenshot(path='screenshot.png')
await tab.take_screenshot(path='full.png', full_page=True)

# PDF
await tab.print_to_pdf(path='page.pdf')

# Download
from pathlib import Path
async with tab.expect_download(keep_file_at=Path('/tmp')) as dl:
    await (await tab.find(text='Download')).click()
print(f"Downloaded to: {dl.file_path}")
```

---

## Exception Handling

```python
from pydoll.exceptions import ElementNotFound, PageLoadTimeout, NetworkError

try:
    element = await tab.find(id='button', timeout=5)
except ElementNotFound:
    print("Element not found")
except PageLoadTimeout:
    print("Page load timeout")

# Retry decorator
from pydoll.decorators import retry

@retry(max_retries=3, exceptions=[ElementNotFound, NetworkError])
async def scrape_page(tab, url):
    await tab.go_to(url)
    return await tab.title
```

---

## Complete Examples

See `examples/` directory for detailed code examples:

| File | Description |
|------|-------------|
| `bypass_cloudflare.py` | Cloudflare WAF bypass |
| `bypass_managed_challenge.py` | Managed Challenge bypass |
| `stealth_scraper.py` | Full anti-detection scraper |
| `concurrent_scraper.py` | Concurrent scraping |
| `screenshot.py` | Batch screenshots |

Common templates in `scripts/templates.py`, includes 8 ready-to-use templates.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Browser not found | `options.binary_location = '/path/to/chrome'` |
| Startup timeout | `options.start_timeout = 20` |
| Docker crash | Add `--no-sandbox` and `--disable-dev-shm-usage` |
| Element not found | Increase `timeout` or use `raise_exc=False` |
| Detected as bot | Enable `humanize=True`, configure browser fingerprint |
| Cloudflare failed | Use `expect_and_bypass_cloudflare_captcha()` |
| Managed Challenge failed | Use `headless=False` + xvfb |

---

## References

- GitHub: https://github.com/autoscrape-labs/pydoll
- Documentation: https://pydoll.tech/
- Chinese Docs: https://autoscrape-labs.github.io/pydoll/

**Important**: When using this library for scraping, please comply with target website's robots.txt and terms of service.
