# Wakeb Skills

AI agent skills that keep the entire team's code consistent — same structure,
same patterns, same conventions — regardless of who prompted the AI.

## Features

### Consistent Code Across the Team

The agent knows every component, composable, controller, model, and service in
the project. Instead of inventing new ones, it reuses what already exists.

### 25 Specialized Skills

| Category          | Count | Examples                                               |
| ----------------- | ----- | ------------------------------------------------------ |
| Core FE Skills    | 7     | Design-to-page, API wiring, full module generation     |
| Core Architecture | 5     | Best Practices, SOLID, Pattern Enforcer                |
| Frontend          | 6     | Performance, i18n/RTL, State Management, Accessibility |
| Backend           | 5     | API Builder, Security Scanner, Database Lifecycle      |
| Full Stack        | 3     | Contract Sync (FE↔BE), E2E Flow Generator, CI/CD       |
| Advanced          | 4     | Code Review, Debugging, Refactoring, Learning System   |

### What It Can Do

- **Full module** (FE + BE) from a simple description — migration, model, controller, resource, config, pages, router, locales
- **Figma to code** — reads the design and uses project components, not raw HTML
- **Automated code review** — checks security, performance, patterns, accessibility
- **Structured debugging** — follows a systematic flow, never guesses
- **FE ↔ BE sync** — ensures fields and endpoints match between front-end and back-end

### Common Mistakes It Prevents

- No `ml-4` or `mr-4` — uses `ms-4` / `me-4` for RTL support
- No hardcoded colors — uses design tokens
- No untranslated strings — every text uses `$t()` in FE and `__()` in BE
- No `response()->json()` — uses `successResponse()` / `failResponse()`
- No duplicate components — checks codebase-index first

---

## Quick Start (VS Code)

> **3 steps and you're ready to go:**

### 1. Copy the Prompt File

```powershell
copy "wakeb.prompt.md" "%APPDATA%\Code\User\prompts\wakeb.prompt.md"
```

### 2. Update the Paths

Open the copied file and update these paths to match your local setup:

```
d:/Jervis Tech/Jervis Labs/Wakeb Skills/   ← path to the Wakeb Skills folder
D:\Wakeb\Vue\aware-v2-dahsboard            ← path to the FE starter project
D:\laragon\www\Starter-Backend             ← path to the BE starter project
```

### 3. Use `@wakeb`

Open Copilot Chat in VS Code (`Ctrl+Shift+I`) and type:

```
@wakeb Create an employees module with name, email, department, and status
```

---

## Usage Examples

### Generate a Full Module

```
@wakeb Create a products module with name (translatable), price, category, image, and status
```

→ Generates: Migration + Model + Controller + Request + Resource + Routes + Config + Pages + Router + Locales

### Convert a Figma Design

```
@wakeb Convert this Figma design to a Vue component: https://figma.com/design/xxx/yyy
```

→ Reads the design, maps to project components, applies design tokens

### CRUD Pages

```
@wakeb Create list, add, and edit pages for departments — fields: name (translatable), description, status
```

### Backend API

```
@wakeb Create a REST API for orders with: customer_id, items (json), total, status, notes
```

### Code Review

```
@wakeb Review this file for pattern compliance and security issues
```

### Debug a Problem

```
@wakeb The table isn't showing data — it renders empty even though the API returns results
```

### Refactoring

```
@wakeb This component is 400 lines — refactor it into smaller pieces
```

---

## What's Inside

```
Wakeb Skills/
├── wakeb-dashboard/          ← Front-end skill (Vue 3 + Vuetify + Tailwind)
│   ├── SKILL.md              ← Main skill file (execution flow + taxonomy)
│   └── references/
│       ├── codebase-index.md     Full FE inventory: components, composables, factories, utils, stores
│       ├── module-scaffold.md    Module scaffolding templates
│       ├── components.md         Component props & Figma-to-component mapping
│       ├── field-utils.md        FieldUtils creators (18 types) & validation rules (30+)
│       ├── design-tokens.md      CSS variable reference for colors/spacing/themes
│       ├── composables.md        useLookupPage full options & composables table
│       ├── factories.md          BaseCrudFactory & TableFactory API
│       ├── patterns.md           Code style, i18n/RTL, routing, API, themes, debugging
│       └── skills/               ← Core agent skills (7)
│           ├── generate-page-from-design.md   Convert designs → Vue pages
│           ├── resolve-component.md           Find existing components
│           ├── enforce-structure.md            Validate file placement & naming
│           ├── generate-module.md              Full-stack module scaffolding
│           ├── reuse-composable.md             Extract reusable logic
│           ├── connect-api.md                  Wire FE ↔ BE endpoints
│           └── validate-output.md              Final quality gate
│
├── wakeb-backend/            ← Back-end skill (Laravel 12 + Sanctum + Spatie)
│   ├── SKILL.md              ← Main skill file (execution flow + taxonomy)
│   └── references/
│       ├── codebase-index.md     Full BE inventory: controllers, models, filters, traits, services
│       ├── module-scaffold.md    Full templates: Model, Controller, Request, Resource, Migration, etc.
│       ├── validation.md         BaseFormRequest & custom rules
│       ├── filters.md            Pipeline pattern & built-in filters
│       ├── models.md             BaseModel, traits, relationships
│       └── patterns.md           Auth flow, Gate+Policy, controller traits, helpers, notifications
│
├── skills/                   ← Extended skills library (18 skills)
│   ├── core/                     Architecture + quality foundations
│   │   ├── best-practices.md         Security, performance, DRY rules
│   │   ├── codebase-pattern-enforcer.md  Pattern compliance gate
│   │   ├── context-awareness.md      Auto-detect project/module/file context
│   │   ├── environment-intelligence.md   Env-specific code (dev/staging/prod)
│   │   └── solid-principles.md       SOLID mapped to Wakeb patterns
│   ├── frontend/                 Vue-specific skills
│   │   ├── accessibility-ux-validator.md  WCAG 2.1 AA + RTL accessibility
│   │   ├── design-enhancement.md     Visual polish without Figma
│   │   ├── i18n-rtl-awareness.md     Arabic-first i18n + RTL rules
│   │   ├── performance-optimizer.md  Route/component/reactivity optimization
│   │   ├── smart-component-generator.md  Component creation decision tree
│   │   └── state-management.md       Pinia store patterns + caching
│   ├── backend/                  Laravel-specific skills
│   │   ├── api-builder.md            REST endpoint patterns
│   │   ├── background-jobs.md        Queue jobs + notifications + scheduler
│   │   ├── database-lifecycle.md     Migrations, seeders, relationships
│   │   ├── project-bootstrapper.md   Module scaffolding (nwidart)
│   │   └── security-scanner.md       OWASP Top 10 validation
│   ├── fullstack/                Cross-stack skills
│   │   ├── cicd-integration.md       GitHub Actions + deployment
│   │   ├── contract-sync.md          FE ↔ BE field/endpoint alignment
│   │   └── e2e-flow-generator.md     Full feature generation flow
│   └── advanced/                 Meta-skills
│       ├── code-review-agent.md      Automated code review
│       ├── debugging-agent.md        Systematic bug diagnosis
│       ├── learning-system.md        Track patterns + corrections
│       └── refactoring-engine.md     Safe refactoring catalog
│
├── wakeb.prompt.md           ← VS Code prompt file (copy to %APPDATA%\Code\User\prompts\)
└── README.md                 ← This file
```

### How It Works

```
User request → @wakeb analyzes → scans existing code → generates compliant code → validates → returns
```

In detail — 7 steps:

1. **Analyze** → understand the request and determine FE, BE, or both
2. **Scan** → read codebase-index to know what already exists
3. **Resolve** → find ready-made components/composables/traits to reuse
4. **Structure** → validate file placement and naming
5. **Generate** → write code using project patterns
6. **Validate** → run quality checklist + anti-pattern detection
7. **Return** → only return code that passes all checks

The **codebase-index** files are the secret — they give the agent a complete
inventory of every component, composable, controller, model, filter, trait,
and service in the project so it NEVER creates duplicates.

## Other IDEs

> The primary setup (VS Code) is in the "Quick Start" section above.
> These sections are for team members using a different IDE.

### VS Code (GitHub Copilot)

The prompt file at `wakeb.prompt.md` turns Copilot into a `@wakeb` agent.

**Setup:**

1. Copy `wakeb.prompt.md` to the prompts folder:

   ```
   %APPDATA%\Code\User\prompts\wakeb.prompt.md
   ```

2. Update the file paths inside the prompt file to point to your local
   `Wakeb Skills` folder and starter project locations.

**Usage:**

1. Open any Wakeb project in VS Code
2. Open Copilot Chat (`Ctrl+Shift+I`)
3. Type `@wakeb` then your request

```
@wakeb Create a products module with name (translatable), price, image, and status
@wakeb Convert this Figma design to a Vue component: [paste Figma URL]
```

The agent auto-detects whether the task is front-end or back-end.

---

### Cursor

Cursor uses rules files in the project root. Since Cursor doesn't support
`{{{ file:// }}}` includes, you reference the skill files as project rules.

**Setup:**

1. In your Wakeb project root, create `.cursor/rules/wakeb.mdc`:

   ```markdown
   ---
   description: "Wakeb Development Agent — Vue 3 + Laravel conventions"
   globs: "**/*"
   alwaysApply: true
   ---

   You are the Wakeb Development Agent. Produce code that looks like it was
   written by the same developer.

   @file wakeb-dashboard/SKILL.md
   @file wakeb-dashboard/references/codebase-index.md
   @file wakeb-backend/SKILL.md
   @file wakeb-backend/references/codebase-index.md
   ```

2. For full context, either:
   - **Symlink** the `Wakeb Skills` folder into your project root
   - **Copy** the skill files into `.cursor/rules/` and reference them with `@file`

3. Alternatively, paste the contents of `SKILL.md` + `codebase-index.md` directly
   into the `.mdc` file body (Cursor has a ~120K token context limit per rule).

**Usage:**

Open Cursor Chat (`Ctrl+L`) or Composer (`Ctrl+I`) — the rules apply automatically
to every request matching the glob pattern.

---

### JetBrains (AI Assistant / GitHub Copilot)

JetBrains IDEs (WebStorm, PhpStorm, IntelliJ) support custom AI instructions.

**Option A — GitHub Copilot Plugin:**

1. Install the GitHub Copilot plugin from the JetBrains Marketplace
2. Create `.github/copilot-instructions.md` in your project root with the
   contents of `wakeb-dashboard/SKILL.md` and/or `wakeb-backend/SKILL.md`
3. Copilot Chat will automatically use these instructions

   ```
   your-project/
   └── .github/
       └── copilot-instructions.md   ← paste SKILL.md content here
   ```

**Option B — JetBrains AI Assistant:**

1. Go to **Settings → AI Assistant → Project-level prompts**
2. Click **+** to add a new prompt
3. Paste the contents of the relevant `SKILL.md` file
4. For reference files, add them as additional prompts or paste key sections
   (like `codebase-index.md`) into the prompt body

**Usage:**

Open AI Chat (`Alt+Enter` or AI Assistant panel) — the project-level prompts
apply automatically.

---

### Windsurf

Windsurf (by Codeium) uses a rules file in the project root.

**Setup:**

1. Create `.windsurfrules` in your project root:

   ```markdown
   You are the Wakeb Development Agent. Produce code that looks like it was
   written by the same developer.

   [Paste the contents of wakeb-dashboard/SKILL.md here]

   [Paste the contents of wakeb-backend/SKILL.md here]

   [Paste the contents of wakeb-dashboard/references/codebase-index.md here]

   [Paste the contents of wakeb-backend/references/codebase-index.md here]
   ```

2. Windsurf doesn't support file includes — paste the full content directly.
   Focus on `SKILL.md` + `codebase-index.md` as the highest-value files.

**Usage:**

Open Cascade (`Ctrl+L`) — the rules apply to every conversation automatically.

---

### Zed

Zed supports assistant instructions via project settings.

**Setup:**

1. Create `.zed/settings.json` in your project root:

   ```json
   {
     "assistant": {
       "default_model": { "provider": "copilot_chat", "model": "gpt-4o" },
       "instructions_files": [".zed/wakeb-instructions.md"]
     }
   }
   ```

2. Create `.zed/wakeb-instructions.md` with the contents of the `SKILL.md`
   files and `codebase-index.md` files.

**Usage:**

Open the Assistant panel (`Ctrl+Shift+A`) — instructions apply automatically.

---

### Any Other IDE / ChatGPT / Claude

For any AI tool that accepts a system prompt or context files:

1. **Copy the content** of the relevant `SKILL.md` file into the system prompt
   or custom instructions field
2. **Add `codebase-index.md`** as additional context — this is the most
   important reference file (prevents duplicate code generation)
3. **Add reference files** as needed based on the task:
   - Module scaffolding → `module-scaffold.md`
   - Form fields → `field-utils.md`
   - Components → `components.md`
   - Filters/validation → `filters.md` + `validation.md`

**Priority order** (if context window is limited):

| Priority | File                        | Why                                                |
| -------- | --------------------------- | -------------------------------------------------- |
| 1        | `SKILL.md`                  | Core conventions, anti-patterns, quality checklist |
| 2        | `codebase-index.md`         | Prevents duplicate code — knows what exists        |
| 3        | `skills/validate-output.md` | Final quality gate                                 |
| 4        | `module-scaffold.md`        | Templates for new modules                          |
| 5        | Other reference files       | As needed for the specific task                    |

---

### Sharing with Team Members

For any IDE, each team member needs to:

1. **Clone or copy** the `Wakeb Skills` folder to their machine
2. **Update paths** to point to their local starter projects:
   - FE: their path to `aware-v2-dahsboard`
   - BE: their path to `Starter-Backend`
3. **Follow the IDE-specific setup** from the sections above

## Updating the Skills

The skills are based on the actual starter project code. When the projects
change (new components, new patterns, new conventions), update the
corresponding reference files to stay in sync:

| What changed                | Update this file                                                  |
| --------------------------- | ----------------------------------------------------------------- |
| New common component        | `wakeb-dashboard/references/codebase-index.md` + `components.md`  |
| New FieldUtils creator      | `wakeb-dashboard/references/codebase-index.md` + `field-utils.md` |
| New composable              | `wakeb-dashboard/references/codebase-index.md` + `composables.md` |
| New design tokens / theme   | `wakeb-dashboard/references/design-tokens.md`                     |
| Factory API change          | `wakeb-dashboard/references/factories.md`                         |
| FE convention change        | `wakeb-dashboard/references/patterns.md`                          |
| New controller or model     | `wakeb-backend/references/codebase-index.md`                      |
| New custom validation rule  | `wakeb-backend/references/codebase-index.md` + `validation.md`    |
| New pipeline filter         | `wakeb-backend/references/codebase-index.md` + `filters.md`       |
| New model trait             | `wakeb-backend/references/codebase-index.md` + `models.md`        |
| BE convention change        | `wakeb-backend/references/patterns.md`                            |
| New module file requirement | `*/references/module-scaffold.md`                                 |
| New skill or skill update   | `wakeb-dashboard/references/skills/*.md` or `skills/**/*.md`      |

After updating reference files, no changes to the prompt file are needed —
it pulls from SKILL.md, references, and extended skills automatically.
