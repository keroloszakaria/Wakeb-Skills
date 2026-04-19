# Component Reference

## Contents

- Figma-to-Code mapping table
- Anti-pattern table (Vuetify direct usage)
- Button props
- Badge props
- Modal props
- Table props
- GenericForm props
- SvgIcon usage
- SteamingHandle props
- Layout composition patterns
- IndexPage theme wrapper
- AddEditPage theme wrapper

---

## Figma → Code Mapping

| Figma Element           | Component            | Usage                                                |
| ----------------------- | -------------------- | ---------------------------------------------------- |
| Any button              | `Button.vue`         | `<Button title="Submit" type="primary" size="md" />` |
| Colored tag/chip/status | `Badge.vue`          | `<Badge label="Active" type="success" isDot />`      |
| Card container          | `Card.vue`           | `<Card :item="item" :headers="headers" />`           |
| Data table              | `Table/index.vue`    | `<Table :headers="headers" :items="items" />`        |
| Modal/Dialog            | `Modal.vue`          | `<Modal v-model="isOpen" title="Title" />`           |
| Add/Edit form modal     | `AddEditModal.vue`   | `<AddEditModal :schema="schema" />`                  |
| Tab navigation          | `Tabs.vue`           | `<Tabs :tabs="items" :selectedTab="active" />`       |
| Form fields             | `GenericForm`        | `<GenericForm :schema="schema" />`                   |
| Accordion               | `Accordion.vue`      | `<Accordion title="Section">content</Accordion>`     |
| Dropdown menu           | `Dropdown.vue`       | `<Dropdown :menuItems="items" />`                    |
| Loading                 | `Loading.vue`        | `<Loading />`                                        |
| Tree view               | `Tree.vue`           | `<Tree :nodes="data" />`                             |
| Steps/Timeline          | `VerticalSteps.vue`  | `<VerticalSteps :steps="steps" />`                   |
| Kanban board            | `Kanban.vue`         | `<Kanban :columns="cols" />`                         |
| Video/camera stream     | `SteamingHandle.vue` | `<SteamingHandle type="video" :videoSrc="url" />`    |
| Inline icon (SVG)       | `SvgIcon`            | `<SvgIcon name="icon-name" size="sm" />`             |

### Anti-Pattern: Vuetify Direct Usage

NEVER use these Vuetify components directly — always use the project wrapper:

| Vuetify Component     | Use Instead                | Reason                                     |
| --------------------- | -------------------------- | ------------------------------------------ |
| `v-data-table-server` | `Table/index.vue`          | Table handles theming, pagination, RTL     |
| `v-pagination`        | (Table handles internally) | Pagination is built into Table component   |
| `v-btn`               | `Button.vue`               | Button handles theming, sizes, icon layout |
| `v-chip`              | `Badge.vue`                | Badge handles status colors and dot style  |
| `v-dialog`            | `Modal.vue`                | Modal handles backdrop, close, loading     |
| `v-text-field`        | `GenericForm` + FieldUtils | Form handles validation, i18n, RTL         |
| `v-select` (non-form) | `Dropdown.vue`             | Dropdown handles menu styling and actions  |

---

## Button.vue

```
Props:
  title: String                             # Button text
  type: 'primary'|'secondary'|'outline'|'danger'|'tertiary'|'link'|'icon'|'table-icon'
  size: 'xl'|'lg'|'md'|'sm'                # Default: 'md'
  icon: String|Object
  iconPosition: 'start'|'end'              # Default: 'start'
  iconSize: String                          # Default: 'sm'
  maxIconWidth: String|Number               # Default: '100%'
  isLoading: Boolean
  disabled: Boolean
  isDropdown: Boolean
  isTooltip: Boolean
  location: String                          # Tooltip location, default: 'top'
  action: 'button'|'submit'|'reset'         # Default: 'button'
  classList: String|Array

Emits: @click
```

---

## Badge.vue

```
Props:
  label: String
  type: 'primary'|'success'|'danger'|'warning'|'info'|'outline'
  icon: String
  isDot: Boolean                            # Default: false
  classList: String
```

---

## Modal.vue

```
Props:
  v-model: Boolean                          # Dialog visibility
  width: String|Number
  title: String
  isCloseable: Boolean                      # Default: true
  persistent: Boolean                       # Default: true
  customAction: Boolean                     # Default: false
  isLoading: Boolean
  isDataLoaded: Boolean                     # Default: true

Emits: @afterLeave
Slots: #title, #header-actions, #content, #actions
```

---

## Table (Table/index.vue)

```
Props:
  headers: Array                            # Column definitions
  items: Array                              # Row data
  pagination: Object                        # { page, itemsPerPage, total }
  tableName: String
  isLoading: Boolean
  showSelect: Boolean                       # Default: false
  showTrashed: Boolean                      # Default: false
  showInactive: Boolean                     # Default: false
  filterSchema: Array                       # Filter form fields
  advancedFilterSchema: Array
  switchType: 'tabs'|'dropdown'             # Default: 'tabs'
  addSearch: Boolean                        # Default: true
  addButtonTitle: String
  addButtonLink: String
  emptyMessage: String                      # Default: 'no_data'
  reloadTable: Boolean
  showHeader: Boolean                       # Default: true
  hidePagination: Boolean                   # Default: false
  showHeaderBtns: Boolean                   # Default: true
  defaultSelectedRows: Array

Emits:
  @searchValue, @tableState, @deleteRows, @openModal, @isTrashed,
  @update:options, @selectedRows, @rowClicked, @searchfilter
```

---

## GenericForm

```
Props:
  schema: Array                             # Field objects array
  formClass: String
  propsToWatch: Array
  stopValidation: Boolean                   # Default: false
  emitOnChange: Boolean                     # Default: false

Emits: @submit, @updateSchema
Exposed: validate() → { valid: bool }, getFieldComponent(type)
```

Supported field types:
`text`, `number`, `password`, `textarea`, `select`, `switchbox`, `checkbox`,
`radioButton`, `datetime`, `phoneInput`, `otp`, `imageUploader`, `combobox`,
`colorPicker`, `editor`, `files`

---

## IndexPage (Theme Wrapper)

```
Props:
  moduleName: String
  moduleNameSingular: String
  headers: Array
  items: Array
  pagination: Object
  isLoading: Boolean
  modalWidth: String|Number
  schema: Array
  actions: Array
  showSelect: Boolean
  showTrashed: Boolean
  showInactive: Boolean
  switchType: String                        # 'tabs' | 'dropdown'
  buttonTitle: String
  modalTitle: String
  isAddLoading: Boolean
  isView: Boolean
  isCreate: Boolean
  skeleton: Object
  isDots: Boolean
  addMode: String                           # 'modal' | 'page'
  viewMode: String                          # 'table' | 'card' | 'custom'
  showExportButton: Boolean

Emits:
  @editRow, @onUpdateOptions, @isTrashed, @searchValue, @openModal,
  @deleteRows, @addEditRow, @addRow, @tableState, @viewRow, @deleteRow,
  @toggleActiveRow, @forceDeleteRow, @exportData

Slots: #[`item.{headerKey}`] for custom column rendering
```

---

## AddEditPage (Theme Wrapper)

```
Props:
  title: String
  form: String                              # Default: 'myForm'
  showBack: Boolean                         # Default: false
  isTitle: Boolean                          # Default: true
  isLast: Boolean                           # Default: false
  isView: Boolean                           # Default: false
  isCreate: Boolean                         # Default: false

Emits: @back, @next
```

---

## SvgIcon (from jervis-icons plugin)

```
Usage:
  <SvgIcon name="icon-name" size="sm" class="text-white" />

Sizes: xs | sm | md | lg | xl

Notes:
  - Globally registered by jervis-icons plugin — no import needed
  - Use for inline icons in custom layouts (cards, lists, stats)
  - For button icons: use Button's `icon` prop instead of SvgIcon
  - Icon names match the jervis-icons library naming convention
```

---

## SteamingHandle.vue

```
Props:
  type: String                              # 'video' | 'audio'
  videoSrc: String                          # HLS stream URL

Usage:
  <SteamingHandle type="video" :videoSrc="streamUrl" />

Notes:
  - Use for ALL video/camera/surveillance streams
  - NEVER use raw <video> element
  - Handles HLS playback, loading states, and error handling
  - Place inside a relative container with overflow-hidden for overlays
  - Combine with glassmorphism overlays for stats display:
    position: absolute; backdrop-filter: blur(8px); background: rgba(0,0,0,0.6)
```

---

## Layout Composition Patterns

### Monitoring Dashboard Layout

Used for pages with video streams, sidebar lists, and data tables:

```vue
<div class="flex flex-col gap-4">
  <!-- 1. Filter Bar -->
  <div class="flex items-center justify-between">
    <Button title="$t('select_dates')" type="outline" icon="calendar" />
    <Dropdown :menuItems="filters" title="$t('filter')" type="outline" />
  </div>

  <!-- 2. Video + Sidebar -->
  <div class="flex gap-4">
    <div class="flex-1 relative rounded-2xl overflow-hidden">
      <SteamingHandle type="video" :videoSrc="url" />
      <!-- Glassmorphism stats overlay -->
      <div class="absolute bottom-0 inset-x-0 p-4"
           style="backdrop-filter: blur(8px); background: rgba(0,0,0,0.6)">
        <Badge label="status" type="success" isDot />
      </div>
    </div>
    <div class="card-gradient flex flex-col min-w-81.75">
      <GenericForm :schema="searchSchema" :emitOnChange="true" />
      <ul class="overflow-y-auto flex flex-col gap-1">
        <li v-for="item in items" :key="item.id" @click="select(item)">
          <!-- item content -->
        </li>
      </ul>
    </div>
  </div>

  <!-- 3. Data Table (project Table, NOT v-data-table-server) -->
  <Table :headers="headers" :items="items" :pagination="pagination" />
</div>
```

### Detail/View Page Layout

```vue
<div class="flex flex-col gap-4">
  <div class="flex items-center gap-3">
    <Button type="icon" icon="arrow-left-01" @click="$router.back()" />
    <h2>{{ $t('item_details') }}</h2>
  </div>
  <div class="flex gap-4">
    <div class="flex-1 tableContainer rounded-2xl p-4">
      <!-- Use GenericForm with viewMode for read-only display -->
    </div>
  </div>
</div>
```

### Gradient Card Sidebar

```css
.card-gradient {
  padding: 2px;
  border-radius: 16px;
  background: linear-gradient(
    140deg,
    var(--gradient-purple),
    var(--gradient-blue),
    var(--gradient-black),
    var(--gradient-black),
    var(--gradient-black),
    var(--gradient-black),
    var(--gradient-black)
  );
}
.card-gradient > div {
  background: linear-gradient(
    240deg,
    var(--gradient-blue),
    var(--gradient-purple),
    var(--gradient-black)
  );
  border-radius: 16px;
}
```
