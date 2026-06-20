# Skills

Skills are callable procedures that extend Claude Code for this project. Invoke them with `/skill-name` in the Claude Code CLI, or Claude may load them automatically when your request matches the skill's description.

Skills are defined in `.claude/commands/` — each `.md` file is one skill. This is equivalent to `.claude/skills/<name>/SKILL.md` in the newer skill directory format; both work the same way.

---

## Project Skills

### /analyze-site

**File:** `.claude/commands/analyze-site.md`

Analyzes a live website and produces a fully-populated `site.config.json`.

**Usage:**
```
/analyze-site [url]
```

If `url` is omitted, reads the `url` field from `site.config.json`.

**What it does:**
1. Navigates to the site (handles SPAs — waits for `networkidle`)
2. Inspects page structure: title, meta description, nav links, forms, H1, CTA buttons
3. Attempts to find the contact page at common paths (`/contact`, `/contact-us`, etc.)
4. Checks responsiveness at 390px viewport
5. Outputs an updated `site.config.json` block and an issues checklist

**When to use:**
- Onboarding the EmberOT site for the first time
- After a major site redesign or restructure
- When nav items or contact form presence has changed

---

### /generate-full-suite

**File:** `.claude/commands/generate-full-suite.md`

Analyzes the live EmberOT site and generates a complete Playwright + TypeScript POM test suite.

**Usage:**
```
/generate-full-suite
```

**What it does:**
1. Reads `site.config.json` for URL and flags
2. Fetches the live site to inspect real HTML (not guesses)
3. Plans page object classes for each discovered page/section
4. Writes page objects to `src/pages/<name>.page.ts`
5. Updates `src/fixtures/site.fixture.ts` with new page objects
6. Writes spec files across all test categories:
   - `tests/smoke/site-availability.spec.ts`
   - `tests/navigation/nav-links.spec.ts`
   - `tests/forms/contact-form.spec.ts` (if `hasContactForm: true`)
   - `tests/functional/<feature>.spec.ts` (one per business feature)
   - `tests/visual/visual-regression.spec.ts`
   - `tests/responsive/layout.spec.ts`
7. Updates `site.config.json` with discovered nav items and description
8. Runs `npx tsc --noEmit` to verify TypeScript

**When to use:**
- First-time test generation for EmberOT
- After a site redesign that invalidates existing selectors
- When adding coverage for a newly discovered page or feature

---

### /run-smoke

**File:** `.claude/commands/run-smoke.md`

Runs the `@smoke` test suite and displays a clean pass/fail summary.

**Usage:**
```
/run-smoke
```

**What it does:**
1. Runs `npm run test:smoke`
2. Parses `test-results/results.json`
3. Displays a formatted table: test name, status (PASS/FAIL/WARN), and duration
4. For failures, shows the error message and a suggested fix
5. Mirrors the underlying exit code (0 = all pass, 1 = any failure)

**Suggested fix table:**

| Failure | Suggestion |
|---------|-----------|
| HTTP 4xx/5xx on load | Check hosting/DNS — site may be down |
| Load time > 10s | Audit with Lighthouse; look for large images |
| Console errors | Open browser DevTools on the live site |
| HTTP (not HTTPS) | Configure SSL/TLS cert; update `site.config.json` URL |
| Missing meta description | Add `<meta name="description">` to site `<head>` |

**When to use:** Before every PR merge, after any site change, as a daily health check.

---

### /update-baseline

**File:** `.claude/commands/update-baseline.md`

Captures new visual regression baseline screenshots after intentional design changes.

**Usage:**
```
/update-baseline
```

**What it does:**
1. Runs `npm run baseline` (`playwright test --grep @visual --update-snapshots`)
2. Lists all updated snapshot files (diff against git)
3. Reminds you to review screenshots before committing

**When to use:**
- After a deliberate site redesign or content update
- After changing viewport sizes in `playwright.config.ts`
- When onboarding the site for the first time (initial baseline capture)

> Baselines live in `__snapshots__/`. Commit them to version control so the team shares the same reference screenshots.

---

### /generate-report

**File:** `.claude/commands/generate-report.md`

Parses the last test run and displays a structured summary.

**Usage:**
```
/generate-report
```

**What it does:**
1. Reads `test-results/results.json`
2. Displays per-suite totals (total / passed / failed / flaky)
3. Lists all failed tests with full error messages
4. Lists flaky tests with retry information
5. Suggests next steps based on results

**When to use:** After a full test run to understand what passed, failed, and needs investigation.

---

## Skill File Format

Skills in `.claude/commands/` are markdown files with optional YAML frontmatter:

```yaml
---
description: >
  One or two sentences describing when Claude should auto-invoke this skill.
  Specific descriptions enable automatic invocation — Claude loads the skill
  when your message matches, even without the /prefix.
---

# /skill-name

Detailed instructions Claude follows when this skill is invoked.
```

### Frontmatter Fields

| Field | Purpose |
|-------|---------|
| `description` | Enables auto-invocation when Claude's context matches |

---

## Adding a New Skill

1. Create `.claude/commands/<skill-name>.md`
2. Add a `description:` frontmatter that describes when to auto-invoke
3. Write step-by-step instructions in the body
4. Document the skill here in `Skills.md`
5. Reference it in `CLAUDE.md` under "Slash Commands" if it's user-invocable

**Naming convention:** Kebab-case, verb-noun: `check-accessibility.md`, `audit-performance.md`

---

## Bundled Claude Code Skills

These are available in every Claude Code session without any project setup:

| Skill | Description |
|-------|-------------|
| `/code-review` | Review code changes for bugs and style issues |
| `/debug` | Debug errors and test failures |
| `/loop` | Repeat a command on a recurring interval |
| `/run` | Launch and drive the app in a browser |
| `/verify` | Confirm a code change works as expected |
| `/batch` | Run parallel subagent tasks |

See the [Claude Code commands reference](https://code.claude.com/docs/en/commands) for the full list.

---

## Skill vs. Agent vs. Rule vs. CLAUDE.md

| Mechanism | Location | Loads | Use for |
|-----------|----------|-------|---------|
| **CLAUDE.md** | `./CLAUDE.md` | Every session | Project-wide rules, architecture facts, do/don't lists |
| **Rule** | `.claude/rules/*.md` | When matching files are opened | Path-specific conventions (e.g., only for `tests/**`) |
| **Skill** | `.claude/commands/*.md` | When invoked | Step-by-step procedures, checklists, repeatable workflows |
| **Agent** | `.claude/agents/*.md` | When auto-routed or invoked | Autonomous tasks that read/write multiple files |
