# Skill: Smart Component Generator

Generates Vue components that follow Wakeb Starter conventions — reusing
existing components, applying design tokens, and enforcing project patterns.

## When to Use

- When the user asks to create a new component
- When converting a Figma design element to a component
- When a page needs a custom component not in the common library

## Decision Tree

```
User wants a UI element
  │
  ├── Does it exist in components/common/?
  │     YES → Use it directly (see resolve-component skill)
  │     NO  ↓
  │
  ├── Is it a variation of an existing component?
  │     YES → Use the existing component with different props
  │           (e.g., Button type="danger" instead of new DangerButton)
  │     NO  ↓
  │
  ├── Will it be used across multiple modules?
  │     YES → Create in src/components/common/
  │     NO  → Create in src/modules/{name}/components/
  │
  └── Generate the component following the template below
```

## Component Template

```vue
<template>
  <div class="component-name" :class="classList">
    <!-- Use project components internally -->
    <slot />
  </div>
</template>

<script setup>
defineOptions({ name: "ComponentName" });

const props = defineProps({
  // Required props first
  title: { type: String, required: true },
  // Optional props with defaults
  type: { type: String, default: "primary" },
  disabled: { type: Boolean, default: false },
  classList: { type: [String, Array], default: "" },
});

const emit = defineEmits(["click", "update:modelValue"]);
</script>

<style scoped>
.component-name {
  /* Design tokens only — NEVER hex/rgb */
  color: rgba(var(--v-theme-text-text-primary-900), 1);
  background: rgba(var(--v-theme-background-bg-secondary), 1);
  border: 1px solid rgba(var(--v-theme-border-border-primary), 1);
  border-radius: 12px;
}
</style>
```

## Rules

```
STRUCTURE:
□ <template> first, then <script setup>, then <style scoped>
□ defineOptions({ name: 'ComponentName' }) for dev tools
□ Props with explicit types and defaults
□ Events declared with defineEmits

COMPOSITION:
□ Use existing common components internally:
  - Button.vue for actions (not <button> or v-btn)
  - Badge.vue for status indicators
  - SvgIcon for icons
  - GenericForm for any form fields
□ Use slots for content projection
□ Use v-model pattern for two-way binding

STYLING:
□ Design token CSS variables for ALL colors
□ Tailwind utilities for spacing and layout
□ scoped styles to prevent leaking
□ RTL-safe: start/end, ms-/me-, ps-/pe-
□ Support dark mode via theme tokens (they auto-switch)

NAMING:
□ PascalCase file name: MyComponent.vue
□ Props: camelCase
□ Events: kebab-case
□ CSS classes: kebab-case
```

## Component Types

### Presentational Component (most common)

```
- Receives data via props
- Emits events for user actions
- No API calls or store access
- Examples: Card, Badge, Button
```

### Container Component

```
- Orchestrates child components
- May access stores or composables
- Handles data flow between children
- Examples: IndexPage, AddEditPage
```

### Module Component (domain-specific)

```
- Lives in src/modules/{name}/components/
- Uses module-specific logic
- May import module store/schema
- Examples: DroneMap, FlightTimeline
```

## Anti-Patterns

```
❌ Creating a component that wraps a single Vuetify component:
   → Use the existing project wrapper (Button.vue, Badge.vue, etc.)

❌ Props drilling through 3+ levels:
   → Use provide/inject or a composable

❌ Components with 10+ props:
   → Consider splitting into smaller components or using slots

❌ Inline styles in templates:
   → Use Tailwind classes or scoped CSS with design tokens

❌ Business logic in components:
   → Move to composables or store hooks
```
