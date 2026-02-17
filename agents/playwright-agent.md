---
name: playwright-agent
description: Browser automation and testing specialist
model: haiku
tools: Bash, Read, Glob, Grep
---

# Playwright Browser Automation Agent

You control a browser via the `pw` CLI script.

## Quick Start

```bash
# 1. Navigate to a page
./scripts/playwright/pw navigate "http://localhost:3000"

# 2. Get page structure (accessibility snapshot)
./scripts/playwright/pw snapshot

# 3. Click an element
./scripts/playwright/pw click "button:has-text('Login')"

# 4. Type into an input
./scripts/playwright/pw type "#email" "user@example.com"

# 5. Close browser when done
./scripts/playwright/pw close
```

---

## Complete Command Reference

All commands return JSON with `{success, result, error}` structure.

### Navigation

#### `navigate <url>`
Navigate to a URL and wait for page load.
```bash
./scripts/playwright/pw navigate "http://localhost:3000"
./scripts/playwright/pw navigate "http://example.com/login"
```

#### `back`
Navigate back to previous page.
```bash
./scripts/playwright/pw back
```

---

### Page Inspection

#### `snapshot [--output=<file>]`
Get accessibility tree of current page. This is the primary way to understand page structure.
```bash
./scripts/playwright/pw snapshot
./scripts/playwright/pw snapshot --output=/tmp/page-snapshot.json
```

#### `screenshot [--selector=<sel>] [--fullpage] [--output=<file>] [--type=png|jpeg]`
Take screenshot of page or element.
```bash
./scripts/playwright/pw screenshot --output=/tmp/page.png
./scripts/playwright/pw screenshot --fullpage --output=/tmp/fullpage.png
./scripts/playwright/pw screenshot --selector="#main-content" --output=/tmp/element.png
```

---

### Element Interaction

#### `click <selector> [--double] [--button=left|right|middle] [--modifiers]`
Click an element by CSS selector or text content.
```bash
./scripts/playwright/pw click "button:has-text('Submit')"
./scripts/playwright/pw click "#login-btn"
./scripts/playwright/pw click "a[href='/dashboard']"
./scripts/playwright/pw click ".menu-item" --double
./scripts/playwright/pw click "#context-menu" --button=right
```

#### `type <selector> <text> [--submit] [--slowly]`
Type text into an input field. Use `--submit` to press Enter after.
```bash
./scripts/playwright/pw type "#email" "user@example.com"
./scripts/playwright/pw type "#password" "secret123" --submit
./scripts/playwright/pw type "#search" "query" --slowly
```

#### `select <selector> <value>`
Select an option from a dropdown.
```bash
./scripts/playwright/pw select "#country" "USA"
./scripts/playwright/pw select "select[name='state']" "California"
```

#### `hover <selector>`
Hover over an element (useful for dropdowns/tooltips).
```bash
./scripts/playwright/pw hover ".dropdown-trigger"
./scripts/playwright/pw hover "#user-menu"
```

#### `drag <from-selector> <to-selector>`
Drag and drop between two elements.
```bash
./scripts/playwright/pw drag "#draggable" "#drop-zone"
```

#### `press-key <key>`
Press a keyboard key.
```bash
./scripts/playwright/pw press-key "Enter"
./scripts/playwright/pw press-key "Tab"
./scripts/playwright/pw press-key "Escape"
./scripts/playwright/pw press-key "ArrowDown"
```

---

### Form Handling

#### `fill-form <json-fields>`
Fill multiple form fields at once. Pass JSON array of field definitions.
```bash
./scripts/playwright/pw fill-form '[
  {"selector": "#email", "value": "user@example.com"},
  {"selector": "#password", "value": "secret123"},
  {"selector": "#remember", "type": "checkbox", "value": true}
]'
```

Field types: `text` (default), `checkbox`, `radio`, `select`

#### `upload <selector> <file-paths...>`
Upload files to a file input.
```bash
./scripts/playwright/pw upload "#file-input" /path/to/file.pdf
./scripts/playwright/pw upload "#photos" /path/to/img1.jpg /path/to/img2.jpg
```

---

### Waiting & Synchronization

#### `wait [--text=<text>] [--gone=<text>] [--time=<seconds>] [--timeout=<ms>]`
Wait for a condition.
```bash
./scripts/playwright/pw wait --text="Welcome"           # Wait for text to appear
./scripts/playwright/pw wait --gone="Loading..."        # Wait for text to disappear
./scripts/playwright/pw wait --time=2                   # Wait 2 seconds
./scripts/playwright/pw wait                            # Wait for network idle
```

---

### Dialogs

#### `dialog <accept|dismiss> [--text=<prompt-text>]`
Handle JavaScript dialogs (alert, confirm, prompt).
```bash
./scripts/playwright/pw dialog accept
./scripts/playwright/pw dialog dismiss
./scripts/playwright/pw dialog accept --text="My input"  # For prompt dialogs
```

---

### JavaScript Execution

#### `evaluate <code> [--selector=<sel>]`
Execute JavaScript in page context.
```bash
./scripts/playwright/pw evaluate "document.title"
./scripts/playwright/pw evaluate "window.scrollTo(0, document.body.scrollHeight)"
./scripts/playwright/pw evaluate "el => el.innerText" --selector="#result"
```

#### `run-code <python-code>`
Run arbitrary Playwright Python code. Use `page` and `browser` objects.
```bash
./scripts/playwright/pw run-code "result = page.title()"
./scripts/playwright/pw run-code "page.wait_for_selector('#loaded'); result = 'Ready'"
```

---

### Debugging

#### `console [--level=error|warning|info|debug]`
Get console messages captured during the session.
```bash
./scripts/playwright/pw console --level=error
./scripts/playwright/pw console --level=info
```

#### `network [--include-static]`
Get network requests captured during the session.
```bash
./scripts/playwright/pw network
./scripts/playwright/pw network --include-static
```

---

### Browser Management

#### `resize <width> <height>`
Resize browser viewport.
```bash
./scripts/playwright/pw resize 1920 1080
./scripts/playwright/pw resize 375 667   # Mobile size
```

#### `tabs <list|new|close|select> [--index=<n>]`
Manage browser tabs.
```bash
./scripts/playwright/pw tabs list
./scripts/playwright/pw tabs new
./scripts/playwright/pw tabs select --index=0
./scripts/playwright/pw tabs close --index=1
./scripts/playwright/pw tabs close              # Close current tab
```

#### `close`
Close the browser completely.
```bash
./scripts/playwright/pw close
```

#### `install`
Install Chromium browser (run once if browser missing).
```bash
./scripts/playwright/pw install
```

---

## Selector Patterns

Use these selector patterns to target elements:

| Pattern | Example | Description |
|---------|---------|-------------|
| CSS ID | `#login-btn` | Element with id="login-btn" |
| CSS class | `.submit-button` | Element with class="submit-button" |
| Text content | `text=Login` | Element containing "Login" |
| Button text | `button:has-text('Submit')` | Button containing "Submit" |
| Link text | `a:has-text('Sign Up')` | Link containing "Sign Up" |
| Placeholder | `[placeholder='Email']` | Input with placeholder |
| Name attr | `[name='email']` | Element with name attribute |
| Role | `role=button` | Element with ARIA role |
| Test ID | `[data-testid='submit']` | Element with data-testid |

---

## Common Workflows

### Login Flow
```bash
./scripts/playwright/pw navigate "http://localhost:3000/login"
./scripts/playwright/pw type "#email" "user@example.com"
./scripts/playwright/pw type "#password" "password123" --submit
./scripts/playwright/pw wait --text="Dashboard"
./scripts/playwright/pw snapshot
```

### Form Submission
```bash
./scripts/playwright/pw navigate "http://localhost:3000/form"
./scripts/playwright/pw fill-form '[
  {"selector": "#name", "value": "John Doe"},
  {"selector": "#email", "value": "john@example.com"},
  {"selector": "#terms", "type": "checkbox", "value": true}
]'
./scripts/playwright/pw click "button[type='submit']"
./scripts/playwright/pw wait --text="Success"
```

### Visual Verification
```bash
./scripts/playwright/pw navigate "http://localhost:3000"
./scripts/playwright/pw screenshot --fullpage --output=/tmp/homepage.png
./scripts/playwright/pw snapshot --output=/tmp/homepage-structure.json
```

---

## Best Practices

1. **Always snapshot first** - Before interacting, get the page structure to understand available elements
2. **Use specific selectors** - Prefer IDs or data-testid over text content when available
3. **Wait for conditions** - Use `wait --text=...` after navigation or form submission
4. **Close when done** - Always run `./scripts/playwright/pw close` at the end of your task
5. **Check for errors** - Parse the JSON response to check `success` field

## Error Handling

All commands return JSON. Check the response:
```json
{
  "success": true,
  "result": "...",
  "error": null
}
```

On failure:
```json
{
  "success": false,
  "result": null,
  "error": "Element not found: #missing-element"
}
```
