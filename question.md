# Complete Playwright Interview Questions

Concept → Challenge format covering all Playwright features and APIs.

---

## 1. LOCATORS

### Concept Questions

**Q1:** What is the difference between `page.locator()` and `page.getByRole()`?
> `locator()` uses CSS/XPath selectors while `getByRole()` uses ARIA roles - more accessible and stable.

**Q2:** Why is `getByRole()` preferred over CSS selectors?
> It mirrors how users interact with the page, survives UI changes, and promotes accessibility.

**Q3:** What does `filter()` do on a locator?
> Narrows down elements by text content or child elements.

**Q4:** Explain the difference between `first()`, `last()`, and `nth()`.
> `first()` - first match, `last()` - last match, `nth(index)` - specific index (0-based).

### Challenges

**Challenge 1:** Locate and click "Delete" button for user "john@test.com" in this table:
```html
<table>
  <tr><td>john@test.com</td><td><button>Edit</button><button>Delete</button></td></tr>
  <tr><td>jane@test.com</td><td><button>Edit</button><button>Delete</button></td></tr>
</table>
```
```typescript
// Solution:
await page.getByRole('row', { name: 'john@test.com' })
    .getByRole('button', { name: 'Delete' })
    .click();
```

**Challenge 2:** Select the 3rd item in a dropdown that doesn't use `<select>`:
```html
<div class="dropdown">
  <div class="option">Option 1</div>
  <div class="option">Option 2</div>
  <div class="option">Option 3</div>
</div>
```
```typescript
// Solution:
await page.locator('.dropdown .option').nth(2).click();
```

**Challenge 3:** Click a button that's inside a card with specific title:
```typescript
// Solution:
await page.locator('.card')
    .filter({ has: page.getByText('Premium Plan') })
    .getByRole('button', { name: 'Subscribe' })
    .click();
```

---

## 2. ACTIONS

### Concept Questions

**Q1:** What's the difference between `fill()` and `type()`?
> `fill()` clears then sets value instantly. `type()` simulates keystrokes with optional delay.

**Q2:** When would you use `press()` vs `keyboard.press()`?
> `press()` is on a locator (element-specific). `keyboard.press()` is global (any focused element).

**Q3:** What does `force: true` do in click actions?
> Bypasses actionability checks - clicks even if element is covered or not visible.

**Q4:** How does `setInputFiles()` handle file uploads?
> Sets files directly on input[type=file] without opening file dialog.

### Challenges

**Challenge 1:** Fill a form with keyboard navigation:
```typescript
// Solution:
await page.getByLabel('Username').fill('john');
await page.keyboard.press('Tab');
await page.getByLabel('Password').fill('secret');
await page.keyboard.press('Enter');
```

**Challenge 2:** Perform Ctrl+Click to open link in new tab:
```typescript
// Solution:
await page.getByRole('link', { name: 'Dashboard' }).click({ modifiers: ['Control'] });
```

**Challenge 3:** Upload multiple files:
```typescript
// Solution:
await page.setInputFiles('input[type="file"]', [
    'files/doc1.pdf',
    'files/doc2.pdf'
]);
```

**Challenge 4:** Clear an input field without using `fill('')`:
```typescript
// Solution:
await page.getByLabel('Search').press('Control+a');
await page.keyboard.press('Backspace');
```

---

## 3. ASSERTIONS (expect)

### Concept Questions

**Q1:** What's the difference between `toBeVisible()` and `toBeAttached()`?
> `toBeVisible()` - element is in DOM and visible. `toBeAttached()` - element is in DOM (may be hidden).

**Q2:** What does `expect.soft()` do?
> Continues test execution even if assertion fails - collects all failures.

**Q3:** How does `toHaveScreenshot()` work?
> Compares current screenshot with baseline. Fails if difference exceeds threshold.

**Q4:** What is `expect.poll()` used for?
> Repeatedly calls a function until it returns expected value or times out.

### Challenges

**Challenge 1:** Verify a list has between 5 and 10 items:
```typescript
// Solution:
const count = await page.locator('.list-item').count();
expect(count).toBeGreaterThanOrEqual(5);
expect(count).toBeLessThanOrEqual(10);
```

**Challenge 2:** Verify element becomes enabled within 10 seconds:
```typescript
// Solution:
await expect(page.getByRole('button', { name: 'Submit' }))
    .toBeEnabled({ timeout: 10000 });
```

**Challenge 3:** Verify API returns 200 using polling:
```typescript
// Solution:
await expect.poll(async () => {
    const response = await page.request.get('/api/status');
    return response.status();
}, { timeout: 30000 }).toBe(200);
```

**Challenge 4:** Soft assertions for multiple fields:
```typescript
// Solution:
await expect.soft(page.getByLabel('Name')).toHaveValue('John');
await expect.soft(page.getByLabel('Email')).toHaveValue('john@test.com');
await expect.soft(page.getByLabel('Phone')).toHaveValue('1234567890');
// Test continues even if some fail
```

---

## 4. NAVIGATION & WAITS

### Concept Questions

**Q1:** What are the different load states in `waitForLoadState()`?
> `load` - window.onload fired. `domcontentloaded` - DOM ready. `networkidle` - no network for 500ms.

**Q2:** When would you use `waitForURL()` vs `waitForNavigation()`?
> `waitForURL()` - wait for specific URL pattern. `waitForNavigation()` - wait for any navigation.

**Q3:** What does `waitForResponse()` return?
> Returns the Response object matching the URL/predicate.

**Q4:** Why should you avoid `waitForTimeout()`?
> It's a hard wait - wastes time or may not be enough. Use explicit waits instead.

### Challenges

**Challenge 1:** Click button and wait for specific API response:
```typescript
// Solution:
const [response] = await Promise.all([
    page.waitForResponse(resp => 
        resp.url().includes('/api/save') && resp.status() === 200
    ),
    page.getByRole('button', { name: 'Save' }).click()
]);
const data = await response.json();
```

**Challenge 2:** Wait for URL to contain "success" or "error":
```typescript
// Solution:
await page.waitForURL(/\/(success|error)$/);
```

**Challenge 3:** Wait for element to disappear:
```typescript
// Solution:
await page.locator('.loading-spinner').waitFor({ state: 'hidden' });
// OR
await expect(page.locator('.loading-spinner')).toBeHidden();
```

**Challenge 4:** Navigate and wait for all resources:
```typescript
// Solution:
await page.goto('/dashboard', { waitUntil: 'networkidle' });
```

---

## 5. FRAMES & IFRAMES

### Concept Questions

**Q1:** What's the difference between `frame()` and `frameLocator()`?
> `frame()` returns Frame object (older). `frameLocator()` returns FrameLocator for chaining with locators.

**Q2:** How do you handle nested iframes?
> Chain `frameLocator()` calls: `page.frameLocator('#outer').frameLocator('#inner')`.

**Q3:** Can you use `getByRole()` inside a frame?
> Yes, with frameLocator: `page.frameLocator('#frame').getByRole('button')`.

### Challenges

**Challenge 1:** Fill form inside iframe:
```typescript
// Solution:
const frame = page.frameLocator('#payment-iframe');
await frame.getByLabel('Card Number').fill('4111111111111111');
await frame.getByLabel('Expiry').fill('12/25');
await frame.getByRole('button', { name: 'Pay' }).click();
```

**Challenge 2:** Get text from nested iframe:
```typescript
// Solution:
const content = await page
    .frameLocator('#outer-frame')
    .frameLocator('#inner-frame')
    .locator('.message')
    .textContent();
```

**Challenge 3:** Handle iframe that loads dynamically:
```typescript
// Solution:
await page.waitForSelector('iframe#dynamic-frame');
const frame = page.frameLocator('#dynamic-frame');
await frame.getByRole('button').click();
```

---

## 6. MULTIPLE PAGES & POPUPS

### Concept Questions

**Q1:** How do you handle a new tab opened by a click?
> Use `Promise.all` with `context.waitForEvent('page')` and the click action.

**Q2:** What's the difference between `page` and `popup` events?
> Both catch new pages. `popup` specifically catches window.open() popups.

**Q3:** How do you switch between multiple pages?
> Store page references, use `page.bringToFront()` to focus.

### Challenges

**Challenge 1:** Handle link that opens new tab:
```typescript
// Solution:
const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByRole('link', { name: 'Open Dashboard' }).click()
]);
await newPage.waitForLoadState();
await expect(newPage).toHaveTitle('Dashboard');
```

**Challenge 2:** Handle popup and extract data:
```typescript
// Solution:
const [popup] = await Promise.all([
    page.waitForEvent('popup'),
    page.getByRole('button', { name: 'View Details' }).click()
]);
const details = await popup.locator('.details').textContent();
await popup.close();
```

**Challenge 3:** Work with multiple tabs:
```typescript
// Solution:
const page1 = await context.newPage();
const page2 = await context.newPage();

await page1.goto('/page1');
await page2.goto('/page2');

// Switch to page1
await page1.bringToFront();
await page1.getByRole('button').click();
```

---

## 7. DIALOGS (Alert, Confirm, Prompt)

### Concept Questions

**Q1:** Why must dialog handlers be set up before triggering the dialog?
> Dialogs are blocking - they freeze JavaScript execution until handled.

**Q2:** What's the difference between `accept()` and `dismiss()`?
> `accept()` clicks OK/Yes. `dismiss()` clicks Cancel/No.

**Q3:** How do you handle a prompt dialog?
> Use `dialog.accept('input text')` to enter value and confirm.

### Challenges

**Challenge 1:** Handle confirm dialog and verify message:
```typescript
// Solution:
page.once('dialog', async dialog => {
    expect(dialog.message()).toContain('Are you sure');
    await dialog.accept();
});
await page.getByRole('button', { name: 'Delete' }).click();
```

**Challenge 2:** Handle prompt and enter text:
```typescript
// Solution:
page.once('dialog', async dialog => {
    expect(dialog.type()).toBe('prompt');
    await dialog.accept('My new name');
});
await page.getByRole('button', { name: 'Rename' }).click();
```

**Challenge 3:** Dismiss alert and verify page state:
```typescript
// Solution:
page.once('dialog', dialog => dialog.dismiss());
await page.getByRole('button', { name: 'Cancel Order' }).click();
await expect(page.getByText('Order not cancelled')).toBeVisible();
```

---

## 8. FILE DOWNLOAD

### Concept Questions

**Q1:** How does Playwright handle file downloads?
> Listen to `download` event, then use `download.saveAs()` or `download.path()`.

**Q2:** What information can you get from a Download object?
> `suggestedFilename()`, `url()`, `path()`, `failure()`.

**Q3:** How do you enable downloads in browser context?
> Set `acceptDownloads: true` in context options (default is true).

### Challenges

**Challenge 1:** Download file and verify filename:
```typescript
// Solution:
const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.getByRole('link', { name: 'Download Report' }).click()
]);
expect(download.suggestedFilename()).toBe('report.pdf');
await download.saveAs('downloads/report.pdf');
```

**Challenge 2:** Download and read file content:
```typescript
// Solution:
const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.getByRole('button', { name: 'Export CSV' }).click()
]);
const path = await download.path();
const content = fs.readFileSync(path!, 'utf-8');
expect(content).toContain('Name,Email');
```

---

## 9. NETWORK INTERCEPTION

### Concept Questions

**Q1:** What's the difference between `route.fulfill()`, `route.continue()`, and `route.abort()`?
> `fulfill()` - mock response. `continue()` - proceed with optional modifications. `abort()` - block request.

**Q2:** Can you modify a response from the actual server?
> Yes, use `route.fetch()` to get real response, modify it, then `route.fulfill()`.

**Q3:** What does `page.unroute()` do?
> Removes a previously set route handler.

**Q4:** How do you mock only specific HTTP methods?
> Check `route.request().method()` in the handler.

### Challenges

**Challenge 1:** Mock API to return custom data:
```typescript
// Solution:
await page.route('**/api/users', route => {
    route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([{ id: 1, name: 'Mock User' }])
    });
});
```

**Challenge 2:** Add authorization header to all API requests:
```typescript
// Solution:
await page.route('**/api/**', route => {
    route.continue({
        headers: {
            ...route.request().headers(),
            'Authorization': 'Bearer test-token'
        }
    });
});
```

**Challenge 3:** Modify response from server:
```typescript
// Solution:
await page.route('**/api/products', async route => {
    const response = await route.fetch();
    const json = await response.json();
    json.products.forEach(p => p.price = 0); // Make all free
    await route.fulfill({ response, json });
});
```

**Challenge 4:** Block all images for faster tests:
```typescript
// Solution:
await page.route('**/*.{png,jpg,jpeg,gif,svg}', route => route.abort());
```

**Challenge 5:** Mock error response:
```typescript
// Solution:
await page.route('**/api/submit', route => {
    route.fulfill({
        status: 500,
        body: JSON.stringify({ error: 'Server Error' })
    });
});
```

---

## 10. REQUEST/RESPONSE MONITORING

### Concept Questions

**Q1:** What events can you listen to for network monitoring?
> `request`, `response`, `requestfailed`, `requestfinished`.

**Q2:** How do you get request body/payload?
> Use `request.postData()` or `request.postDataJSON()`.

**Q3:** How do you get response body?
> Use `response.json()`, `response.text()`, or `response.body()`.

### Challenges

**Challenge 1:** Log all API calls:
```typescript
// Solution:
page.on('request', request => {
    if (request.url().includes('/api/')) {
        console.log(`>> ${request.method()} ${request.url()}`);
    }
});

page.on('response', response => {
    if (response.url().includes('/api/')) {
        console.log(`<< ${response.status()} ${response.url()}`);
    }
});
```

**Challenge 2:** Capture specific API response:
```typescript
// Solution:
const responsePromise = page.waitForResponse('**/api/user/profile');
await page.getByRole('button', { name: 'Load Profile' }).click();
const response = await responsePromise;
const userData = await response.json();
expect(userData.name).toBe('John');
```

**Challenge 3:** Track failed requests:
```typescript
// Solution:
const failedRequests: string[] = [];
page.on('requestfailed', request => {
    failedRequests.push(`${request.url()} - ${request.failure()?.errorText}`);
});

// After test
expect(failedRequests).toHaveLength(0);
```

---

## 11. BROWSER CONTEXT

### Concept Questions

**Q1:** What is a BrowserContext?
> Isolated browser session with separate cookies, storage, and cache. Like incognito window.

**Q2:** How do you persist authentication across tests?
> Save state with `context.storageState()`, load with context option `storageState`.

**Q3:** What's the difference between browser and context level settings?
> Browser settings are shared. Context settings are isolated per context.

### Challenges

**Challenge 1:** Create context with custom viewport and locale:
```typescript
// Solution:
const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 },
    locale: 'de-DE',
    timezoneId: 'Europe/Berlin'
});
```

**Challenge 2:** Save and reuse authentication:
```typescript
// Solution:
// Save after login
await page.goto('/login');
await page.fill('#username', 'admin');
await page.fill('#password', 'secret');
await page.click('#login');
await context.storageState({ path: 'auth.json' });

// Reuse in new context
const authContext = await browser.newContext({
    storageState: 'auth.json'
});
```

**Challenge 3:** Test with multiple users simultaneously:
```typescript
// Solution:
const adminContext = await browser.newContext({ storageState: 'admin-auth.json' });
const userContext = await browser.newContext({ storageState: 'user-auth.json' });

const adminPage = await adminContext.newPage();
const userPage = await userContext.newPage();

await adminPage.goto('/admin/dashboard');
await userPage.goto('/user/dashboard');
```

---

## 12. COOKIES & STORAGE

### Concept Questions

**Q1:** How do you add cookies to a context?
> Use `context.addCookies([{ name, value, domain, path }])`.

**Q2:** How do you access localStorage?
> Use `page.evaluate(() => localStorage.getItem('key'))`.

**Q3:** What does `storageState` include?
> Cookies and localStorage/sessionStorage for all origins.

### Challenges

**Challenge 1:** Set authentication cookie:
```typescript
// Solution:
await context.addCookies([{
    name: 'session',
    value: 'abc123',
    domain: 'example.com',
    path: '/'
}]);
```

**Challenge 2:** Read and verify localStorage:
```typescript
// Solution:
const token = await page.evaluate(() => localStorage.getItem('authToken'));
expect(token).toBeTruthy();
```

**Challenge 3:** Clear all storage between tests:
```typescript
// Solution:
await context.clearCookies();
await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
});
```

---

## 13. GEOLOCATION & PERMISSIONS

### Concept Questions

**Q1:** How do you set geolocation in Playwright?
> Use `context.setGeolocation({ latitude, longitude })` with `permissions: ['geolocation']`.

**Q2:** What permissions can be granted?
> `geolocation`, `notifications`, `camera`, `microphone`, etc.

### Challenges

**Challenge 1:** Test location-based feature:
```typescript
// Solution:
const context = await browser.newContext({
    geolocation: { latitude: 40.7128, longitude: -74.0060 },
    permissions: ['geolocation']
});
const page = await context.newPage();
await page.goto('/store-locator');
await expect(page.getByText('New York')).toBeVisible();
```

**Challenge 2:** Change location during test:
```typescript
// Solution:
await context.setGeolocation({ latitude: 51.5074, longitude: -0.1278 });
await page.reload();
await expect(page.getByText('London')).toBeVisible();
```

---

## 14. MOBILE EMULATION

### Concept Questions

**Q1:** How do you emulate a mobile device?
> Use `devices` from Playwright: `browser.newContext({ ...devices['iPhone 13'] })`.

**Q2:** What properties does device emulation set?
> viewport, userAgent, deviceScaleFactor, isMobile, hasTouch.

**Q3:** How do you test touch interactions?
> Use `page.tap()` or `page.touchscreen.tap(x, y)`.

### Challenges

**Challenge 1:** Test responsive menu on mobile:
```typescript
// Solution:
import { devices } from '@playwright/test';

const context = await browser.newContext({
    ...devices['iPhone 13']
});
const page = await context.newPage();
await page.goto('/');
await page.tap('.hamburger-menu');
await expect(page.getByRole('navigation')).toBeVisible();
```

**Challenge 2:** Test both desktop and mobile viewports:
```typescript
// Solution:
// In playwright.config.ts
projects: [
    { name: 'Desktop Chrome', use: { ...devices['Desktop Chrome'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } },
]
```

---

## 15. SCREENSHOTS & VIDEOS

### Concept Questions

**Q1:** What screenshot options are available?
> `fullPage`, `clip`, `path`, `type` (png/jpeg), `mask`.

**Q2:** When are videos recorded?
> Based on `video` option: `on`, `off`, `retain-on-failure`, `on-first-retry`.

**Q3:** How do you mask dynamic content in screenshots?
> Use `mask` option with locators: `mask: [page.locator('.timestamp')]`.

### Challenges

**Challenge 1:** Take screenshot hiding sensitive data:
```typescript
// Solution:
await page.screenshot({
    path: 'dashboard.png',
    fullPage: true,
    mask: [
        page.locator('.credit-card'),
        page.locator('.ssn')
    ]
});
```

**Challenge 2:** Screenshot specific element:
```typescript
// Solution:
await page.locator('.chart').screenshot({ path: 'chart.png' });
```

**Challenge 3:** Visual comparison with threshold:
```typescript
// Solution:
await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixelRatio: 0.1,
    animations: 'disabled'
});
```

---

## 16. TRACING

### Concept Questions

**Q1:** What does a trace include?
> Screenshots, DOM snapshots, network requests, console logs, action timeline.

**Q2:** What are trace options?
> `screenshots`, `snapshots`, `sources`.

**Q3:** How do you view a trace?
> `npx playwright show-trace trace.zip` or upload to trace.playwright.dev.

### Challenges

**Challenge 1:** Enable tracing for debugging:
```typescript
// Solution:
await context.tracing.start({ screenshots: true, snapshots: true });

// ... test actions ...

await context.tracing.stop({ path: 'trace.zip' });
```

**Challenge 2:** Configure tracing in config:
```typescript
// Solution:
// playwright.config.ts
use: {
    trace: 'retain-on-failure', // Only save trace on failure
}
```

---

## 17. API TESTING

### Concept Questions

**Q1:** How do you make API requests in Playwright?
> Use `page.request` or `request` fixture for standalone API calls.

**Q2:** Can you combine UI and API tests?
> Yes, use API to setup data, then verify in UI.

**Q3:** How do you send different content types?
> Use `data` for JSON, `form` for form data, `multipart` for files.

### Challenges

**Challenge 1:** Test REST API:
```typescript
// Solution:
const response = await request.post('/api/users', {
    data: { name: 'John', email: 'john@test.com' }
});
expect(response.ok()).toBeTruthy();
const user = await response.json();
expect(user.id).toBeDefined();
```

**Challenge 2:** API setup for UI test:
```typescript
// Solution:
// Create user via API
const response = await request.post('/api/users', {
    data: { name: 'Test User' }
});
const { id } = await response.json();

// Verify in UI
await page.goto(`/users/${id}`);
await expect(page.getByText('Test User')).toBeVisible();
```

**Challenge 3:** Upload file via API:
```typescript
// Solution:
const response = await request.post('/api/upload', {
    multipart: {
        file: {
            name: 'test.txt',
            mimeType: 'text/plain',
            buffer: Buffer.from('file content')
        }
    }
});
```

---

## 18. TEST HOOKS

### Concept Questions

**Q1:** What hooks are available?
> `beforeAll`, `afterAll`, `beforeEach`, `afterEach`.

**Q2:** What's the difference between `beforeAll` and `beforeEach`?
> `beforeAll` runs once before all tests. `beforeEach` runs before each test.

**Q3:** Do hooks in `describe` blocks affect other blocks?
> No, hooks are scoped to their describe block.

### Challenges

**Challenge 1:** Setup and teardown:
```typescript
// Solution:
test.beforeAll(async () => {
    // Seed database
});

test.beforeEach(async ({ page }) => {
    await page.goto('/');
});

test.afterEach(async ({ page }, testInfo) => {
    if (testInfo.status !== 'passed') {
        await page.screenshot({ path: `failure-${testInfo.title}.png` });
    }
});

test.afterAll(async () => {
    // Cleanup database
});
```

---

## 19. FIXTURES

### Concept Questions

**Q1:** What are fixtures in Playwright?
> Reusable setup/teardown logic that's injected into tests.

**Q2:** What's the difference between test-scoped and worker-scoped fixtures?
> Test-scoped: created per test. Worker-scoped: shared across tests in worker.

**Q3:** How do you override built-in fixtures?
> Extend and redefine: `page: async ({ page }, use) => { ... }`.

### Challenges

**Challenge 1:** Create page object fixture:
```typescript
// Solution:
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

const test = base.extend<{ loginPage: LoginPage }>({
    loginPage: async ({ page }, use) => {
        await use(new LoginPage(page));
    }
});

test('login test', async ({ loginPage }) => {
    await loginPage.goto();
    await loginPage.login('user', 'pass');
});
```

**Challenge 2:** Create authenticated user fixture:
```typescript
// Solution:
const test = base.extend<{ authedPage: Page }>({
    authedPage: async ({ browser }, use) => {
        const context = await browser.newContext({
            storageState: 'auth.json'
        });
        const page = await context.newPage();
        await use(page);
        await context.close();
    }
});
```

---

## 20. PARALLEL EXECUTION & SHARDING

### Concept Questions

**Q1:** How does Playwright run tests in parallel?
> Uses worker processes, each with isolated browser context.

**Q2:** What is sharding?
> Splitting tests across multiple machines for CI.

**Q3:** How do you make tests run serially?
> Use `test.describe.configure({ mode: 'serial' })`.

### Challenges

**Challenge 1:** Configure parallel execution:
```typescript
// Solution:
// playwright.config.ts
export default {
    workers: process.env.CI ? 4 : 2,
    fullyParallel: true,
};
```

**Challenge 2:** Run dependent tests serially:
```typescript
// Solution:
test.describe.configure({ mode: 'serial' });

test.describe('Order flow', () => {
    test('add to cart', async ({ page }) => { });
    test('checkout', async ({ page }) => { });
    test('payment', async ({ page }) => { });
});
```

**Challenge 3:** Shard tests in CI:
```bash
# Solution:
# Run on 4 machines
npx playwright test --shard=1/4
npx playwright test --shard=2/4
npx playwright test --shard=3/4
npx playwright test --shard=4/4
```

---

## 21. REPORTERS

### Concept Questions

**Q1:** What built-in reporters are available?
> `list`, `dot`, `line`, `html`, `json`, `junit`.

**Q2:** Can you use multiple reporters?
> Yes, configure as array: `reporter: [['html'], ['json', { outputFile: 'results.json' }]]`.

**Q3:** How do you create custom reporters?
> Implement Reporter interface with methods like `onTestBegin`, `onTestEnd`.

### Challenges

**Challenge 1:** Configure multiple reporters:
```typescript
// Solution:
// playwright.config.ts
reporter: [
    ['list'],
    ['html', { open: 'never' }],
    ['junit', { outputFile: 'results.xml' }]
]
```

---

## 22. RETRIES & TIMEOUTS

### Concept Questions

**Q1:** How do retries work?
> Failed tests are re-run up to `retries` count.

**Q2:** What timeouts can be configured?
> Test timeout, expect timeout, action timeout, navigation timeout.

**Q3:** How do you identify retried tests?
> Use `testInfo.retry` - 0 for first attempt, 1+ for retries.

### Challenges

**Challenge 1:** Configure retries with different trace behavior:
```typescript
// Solution:
// playwright.config.ts
export default {
    retries: 2,
    use: {
        trace: 'on-first-retry', // Capture trace on first retry
    }
};
```

**Challenge 2:** Set timeout per test:
```typescript
// Solution:
test('slow test', async ({ page }) => {
    test.setTimeout(120000); // 2 minutes
    // ... long running test
});
```

---

## 23. TAGS & FILTERING

### Concept Questions

**Q1:** How do you tag tests?
> Add `@tag` in test title: `test('login @smoke', ...)`.

**Q2:** How do you run specific tags?
> Use `--grep`: `npx playwright test --grep @smoke`.

**Q3:** How do you exclude tags?
> Use `--grep-invert`: `npx playwright test --grep-invert @slow`.

### Challenges

**Challenge 1:** Tag and filter tests:
```typescript
// Solution:
test('user login @smoke @auth', async ({ page }) => { });
test('admin dashboard @admin', async ({ page }) => { });
test('report generation @slow', async ({ page }) => { });

// Run: npx playwright test --grep "@smoke|@auth"
// Exclude: npx playwright test --grep-invert @slow
```

---

## 24. GLOBAL SETUP/TEARDOWN

### Concept Questions

**Q1:** What is global setup?
> Code that runs once before all tests start.

**Q2:** Common use cases?
> Database seeding, authentication state generation, server startup.

### Challenges

**Challenge 1:** Authenticate once for all tests:
```typescript
// Solution:
// global-setup.ts
import { chromium } from '@playwright/test';

export default async function globalSetup() {
    const browser = await chromium.launch();
    const page = await browser.newPage();
    
    await page.goto('/login');
    await page.fill('#username', 'admin');
    await page.fill('#password', 'password');
    await page.click('#submit');
    
    await page.context().storageState({ path: 'auth.json' });
    await browser.close();
}

// playwright.config.ts
export default {
    globalSetup: require.resolve('./global-setup'),
    use: {
        storageState: 'auth.json'
    }
};
```

---

## 25. ANNOTATIONS

### Concept Questions

**Q1:** What annotations are available?
> `skip`, `fixme`, `slow`, `only`, `fail`.

**Q2:** When would you use `test.fixme()`?
> To mark a test as known to fail - skipped but tracked.

**Q3:** What does `test.slow()` do?
> Triples the timeout for the test.

### Challenges

**Challenge 1:** Conditional skip:
```typescript
// Solution:
test('desktop only feature', async ({ page, isMobile }) => {
    test.skip(isMobile, 'Feature not available on mobile');
    // ... test code
});
```

**Challenge 2:** Skip on specific browser:
```typescript
// Solution:
test('webkit specific', async ({ page, browserName }) => {
    test.skip(browserName !== 'webkit', 'WebKit only test');
    // ... test code
});
```

---

## 26. CLOCK / TIME MANIPULATION

### Concept Questions

**Q1:** How do you control time in Playwright?
> Use `page.clock` API to mock Date and timers.

**Q2:** What can you do with clock API?
> `setFixedTime()`, `install()`, `fastForward()`, `runFor()`.

### Challenges

**Challenge 1:** Test time-based feature:
```typescript
// Solution:
await page.clock.setFixedTime(new Date('2024-01-01T10:00:00'));
await page.goto('/dashboard');
await expect(page.getByText('Good morning')).toBeVisible();

await page.clock.setFixedTime(new Date('2024-01-01T20:00:00'));
await page.reload();
await expect(page.getByText('Good evening')).toBeVisible();
```

**Challenge 2:** Test session timeout:
```typescript
// Solution:
await page.clock.install();
await page.goto('/app');
await page.clock.fastForward('30:00'); // 30 minutes
await expect(page.getByText('Session expired')).toBeVisible();
```

---

## Quick Reference

| Feature | Key API |
|---------|---------|
| Locators | `getByRole`, `getByText`, `getByLabel`, `locator`, `filter` |
| Actions | `click`, `fill`, `type`, `press`, `check`, `selectOption` |
| Assertions | `expect`, `toBeVisible`, `toHaveText`, `toHaveCount` |
| Waits | `waitForSelector`, `waitForLoadState`, `waitForResponse` |
| Network | `route`, `fulfill`, `continue`, `abort` |
| Context | `newContext`, `storageState`, `addCookies` |
| Tracing | `tracing.start`, `tracing.stop` |
| API | `request.get`, `request.post`, `response.json` |

---

*Use these questions progressively - concept first, then challenge. Evaluate both understanding and practical ability.*


# TypeScript Problem Solving Challenges

Concept → Challenge format for testing TypeScript skills.

---

## 1. TYPES & TYPE INFERENCE

### Concept Questions

**Q1:** What's the difference between `type` and `interface`?
> `interface` is extendable and mergeable. `type` can represent unions, primitives, tuples.

**Q2:** When does TypeScript infer types?
> When variables are initialized, function return values, and array literals.

**Q3:** What does `readonly` do?
> Makes property immutable after initialization.

### Challenges

**Challenge 1:** Fix the type errors:
```typescript
// Problem:
const user = {
    name: "John",
    age: 30
};
user.email = "john@test.com"; // Error!

// Solution:
interface User {
    name: string;
    age: number;
    email?: string;
}
const user: User = { name: "John", age: 30 };
user.email = "john@test.com"; // OK
```

**Challenge 2:** Create a type for API response:
```typescript
// Solution:
interface ApiResponse<T> {
    data: T;
    status: number;
    message: string;
    timestamp: Date;
}

interface User {
    id: number;
    name: string;
}

const response: ApiResponse<User> = {
    data: { id: 1, name: "John" },
    status: 200,
    message: "Success",
    timestamp: new Date()
};
```

---

## 2. UNION & INTERSECTION TYPES

### Concept Questions

**Q1:** What is a union type?
> A type that can be one of several types: `string | number`.

**Q2:** What is an intersection type?
> A type that combines multiple types: `TypeA & TypeB`.

**Q3:** How do you narrow a union type?
> Using type guards: `typeof`, `instanceof`, `in`, or custom type predicates.

### Challenges

**Challenge 1:** Handle different response types:
```typescript
// Solution:
type SuccessResponse = { status: 'success'; data: any };
type ErrorResponse = { status: 'error'; message: string };
type Response = SuccessResponse | ErrorResponse;

function handleResponse(response: Response) {
    if (response.status === 'success') {
        console.log(response.data); // TypeScript knows it's SuccessResponse
    } else {
        console.log(response.message); // TypeScript knows it's ErrorResponse
    }
}
```

**Challenge 2:** Combine types with intersection:
```typescript
// Solution:
type Timestamped = { createdAt: Date; updatedAt: Date };
type Identifiable = { id: string };

type Entity<T> = T & Timestamped & Identifiable;

interface UserData {
    name: string;
    email: string;
}

type User = Entity<UserData>;
// User has: name, email, id, createdAt, updatedAt
```

---

## 3. GENERICS

### Concept Questions

**Q1:** What are generics?
> Type variables that allow creating reusable components that work with any type.

**Q2:** What is a generic constraint?
> Limiting generic types using `extends`: `<T extends SomeType>`.

**Q3:** What does `keyof` do?
> Creates a union type of all keys of an object type.

### Challenges

**Challenge 1:** Create a generic function to get property:
```typescript
// Solution:
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const user = { name: "John", age: 30 };
const name = getProperty(user, "name"); // type: string
const age = getProperty(user, "age");   // type: number
// getProperty(user, "email"); // Error: "email" not in keyof user
```

**Challenge 2:** Create a generic Stack class:
```typescript
// Solution:
class Stack<T> {
    private items: T[] = [];
    
    push(item: T): void {
        this.items.push(item);
    }
    
    pop(): T | undefined {
        return this.items.pop();
    }
    
    peek(): T | undefined {
        return this.items[this.items.length - 1];
    }
    
    isEmpty(): boolean {
        return this.items.length === 0;
    }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2
```

**Challenge 3:** Generic function with multiple type parameters:
```typescript
// Solution:
function merge<T, U>(obj1: T, obj2: U): T & U {
    return { ...obj1, ...obj2 };
}

const result = merge({ name: "John" }, { age: 30 });
// result type: { name: string } & { age: number }
```

---

## 4. UTILITY TYPES

### Concept Questions

**Q1:** What does `Partial<T>` do?
> Makes all properties optional.

**Q2:** What does `Pick<T, K>` do?
> Creates type with only selected keys.

**Q3:** What does `Omit<T, K>` do?
> Creates type excluding specified keys.

**Q4:** What does `Record<K, V>` do?
> Creates object type with keys K and values V.

### Challenges

**Challenge 1:** Use Partial for update function:
```typescript
// Solution:
interface User {
    id: number;
    name: string;
    email: string;
    age: number;
}

function updateUser(id: number, updates: Partial<User>): User {
    const user = getUserById(id);
    return { ...user, ...updates };
}

updateUser(1, { name: "Jane" }); // Only update name
updateUser(1, { age: 31, email: "jane@test.com" }); // Update multiple
```

**Challenge 2:** Create type excluding sensitive fields:
```typescript
// Solution:
interface UserRecord {
    id: number;
    name: string;
    email: string;
    password: string;
    ssn: string;
}

type PublicUser = Omit<UserRecord, 'password' | 'ssn'>;
// PublicUser: { id: number; name: string; email: string }
```

**Challenge 3:** Create dictionary type:
```typescript
// Solution:
type UserRole = 'admin' | 'user' | 'guest';

type RolePermissions = Record<UserRole, string[]>;

const permissions: RolePermissions = {
    admin: ['read', 'write', 'delete'],
    user: ['read', 'write'],
    guest: ['read']
};
```

---

## 5. TYPE GUARDS

### Concept Questions

**Q1:** What is a type guard?
> A runtime check that narrows the type within a conditional block.

**Q2:** What is a type predicate?
> Return type `value is Type` that tells TypeScript the narrowed type.

**Q3:** When would you use `instanceof` vs `typeof`?
> `typeof` for primitives. `instanceof` for class instances.

### Challenges

**Challenge 1:** Custom type guard:
```typescript
// Solution:
interface Cat {
    meow(): void;
}

interface Dog {
    bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
    return (animal as Cat).meow !== undefined;
}

function makeSound(animal: Cat | Dog) {
    if (isCat(animal)) {
        animal.meow(); // TypeScript knows it's Cat
    } else {
        animal.bark(); // TypeScript knows it's Dog
    }
}
```

**Challenge 2:** Discriminated union:
```typescript
// Solution:
interface Square {
    kind: 'square';
    size: number;
}

interface Circle {
    kind: 'circle';
    radius: number;
}

interface Rectangle {
    kind: 'rectangle';
    width: number;
    height: number;
}

type Shape = Square | Circle | Rectangle;

function getArea(shape: Shape): number {
    switch (shape.kind) {
        case 'square':
            return shape.size ** 2;
        case 'circle':
            return Math.PI * shape.radius ** 2;
        case 'rectangle':
            return shape.width * shape.height;
    }
}
```

---

## 6. MAPPED TYPES

### Concept Questions

**Q1:** What is a mapped type?
> A type that transforms properties of another type.

**Q2:** What does `[K in keyof T]` mean?
> Iterates over all keys of type T.

**Q3:** What does `-?` do in mapped types?
> Removes optional modifier (makes required).

### Challenges

**Challenge 1:** Create Readonly type from scratch:
```typescript
// Solution:
type MyReadonly<T> = {
    readonly [K in keyof T]: T[K];
};

interface User {
    name: string;
    age: number;
}

type ReadonlyUser = MyReadonly<User>;
// All properties are readonly
```

**Challenge 2:** Create type that makes all properties nullable:
```typescript
// Solution:
type Nullable<T> = {
    [K in keyof T]: T[K] | null;
};

interface Config {
    host: string;
    port: number;
}

type NullableConfig = Nullable<Config>;
// { host: string | null; port: number | null }
```

**Challenge 3:** Create getters type:
```typescript
// Solution:
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }
```

---

## 7. CONDITIONAL TYPES

### Concept Questions

**Q1:** What is a conditional type?
> A type that selects one of two types based on a condition: `T extends U ? X : Y`.

**Q2:** What does `infer` do?
> Extracts a type from within a conditional type.

**Q3:** What are distributive conditional types?
> When conditional type is applied to each member of a union separately.

### Challenges

**Challenge 1:** Extract return type:
```typescript
// Solution:
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet(): string { return "Hello"; }
type GreetReturn = MyReturnType<typeof greet>; // string
```

**Challenge 2:** Extract array element type:
```typescript
// Solution:
type ElementType<T> = T extends (infer U)[] ? U : T;

type NumArray = ElementType<number[]>; // number
type StrArray = ElementType<string[]>; // string
type NotArray = ElementType<boolean>;  // boolean
```

**Challenge 3:** Create NonNullable from scratch:
```typescript
// Solution:
type MyNonNullable<T> = T extends null | undefined ? never : T;

type MaybeString = string | null | undefined;
type DefiniteString = MyNonNullable<MaybeString>; // string
```

---

## 8. ASYNC TYPES

### Concept Questions

**Q1:** How do you type a Promise?
> `Promise<T>` where T is the resolved value type.

**Q2:** What does `Awaited<T>` do?
> Extracts the type that a Promise resolves to.

**Q3:** How do you type async function return?
> Return type is `Promise<T>` where T is the actual return value.

### Challenges

**Challenge 1:** Type async function:
```typescript
// Solution:
interface User {
    id: number;
    name: string;
}

async function fetchUser(id: number): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

async function fetchUsers(): Promise<User[]> {
    const response = await fetch('/api/users');
    return response.json();
}
```

**Challenge 2:** Promise.all typing:
```typescript
// Solution:
async function fetchData(): Promise<[User, Post[]]> {
    const [user, posts] = await Promise.all([
        fetchUser(1),
        fetchPosts()
    ]);
    return [user, posts];
}
```

---

## 9. FUNCTION OVERLOADS

### Concept Questions

**Q1:** What are function overloads?
> Multiple function signatures for the same function with different parameter/return types.

**Q2:** How do overloads work in TypeScript?
> Declare multiple signatures, then one implementation that handles all cases.

### Challenges

**Challenge 1:** Overload for different input types:
```typescript
// Solution:
function format(value: string): string;
function format(value: number): string;
function format(value: Date): string;
function format(value: string | number | Date): string {
    if (typeof value === 'string') return value.toUpperCase();
    if (typeof value === 'number') return value.toFixed(2);
    return value.toISOString();
}

format("hello");     // returns string
format(42.123);      // returns string
format(new Date());  // returns string
```

**Challenge 2:** Overload with different return types:
```typescript
// Solution:
function parse(input: string, asNumber: true): number;
function parse(input: string, asNumber: false): string;
function parse(input: string, asNumber: boolean): string | number {
    return asNumber ? parseFloat(input) : input;
}

const num = parse("42", true);   // type: number
const str = parse("42", false);  // type: string
```

---

## 10. PROBLEM SOLVING

### Challenge 1: Implement groupBy
```typescript
// Problem: Group array items by a key
function groupBy<T, K extends keyof T>(arr: T[], key: K): Record<string, T[]> {
    // Your solution
}

// Test:
const people = [
    { name: 'John', dept: 'IT' },
    { name: 'Jane', dept: 'HR' },
    { name: 'Bob', dept: 'IT' }
];
console.log(groupBy(people, 'dept'));
// { IT: [{name:'John',...}, {name:'Bob',...}], HR: [{name:'Jane',...}] }
```
<details><summary>Solution</summary>

```typescript
function groupBy<T, K extends keyof T>(arr: T[], key: K): Record<string, T[]> {
    return arr.reduce((acc, item) => {
        const groupKey = String(item[key]);
        if (!acc[groupKey]) acc[groupKey] = [];
        acc[groupKey].push(item);
        return acc;
    }, {} as Record<string, T[]>);
}
```
</details>

### Challenge 2: Implement debounce with types
```typescript
// Problem: Create typed debounce function
function debounce<T extends (...args: any[]) => any>(
    fn: T,
    ms: number
): (...args: Parameters<T>) => void {
    // Your solution
}

// Test:
const log = debounce((msg: string) => console.log(msg), 300);
log("a"); log("b"); log("c"); // Only "c" after 300ms
```
<details><summary>Solution</summary>

```typescript
function debounce<T extends (...args: any[]) => any>(
    fn: T,
    ms: number
): (...args: Parameters<T>) => void {
    let timeoutId: ReturnType<typeof setTimeout>;
    return (...args: Parameters<T>) => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn(...args), ms);
    };
}
```
</details>

### Challenge 3: Implement deep clone with types
```typescript
// Problem: Deep clone an object preserving types
function deepClone<T>(obj: T): T {
    // Your solution
}

// Test:
const original = { a: 1, b: { c: 2 } };
const cloned = deepClone(original);
cloned.b.c = 99;
console.log(original.b.c); // 2
```
<details><summary>Solution</summary>

```typescript
function deepClone<T>(obj: T): T {
    if (obj === null || typeof obj !== 'object') return obj;
    if (Array.isArray(obj)) return obj.map(deepClone) as T;
    
    const cloned = {} as T;
    for (const key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
            cloned[key] = deepClone(obj[key]);
        }
    }
    return cloned;
}
```
</details>

### Challenge 4: Implement memoize with types
```typescript
// Problem: Memoize function with proper typing
function memoize<T extends (...args: any[]) => any>(fn: T): T {
    // Your solution
}

// Test:
let calls = 0;
const expensive = (n: number) => { calls++; return n * 2; };
const memoized = memoize(expensive);
memoized(5); // calls: 1
memoized(5); // calls: 1 (cached)
```
<details><summary>Solution</summary>

```typescript
function memoize<T extends (...args: any[]) => any>(fn: T): T {
    const cache = new Map<string, ReturnType<T>>();
    return ((...args: Parameters<T>): ReturnType<T> => {
        const key = JSON.stringify(args);
        if (cache.has(key)) return cache.get(key)!;
        const result = fn(...args);
        cache.set(key, result);
        return result;
    }) as T;
}
```
</details>

### Challenge 5: Implement pick function
```typescript
// Problem: Pick specific keys from object
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    // Your solution
}

// Test:
const user = { id: 1, name: 'John', email: 'john@test.com', age: 30 };
const picked = pick(user, ['name', 'email']);
// { name: 'John', email: 'john@test.com' }
```
<details><summary>Solution</summary>

```typescript
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
    const result = {} as Pick<T, K>;
    for (const key of keys) {
        result[key] = obj[key];
    }
    return result;
}
```
</details>

### Challenge 6: Implement omit function
```typescript
// Problem: Omit specific keys from object
function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
    // Your solution
}

// Test:
const user = { id: 1, name: 'John', password: 'secret' };
const safe = omit(user, ['password']);
// { id: 1, name: 'John' }
```
<details><summary>Solution</summary>

```typescript
function omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
    const result = { ...obj };
    for (const key of keys) {
        delete result[key];
    }
    return result as Omit<T, K>;
}
```
</details>

### Challenge 7: Implement flatten object
```typescript
// Problem: Flatten nested object
function flatten(obj: object, prefix = ''): Record<string, any> {
    // Your solution
}

// Test:
const nested = { a: 1, b: { c: 2, d: { e: 3 } } };
console.log(flatten(nested));
// { 'a': 1, 'b.c': 2, 'b.d.e': 3 }
```
<details><summary>Solution</summary>

```typescript
function flatten(obj: object, prefix = ''): Record<string, any> {
    const result: Record<string, any> = {};
    for (const [key, value] of Object.entries(obj)) {
        const newKey = prefix ? `${prefix}.${key}` : key;
        if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
            Object.assign(result, flatten(value, newKey));
        } else {
            result[newKey] = value;
        }
    }
    return result;
}
```
</details>

### Challenge 8: Implement retry function
```typescript
// Problem: Retry async function with backoff
async function retry<T>(
    fn: () => Promise<T>,
    retries: number,
    delay: number
): Promise<T> {
    // Your solution
}

// Test:
let attempts = 0;
const unstable = async () => {
    attempts++;
    if (attempts < 3) throw new Error('Failed');
    return 'Success';
};
const result = await retry(unstable, 5, 100);
```
<details><summary>Solution</summary>

```typescript
async function retry<T>(
    fn: () => Promise<T>,
    retries: number,
    delay: number
): Promise<T> {
    try {
        return await fn();
    } catch (error) {
        if (retries <= 0) throw error;
        await new Promise(r => setTimeout(r, delay));
        return retry(fn, retries - 1, delay * 2);
    }
}
```
</details>

### Challenge 9: Implement event emitter
```typescript
// Problem: Type-safe event emitter
class EventEmitter<Events extends Record<string, any>> {
    // Your implementation
    on<K extends keyof Events>(event: K, handler: (data: Events[K]) => void): void;
    emit<K extends keyof Events>(event: K, data: Events[K]): void;
}

// Test:
interface AppEvents {
    login: { userId: string };
    logout: void;
}

const emitter = new EventEmitter<AppEvents>();
emitter.on('login', ({ userId }) => console.log(userId));
emitter.emit('login', { userId: '123' });
```
<details><summary>Solution</summary>

```typescript
class EventEmitter<Events extends Record<string, any>> {
    private listeners: { [K in keyof Events]?: Array<(data: Events[K]) => void> } = {};

    on<K extends keyof Events>(event: K, handler: (data: Events[K]) => void): void {
        if (!this.listeners[event]) this.listeners[event] = [];
        this.listeners[event]!.push(handler);
    }

    emit<K extends keyof Events>(event: K, data: Events[K]): void {
        this.listeners[event]?.forEach(handler => handler(data));
    }
}
```
</details>

### Challenge 10: Implement pipe function
```typescript
// Problem: Compose functions left to right
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
    // Your solution
}

// Test:
const addOne = (x: number) => x + 1;
const double = (x: number) => x * 2;
const square = (x: number) => x * x;

const compute = pipe(addOne, double, square);
console.log(compute(2)); // ((2+1)*2)^2 = 36
```
<details><summary>Solution</summary>

```typescript
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
    return (arg: T) => fns.reduce((result, fn) => fn(result), arg);
}
```
</details>

---

## Quick Type Reference

| Utility | Description |
|---------|-------------|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties readonly |
| `Pick<T, K>` | Only selected keys |
| `Omit<T, K>` | Exclude selected keys |
| `Record<K, V>` | Object with K keys, V values |
| `Exclude<T, U>` | Remove U from T union |
| `Extract<T, U>` | Keep only U from T union |
| `NonNullable<T>` | Remove null/undefined |
| `ReturnType<T>` | Function return type |
| `Parameters<T>` | Function parameters tuple |
| `Awaited<T>` | Unwrap Promise type |

---

## Scoring Guide

| Level | Expected |
|-------|----------|
| Junior | Concept questions + Basic challenges |
| Mid | All concepts + Half problem solving |
| Senior | All sections + Clean implementation |

---

*Focus on type safety and proper TypeScript usage, not just making it work.*
