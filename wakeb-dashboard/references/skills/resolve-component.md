# Skill: resolve-component

Finds the best matching component from the codebase for a given UI element.
Prevents duplicate component creation by surfacing what already exists.

## When to Use

- Before creating any new component
- When mapping a Figma element to code
- When the user asks "do we have a component for X?"
- When generating any template code that includes UI elements

## Input

| Field     | Type   | Description                                                         |
| --------- | ------ | ------------------------------------------------------------------- |
| `element` | string | What the user needs ("button", "status tag", "modal", "data table") |
| `context` | string | Where it will be used (page type, module)                           |

## Execution

### Step 1 — Search the codebase index

Consult [codebase-index.md](codebase-index.md) in this order:

1. **Common components** — `src/components/common/` (highest priority)
2. **Theme components** — `src/themes/{active}/` (layout-level)
3. **FieldUtils creators** — `@/utils/FieldUtils` (for form fields)
4. **Module-specific components** — `src/modules/{name}/components/` (last resort)

### Step 2 — Match by function, not by name

The user might say "tag" but mean `Badge.vue`. Match by what the element DOES:

| User says...                      | Component                              | Why                                 |
| --------------------------------- | -------------------------------------- | ----------------------------------- |
| "tag", "chip", "status", "label"  | `Badge.vue`                            | Colored status indicator            |
| "popup", "dialog", "overlay"      | `Modal.vue`                            | Dialog container                    |
| "form popup", "edit dialog"       | `AddEditModal.vue`                     | Form in modal/drawer                |
| "dropdown", "menu"                | `Dropdown.vue`                         | Action menu                         |
| "data grid", "list"               | `Table/index.vue`                      | Data display with sorting/filtering |
| "expand", "collapsible"           | `Accordion.vue`                        | Toggle content visibility           |
| "progress", "timeline"            | `VerticalSteps.vue`                    | Step progress display               |
| "drag board", "columns"           | `Kanban.vue`                           | Drag-drop board                     |
| "text field", "input"             | `createTextField` (FieldUtils)         | Form text input                     |
| "select", "dropdown input"        | `createSelectField` (FieldUtils)       | Form select input                   |
| "file upload", "image"            | `createImageInput` / `createFileInput` | File upload field                   |
| "rich text", "editor"             | `createEditorField`                    | WYSIWYG editor                      |
| "map", "location"                 | `createMapField`                       | Map location picker                 |
| "tooltip button"                  | `TooltipBtn.vue`                       | Button with hover tooltip           |
| "action menu", "row actions"      | `TableAction.vue`                      | Table row action dots/buttons       |
| "skeleton", "loading placeholder" | `PageSkeleton.vue`                     | Config-driven loading state         |
| "card grid", "card list"          | `CardView.vue`                         | Multi-card container                |
| "tree", "hierarchy"               | `Tree.vue`                             | Nested tree structure               |
| "tabs", "tab navigation"          | `Tabs.vue`                             | Tab switching                       |

### Step 3 — Return usage example

Provide a working code snippet showing:

- Import statement
- Props with real values
- Common slots if applicable

## Output

```
Component: Badge.vue
Path: @/components/common/Badge.vue
Props: label (String), type (primary|success|danger|warning|info|outline), icon (String), isDot (Boolean)

Usage:
<Badge :label="$t('active')" type="success" />
<Badge :label="$t('inactive')" type="danger" />
<Badge :label="item.status" :type="statusType(item)" icon="mdi-circle" />
```

## Rules

1. If a common component exists, use it — NEVER create a duplicate
2. If a FieldUtils creator exists for a form element, use it — NEVER create a raw input
3. If the element doesn't match anything, check module-specific components before creating new
4. New components go in `src/components/common/` only if they'll be reused across 2+ modules
5. Module-specific components go in `src/modules/{name}/components/`
