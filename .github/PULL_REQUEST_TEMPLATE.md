## What does this PR do?

<!-- Describe the test coverage added, updated, or removed. -->

## Related Issue

<!-- Link an issue if applicable: Closes #123 -->

---

## Test Type

- [ ] `@smoke` — site availability / basic load
- [ ] `@navigation` — nav links, routing, mobile menu
- [ ] `@forms` — form fields and validation (no submission)
- [ ] `@functional` — business features and content sections
- [ ] `@visual` — screenshot regression baselines
- [ ] `@responsive` — layout at different viewport sizes
- [ ] Framework / infrastructure change (no test logic changes)

---

## Checklist

### Code Quality
- [ ] `npm run typecheck` passes — TypeScript compiles with no errors
- [ ] `npm run lint` passes — no ESLint errors or warnings
- [ ] New page object(s) added to `src/pages/` if new selectors are needed
- [ ] Fixture updated in `src/fixtures/site.fixture.ts` if a new page object was added

### Test Quality
- [ ] All new tests are tagged with at least one `@tag`
- [ ] No `page.waitForTimeout()` calls — Playwright auto-waiting used instead
- [ ] No hardcoded URLs — `baseURL` or `siteConfig.url` used
- [ ] No form submissions in any test
- [ ] Each test is independent (no shared state between tests)
- [ ] Selectors are based on real HTML from the live site, not guesses

### Visual Tests (if applicable)
- [ ] `npm run baseline` run after intentional layout changes
- [ ] New/updated baseline screenshots reviewed visually before committing
- [ ] `__snapshots__/` changes included in this PR

### CI
- [ ] `npm run test:smoke` passes locally
- [ ] CI checks are green (or failures are explained below)

---

## Test Run Output

```
# Paste a snippet from: npm run test:smoke
```

---

## Notes for Reviewers

<!-- Anything the reviewer should pay attention to: flaky tests, known issues, follow-ups -->
