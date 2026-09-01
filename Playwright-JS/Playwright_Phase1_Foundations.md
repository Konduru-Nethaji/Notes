# Playwright Notes — Phase 1: Foundations

> Revision notes for Playwright fundamentals. Each topic has a plain explanation \+ runnable example, so you can revise quickly and reuse snippets as a starting point for your own scripts.

---

## Table of Contents

1. [Setup & Installation](#1-setup--installation)  
2. [Core Concepts](#2.-core-concepts)  
3. [Locators (the heart of Playwright)](#3.-locators-\(the-heart-of-playwright\))  
4. [Actions](#4.-actions)  
5. [Assertions](#5.-assertions)  
6. [Quick Reference Cheat Sheet](#6.-quick-reference-cheat-sheet)

---

## 1\. Setup & Installation

### Installing via `npm init playwright@latest`

```shell
npm init playwright@latest
```

This single command sets up an entire Playwright project for you. It will ask you:

- TypeScript or JavaScript? (choose JavaScript for now, since that's what you've been learning)  
- Where to put tests? (default: `tests/`)  
- Add a GitHub Actions workflow? (optional, say No for now)  
- Install Playwright browsers? (say Yes)

### Project structure

After setup, you'll see:

```
my-project/
├── tests/
│   └── example.spec.js        # sample test file
├── tests-examples/
│   └── demo-todo-app.spec.js  # a full sample test (safe to delete)
├── playwright.config.js       # main configuration file
├── package.json
├── package-lock.json
└── .gitignore
```

**`playwright.config.js`** — controls how your tests run: which browsers, timeouts, base URL, retries, reporters, etc.

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30 * 1000,          // 30 seconds max per test
  retries: 1,                   // retry failing tests once
  use: {
    baseURL: 'https://example.com',
    headless: true,             // run browser without a visible UI
    screenshot: 'only-on-failure',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
  ],
});
```

**`.gitignore`** — Playwright adds entries so you don't commit noise to git:

```
node_modules/
test-results/
playwright-report/
playwright/.cache/
```

### Installing browsers

Playwright ships its own tested browser binaries, separate from the browsers on your machine — this is why tests behave consistently across environments.

```shell
npx playwright install                 # installs all 3 browsers
npx playwright install chromium        # installs only Chromium
npx playwright install --with-deps     # also installs OS-level dependencies (useful on Linux/CI)
```

- **Chromium** → stands in for Google Chrome / Microsoft Edge  
- **Firefox** → Mozilla Firefox  
- **WebKit** → stands in for Safari

### Running your first test

```javascript
// tests/example.spec.js
import { test, expect } from '@playwright/test';

test('has title', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});
```

```shell
npx playwright test                     # run all tests, headless
npx playwright test --headed            # run with visible browser
npx playwright test example.spec.js     # run one file
npx playwright test -g "has title"      # run tests matching a name
npx playwright test --project=firefox   # run on one browser only
npx playwright show-report              # open the HTML report after a run
npx playwright test --debug             # run in debug/step-through mode
```

---

## 2\. Core Concepts

### Browser, BrowserContext, Page — the object hierarchy

Think of it like this: **Browser → Context → Page**

```
Browser (one installed browser engine, e.g. Chromium)
  └── BrowserContext (an isolated "incognito-like" session — its own cookies, storage)
        └── Page (a single tab)
```

```javascript
import { chromium } from '@playwright/test';

const browser = await chromium.launch();          // launches ONE browser instance
const context = await browser.newContext();       // isolated session (like a private window)
const page = await context.newPage();              // one tab inside that session

await page.goto('https://example.com');
await browser.close();
```

- **Browser** — the actual browser process. Expensive to start, so usually launched once.  
- **BrowserContext** — a fully isolated environment (cookies, localStorage, cache are separate per context). You can open many contexts from one browser — useful for testing multi-user scenarios (e.g. two users chatting) without opening two full browsers.  
- **Page** — an actual tab/page inside a context. Most of your test code interacts with `page`.

**In the Playwright Test runner** (`@playwright/test`), you don't usually create these manually — they're provided to you automatically as **fixtures**:

```javascript
import { test, expect } from '@playwright/test';

test('example', async ({ page }) => {
  // 'page' is already created for you — a fresh context + page per test
  await page.goto('https://example.com');
});
```

### Headless vs Headed mode

| Mode | Meaning | When to use |
| :---- | :---- | :---- |
| **Headless** | Browser runs with no visible UI, faster | CI pipelines, running many tests quickly |
| **Headed** | Browser window actually opens and you can watch it | Debugging, writing new tests, demos |

```javascript
// playwright.config.js
use: {
  headless: false,   // forces headed mode for every test
}
```

```shell
npx playwright test --headed     # override for one run, without editing config
```

### Test runner basics (`@playwright/test`)

```javascript
import { test, expect } from '@playwright/test';

test.describe('Login feature', () => {

  test.beforeEach(async ({ page }) => {
    // runs before EVERY test in this describe block
    await page.goto('https://example.com/login');
  });

  test.afterEach(async ({ page }) => {
    // runs after EVERY test — good for cleanup, logging, screenshots
    console.log('Test finished');
  });

  test('valid login shows dashboard', async ({ page }) => {
    await page.fill('#username', 'admin');
    await page.fill('#password', '1234');
    await page.click('#loginBtn');
    await expect(page.locator('.dashboard')).toBeVisible();
  });

  test('invalid login shows error', async ({ page }) => {
    await page.fill('#username', 'wrong');
    await page.fill('#password', 'wrong');
    await page.click('#loginBtn');
    await expect(page.locator('.error-msg')).toBeVisible();
  });

});
```

- **`test()`** — defines one test case.  
- **`test.describe()`** — groups related tests (like a test suite). Purely organizational.  
- **`test.beforeEach()` / `test.afterEach()`** — setup/teardown that runs around each test inside the describe block.  
- **`test.beforeAll()` / `test.afterAll()`** — run once for the whole describe block, not per test (rarely used, since Playwright prefers test isolation).

---

## 3\. Locators (the heart of Playwright)

### `page.locator()` vs old-style `page.$()`

```javascript
// OLD style (ElementHandle) — avoid this
const button = await page.$('#submit');
await button.click();
```

Problem: `page.$()` grabs the element **once, immediately**. If the element isn't there yet, or gets re-rendered, your reference becomes stale and the test fails or throws.

```javascript
// MODERN style (Locator) — always prefer this
const button = page.locator('#submit');
await button.click();
```

A **Locator** is not "the element" — it's a **recipe for finding the element**, re-evaluated every time you act on it. This is what gives Playwright its auto-waiting and reliability.

### Built-in role/text-based locators

Playwright recommends locating elements the way a real user (or screen reader) would — by role, label, or visible text — instead of brittle CSS classes.

```javascript
// getByRole — the most recommended, matches accessibility roles
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByRole('checkbox', { name: 'Remember me' }).check();
await page.getByRole('link', { name: 'Home' }).click();

// getByText — matches visible text on the page
await page.getByText('Welcome back').isVisible();

// getByLabel — matches a form field by its associated <label>
await page.getByLabel('Username').fill('admin');

// getByPlaceholder — matches an input by its placeholder text
await page.getByPlaceholder('Enter your email').fill('test@test.com');

// getByTestId — matches a custom data-testid attribute (great when devs add these for tests)
await page.getByTestId('submit-button').click();

// getByTitle — matches the 'title' attribute
await page.getByTitle('Close').click();

// getByAltText — matches an image's alt attribute
await page.getByAltText('Company Logo').click();
```

**Why these are preferred:** they describe *what the element means to a user*, not *how it's implemented*. If a developer changes a CSS class name, these locators still work.

### CSS and XPath selectors

```javascript
// CSS selector
await page.locator('.login-form button.submit').click();
await page.locator('#username').fill('admin');

// XPath selector
await page.locator('xpath=//button[text()="Submit"]').click();
```

**When to use:**

- Use CSS selectors when there's no accessible role/label/testid available, and the CSS is simple and stable (an id or a stable class).  
- Use XPath only as a last resort — e.g. when you need to select based on text content in a complex way, or navigate up the DOM (`parent::`), which CSS can't do.

**When to avoid:**

- Avoid selectors based on deeply nested structure (`div > div > span:nth-child(3)`) — these break the moment the page layout changes even slightly.

### Chaining and filtering locators

```javascript
// .filter() — narrows a locator to elements matching extra conditions
const row = page.locator('tr').filter({ hasText: 'Ravi Kumar' });
await row.getByRole('button', { name: 'Delete' }).click();

// .first(), .last(), .nth() — pick a specific match out of several
await page.locator('.product-card').first().click();
await page.locator('.product-card').last().click();
await page.locator('.product-card').nth(2).click();  // 0-indexed, so this is the 3rd card

// Chaining locators — narrows scope step by step (like nested CSS)
const table = page.locator('#userTable');
const secondRow = table.locator('tr').nth(1);
await secondRow.locator('button.edit').click();
```

### Auto-waiting and web-first assertions

Traditional automation tools needed manual `sleep(2000)` calls because the page might not be ready yet. Playwright avoids this almost entirely.

**Auto-waiting** — before performing an action, Playwright automatically waits for the element to:

- be attached to the DOM  
- be visible  
- be stable (not animating)  
- be enabled (not disabled)  
- receive events (not covered by another element)

```javascript
// No manual wait needed — Playwright waits internally until #submit
// is visible & clickable before clicking it.
await page.locator('#submit').click();
```

**Web-first assertions** — `expect(locator)` doesn't just check once; it **retries** until the condition is true or a timeout is hit. This removes the need for manual waits before assertions too.

```javascript
// This will keep checking for up to the timeout, not just check once immediately
await expect(page.locator('.toast-message')).toBeVisible();
```

This is why you almost never need something like:

```javascript
await page.waitForTimeout(3000); // ❌ avoid — a fixed sleep, wastes time or isn't long enough
```

---

## 4\. Actions

### Clicking, typing, filling forms

```javascript
await page.locator('#submitBtn').click();
await page.locator('#submitBtn').dblclick();

// fill() -> clears the field first, then types the value instantly
await page.locator('#username').fill('admin');

// type() (legacy, use fill() instead unless you need real per-character keystrokes)
await page.locator('#search').pressSequentially('playwright', { delay: 100 });
```

### Checkboxes and radio buttons

```javascript
await page.getByRole('checkbox', { name: 'Accept terms' }).check();
await page.getByRole('checkbox', { name: 'Accept terms' }).uncheck();

const isChecked = await page.getByRole('checkbox', { name: 'Accept terms' }).isChecked();

await page.getByRole('radio', { name: 'Male' }).check();
```

### Dropdowns (native `<select>`)

```javascript
// Select by visible label
await page.locator('#country').selectOption({ label: 'India' });

// Select by value attribute
await page.locator('#country').selectOption('IN');

// Select multiple options (for multi-select dropdowns)
await page.locator('#skills').selectOption(['js', 'python']);
```

### Custom widgets (non-native dropdowns built with divs/spans)

These aren't real `<select>` elements, so `selectOption()` won't work — you interact with them like a normal user would: click to open, then click the option.

```javascript
await page.locator('.custom-dropdown').click();          // opens the dropdown
await page.getByText('United Kingdom').click();          // clicks the option from the opened list
```

### Keyboard and mouse events

```javascript
// Keyboard
await page.locator('#search').press('Enter');
await page.keyboard.press('Control+A');   // works on whatever is currently focused
await page.keyboard.type('hello world');

// Mouse hover
await page.locator('.menu-item').hover();

// Drag and drop
await page.locator('#source').dragTo(page.locator('#target'));

// Manual mouse control (rarely needed, but available)
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.up();
```

### File uploads and downloads

```javascript
// Upload
await page.locator('input[type="file"]').setInputFiles('path/to/file.pdf');

// Upload multiple files
await page.locator('input[type="file"]').setInputFiles(['file1.png', 'file2.png']);

// Remove selected files
await page.locator('input[type="file"]').setInputFiles([]);

// Download — Playwright waits for the download event
const [download] = await Promise.all([
  page.waitForEvent('download'),
  page.locator('#downloadBtn').click(),
]);
console.log(await download.path());   // where it was saved temporarily
await download.saveAs('/my/folder/report.pdf');
```

### Date pickers

Date pickers vary wildly between apps (native HTML5 input vs custom calendar widgets), so the approach depends on implementation:

```javascript
// Native HTML5 date input — just fill it directly in the required format
await page.locator('input[type="date"]').fill('2026-09-01');

// Custom calendar widget — click through the UI like a real user
await page.locator('.date-picker-input').click();
await page.getByText('15', { exact: true }).click();  // click day "15" on the calendar
```

---

## 5\. Assertions

Playwright uses `expect()` (built on top of Jest-style assertions) combined with **web-first, auto-retrying** checks.

```javascript
import { test, expect } from '@playwright/test';

test('assertions example', async ({ page }) => {
  await page.goto('https://example.com');

  // Visibility
  await expect(page.locator('.welcome-banner')).toBeVisible();
  await expect(page.locator('.loading-spinner')).toBeHidden();

  // Text content
  await expect(page.locator('h1')).toHaveText('Welcome');
  await expect(page.locator('.description')).toContainText('sample');

  // Input value
  await expect(page.locator('#username')).toHaveValue('admin');

  // Attribute / class / count
  await expect(page.locator('.tab')).toHaveAttribute('aria-selected', 'true');
  await expect(page.locator('.tab.active')).toHaveClass(/active/);
  await expect(page.locator('.product-card')).toHaveCount(5);

  // Enabled/disabled
  await expect(page.locator('#submitBtn')).toBeEnabled();
  await expect(page.locator('#submitBtn')).toBeDisabled();

  // Page-level assertions
  await expect(page).toHaveTitle('Example Domain');
  await expect(page).toHaveURL(/example\.com/);
});
```

### Soft assertions vs hard assertions

```javascript
// HARD assertion (default) -> test STOPS immediately if it fails
await expect(page.locator('.error')).toBeVisible();

// SOFT assertion -> test CONTINUES even if this fails, but the test is
// still marked as failed at the end. Useful when you want to check
// several things in one go without stopping at the first failure.
await expect.soft(page.locator('.title')).toHaveText('Dashboard');
await expect.soft(page.locator('.subtitle')).toHaveText('Overview');

// You can check if any soft assertions failed so far
test.info().errors; // list of collected soft assertion failures
```

**When to use soft assertions:** verifying multiple independent pieces of UI on one page (e.g., checking 5 different labels are correct) — you'd rather see *all 5 results* in one test run than have the test stop after the first mismatch.

### `expect.poll()` for polling custom conditions

Use this when you need to assert on something that **isn't a locator** — like a value from an API call, a database check, or a custom function — but still want Playwright's auto-retry behavior.

```javascript
let attempts = 0;

async function getJobStatus() {
  attempts++;
  return attempts >= 3 ? 'completed' : 'processing';
}

await expect.poll(async () => {
  return await getJobStatus();
}, {
  message: 'waiting for job to complete',
  timeout: 10000,
  intervals: [1000, 2000, 3000], // custom retry intervals
}).toBe('completed');
```

This keeps calling `getJobStatus()` repeatedly until it returns `'completed'` or the timeout is reached — instead of you writing a manual polling loop with `setTimeout`.

---

## 6\. Quick Reference Cheat Sheet

```javascript
// SETUP
npx playwright install                    // install browsers
npx playwright test                        // run all tests
npx playwright test --headed --debug       // run visibly, step through

// LOCATORS
page.getByRole('button', { name: 'Submit' })
page.getByText('Welcome')
page.getByLabel('Username')
page.getByPlaceholder('Search...')
page.getByTestId('submit-btn')
page.locator('.css-selector')
page.locator('xpath=//div')

// FILTER / NARROW
locator.filter({ hasText: 'text' })
locator.first() / .last() / .nth(2)

// ACTIONS
await locator.click();
await locator.fill('text');
await locator.check();
await locator.selectOption('value');
await locator.hover();
await locator.press('Enter');
await locator.setInputFiles('path');

// ASSERTIONS
await expect(locator).toBeVisible();
await expect(locator).toHaveText('text');
await expect(locator).toHaveValue('text');
await expect(locator).toHaveCount(3);
await expect(page).toHaveURL(/pattern/);
await expect.soft(locator).toBeVisible();
await expect.poll(async () => value).toBe(expected);
```

---

## Suggested Practice Order

1. Run `npm init playwright@latest` on a throwaway folder and explore the generated files.  
2. Rewrite the sample `example.spec.js` test using `getByRole` instead of CSS selectors.  
3. Build a small login test using: `goto`, `fill`, `click`, and an assertion.  
4. Practice `.filter()` and `.nth()` on a page with a list/table (e.g. a demo shopping site).  
5. Deliberately write a wrong assertion and observe how Playwright's error message/trace helps you debug it — this builds intuition fast.  
6. Try `--debug` mode once, to see the Playwright Inspector step through your test live.

---

*Tip: Ask me for a small hands-on exercise (e.g. "give me a mini login+dropdown+assertion test to write myself using a public demo site"), or paste your own Playwright script here for review.*  
