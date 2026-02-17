# Example Workflows

Common automation patterns using the Playwright skill.

## Authentication Flows

### Basic Login

```bash
# Navigate to login page
./scripts/playwright/pw navigate "http://localhost:3000/login"

# Fill credentials
./scripts/playwright/pw type "#email" "user@example.com"
./scripts/playwright/pw type "#password" "secretpassword" --submit

# Wait for redirect
./scripts/playwright/pw wait --text="Dashboard"

# Verify logged in
./scripts/playwright/pw snapshot
```

### Login with Remember Me

```bash
./scripts/playwright/pw navigate "http://localhost:3000/login"

./scripts/playwright/pw fill-form '[
  {"selector": "#email", "value": "user@example.com"},
  {"selector": "#password", "value": "secretpassword"},
  {"selector": "#remember-me", "type": "checkbox", "value": true}
]'

./scripts/playwright/pw click "button[type='submit']"
./scripts/playwright/pw wait --text="Welcome"
```

### OAuth/Social Login

```bash
./scripts/playwright/pw navigate "http://localhost:3000/login"

# Click OAuth button
./scripts/playwright/pw click "button:has-text('Continue with Google')"

# Handle OAuth popup (if new tab)
./scripts/playwright/pw tabs list
./scripts/playwright/pw tabs select --index=1

# Fill OAuth credentials in popup
./scripts/playwright/pw type "#identifierId" "user@gmail.com"
./scripts/playwright/pw press-key "Enter"
./scripts/playwright/pw wait --time=2
./scripts/playwright/pw type "[name='password']" "password"
./scripts/playwright/pw press-key "Enter"

# Return to main tab
./scripts/playwright/pw tabs select --index=0
./scripts/playwright/pw wait --text="Dashboard"
```

---

## Form Testing

### Multi-Step Form

```bash
# Step 1: Personal Info
./scripts/playwright/pw navigate "http://localhost:3000/signup"

./scripts/playwright/pw fill-form '[
  {"selector": "#firstName", "value": "John"},
  {"selector": "#lastName", "value": "Doe"},
  {"selector": "#email", "value": "john@example.com"}
]'
./scripts/playwright/pw click "button:has-text('Next')"

# Step 2: Address
./scripts/playwright/pw wait --text="Address"
./scripts/playwright/pw fill-form '[
  {"selector": "#street", "value": "123 Main St"},
  {"selector": "#city", "value": "San Francisco"},
  {"selector": "#state", "value": "CA"},
  {"selector": "#zip", "value": "94102"}
]'
./scripts/playwright/pw click "button:has-text('Next')"

# Step 3: Review & Submit
./scripts/playwright/pw wait --text="Review"
./scripts/playwright/pw click "button:has-text('Submit')"
./scripts/playwright/pw wait --text="Success"
```

### Form Validation Testing

```bash
./scripts/playwright/pw navigate "http://localhost:3000/contact"

# Submit empty form
./scripts/playwright/pw click "button[type='submit']"

# Check for validation errors
./scripts/playwright/pw snapshot
# Look for error messages in the output

# Fill with invalid email
./scripts/playwright/pw type "#email" "not-an-email"
./scripts/playwright/pw click "button[type='submit']"

# Check for email validation error
./scripts/playwright/pw evaluate "document.querySelector('.error-message')?.textContent"
```

---

## E2E User Journeys

### E-commerce Checkout

```bash
# Browse products
./scripts/playwright/pw navigate "http://localhost:3000/products"
./scripts/playwright/pw snapshot

# Add to cart
./scripts/playwright/pw click "[data-testid='product-1'] button:has-text('Add to Cart')"
./scripts/playwright/pw wait --text="Added to cart"

# Go to cart
./scripts/playwright/pw click "a:has-text('Cart')"
./scripts/playwright/pw wait --text="Shopping Cart"

# Proceed to checkout
./scripts/playwright/pw click "button:has-text('Checkout')"

# Fill shipping info
./scripts/playwright/pw fill-form '[
  {"selector": "#name", "value": "John Doe"},
  {"selector": "#address", "value": "123 Main St"},
  {"selector": "#city", "value": "San Francisco"},
  {"selector": "#zip", "value": "94102"}
]'
./scripts/playwright/pw click "button:has-text('Continue to Payment')"

# Fill payment (test card)
./scripts/playwright/pw type "#card-number" "4242424242424242"
./scripts/playwright/pw type "#expiry" "12/25"
./scripts/playwright/pw type "#cvc" "123"

# Place order
./scripts/playwright/pw click "button:has-text('Place Order')"
./scripts/playwright/pw wait --text="Order Confirmed"

# Screenshot confirmation
./scripts/playwright/pw screenshot --output=/tmp/order-confirmation.png
```

---

## Visual Testing

### Responsive Design Testing

```bash
./scripts/playwright/pw navigate "http://localhost:3000"

# Desktop
./scripts/playwright/pw resize 1920 1080
./scripts/playwright/pw screenshot --fullpage --output=/tmp/desktop.png

# Tablet
./scripts/playwright/pw resize 768 1024
./scripts/playwright/pw screenshot --fullpage --output=/tmp/tablet.png

# Mobile
./scripts/playwright/pw resize 375 667
./scripts/playwright/pw screenshot --fullpage --output=/tmp/mobile.png
```

### Before/After Comparison

```bash
# Capture "before" state
./scripts/playwright/pw navigate "http://localhost:3000/dashboard"
./scripts/playwright/pw screenshot --output=/tmp/before.png
./scripts/playwright/pw snapshot --output=/tmp/before.json

# Perform action
./scripts/playwright/pw click "button:has-text('Toggle Dark Mode')"
./scripts/playwright/pw wait --time=0.5

# Capture "after" state
./scripts/playwright/pw screenshot --output=/tmp/after.png
./scripts/playwright/pw snapshot --output=/tmp/after.json
```

---

## Debugging

### Capture Console Errors

```bash
./scripts/playwright/pw navigate "http://localhost:3000"

# Interact with page
./scripts/playwright/pw click "button:has-text('Load Data')"
./scripts/playwright/pw wait --time=2

# Check for errors
./scripts/playwright/pw console --level=error
```

### Monitor Network Requests

```bash
./scripts/playwright/pw navigate "http://localhost:3000"

# Trigger API calls
./scripts/playwright/pw click "button:has-text('Fetch Users')"
./scripts/playwright/pw wait --time=2

# Check network activity
./scripts/playwright/pw network
```

### Debug with JavaScript

```bash
# Get all form values
./scripts/playwright/pw evaluate "Object.fromEntries(new FormData(document.querySelector('form')))"

# Check element visibility
./scripts/playwright/pw evaluate "document.querySelector('#modal').style.display"

# Get computed styles
./scripts/playwright/pw evaluate "getComputedStyle(document.querySelector('.button')).backgroundColor"

# Scroll to element
./scripts/playwright/pw evaluate "document.querySelector('#footer').scrollIntoView()"
```

---

## Handling Edge Cases

### Dialogs

```bash
# Alert
./scripts/playwright/pw dialog accept
./scripts/playwright/pw click "button:has-text('Show Alert')"

# Confirm (accept)
./scripts/playwright/pw dialog accept
./scripts/playwright/pw click "button:has-text('Delete Item')"

# Confirm (dismiss)
./scripts/playwright/pw dialog dismiss
./scripts/playwright/pw click "button:has-text('Delete Item')"

# Prompt
./scripts/playwright/pw dialog accept --text="My Response"
./scripts/playwright/pw click "button:has-text('Enter Name')"
```

### File Upload

```bash
./scripts/playwright/pw navigate "http://localhost:3000/upload"

# Single file
./scripts/playwright/pw upload "#file-input" /path/to/document.pdf

# Multiple files
./scripts/playwright/pw upload "#photos" /path/to/img1.jpg /path/to/img2.jpg /path/to/img3.jpg

# Verify upload
./scripts/playwright/pw wait --text="Upload complete"
```

### Dropdown with Search

```bash
# Open dropdown
./scripts/playwright/pw click "#country-select"

# Type to filter
./scripts/playwright/pw type "#country-select input" "United"

# Select from filtered results
./scripts/playwright/pw click "[data-value='US']"
```

---

## Tips

1. **Always snapshot first** - Understand page structure before interacting
2. **Use specific selectors** - `#id` > `[data-testid]` > `.class` > `text=`
3. **Wait appropriately** - Use `--text` for content, `--time` for animations
4. **Check JSON responses** - Parse `success` field to handle errors
5. **Close browser when done** - Free up resources with `./scripts/playwright/pw close`
