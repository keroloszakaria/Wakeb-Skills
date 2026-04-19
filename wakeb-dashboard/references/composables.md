# Composables Reference

## Contents

- useLookupPage (master composable)
- Composables quick reference table
- Global functions

---

## useLookupPage — Master Composable

The most important composable. Auto-wires everything for a CRUD page: store, schema, table factory, enums, models, breadcrumbs, and all CRUD actions.

### Full Options

```js
const result = await useLookupPage({
  // Required
  itemKey: "moduleName",

  // Structure
  structureType: "module", // 'module' | 'page'
  folder: null, // For 'page' type: folder path

  // Store & Schema (auto-created if null)
  STORE: null,
  SCHEMA: null,

  // Display
  headersConfig: config.headers, // Table column definitions
  viewMode: "table", // 'table' | 'card'
  mode: "modal", // 'modal' | 'page' (add/edit mode)
  pageType: "index", // 'index' | 'add' | 'edit' | 'view'

  // Data fetching
  enums: [{ name: "enum.key" }],
  models: [{ name: "roles" }],
  configs: [],
  fetchDetailedItem: false,

  // Features
  ignoredActions: [], // Skip: ['add', 'edit', 'delete']
  nameKey: "name",
  skipBreadcrumbs: false,
  steps: null, // StepDefinition[] for stepper forms
  stepperIcon: null,
});
```

### Full Return Value

```js
const {
  t, // i18n translator
  isRendered, // Ref<boolean>
  headers, // Ref<HeaderConfig[]>
  STORE, // BaseCrudReturn (items, pagination, loading, error)
  moduleName, // snake_case
  moduleNameSingular, // singular form
  viewMode,

  // Modal/Form state
  isView, // Ref<boolean>
  isShowModal, // Ref<boolean>
  isCreate, // Ref<boolean>
  schema, // Ref<any>
  steps, // StepDefinition[] | null
  modalTitle, // ComputedRef<string>
  addEditLoading, // ComputedRef<boolean>

  // CRUD Methods
  getItems, // (params?) => Promise
  handleModal, // ({ createMode, viewMode, setItem, selectedItem }) => Promise
  submitForm, // (data) => Promise<{ errors? }>

  // Row Actions
  addRow, // (item?) => void
  editRow, // (item) => void
  viewRow, // (item) => void
  deleteRow, // (item) => Promise
  deleteRows, // (ids[]) => Promise
  toggleActiveRow, // (item) => Promise
  forceDeleteRow, // (item) => Promise
} = await useLookupPage(options);
```

---

## Composables Quick Reference

| Composable             | Import                               | Purpose                      |
| ---------------------- | ------------------------------------ | ---------------------------- |
| `useAlert`             | `@/composables/useAlert`             | Alerts & confirmations       |
| `useLookupPage`        | `@/composables/useLookupPage`        | Full CRUD page orchestration |
| `useDateTimeFormatter` | `@/composables/useDateTimeFormatter` | Date/time formatting         |
| `useNumberConverter`   | `@/composables/useNumberConverter`   | Number localization          |
| `useAutoTranslate`     | `@/composables/useAutoTranslate`     | Auto-translate fields        |
| `useLanguageSwitcher`  | `@/composables/useLanguageSwitcher`  | Switch ar/en locale          |
| `useIsDark`            | `@/composables/useIsDark`            | Dark mode toggle             |
| `useStorage`           | `@/composables/useStorage`           | localStorage wrapper         |
| `useTableActions`      | `@/composables/useTableActions`      | Bulk select/delete           |
| `useTextTruncator`     | `@/composables/useTextTruncator`     | Truncate text                |
| `useFileUpload`        | `@/composables/useFileUpload`        | File upload to API           |
| `useVoiceRecorder`     | `@/composables/useVoiceRecorder`     | Audio recording              |
| `useResizableSidebar`  | `@/composables/useResizableSidebar`  | Sidebar resize               |
| `useCookies`           | `@/composables/useCookies`           | Cookie management            |
| `useAppSettings`       | `@/composables/useAppSettings`       | App theme/favicon            |

---

## Global Functions (no import needed)

```js
globalThis.t("key"); // i18n translate
globalThis.useThemedComponent("name"); // Get themed component
globalThis.useThemedConfig(); // Get theme config
globalThis.useModuleComponent("name"); // Get module component
```
