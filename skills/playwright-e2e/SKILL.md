# Playwright E2E

> End-to-end testing for modern web applications.

## Trigger

- User needs end-to-end tests
- User needs cross-browser testing
- User needs to test user flows
- User needs visual regression tests

## Prerequisites

```bash
npm init playwright@latest
```

## Instructions

### Step 1: Basic Test Structure

```typescript
// tests/home.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Home Page', () => {
  test('should display welcome message', async ({ page }) => {
    await page.goto('/');

    await expect(page.getByRole('heading', { name: /welcome/i })).toBeVisible();
  });

  test('should navigate to about page', async ({ page }) => {
    await page.goto('/');

    await page.getByRole('link', { name: /about/i }).click();

    await expect(page).toHaveURL('/about');
    await expect(page.getByRole('heading', { name: /about us/i })).toBeVisible();
  });
});
```

### Step 2: Locators (Priority Order)

```typescript
test.describe('Locators', () => {
  test('uses role locator (preferred)', async ({ page }) => {
    // Buttons
    await page.getByRole('button', { name: 'Submit' }).click();

    // Links
    await page.getByRole('link', { name: 'Home' }).click();

    // Form elements
    await page.getByRole('textbox', { name: 'Email' }).fill('test@example.com');
    await page.getByRole('checkbox', { name: 'Accept terms' }).check();
    await page.getByRole('combobox', { name: 'Country' }).selectOption('US');

    // Headings
    await expect(page.getByRole('heading', { level: 1 })).toHaveText('Welcome');
  });

  test('uses label locator', async ({ page }) => {
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('secret123');
  });

  test('uses placeholder locator', async ({ page }) => {
    await page.getByPlaceholder('Enter your email').fill('test@example.com');
  });

  test('uses text locator', async ({ page }) => {
    await page.getByText('Welcome back').click();
    await page.getByText(/sign up/i).click(); // Regex for case-insensitive
  });

  test('uses alt text locator', async ({ page }) => {
    await page.getByAltText('User avatar').click();
  });

  test('uses test id (last resort)', async ({ page }) => {
    await page.getByTestId('submit-button').click();
  });

  test('chains locators', async ({ page }) => {
    const dialog = page.getByRole('dialog');
    await dialog.getByRole('button', { name: 'Confirm' }).click();
  });

  test('filters locators', async ({ page }) => {
    await page
      .getByRole('listitem')
      .filter({ hasText: 'Product 1' })
      .getByRole('button', { name: 'Add to cart' })
      .click();
  });
});
```

### Step 3: Actions

```typescript
test.describe('Actions', () => {
  test('clicks elements', async ({ page }) => {
    await page.getByRole('button').click();
    await page.getByRole('button').dblclick();
    await page.getByRole('button').click({ button: 'right' }); // Right click
    await page.getByRole('button').click({ modifiers: ['Shift'] }); // Shift+click
  });

  test('fills inputs', async ({ page }) => {
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Email').clear();
    await page.getByLabel('Email').pressSequentially('slow typing', { delay: 100 });
  });

  test('selects options', async ({ page }) => {
    await page.getByRole('combobox').selectOption('value');
    await page.getByRole('combobox').selectOption({ label: 'Label' });
    await page.getByRole('combobox').selectOption(['option1', 'option2']); // Multi-select
  });

  test('checks/unchecks', async ({ page }) => {
    await page.getByRole('checkbox').check();
    await page.getByRole('checkbox').uncheck();
    await page.getByRole('radio', { name: 'Option A' }).check();
  });

  test('uploads files', async ({ page }) => {
    await page.getByLabel('Upload').setInputFiles('file.pdf');
    await page.getByLabel('Upload').setInputFiles(['file1.pdf', 'file2.pdf']);
    await page.getByLabel('Upload').setInputFiles([]); // Clear
  });

  test('drags and drops', async ({ page }) => {
    await page.getByTestId('source').dragTo(page.getByTestId('target'));
  });

  test('hovers', async ({ page }) => {
    await page.getByRole('button').hover();
  });

  test('focuses', async ({ page }) => {
    await page.getByLabel('Email').focus();
  });

  test('presses keys', async ({ page }) => {
    await page.keyboard.press('Enter');
    await page.keyboard.press('Control+A');
    await page.getByLabel('Search').press('Enter');
  });
});
```

### Step 4: Assertions

```typescript
test.describe('Assertions', () => {
  test('asserts visibility', async ({ page }) => {
    await expect(page.getByRole('heading')).toBeVisible();
    await expect(page.getByRole('dialog')).toBeHidden();
  });

  test('asserts text', async ({ page }) => {
    await expect(page.getByRole('heading')).toHaveText('Welcome');
    await expect(page.getByRole('heading')).toContainText('Welc');
  });

  test('asserts attributes', async ({ page }) => {
    await expect(page.getByRole('link')).toHaveAttribute('href', '/home');
    await expect(page.getByRole('textbox')).toHaveValue('test@example.com');
    await expect(page.getByRole('textbox')).toBeEmpty();
  });

  test('asserts state', async ({ page }) => {
    await expect(page.getByRole('button')).toBeEnabled();
    await expect(page.getByRole('button')).toBeDisabled();
    await expect(page.getByRole('checkbox')).toBeChecked();
    await expect(page.getByRole('textbox')).toBeFocused();
  });

  test('asserts count', async ({ page }) => {
    await expect(page.getByRole('listitem')).toHaveCount(5);
  });

  test('asserts URL', async ({ page }) => {
    await expect(page).toHaveURL('/dashboard');
    await expect(page).toHaveURL(/.*dashboard/);
  });

  test('asserts title', async ({ page }) => {
    await expect(page).toHaveTitle('Dashboard | My App');
  });

  test('asserts CSS', async ({ page }) => {
    await expect(page.getByRole('button')).toHaveCSS('background-color', 'rgb(0, 0, 255)');
  });

  test('soft assertions (continue on failure)', async ({ page }) => {
    await expect.soft(page.getByRole('heading')).toHaveText('Welcome');
    await expect.soft(page.getByRole('button')).toBeVisible();
    // Test continues even if assertions fail
  });
});
```

### Step 5: Page Object Model

```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
    this.errorMessage = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toHaveText(message);
  }
}

// pages/DashboardPage.ts
export class DashboardPage {
  readonly page: Page;
  readonly heading: Locator;
  readonly logoutButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.heading = page.getByRole('heading', { level: 1 });
    this.logoutButton = page.getByRole('button', { name: 'Logout' });
  }

  async expectToBeVisible() {
    await expect(this.heading).toContainText('Dashboard');
  }

  async logout() {
    await this.logoutButton.click();
  }
}
```

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('Login Flow', () => {
  test('successful login', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.login('user@example.com', 'password123');

    await dashboardPage.expectToBeVisible();
  });

  test('shows error for invalid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.goto();
    await loginPage.login('wrong@example.com', 'wrongpassword');

    await loginPage.expectError('Invalid email or password');
  });
});
```

### Step 6: Fixtures

```typescript
// fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';
import { DashboardPage } from './pages/DashboardPage';

type Fixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
  authenticatedPage: void;
};

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  dashboardPage: async ({ page }, use) => {
    const dashboardPage = new DashboardPage(page);
    await use(dashboardPage);
  },

  authenticatedPage: async ({ page }, use) => {
    // Login before test
    await page.goto('/login');
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    await page.waitForURL('/dashboard');

    await use();

    // Logout after test (cleanup)
    await page.getByRole('button', { name: 'Logout' }).click();
  },
});

export { expect } from '@playwright/test';
```

```typescript
// tests/dashboard.spec.ts
import { test, expect } from '../fixtures';

test.describe('Dashboard', () => {
  test('shows user profile', async ({ page, authenticatedPage }) => {
    // Already logged in via fixture
    await expect(page.getByText('Welcome, User')).toBeVisible();
  });
});
```

### Step 7: API Testing

```typescript
import { test, expect } from '@playwright/test';

test.describe('API Tests', () => {
  test('creates user via API', async ({ request }) => {
    const response = await request.post('/api/users', {
      data: {
        name: 'John Doe',
        email: 'john@example.com',
      },
    });

    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(201);

    const user = await response.json();
    expect(user).toHaveProperty('id');
    expect(user.name).toBe('John Doe');
  });

  test('gets user list', async ({ request }) => {
    const response = await request.get('/api/users');

    expect(response.ok()).toBeTruthy();

    const users = await response.json();
    expect(users.length).toBeGreaterThan(0);
  });

  test('combines API and UI', async ({ page, request }) => {
    // Create user via API
    await request.post('/api/users', {
      data: { name: 'Jane', email: 'jane@example.com' },
    });

    // Verify in UI
    await page.goto('/users');
    await expect(page.getByText('Jane')).toBeVisible();
  });
});
```

### Step 8: Visual Regression

```typescript
test.describe('Visual Tests', () => {
  test('homepage screenshot', async ({ page }) => {
    await page.goto('/');

    await expect(page).toHaveScreenshot('homepage.png');
  });

  test('component screenshot', async ({ page }) => {
    await page.goto('/components');

    await expect(page.getByTestId('card')).toHaveScreenshot('card.png');
  });

  test('full page screenshot', async ({ page }) => {
    await page.goto('/');

    await expect(page).toHaveScreenshot('full-page.png', { fullPage: true });
  });

  test('screenshot with options', async ({ page }) => {
    await page.goto('/');

    await expect(page).toHaveScreenshot('homepage.png', {
      maxDiffPixels: 100,
      threshold: 0.2,
      mask: [page.getByTestId('dynamic-content')],
    });
  });
});
```

### Step 9: Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }],
  ],

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Common Commands

```bash
# Run all tests
npx playwright test

# Run specific file
npx playwright test tests/login.spec.ts

# Run with UI mode
npx playwright test --ui

# Run headed (see browser)
npx playwright test --headed

# Debug mode
npx playwright test --debug

# Update snapshots
npx playwright test --update-snapshots

# Show report
npx playwright show-report
```

## Success Criteria

- [ ] Tests use role-based locators
- [ ] Page Object Model for reusable code
- [ ] Assertions wait for conditions
- [ ] Tests are independent
- [ ] Configuration supports multiple browsers

## Common Pitfalls

- Don't use CSS selectors when role locators work
- Don't forget to await all Playwright actions
- Don't hardcode timeouts (use auto-waiting)
- Don't share state between tests
- Don't forget to configure baseURL
- Don't skip visual regression for UI-heavy apps
