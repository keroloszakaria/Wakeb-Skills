---
name: wakeb-dashboard
description: >
  Generates production-ready Vue 3 dashboard code that matches the Wakeb Starter
  Kit conventions exactly — so every team member's output looks like it was
  written by the same developer. Handles module scaffolding, Figma-to-code
  conversion, CRUD pages with factory patterns, form schemas, design token
  mapping, and Arabic-first RTL/i18n support. Use this skill whenever creating
  modules, building CRUD pages, adding forms or fields, converting Figma designs
  to Vue components, working with design tokens or themes, generating stores or
  route configs, or doing any Vue/Vuetify/Tailwind dashboard work — even if the
  user doesn't explicitly say "Wakeb", "starter kit", or "aware". Also use it
  for questions about project structure, component usage, composable APIs,
  factory patterns, or how something is done in the starter.
---

# Wakeb Dashboard Skill

Vue 3 + Vuetify 3 + Tailwind CSS v4 + Pinia + Vite dashboard starter kit.
Arabic-first (RTL), English supported.

This skill exists to keep the entire team's code consistent. Every module,
page, store, and component follows the same structure — so code reviews are
faster, onboarding is easier, and bugs are fewer.

### When to Use

- Creating a new feature module (users, orders, products, settings, etc.)
- Building index/list pages, add/edit pages, or detail views
- Adding or modifying form fields and validation rules
- Converting a Figma design or mockup into Vue components
- Wiring up a Pinia store for a new API endpoint
- Working with design tokens, themes, or color variables
- Setting up routes, permissions, or sidebar navigation
- Generating locale files (ar/en) for a module
- Debugging CRUD issues in the Wakeb starter project
- Asking how something works in the starter or what pattern to follow
- Any Vue/Vuetify/Tailwind dashboard code generation

## Execution Flow

Every code generation request follows these 7 steps in order:

1. **Analyze request** — Classify what the user needs (module, page, component, fix)
2. **Scan codebase** — Consult [codebase-index.md](references/codebase-index.md) to know what already exists
3. **Resolve reusable parts** — Find existing components, composables, and utilities that match (see [resolve-component](references/skills/resolve-component.md))
4. **Generate structure** — Place files in correct locations (see [enforce-structure](references/skills/enforce-structure.md))
5. **Generate code** — Write code using project factories, patterns, and conventions
6. **Validate output** — Run full validation checklist (see [validate-output](references/skills/validate-output.md))
7. **Return result** — Only return code that passes all checks

## Skills Taxonomy

| Category           | Skills                                                                      | Purpose                                                            |
| ------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| UI Generation      | [generate-page-from-design](references/skills/generate-page-from-design.md) | Convert designs/descriptions to Vue pages using project components |
| Component Reuse    | [resolve-component](references/skills/resolve-component.md)                 | Find existing components instead of creating duplicates            |
| Module Scaffolding | [generate-module](references/skills/generate-module.md)                     | Full-stack module generation (FE + BE aligned)                     |
| Refactoring        | [reuse-composable](references/skills/reuse-composable.md)                   | Extract and reuse shared logic via composables                     |
| API Integration    | [connect-api](references/skills/connect-api.md)                             | Wire frontend stores to backend endpoints                          |
| Validation & QA    | [validate-output](references/skills/validate-output.md)                     | Final check on all generated code                                  |
| Structure          | [enforce-structure](references/skills/enforce-structure.md)                 | Verify file placement and naming conventions                       |

**Codebase Awareness:** [references/codebase-index.md](references/codebase-index.md) — full inventory of every component, composable, factory, utility, and store. Consult BEFORE generating code.

## Architecture

```
src/
├── components/common/       # Button, Badge, Modal, Table, GenericForm...
├── composables/             # useLookupPage, useAlert, useStorage...
├── Factory/                 # ModuleConfigFactory, BaseCrudFactory, TableFactory
├── modules/{name}/          # Feature modules (domain-driven)
│   ├── config.js            # createModuleConfig({...})
│   ├── router/index.js      # Route definitions with permissions
│   ├── stores/              # Pinia stores via useCrudFactory
│   ├── views/               # IndexView, AddEditView
│   ├── schema/index.js      # Form fields via FieldUtils
│   └── locales/             # ar.json, en.json
├── themes/{name}/           # IndexPage, AddEditPage, Sidebar, tokens
├── services/api.js          # HTTP wrapper (jervis-connect)
└── utils/                   # FieldUtils, validationRules, formDataHandler
```

## Decision Protocol

Match the user's request to the right workflow. When in doubt, start with
the module scaffold — it covers the common case and produces all required files.

| User says...                              | Workflow                    | Priority |
| ----------------------------------------- | --------------------------- | -------- |
| "Create module X" / "new feature"         | Module scaffold             | CRITICAL |
| "Convert Figma design" / shares Figma URL | Figma-to-code               | CRITICAL |
| "Monitoring page" / video + data layout   | Monitoring dashboard pattern | CRITICAL |
| "Detail/view page" / profile / info       | Detail page pattern          | HIGH     |
| "Add form/field" / "edit schema"          | Form schema                 | HIGH     |
| "Create page" / "build view"              | Page template               | HIGH     |
| "Fix bug" / "error" / "not working"       | Debugging (see patterns.md) | MEDIUM   |
| Any code output                           | Run quality checklist first | ALWAYS   |

## Module Scaffold Workflow

Generate ALL 7 files for a new module. See [references/module-scaffold.md](references/module-scaffold.md) for complete templates.

**Why all 7 files?** The starter kit auto-discovers modules by convention. Missing
any file (config, router, store, schema, views, locales) breaks the module
loader or leaves gaps the developer has to fill manually.

**Key rules:**

- Config uses `createModuleConfig()` from `@/Factory/ModuleConfigFactory.ts`
- Store uses `useCrudFactory()` from `@/Factory/BaseCrudFactory`
- Index view uses `useLookupPage()` — handles table, pagination, search,
  modals, and CRUD actions automatically. Wiring these manually duplicates
  logic the composable already provides and introduces bugs.
- Schema uses field creators from `@/utils/FieldUtils`
- Always create both `ar.json` and `en.json` locale files
- Router meta includes `permissions: ['module.action']`

**Quick index view pattern:**

```vue
<script setup>
import { useLookupPage } from "@/composables/useLookupPage";
import config from "../config";

const IndexPage = useThemedComponent("IndexPage");

const {
  STORE,
  headers,
  getItems,
  addRow,
  editRow,
  viewRow,
  deleteRow,
  deleteRows,
  moduleName,
  moduleNameSingular,
  isShowModal,
  isView,
  isCreate,
  schema,
  modalTitle,
  submitForm,
} = await useLookupPage({
  itemKey: config.module,
  structureType: "module",
  mode: config.crud.mode,
  enums: config.enums,
  models: config.models,
  headersConfig: config.headers,
});
</script>
```

For full `useLookupPage` options and return value: [references/composables.md](references/composables.md)

## Figma-to-Code Workflow

This workflow follows the official Figma implement-design approach, adapted
for the Wakeb component library. The goal is 1:1 visual parity while reusing
project components (not raw HTML) so themes, RTL, and accessibility work
automatically.

**Step 1 — Fetch the design.** If the user provides a Figma URL, use the
Figma MCP tools (`get_design_context`, `get_screenshot`) to fetch the design
data, code hints, and a screenshot. This gives you accurate spacing, colors,
and component structure instead of guessing from a description.

**Step 2 — Map to project components.** Raw HTML (`<button>`, `<table>`,
`<input>`) duplicates logic that common components already handle — theming,
RTL direction, validation, and accessibility. Using them means the developer
has to maintain that logic separately, which always drifts:

| Figma Element   | Component                                                                |
| --------------- | ------------------------------------------------------------------------ |
| Button          | `Button.vue` — type: primary/secondary/outline/danger/tertiary/link/icon |
| Tag/Chip/Status | `Badge.vue` — type: primary/success/danger/warning/info/outline          |
| Table           | `Table/index.vue` — NEVER `v-data-table-server`                          |
| Pagination      | (handled by Table internally) — NEVER `v-pagination`                     |
| Modal/Dialog    | `Modal.vue` — NEVER `v-dialog`                                          |
| Form            | `GenericForm` with FieldUtils schema — NEVER raw inputs                  |
| Search input    | `GenericForm` + `createTextField` — NEVER `v-text-field`                 |
| Card            | `Card.vue`                                                               |
| Video/Camera    | `SteamingHandle.vue` — NEVER raw `<video>`                               |
| Inline icon     | `SvgIcon` from jervis-icons — NEVER raw `<svg>` or `<img>`              |
| Filter dropdown | `Dropdown.vue` — NEVER `v-select` for non-form dropdowns                |

Full component props and mapping table: [references/components.md](references/components.md)

**Step 3 — Design tokens for ALL colors.** The project supports multiple
themes (aware, sar, etc.) via CSS variables. Hardcoding hex/rgb values means
the component looks correct in one theme but breaks in others:

```css
/* Correct — adapts to any theme */
color: rgba(var(--v-theme-text-text-primary-900), 1)
background: rgba(var(--v-theme-background-bg-primary), 1)
border-color: rgba(var(--v-theme-border-border-primary), 1)
```

Full token reference: [references/design-tokens.md](references/design-tokens.md)

**Step 4 — Layout with Vuetify grid + Tailwind.** Use `v-row`/`v-col` for
responsive layout and Tailwind utilities for spacing.

**Step 5 — Validate.** Compare the rendered result against the Figma
screenshot. Check spacing, colors, typography, and responsive behavior.

## Form Schema Workflow

Use field creators from `@/utils/FieldUtils`:

```js
import {
  createTextField,
  createSelectField,
  createImageInput,
} from "@/utils/FieldUtils";

const schema = ref([
  createTextField({
    key: "name",
    label: "name",
    required: true,
    cols: { md: 6, lg: 6 },
  }),
  createSelectField({ key: "status", label: "status", options: statusOptions }),
  createImageInput({ key: "avatar", label: "avatar", required: false }),
]);
```

Available creators: `createTextField`, `createTextAreaField`, `createSelectField`,
`createComboBoxField`, `createNumberField`, `createDateTimeField`, `createCheckBoxField`,
`createRadioButtonField`, `createImageInput`, `createPasswordField`, `createPhoneField`,
`createEditorField`, `createColorField`, `createOtpInput`, `createMapField`, `createButton`

Full signatures and validation rules: [references/field-utils.md](references/field-utils.md)

## CRUD & Factory Patterns

**Store:**

```js
const store = useCrudFactory("endpoint-name", {
  cache: { enabled: true },
  hooks: { beforeCreate: (data) => data, afterGetAll: (res) => {} },
});
```

**Table Factory** is wrapped by `useLookupPage` — prefer it over manual `TableFactory`.

Full factory API: [references/factories.md](references/factories.md)

## Code Conventions

- `<script setup>` only — the entire codebase uses Composition API; Options API
  would create an inconsistent pattern that confuses grep-based refactoring
- All text in `$t('key')` or `t('key')` — the project ships in Arabic and English;
  bare strings mean the UI breaks for half the users
- RTL: `start`/`end`, `ms-`/`me-`, `ps-`/`pe-` — `left`/`right` renders backward
  in Arabic because the reading direction is flipped
- Prettier: no semicolons, single quotes, 2 spaces — enforced by project config;
  mixing styles causes noisy diffs and merge conflicts
- Components from `src/components/common/` — they carry theming, RTL, and a11y
  behavior that raw HTML elements don't
- Colors from `rgba(var(--v-theme-*), 1)` — the project supports theme switching;
  hardcoded colors look right in one theme and wrong in all others
- Layout: `v-row`/`v-col` + Tailwind spacing

Full conventions, debugging, API, routing, themes, and env vars: [references/patterns.md](references/patterns.md)

## Anti-Patterns

These patterns cause real problems — broken themes, broken RTL, duplicated
logic, noisy diffs, or code that looks alien next to the rest of the project:

| Do NOT                                     | Do instead                              | Why it matters                                   |
| ------------------------------------------ | --------------------------------------- | ------------------------------------------------ |
| Use Options API                            | `<script setup>` only                   | Breaks codebase consistency and grep refactoring |
| Hardcode colors (`#fff`, `rgb(...)`)       | Design token CSS variables              | Breaks when theme changes                        |
| Use `left`/`right`/`ml-`/`mr-`/`pl-`/`pr-` | `start`/`end`/`ms-`/`me-`/`ps-`/`pe-`   | Renders backward in Arabic RTL                   |
| Wire CRUD manually (axios + ref + watch)   | `useLookupPage()` composable            | Duplicates 200+ lines the composable handles     |
| Write raw `<button>`/`<table>`/`<input>`   | Project components from `common/`       | Loses theming, RTL, and a11y behavior            |
| Use `v-data-table-server`                  | `Table/index.vue`                       | Table handles theming, pagination, RTL           |
| Use `v-pagination`                         | (Table handles internally)              | Pagination is built into Table component         |
| Use `v-btn`                                | `Button.vue`                            | Button handles theming, sizes, icon layout       |
| Use `v-chip`                               | `Badge.vue`                             | Badge handles status colors and dot style        |
| Use `v-dialog`                             | `Modal.vue`                             | Modal handles backdrop, close, loading           |
| Use `v-text-field` or raw `<input>`        | `GenericForm` + `createTextField`       | Form handles validation, i18n, RTL               |
| Use raw `<video>`                          | `SteamingHandle.vue`                    | Handles HLS playback and error states            |
| Put plain strings in templates             | `$t('key')` with locale files           | Shows English-only to Arabic users               |
| Create store without factory               | `useCrudFactory()` from BaseCrudFactory | Loses caching, hooks, and error handling         |
| Use semicolons or double quotes            | Prettier: no semicolons, single quotes  | Causes noisy diffs and merge conflicts           |

## Quick Reference

Common operations at a glance — read the linked reference for full details.

| Task                       | Key API / File                          | Reference                                           |
| -------------------------- | --------------------------------------- | --------------------------------------------------- |
| Create module config       | `createModuleConfig()`                  | [module-scaffold.md](references/module-scaffold.md) |
| Create store               | `useCrudFactory('endpoint')`            | [factories.md](references/factories.md)             |
| Build index page           | `useLookupPage({ itemKey, ... })`       | [composables.md](references/composables.md)         |
| Add form field             | `createTextField({ key, label })`       | [field-utils.md](references/field-utils.md)         |
| Map Figma → component      | See mapping table above                 | [components.md](references/components.md)           |
| Apply colors               | `rgba(var(--v-theme-*), 1)`             | [design-tokens.md](references/design-tokens.md)     |
| Add validation             | `rules: 'required\|email\|max:255'`     | [field-utils.md](references/field-utils.md)         |
| Configure table headers    | `config.headers` array                  | [module-scaffold.md](references/module-scaffold.md) |
| Add route with permissions | `meta: { permissions: ['mod.action'] }` | [patterns.md](references/patterns.md)               |

## Quality Checklist

Run BEFORE outputting any code:

```
Task Progress:
- [ ] Using <script setup> (Composition API only)
- [ ] All text wrapped in $t() / t()
- [ ] Colors from design tokens (CSS variables)
- [ ] Layout with v-row/v-col + Tailwind
- [ ] RTL-compatible (start/end, ms/me, ps/pe)
- [ ] Components from src/components/common/
- [ ] Prettier format (no semicolons, single quotes)
```

**For modules, also check:**

```
- [ ] Config uses createModuleConfig()
- [ ] Store uses useCrudFactory()
- [ ] View uses useLookupPage() (NOT manual wiring)
- [ ] Schema uses FieldUtils creators
- [ ] Both ar.json + en.json locale files
- [ ] Router with permissions in meta
- [ ] Sidebar config with order + icon
```

**For Figma conversion, also check:**

```
- [ ] Design tokens for all colors/spacing (no hex/rgb hardcoding)
- [ ] Responsive grid (mobile-first with md/lg)
- [ ] Interactive states use token variables
- [ ] No v-data-table-server (use Table/index.vue)
- [ ] No v-pagination (Table handles internally)
- [ ] No v-btn, v-chip, v-dialog, v-text-field (use project wrappers)
- [ ] No raw <video> (use SteamingHandle.vue)
- [ ] SvgIcon for inline icons (from jervis-icons)
- [ ] Gradient borders use CSS vars (--gradient-purple, --gradient-blue, --gradient-black)
- [ ] Glassmorphism overlays use backdrop-filter: blur() + rgba bg
```
