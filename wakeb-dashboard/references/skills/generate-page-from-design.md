# Skill: generate-page-from-design

Takes a UI description, mockup, or Figma structure and generates a complete
page using ONLY existing project components. Never creates raw HTML when a
project component exists.

## When to Use

- User shares a Figma URL or screenshot
- User describes a page layout ("table with filters", "form with tabs")
- User says "convert this design" or "build this page"

## Input

| Field       | Type                                       | Description                       |
| ----------- | ------------------------------------------ | --------------------------------- |
| `design`    | Figma URL, screenshot, or text description | The UI to implement               |
| `page_type` | `index` / `add-edit` / `detail` / `custom` | What kind of page                 |
| `module`    | string                                     | Which module this page belongs to |

## Execution

### Step 1 — Understand the design

If Figma URL: use `get_design_context` + `get_screenshot` from Figma MCP.
If description: identify the UI elements (tables, forms, cards, modals, buttons, badges).

### Step 2 — Scan codebase index

Read [codebase-index.md](codebase-index.md) and identify:

- Which common components match the design elements
- Which composable handles the page type
- Which theme components provide the layout

### Step 3 — Resolve components

Map EVERY visual element to an existing component:

| Design Element       | Resolve To                            | Never Do                             |
| -------------------- | ------------------------------------- | ------------------------------------ |
| Data table           | `Table/index.vue` via `useLookupPage` | `v-data-table-server`, raw `<table>` |
| Pagination           | (handled by Table internally)         | `v-pagination` directly              |
| Form                 | `GenericForm` + FieldUtils schema     | Raw `<form>` + `<input>`             |
| Search input         | `GenericForm` + `createTextField`     | Raw `<input>` or `v-text-field`      |
| Modal/Dialog         | `Modal.vue` or `AddEditModal.vue`     | `v-dialog` or custom dialog          |
| Button               | `Button.vue`                          | `v-btn` or raw `<button>`            |
| Status tag           | `Badge.vue`                           | `v-chip` or custom `<span>`          |
| Card                 | `Card.vue`                            | Custom `<div>` card                  |
| Dropdown/filter menu | `Dropdown.vue`                        | `v-select` for non-form dropdowns    |
| Tabs                 | `Tabs.vue`                            | Custom tab implementation            |
| Accordion            | `Accordion.vue`                       | Custom collapsible                   |
| Loading              | `Loading.vue`                         | Custom spinner                       |
| Tree                 | `Tree.vue`                            | Custom tree                          |
| Stepper              | `VerticalSteps.vue`                   | Custom stepper                       |
| Video/camera stream  | `SteamingHandle.vue`                  | Raw `<video>` element                |
| Inline icon          | `SvgIcon` (from jervis-icons)         | Raw `<svg>` or `<img>` for icons     |

### Step 4 — Apply design tokens

Replace ALL colors from the design with CSS variables:

```
Figma #hex → rgba(var(--v-theme-{token-name}), 1)
```

Reference: [design-tokens.md](design-tokens.md)

### Step 5 — Generate the page

For **index pages**: use `useLookupPage()` — generates table + modal + pagination
automatically. Only define config, headers, and schema.

For **add/edit pages**: use `AddEditModal` or a standalone form with `GenericForm`

- FieldUtils schema.

For **detail pages**: use themed `IndexPage` layout with `Card.vue` for sections.

For **monitoring/dashboard pages** (video + sidebar + table): compose from
`SteamingHandle` (video), sidebar with `GenericForm` search + item list,
`Table/index.vue` for data, and glassmorphism overlays for stats. Use
`.card-gradient` CSS class for gradient-border sidebar cards. Use
`backdrop-filter: blur(8px)` with `rgba(0,0,0,0.6)` for overlay cards.

For **custom pages**: compose from common components, use `v-row`/`v-col` for
layout, Tailwind for spacing.

### Step 6 — Validate

```
- [ ] Every visual element uses a project component (no raw HTML)
- [ ] No v-data-table-server (use Table/index.vue)
- [ ] No v-pagination (Table handles pagination internally)
- [ ] No v-btn (use Button.vue)
- [ ] No v-chip (use Badge.vue)
- [ ] No v-dialog (use Modal.vue)
- [ ] No v-text-field or raw <input> (use GenericForm + createTextField)
- [ ] No raw <video> (use SteamingHandle.vue)
- [ ] All colors from design tokens (no hex/rgb hardcoding)
- [ ] Gradient borders use CSS vars (--gradient-purple, --gradient-blue, --gradient-black)
- [ ] RTL-safe (start/end, ms/me, ps/pe)
- [ ] All text in $t() / t()
- [ ] <script setup> with Composition API
- [ ] Layout matches design with v-row/v-col + Tailwind
- [ ] SvgIcon from jervis-icons for inline icons
```

## Output

A complete `.vue` file (or set of files) using `<script setup>`, project
components, design tokens, and i18n. The page should be visually identical to
the design while being architecturally consistent with the codebase.

## Example

**Input:** "Create an orders page with a table showing order number, customer,
status, total, and date. Status should be a colored badge."

**Output:**

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

<template>
  <component
    :is="IndexPage"
    v-bind="{
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
      STORE,
    }"
  />
</template>
```

The config.headers defines status with Badge component rendering.
