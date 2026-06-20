# GitHub Copilot Instructions — EmberOT QA Framework

This is a Playwright + TypeScript regression test suite for https://www.emberot.com.
It uses the Page Object Model (POM) with OOP inheritance from `BasePage`.

---

## Architecture

- Page objects → `src/pages/` — extend `BasePage`, typed `readonly Locator` properties, action methods only
- Tests → `tests/<category>/` — import `test`/`expect` from `@fixtures/site.fixture`
- Fixtures → `src/fixtures/site.fixture.ts` — pre-constructs all page objects
- Site config → `site.config.json` — single source of truth for URL and feature flags

## Code Rules

- TypeScript strict mode — no implicit `any`
- All `Locator` properties are `readonly`
- Method return types must be explicit (`Promise<void>`, `Promise<string>`, etc.)
- Use `page.getByRole()` and `page.getByText()` before CSS selectors
- All tests tagged: `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, or `@responsive`
- Import `test`/`expect` from `@fixtures/site.fixture` — never from `@playwright/test` directly

## Never Generate

- Form submissions in tests
- `page.waitForTimeout()` calls — use Playwright auto-waiting
- Hardcoded URLs — use `baseURL` from Playwright config or `siteConfig.url` from the fixture
- `expect()` inside page object methods — assertions belong in test files only
- `any` types
- Direct `page.locator()` calls in test bodies — use page object properties

## Page Object Pattern

```typescript
import { Page, Locator } from '@playwright/test';
import { BasePage } from './base.page';
import type { SiteConfig } from '@site-types/site-config.types';

export class ExamplePage extends BasePage {
  readonly heading: Locator;
  readonly ctaButton: Locator;

  constructor(page: Page, config: SiteConfig) {
    super(page, config);
    this.heading = page.locator('h1');
    this.ctaButton = page.getByRole('link', { name: /get started/i }).first();
  }

  async clickCta(): Promise<void> {
    await this.ctaButton.click();
    await this.waitForLoad();
  }
}
```

## Test Pattern

```typescript
import { test, expect } from '@fixtures/site.fixture';

test.describe('Example Page', () => {
  test.beforeEach(async ({ homePage }) => {
    await homePage.navigate();
  });

  test('heading is visible @smoke', async ({ homePage }) => {
    await expect(homePage.heroHeading).toBeVisible();
  });
});
```
