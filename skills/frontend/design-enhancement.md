# Skill: Design Enhancement

Improves the visual quality of generated UI when no Figma design is provided.
Uses the project's design token system and component library to create
polished, consistent interfaces.

## When to Use

- When generating UI without a Figma design reference
- When the user wants a page/component to "look good" without specific design specs
- When reviewing generated pages for visual quality
- After creating a page — apply as a finishing pass

## Enhancement Strategy

### 1. Spacing & Layout

```
□ Consistent spacing using the token scale:
  - Section gaps: gap-6 (24px)
  - Card padding: p-4 (16px) or p-6 (24px)
  - Between form fields: gap-4 (16px)
  - Between buttons: gap-2 (8px) or gap-3 (12px)
  - Page padding: pa-6

□ Grid alignment:
  - Cards in a grid: v-row with v-col cols="12" md="6" lg="4"
  - Stats row: v-col cols="6" md="3" (4 items per row on desktop)
  - Forms: 2-column on desktop (cols="12" md="6"), full-width on mobile

□ Max-width for readability:
  - Forms: max-width 800px centered
  - Content blocks: max-width 1200px
  - Full-width for tables and dashboards
```

### 2. Typography

```
□ Use design token typography:
  - Page title: text-xl font-bold text-text-primary-900
  - Section title: text-lg font-semibold
  - Body text: text-sm (default)
  - Labels: text-xs text-text-secondary-700
  - Values: text-sm font-medium text-text-primary-900

□ Line height for readability:
  - Body text: leading-relaxed (1.625)
  - Headings: leading-tight (1.25)
```

### 3. Cards & Containers

```vue
<!-- Standard card wrapper for content sections -->
<div
  class="bg-background-bg-primary rounded-xl border border-border-border-primary p-6"
>
  <!-- Section title -->
  <h3 class="text-lg font-semibold text-text-text-primary-900 mb-4">
    {{ $t('section.title') }}
  </h3>

  <!-- Content -->
  <slot />
</div>
```

```
□ Card border radius: rounded-xl (12px) for main cards, rounded-lg (8px) for nested
□ Card shadow: only for elevated cards (shadow-sm), most cards use border only
□ Card background: bg-background-bg-primary (white in light, dark in dark mode)
□ Nested containers: bg-background-bg-secondary for visual depth
```

### 4. Status Indicators

```
□ Use Badge component with correct type:
  - Active/Success: type="success"
  - Warning/Pending: type="warning"
  - Error/Rejected: type="error"
  - Info/New: type="info"
  - Default/Draft: type="default"

□ Badge with dot indicator for subtle status:
  <Badge :label="$t('status.active')" type="success" isDot />
```

### 5. Tables

```
□ Table headers: text-xs uppercase text-text-secondary-700
□ Table rows: hover state via Table component (automatic)
□ Actions column: use ActionMenu component (3-dot menu)
□ Status column: use Badge component
□ Date column: use formatDate utility
□ Empty state: meaningful message with $t()
```

### 6. Forms

```
□ Field grouping: related fields in same v-row
□ Required indicator: schema required prop (auto-adds asterisk)
□ Help text: schema description prop
□ Field width: full-width inputs (cols="12" md="6" for side-by-side)
□ Submit area: sticky bottom with border-top on long forms
□ Action buttons: primary on end, cancel/back on start
```

### 7. Empty & Loading States

```
□ Empty state pattern:
  <div class="flex flex-col items-center justify-center py-16 gap-4">
    <SvgIcon name="empty-state" size="64" />
    <p class="text-text-secondary-700 text-sm">{{ $t('no_data') }}</p>
    <Button type="primary" :title="$t('actions.create')" @click="create" />
  </div>

□ Loading state:
  - Tables: isLoading prop (shows built-in skeleton)
  - Pages: Loading.vue component
  - Buttons: isLoading prop (shows spinner, disables click)
```

### 8. Dark Mode Compatibility

```
□ All colors use design tokens (auto-switch in dark mode)
□ No hardcoded hex/rgb values
□ Shadows reduce in dark mode (handled by tokens)
□ Borders lighten in dark mode (handled by tokens)

□ If you must use a color:
  - Light: rgba(var(--v-theme-{token-name}), 1)
  - With opacity: rgba(var(--v-theme-{token-name}), 0.5)
```

## Quick Enhancement Recipes

### Stats Card Row

```vue
<v-row>
  <v-col v-for="stat in stats" :key="stat.key" cols="6" md="3">
    <div class="bg-background-bg-primary rounded-xl border border-border-border-primary p-4 flex flex-col gap-1">
      <span class="text-xs text-text-text-secondary-700">{{ $t(stat.label) }}</span>
      <span class="text-2xl font-bold text-text-text-primary-900">{{ stat.value }}</span>
    </div>
  </v-col>
</v-row>
```

### Section with Header + Action

```vue
<div class="flex items-center justify-between mb-4">
  <h3 class="text-lg font-semibold text-text-text-primary-900">{{ $t('section.title') }}</h3>
  <Button type="primary" size="sm" :title="$t('actions.add')" @click="add" />
</div>
```
