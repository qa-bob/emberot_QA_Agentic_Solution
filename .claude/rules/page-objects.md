---
paths:
  - "src/pages/**/*.ts"
---

# Page Object Model Rules

These rules apply whenever Claude is working on any file under `src/pages/`.

## Class Structure

- Every page class must extend `BasePage` from `./base.page`
- Constructor signature: `constructor(page: Page, config: SiteConfig)`
- Always call `super(page, config)` as the first statement in the constructor
- Export the class as a named export (not default)

## Locators

- Declare all key elements as `readonly Locator` properties on the class
- Assign locators in the constructor body using `this.page.locator(...)` etc.
- Prefer: `page.getByRole()` > `page.getByText()` > `page.getByLabel()` > CSS class selector
- Avoid `page.locator('#id')` unless the element genuinely has a stable, semantic ID
- For lists of similar elements, use a single `Locator` pointing to the container or all items

## Methods

- Methods represent user actions: `clickCta()`, `openMobileMenu()`, `fillEmailField(email)`
- Return types must be explicit: `Promise<void>`, `Promise<string>`, `Promise<boolean>`
- Never put `expect()` or assertion calls inside page object methods
- Never call `page.waitForTimeout()` — use `this.waitForLoad()` (inherited from `BasePage`) or `waitForSelector`
- For navigation methods, call `await this.waitForLoad()` after the navigation action

## Typing

- All properties must have explicit types — no implicit `any`
- Use `Locator` from `@playwright/test` for all element references
- Use `SiteConfig` from `@site-types/site-config.types` for the config parameter

## Fixture Registration

- After creating a new page object class, add it to `src/fixtures/site.fixture.ts`
- Add a fixture property of the class type to the `Fixtures` interface
- Wire the fixture using `base.extend<Fixtures>({ ... })` following the existing pattern
