# EmberOT — QA Agentic Solution

> Playwright + TypeScript automated regression framework for [EmberOT](https://www.emberot.com/) — powered by Claude Code agentic AI.

---

## What This Repo Does

This repository is an AI-powered, agentic QA test suite that continuously validates the EmberOT website (`https://www.emberot.com`). It uses:

- **Playwright** for browser automation across desktop, tablet, and mobile viewports
- **TypeScript** (strict mode) for type-safe test code
- **Page Object Model (POM)** — every page has its own class; no raw locators in test bodies
- **Object-Oriented Programming (OOP)** — page classes extend `BasePage`, sharing navigation, screenshot, and wait utilities
- **Claude Code** sub-agents and skills to generate, analyze, and maintain tests automatically

---

## Company Profile

| Field | Detail |
|-------|--------|
| **Company** | EmberOT |
| **Product** | Cybersecurity software for critical infrastructure protection |
| **Website** | https://www.emberot.com |
| **LinkedIn** | [linkedin.com/company/emberot](https://www.linkedin.com/company/emberot/) |
| **Founded** | 2023 |
| **Employees** | 1–10 |

---

## Development Environment Setup

### Prerequisites

| Tool | Minimum Version | How to check |
|------|-----------------|--------------|
| Node.js | 18.x | `node --version` |
| npm | 9.x | `npm --version` |
| Git | 2.x | `git --version` |
| Claude Code | latest | `claude --version` |

> Claude Code is not required to run tests, but is required to use the AI-powered skills and agents.

### Step-by-Step Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-org/emberot-QA-Agentic-Solution.git
cd emberot-QA-Agentic-Solution

# 2. Install Node dependencies
npm install

# 3. Install Playwright browsers (Chromium is required; add Firefox/WebKit optionally)
npx playwright install --with-deps chromium

# 4. Verify the TypeScript config compiles cleanly
npm run typecheck

# 5. Run the smoke suite as a quick sanity check
npm run test:smoke
```

### Environment Variables

Copy `.env.example` to `.env` and set values if needed:

```bash
SITE_URL=https://www.emberot.com   # Override the URL from site.config.json
CI=true                            # Set automatically in GitHub Actions
```

---

## Architecture

### Page Object Model (POM)

Every page or major section has its own TypeScript class in `src/pages/`. The hierarchy:

```
BasePage  (src/pages/base.page.ts)
├── HomePage          (src/pages/home.page.ts)
├── NavigationPage    (src/pages/navigation.page.ts)
├── ContactFormPage   (src/pages/contact.page.ts)
└── <DiscoveredPage>  (src/pages/<name>.page.ts — one per discovered page)
```

Each page class:
- Extends `BasePage` — inheriting `navigate()`, `waitForLoad()`, `isResponsive()`, etc.
- Declares all key elements as `readonly Locator` properties
- Exposes user-action methods (click, fill, open, close) — never assertions
- Has no `expect()` calls — assertions belong in test files

### OOP Example

```typescript
import { Page, Locator } from '@playwright/test';
import { BasePage } from './base.page';
import type { SiteConfig } from '@site-types/site-config.types';

export class ServicesPage extends BasePage {
  readonly heroHeading: Locator;
  readonly serviceCards: Locator;
  readonly ctaButton: Locator;

  constructor(page: Page, config: SiteConfig) {
    super(page, config);                                // inherits shared utilities
    this.heroHeading = page.locator('h1');
    this.serviceCards = page.locator('[class*="service-card"]');
    this.ctaButton = page.getByRole('link', { name: /get started/i }).first();
  }

  async clickCta(): Promise<void> {
    await this.ctaButton.click();
    await this.waitForLoad();
  }
}
```

### Fixture System

All test files import `test` and `expect` from the custom fixture — never from `@playwright/test` directly:

```typescript
import { test, expect } from '@fixtures/site.fixture';

test('services page loads @smoke', async ({ servicesPage }) => {
  await expect(servicesPage.heroHeading).toBeVisible();
});
```

The fixture pre-constructs every page object and the site config. Tests receive them without boilerplate.

---

## Project Structure

```
site.config.json            ← target URL, flags, nav expectations
playwright.config.ts        ← browser projects, timeouts, reporters
tsconfig.json               ← strict TypeScript config

src/
  pages/
    base.page.ts            ← BasePage base class
    home.page.ts            ← HomePage
    navigation.page.ts      ← NavigationPage
    contact.page.ts         ← ContactFormPage
    <name>.page.ts          ← one class per additional discovered page
  fixtures/
    site.fixture.ts         ← custom test() with all page objects pre-wired
  utils/
    link-checker.ts         ← HTTP link validation helper
    visual-helper.ts        ← screenshot comparison utilities
  types/
    site-config.types.ts    ← SiteConfig interface + loadSiteConfig()

tests/
  smoke/
    site-availability.spec.ts
  navigation/
    nav-links.spec.ts
  forms/
    contact-form.spec.ts
  functional/
    <feature>.spec.ts       ← one file per business feature area
  visual/
    visual-regression.spec.ts
  responsive/
    layout.spec.ts

.claude/
  agents/
    site-analyzer.md        ← sub-agent: crawl site, update config
    test-generator.md       ← sub-agent: generate site-specific tests
  commands/
    analyze-site.md         ← /analyze-site skill
    generate-full-suite.md  ← /generate-full-suite skill
    run-smoke.md            ← /run-smoke skill
    update-baseline.md      ← /update-baseline skill
    generate-report.md      ← /generate-report skill
  hooks/
    pre-test.sh             ← site reachability check before test runs
  settings.json             ← Claude Code permissions and hooks config
  rules/
    testing.md              ← path-scoped rules for test files
    page-objects.md         ← path-scoped rules for page object files

.github/
  workflows/
    playwright.yml          ← CI/CD pipeline
  copilot-instructions.md   ← GitHub Copilot project conventions
  CODEOWNERS               ← who reviews test changes
  PULL_REQUEST_TEMPLATE.md  ← PR checklist
  ISSUE_TEMPLATE/
    bug-report.yml          ← template for reporting test failures
    test-request.yml        ← template for requesting new coverage
```

---

## Test Categories

| Folder | Tag | What It Tests |
|--------|-----|---------------|
| `tests/smoke/` | `@smoke` | Site loads, title exists, no JS errors, HTTPS |
| `tests/navigation/` | `@navigation` | Nav links, routing, mobile menu, footer links |
| `tests/forms/` | `@forms` | Form fields, validation messages — no submission |
| `tests/functional/` | `@functional` | Business features: CTAs, content sections, interactive elements |
| `tests/visual/` | `@visual` | Screenshot regression baselines |
| `tests/responsive/` | `@responsive` | Layout at 390px / 768px / 1280px viewports |

---

## Running Tests

```bash
npm test                     # all tests, all browser projects
npm run test:smoke           # @smoke only — fastest, run first
npm run test:navigation      # @navigation only
npm run test:forms           # @forms only
npm run test:visual          # @visual only
npm run test:responsive      # @responsive only
npm run baseline             # update visual regression snapshots
npm run report               # open Playwright HTML report in browser
npm run lint                 # ESLint check
npm run typecheck            # TypeScript compiler check (no emit)
```

---

## Browser Projects

Playwright runs tests against three viewport configurations:

| Project | Browser | Viewport |
|---------|---------|----------|
| `chromium-desktop` | Chrome | 1280 × 720 |
| `mobile-chrome` | Chrome (Pixel 5) | 390 × 844 |
| `tablet` | Chrome (iPad Mini) | 768 × 1024 |

---

## Claude Code Skills

Invoke these from the Claude Code CLI with `/skill-name`. See `Skills.md` for full documentation.

| Skill | Command | Description |
|-------|---------|-------------|
| Analyze site | `/analyze-site` | Crawl the live site, update `site.config.json` |
| Generate tests | `/generate-full-suite` | Build complete POM + test suite from live site |
| Run smoke | `/run-smoke` | Execute `@smoke` tests and show formatted results |
| Update baselines | `/update-baseline` | Refresh visual regression snapshots |
| Generate report | `/generate-report` | Parse and display full test run results |

---

## Claude Code Agents

Sub-agents run autonomously for specialized tasks. See `AGENTS.md` for full documentation.

| Agent | File | Role |
|-------|------|------|
| `site-analyzer` | `.claude/agents/site-analyzer.md` | Crawl site, populate `site.config.json` |
| `test-generator` | `.claude/agents/test-generator.md` | Generate site-specific Playwright tests |

---

## .github Folder

| Path | Purpose |
|------|---------|
| `.github/workflows/playwright.yml` | CI pipeline — runs full test suite on push/PR and daily |
| `.github/copilot-instructions.md` | GitHub Copilot project conventions (equivalent of CLAUDE.md for Copilot) |
| `.github/CODEOWNERS` | Defines who reviews test changes |
| `.github/PULL_REQUEST_TEMPLATE.md` | Checklist contributors fill in for every PR |
| `.github/ISSUE_TEMPLATE/bug-report.yml` | Structured form for reporting test failures |
| `.github/ISSUE_TEMPLATE/test-request.yml` | Structured form for requesting new test coverage |

---

## Contributor Rules

### Do

- Follow the POM pattern — all locators live in page object classes, never in test bodies
- Tag every `test()` call with at least one: `@smoke`, `@navigation`, `@forms`, `@functional`, `@visual`, `@responsive`
- Use `baseURL` from Playwright config — never hardcode the site URL
- Import `test` and `expect` from `@fixtures/site.fixture`, not from `@playwright/test`
- Use `page.getByRole()` or `page.getByText()` before falling back to CSS selectors
- Run `npm run typecheck` before pushing — TypeScript must compile cleanly
- Write selectors based on real HTML from the live site, not guesses

### Do Not

- Submit any form in a test
- Create accounts or log in (unless `auth.required: true` in `site.config.json`)
- Put `expect()` assertions inside page object methods
- Use `page.waitForTimeout()` — rely on Playwright's auto-waiting and `waitForSelector`
- Use the `any` type without an explicit comment justifying it
- Hardcode `https://www.emberot.com` anywhere — always use `baseURL` or `siteConfig.url`
- Merge without a passing smoke run

### Branch Naming

```
feat/test-<feature-name>     # new test coverage
fix/<test-name>-timeout      # fixing a failing test
chore/update-baselines       # visual snapshot updates
chore/update-pom-<page>      # page object updates
```

### Pull Request Process

1. Branch from `main`
2. Write tests following POM conventions
3. Run `npm run typecheck && npm run test:smoke` locally
4. Open a PR using the provided template
5. CI must pass before merge (smoke tests on PR, full suite on merge to main)

---

## CI/CD Pipeline

GitHub Actions runs on every push and pull request to `main`:

- **On PR:** TypeScript check + smoke tests across all three browser projects
- **On merge to main:** Full test suite across all projects
- **Daily schedule:** Full suite at 06:00 UTC to catch site changes

Artifacts uploaded per run: HTML report (`playwright-report/`) and JSON results (`test-results/`), retained for 30 days.

---

## Configuration Reference

| File | Purpose |
|------|---------|
| `site.config.json` | Target site URL, flags (`hasContactForm`, `skipVisual`, etc.) |
| `playwright.config.ts` | Browser projects, global timeout, reporters, snapshot dir |
| `tsconfig.json` | TypeScript strict mode, path aliases (`@pages`, `@fixtures`, etc.) |
| `.env.example` | Environment variable template |
| `.claude/settings.json` | Claude Code permissions and hook configuration |

---

## Generating Tests with Claude Code

The fastest way to bootstrap coverage for a new page or feature:

```bash
# In the Claude Code CLI
/generate-full-suite   # analyzes live site, writes all POM + tests
/analyze-site          # updates site.config.json only
```

Or ask Claude directly:
- "Generate functional tests for the EmberOT pricing page"
- "Add a page object for the About page and write smoke tests for it"
- "Update the contact form tests after the new field was added"

---

*Part of the Phoenix Startup QA Agentic Solutions project.*
