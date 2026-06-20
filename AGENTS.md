# Agents

This repository uses Claude Code sub-agents to automate specialized QA tasks. Agents live in `.claude/agents/` and are invoked automatically by Claude Code based on context, or explicitly by asking Claude to perform a task.

> **Note for Claude Code:** Claude Code reads `CLAUDE.md`, not this file, for session instructions. This file is documentation for human contributors and is imported by `CLAUDE.md` via the `@AGENTS.md` reference.

---

## What Are Sub-Agents?

Sub-agents are specialized AI workers defined in `.claude/agents/*.md`. Each agent has:

- **YAML frontmatter** — `name`, `description`, `model`, and `tools` fields
- **Markdown body** — detailed step-by-step instructions the agent follows
- **Auto-routing** — Claude reads the `description` to decide when to invoke the agent automatically
- **Explicit invocation** — you can always ask Claude to "use the site-analyzer agent" or invoke via a skill

---

## Available Agents

### site-analyzer

**File:** `.claude/agents/site-analyzer.md`
**Model:** `claude-sonnet-4-6`
**Tools:** `Read`, `Write`, `WebFetch`, `Bash`

Crawls a live website and produces a fully-populated `site.config.json`. Use this agent when:
- Onboarding the EmberOT site for the first time
- Verifying an existing `site.config.json` is still accurate after a site redesign
- Running `/analyze-site` from the Claude Code CLI

**What it does:**
1. Resolves the canonical URL (follows all redirects)
2. Navigates the homepage with `waitUntil: 'networkidle'` (handles SPAs and lazy hydration)
3. Dismisses cookie consent banners before inspection
4. Extracts all navigation links, form presence, meta description, and industry keywords
5. Checks for a contact form at `/contact`, `/contact-us`, `/get-in-touch`
6. Outputs a complete `site.config.json` with a confidence rating and issues checklist

**Invoke by:**
```
/analyze-site
```
or by asking:
```
Analyze the EmberOT website and update site.config.json
```

---

### test-generator

**File:** `.claude/agents/test-generator.md`
**Model:** `claude-sonnet-4-6`
**Tools:** `Read`, `Write`, `WebFetch`, `Bash`

Reads `site.config.json` and generates site-specific Playwright test files that extend the shared framework's generic tests.

**Use when:**
- EmberOT has unique functionality not covered by the generic suites
- A specific page (pricing, features, security solutions) needs dedicated tests
- Writing regression tests for a recently discovered bug
- Running `/generate-full-suite` from the Claude Code CLI

**What it does:**
1. Reads and parses `site.config.json` for site structure
2. Identifies coverage gaps by comparing the site's pages against existing test files
3. Plans page object additions before writing code
4. Generates spec files in `tests/custom/<scenario-name>.spec.ts`
5. Adds locators to existing page objects in `src/pages/` as needed
6. Tags all tests appropriately (`@custom` plus a category tag)
7. Validates TypeScript compiles with `npx tsc --noEmit`

**Invoke by:**
```
/generate-full-suite
```
or by asking:
```
Generate tests for the EmberOT features page
Write regression tests for the contact form validation
```

---

## Agent File Format

Agents in `.claude/agents/` use YAML frontmatter followed by a markdown body:

```yaml
---
name: agent-name
description: >
  One or two sentences describing when Claude should auto-invoke this agent.
  Be specific — this text drives automatic routing.
model: claude-sonnet-4-6
tools:
  - Read
  - Write
  - WebFetch
  - Bash
---

# Agent: agent-name

## Role
...

## Step-by-step instructions
...
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier — matches the filename |
| `description` | Yes | Drives auto-invocation; be specific about the trigger |
| `model` | No | Override model for this agent (defaults to session model) |
| `tools` | No | Restrict tools available to this agent |

---

## Adding a New Agent

1. Create `.claude/agents/<agent-name>.md`
2. Add YAML frontmatter with `name` and `description` at minimum
3. Write detailed, step-by-step instructions in the body
4. Document the agent in this file (`AGENTS.md`)
5. If the agent has a corresponding user-invocable skill, document it in `Skills.md` too

**Naming convention:** `<verb>-<noun>.md` — e.g., `page-auditor.md`, `selector-updater.md`

---

## Relationship to Skills

| Concept | Location | Invoked by |
|---------|----------|------------|
| **Agent** | `.claude/agents/*.md` | Claude automatically, or via skills |
| **Skill** | `.claude/commands/*.md` | User typing `/skill-name`, or Claude auto-routing |

Skills and agents work together. For example, `/generate-full-suite` is a skill that orchestrates both the `site-analyzer` and `test-generator` agents in sequence.

---

## Agent Execution Context

Agents in this repo have access to:
- **`site.config.json`** — always read this first
- **Live website** — via `WebFetch` for HTML inspection
- **Local file system** — via `Read`/`Write` for page objects and test files
- **Shell** — via `Bash` for TypeScript checks (`npx tsc --noEmit`)

Agents must follow the same constraints as the framework:
- Never submit forms
- Never create accounts or authenticate
- Never hardcode the site URL — read it from `site.config.json`
- Write selectors based on real HTML, not assumptions
