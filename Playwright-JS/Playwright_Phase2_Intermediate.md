# Playwright Notes — Phase 2: Intermediate Skills

> Revision notes for intermediate Playwright topics. Each topic has a plain explanation \+ runnable example, so you can revise quickly and reuse snippets as a starting point for your own scripts.

---

## Table of Contents

6. [Test Organization](#6.-test-organization)  
7. [Navigation & Waiting](#7-navigation--waiting)  
8. [Working with Multiple Contexts](#8.-working-with-multiple-contexts)  
9. [Network Interception & Mocking](#9-network-interception--mocking)  
10. [Authentication & State Management](#10-authentication--state-management)  
11. [Configuration](#11.-configuration)  
12. [Quick Reference Cheat Sheet](#12.-quick-reference-cheat-sheet)

---

## 6\. Test Organization

### Fixtures — built-in fixtures and custom fixtures

A **fixture** is something the test needs to run — Playwright creates it, hands it to your test function, and cleans it up afterward automatically.

```javascript
import { test, expect } from '@playwright/test';

// 'page', 'context', and 'browser' below are BUILT-IN fixtures.
// You never create them yourself — just ask for them as parameters.
test('built-in fixtures', async ({ page, context, browser }) => {
  console.log(browser.version());       // the Browser instance
  console.log(context);                  // the BrowserContext (isolated session)
  await page.goto('https://example.com'); // the Page (tab) — most commonly used
});
```

| Built-in fixture | What it gives you |
| :---- | :---- |
| `page` | A fresh browser tab, isolated per test |
| `context` | The isolated session (cookies/storage) the page belongs to |
| `browser` | The shared browser instance for the whole worker |
| `browserName` | String: `'chromium'`, `'firefox'`, or `'webkit'` |
| `request` | An API-testing client, no browser needed (see Section 9\) |

**Custom fixtures** — you define your own reusable setup, e.g. a logged-in page, or a Page Object. This avoids repeating the same setup code (like login) in every single test.

```javascript
// fixtures.js
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

// Extend the base 'test' with your own fixture called 'loginPage'
export const test = base.extend({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await page.goto('https://example.com/login');
    await use(loginPage);   // hand it to the test
    // (any code after 'use' would run as teardown, after the test finishes)
  },
});

export { expect } from '@playwright/test';
```

```javascript
// login.spec.js
import { test, expect } from './fixtures';

test('user logs in', async ({ loginPage, page }) => {
  await loginPage.login('admin', '1234');
  await expect(page.locator('.dashboard')).toBeVisible();
});
```

### Page Object Model (POM) pattern

Each page/screen of the app becomes a class. Locators and actions live inside the class; the test file only describes *what* to do, not *how* to find elements.

```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.username = page.getByLabel('Username');
    this.password = page.getByLabel('Password');
    this.loginBtn = page.getByRole('button', { name: 'Login' });
  }

  async login(username, password) {
    await this.username.fill(username);
    await this.password.fill(password);
    await this.loginBtn.click();
  }
}
```

```javascript
// login.spec.js
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('valid login', async ({ page }) => {
  await page.goto('https://example.com/login');
  const loginPage = new LoginPage(page);
  await loginPage.login('admin', '1234');
  await expect(page.locator('.dashboard')).toBeVisible();
});
```

**Why use POM:** if the login form's markup changes, you fix it in **one file** (`LoginPage.js`) instead of every test file that logs in.

### Grouping, tagging, and controlling which tests run

```javascript
import { test, expect } from '@playwright/test';

test.describe('Checkout flow', () => {

  test('add item to cart', async ({ page }) => { /* ... */ });

  test('apply discount code', async ({ page }) => { /* ... */ });

  // test.skip -> never runs, useful for temporarily disabling a broken test
  test.skip('legacy payment method', async ({ page }) => { /* ... */ });

  // test.only -> ONLY this test runs in the file (great for focused debugging,
  // but remove before committing — easy to forget and accidentally skip everything else)
  test.only('debugging this one test', async ({ page }) => { /* ... */ });

  // test.fixme -> marks test as "known broken, fix later" — reported separately, doesn't fail the run
  test.fixme('flaky payment gateway test', async ({ page }) => { /* ... */ });

});
```

**Tagging tests** — attach tags like `@smoke` or `@regression` so you can run subsets:

```javascript
test('checkout completes successfully @smoke', async ({ page }) => { /* ... */ });
```

```shell
npx playwright test --grep @smoke        # runs only tests with "@smoke" in the title
npx playwright test --grep-invert @slow  # runs everything EXCEPT tests tagged @slow
```

Conditional skipping (e.g. skip a test only on a specific browser):

```javascript
test('webkit-only bug check', async ({ page, browserName }) => {
  test.skip(browserName !== 'webkit', 'This bug only reproduces in WebKit');
  // ...
});
```

---

## 7\. Navigation & Waiting

### `page.goto()` and waiting for navigation

```javascript
await page.goto('https://example.com');

// Control what "loaded" means before goto() resolves
await page.goto('https://example.com', { waitUntil: 'load' });        // full page load (default-ish)
await page.goto('https://example.com', { waitUntil: 'domcontentloaded' }); // DOM ready, faster
await page.goto('https://example.com', { waitUntil: 'networkidle' });  // no network activity for 500ms
```

- **`load`** — waits for the full `load` event (images, styles, everything).  
- **`domcontentloaded`** — waits only for HTML/DOM to be parsed — faster, but scripts/images might still be loading.  
- **`networkidle`** — waits until there's been no network traffic for a short window. Useful for pages that fetch data via API calls after the initial load — but can be slow/unreliable on pages with constant background polling (e.g. analytics beacons), so use with care.

**Waiting for navigation triggered by a click** (e.g. clicking a link that goes to a new page):

```javascript
// Playwright usually handles this automatically, but if you need to be explicit:
await Promise.all([
  page.waitForNavigation(),          // (legacy API, still works)
  page.locator('#nextPageLink').click(),
]);

// Modern equivalent — often not even needed, Playwright auto-waits for the next action
await page.locator('#nextPageLink').click();
await page.waitForURL('**/next-page');
```

### `waitForSelector`, `waitForResponse`, `waitForRequest`

```javascript
// waitForSelector — wait for an element to reach a certain state
await page.waitForSelector('.results-loaded');                    // default: visible
await page.waitForSelector('.spinner', { state: 'hidden' });      // wait until spinner disappears
await page.waitForSelector('.item', { state: 'attached' });       // exists in DOM (may not be visible)
```

> Note: with modern Playwright, `locator.click()` etc. already auto-wait, so `waitForSelector` is mostly needed for less common cases — like waiting for something to *disappear*, or waiting before doing a non-action check.

```javascript
// waitForResponse — wait for a specific network response (very useful after an action
// that triggers an API call, e.g. clicking "Save" triggers a POST request)
const [response] = await Promise.all([
  page.waitForResponse(resp => resp.url().includes('/api/save') && resp.status() === 200),
  page.locator('#saveBtn').click(),
]);
const body = await response.json();
console.log(body);

// waitForRequest — similar, but waits for the REQUEST to be sent (not the response)
const [request] = await Promise.all([
  page.waitForRequest(req => req.url().includes('/api/analytics')),
  page.locator('#submitBtn').click(),
]);
console.log(request.method(), request.url());
```

### Handling SPA behavior, dynamic content, and race conditions

Single Page Applications (React/Angular/Vue) often update content **without a full page navigation** — clicking a tab just re-renders part of the DOM via JavaScript. This can cause race conditions if your test doesn't wait for the right thing.

```javascript
// PROBLEM: clicking a tab, then immediately expecting new content — but the app takes
// a moment to fetch and render it. A naive script might check too early and fail.

// SOLUTION 1: let a web-first assertion do the retrying for you (usually enough)
await page.locator('.tab-orders').click();
await expect(page.locator('.orders-table')).toBeVisible();  // retries until visible

// SOLUTION 2: wait for the underlying API call that populates the content
const [response] = await Promise.all([
  page.waitForResponse('**/api/orders'),
  page.locator('.tab-orders').click(),
]);
await expect(page.locator('.orders-table')).toBeVisible();

// AVOID: fixed sleeps — they're either too short (flaky) or too long (slow)
await page.waitForTimeout(2000); // ❌ last resort only, avoid if possible
```

**Rule of thumb:** prefer asserting on the *result* (an element appearing, text changing) over waiting a fixed amount of time. Playwright's web-first assertions handle 90% of SPA timing issues on their own.

---

## 8\. Working with Multiple Contexts

### Multiple tabs/windows (within the same context)

```javascript
// Tab opened by clicking a link with target="_blank"
const [newTab] = await Promise.all([
  context.waitForEvent('page'),
  page.locator('a[target="_blank"]').click(),
]);
await newTab.waitForLoadState();
console.log(await newTab.title());

// Opening a new tab yourself
const anotherTab = await context.newPage();
await anotherTab.goto('https://example.com/settings');
```

### Multiple browser contexts (simulating different users/sessions)

Each `BrowserContext` is fully isolated — separate cookies, localStorage, and sessions. This is the standard way to test multi-user interactions (e.g. a chat app, or an admin approving something a regular user submitted) **within a single test**, without needing two separate browsers.

```javascript
import { chromium } from '@playwright/test';

const browser = await chromium.launch();

// User A's session
const userAContext = await browser.newContext();
const userAPage = await userAContext.newPage();
await userAPage.goto('https://example.com/login');
// ... log in as User A

// User B's session — completely separate cookies/storage
const userBContext = await browser.newContext();
const userBPage = await userBContext.newPage();
await userBPage.goto('https://example.com/login');
// ... log in as User B

// Now you can simulate: User A sends a message, User B sees it appear
await userAPage.locator('#messageBox').fill('Hello!');
await userAPage.locator('#sendBtn').click();
await expect(userBPage.locator('.chat-message').last()).toHaveText('Hello!');

await userAContext.close();
await userBContext.close();
await browser.close();
```

### iframes

An iframe is a "page within a page" — elements inside it aren't part of the main page's DOM, so you need to target the frame first.

```javascript
// By selector pointing to the <iframe> element
const frame = page.frameLocator('iframe#payment-frame');
await frame.locator('#cardNumber').fill('4111111111111111');

// By frame name or URL, if there's no easy selector
const frameByUrl = page.frame({ url: /checkout-frame/ });
await frameByUrl.locator('#cvv').fill('123');
```

### Shadow DOM

Shadow DOM is used by Web Components to encapsulate their internal markup/styles. The good news: **Playwright's locators pierce shadow DOM automatically** — you usually don't need any special syntax.

```javascript
// This works even if '.custom-button' is inside a shadow root —
// Playwright looks inside open shadow roots by default.
await page.locator('.custom-button').click();
```

This is a major advantage over older tools like Selenium, which need extra JavaScript workarounds to pierce shadow DOM.

---

## 9\. Network Interception & Mocking

### `page.route()` to intercept and mock API responses

```javascript
// Intercept a specific API call and return fake/mock data instead of hitting the real server
await page.route('**/api/users', async (route) => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([{ id: 1, name: 'Mock User' }]),
  });
});

await page.goto('https://example.com/users');
// The page will now render using the mocked response, not the real API
```

**Why mock:** test how the UI behaves for scenarios that are hard to set up for real — empty lists, server errors, slow responses, edge-case data — without needing the backend to actually produce them.

```javascript
// Simulate a server error
await page.route('**/api/checkout', route => {
  route.fulfill({ status: 500, body: 'Internal Server Error' });
});

// Simulate a slow network (delay before responding)
await page.route('**/api/data', async (route) => {
  await new Promise(resolve => setTimeout(resolve, 3000));
  await route.continue();
});
```

### Modifying requests/responses (instead of fully replacing them)

```javascript
// Modify an outgoing request before it's sent
await page.route('**/api/search*', (route) => {
  const url = new URL(route.request().url());
  url.searchParams.set('debug', 'true');   // add a query param
  route.continue({ url: url.toString() });
});

// Modify a real response before it reaches the page (e.g. tweak one field)
await page.route('**/api/profile', async (route) => {
  const response = await route.fetch();          // get the REAL response
  const json = await response.json();
  json.isAdmin = true;                            // tamper with one field
  await route.fulfill({ response, json });
});
```

### Blocking resources (images, ads) for faster tests

```javascript
await page.route('**/*.{png,jpg,jpeg,svg,gif}', route => route.abort());
await page.route('**/*ads*', route => route.abort());
await page.route('**/*analytics*', route => route.abort());
```

Blocking unnecessary resources can noticeably speed up test runs, since the browser doesn't waste time downloading/rendering things your test doesn't care about.

### API testing with Playwright's `request` fixture (no browser needed)

Playwright can also test APIs directly — useful for setting up test data quickly, or testing backend endpoints without the overhead of launching a browser.

```javascript
import { test, expect } from '@playwright/test';

test('API test - get users', async ({ request }) => {
  const response = await request.get('https://api.example.com/users');
  expect(response.status()).toBe(200);

  const body = await response.json();
  expect(body.length).toBeGreaterThan(0);
});

test('API test - create user', async ({ request }) => {
  const response = await request.post('https://api.example.com/users', {
    data: { name: 'Ravi', email: 'ravi@test.com' },
  });
  expect(response.status()).toBe(201);
});
```

**Common pattern:** use the `request` fixture to create test data via API (fast) before the actual UI test runs (slow), instead of clicking through the UI just to set up preconditions.

```javascript
test('edit a user created via API', async ({ request, page }) => {
  // Fast setup via API
  const createResp = await request.post('/api/users', { data: { name: 'Temp User' } });
  const { id } = await createResp.json();

  // Now test the actual UI behavior
  await page.goto(`/users/${id}/edit`);
  await page.getByLabel('Name').fill('Updated Name');
  await page.getByRole('button', { name: 'Save' }).click();
  await expect(page.locator('.success-msg')).toBeVisible();
});
```

---

## 10\. Authentication & State Management

### Storing and reusing login state (`storageState`)

Logging in through the UI for *every single test* is slow. Instead, log in **once**, save the resulting cookies/storage to a file, and reuse it across tests.

```javascript
// auth-setup.js — run once to log in and save the session
import { chromium } from '@playwright/test';

const browser = await chromium.launch();
const page = await browser.newPage();

await page.goto('https://example.com/login');
await page.getByLabel('Username').fill('admin');
await page.getByLabel('Password').fill('1234');
await page.getByRole('button', { name: 'Login' }).click();
await page.waitForURL('**/dashboard');

// Save cookies + localStorage to a file
await page.context().storageState({ path: 'auth.json' });
await browser.close();
```

```javascript
// Now any test can reuse this saved session — starts already logged in
import { test } from '@playwright/test';

test.use({ storageState: 'auth.json' });

test('dashboard is visible without logging in again', async ({ page }) => {
  await page.goto('https://example.com/dashboard');
  // no login step needed — session was restored from auth.json
});
```

### Global setup/teardown for auth

Rather than manually running a setup script, Playwright can run it automatically once before all tests, via `playwright.config.js`.

```javascript
// global-setup.js
import { chromium } from '@playwright/test';

export default async function globalSetup() {
  const browser = await chromium.launch();
  const page = await browser.newPage();

  await page.goto('https://example.com/login');
  await page.getByLabel('Username').fill('admin');
  await page.getByLabel('Password').fill('1234');
  await page.getByRole('button', { name: 'Login' }).click();
  await page.context().storageState({ path: 'auth.json' });

  await browser.close();
}
```

```javascript
// playwright.config.js
export default defineConfig({
  globalSetup: './global-setup.js',
  use: {
    storageState: 'auth.json',   // every test starts already authenticated
  },
});
```

`globalTeardown` works the same way — point to a file in the config, useful for cleaning up test data created during the run (e.g. deleting a test user via API).

### Handling cookies, local storage, session storage

```javascript
// Cookies
const cookies = await context.cookies();
await context.addCookies([
  { name: 'session_id', value: 'abc123', domain: 'example.com', path: '/' },
]);
await context.clearCookies();

// localStorage / sessionStorage — accessed via page.evaluate (runs JS in the browser)
await page.evaluate(() => {
  localStorage.setItem('theme', 'dark');
  sessionStorage.setItem('tempFlag', 'true');
});

const theme = await page.evaluate(() => localStorage.getItem('theme'));
console.log(theme); // "dark"
```

---

## 11\. Configuration

### `playwright.config.js` deep dive

```javascript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30000,              // max time per test (ms)
  expect: {
    timeout: 5000,              // max time for each expect() to succeed
  },
  fullyParallel: true,          // run tests in files fully in parallel
  retries: process.env.CI ? 2 : 0,   // retry failing tests more aggressively on CI
  workers: process.env.CI ? 2 : undefined, // number of parallel workers

  reporter: [
    ['html'],                                  // interactive HTML report
    ['list'],                                  // console output while running
    ['junit', { outputFile: 'results.xml' }],  // for CI dashboards (Jenkins, etc.)
  ],

  use: {
    baseURL: 'https://staging.example.com',
    trace: 'on-first-retry',       // capture a trace ONLY when a test is retried after failing
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  // 'projects' = run the SAME tests across multiple browsers/devices
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 7'] } },
  ],
});
```

- **`projects`** is how you get multi-browser AND multi-device runs from one test suite — Playwright runs every test once per project.  
- **`retries`** is often set higher in CI than locally, since CI environments can be flakier (shared machines, network variance).  
- **`trace: 'on-first-retry'`** is a good default — traces are detailed (and heavier) so you usually only want them when something actually failed.

### Environment variables and `.env` handling

```shell
npm install dotenv --save-dev
```

```
# .env
BASE_URL=https://staging.example.com
TEST_USERNAME=admin
TEST_PASSWORD=1234
```

```javascript
// playwright.config.js
import 'dotenv/config';   // loads .env into process.env automatically

export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL,
  },
});
```

```javascript
// inside a test
const username = process.env.TEST_USERNAME;
const password = process.env.TEST_PASSWORD;
await page.getByLabel('Username').fill(username);
await page.getByLabel('Password').fill(password);
```

> **Tip:** add `.env` to `.gitignore` — never commit real credentials. Commit a `.env.example` with placeholder values instead, so teammates know which variables are needed.

### Running tests against different environments (staging, prod)

**Option A — separate `.env` files per environment:**

```
.env.staging
.env.prod
```

```shell
BASE_URL=$(grep BASE_URL .env.staging | cut -d '=' -f2) npx playwright test
```

**Option B — pass the environment name and branch inside the config:**

```javascript
// playwright.config.js
const ENV = process.env.TEST_ENV || 'staging';

const environments = {
  staging: 'https://staging.example.com',
  prod: 'https://www.example.com',
};

export default defineConfig({
  use: {
    baseURL: environments[ENV],
  },
});
```

```shell
TEST_ENV=prod npx playwright test
TEST_ENV=staging npx playwright test
```

Then in tests, always use relative paths with `baseURL` already set, so switching environments requires no code changes:

```javascript
await page.goto('/login');   // resolves to baseURL + '/login'
```

---

## 12\. Quick Reference Cheat Sheet

```javascript
// FIXTURES & POM
test.extend({ myFixture: async ({page}, use) => { ...; await use(value); } })
class LoginPage { constructor(page) { this.page = page; } async login() {...} }

// TEST CONTROL
test.describe('group', () => { ... });
test.skip('name', async () => {...});
test.only('name', async () => {...});
test.fixme('name', async () => {...});
npx playwright test --grep @smoke

// NAVIGATION & WAITING
await page.goto(url, { waitUntil: 'networkidle' });
await page.waitForSelector('.el', { state: 'hidden' });
await page.waitForResponse(url => url.includes('/api/save'));
await expect(locator).toBeVisible();  // prefer this over manual waits

// MULTIPLE CONTEXTS
const context2 = await browser.newContext();
const frame = page.frameLocator('iframe#name');

// NETWORK MOCKING
await page.route('**/api/users', route => route.fulfill({ status: 200, body: '...' }));
await page.route('**/*.png', route => route.abort());
const response = await request.get('/api/users');  // pure API test, no browser

// AUTH / STATE
await page.context().storageState({ path: 'auth.json' });
test.use({ storageState: 'auth.json' });
globalSetup: './global-setup.js'   // in playwright.config.js

// CONFIG
projects: [{ name: 'chromium', use: {...devices['Desktop Chrome']} }]
retries: process.env.CI ? 2 : 0
TEST_ENV=staging npx playwright test
```

---

## Suggested Practice Order

1. Convert your Phase 1 login test into the **Page Object Model** pattern.  
2. Write a **custom fixture** that logs in automatically before every test.  
3. Practice `waitForResponse` by clicking something on a real demo site that triggers an API call, and logging the response body.  
4. Try **mocking an API response** with `page.route()` to force an error state in the UI, and assert the error message appears correctly.  
5. Save a logged-in session with `storageState`, then write a second test that reuses it and skips the login step entirely.  
6. Add a second `project` (e.g. Firefox) to your config and run the same test on both browsers.

---

*Tip: Ask me for a small hands-on exercise (e.g. "give me a mini exercise combining POM \+ custom fixture \+ API mocking"), or paste your own Playwright script here for review.*  
