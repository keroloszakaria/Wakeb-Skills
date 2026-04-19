# Skill: Accessibility + UX Validator

Validates generated UI code for accessibility (WCAG 2.1 AA) and UX quality,
adapted to the Wakeb Starter component library and Arabic-first RTL layout.

## When to Use

- After generating any UI code (automatic check)
- When the user asks about accessibility
- When reviewing existing pages for compliance
- When converting Figma designs (check design gaps)

## Accessibility Checklist

### Keyboard Navigation

```
□ All interactive elements are keyboard accessible:
  - Button.vue handles keyboard events automatically
  - Modal.vue traps focus automatically
  - Table/index.vue supports keyboard navigation
  - GenericForm handles tab order automatically

□ Focus indicators visible:
  - Project components include focus styles via design tokens
  - Custom components must add: outline: 2px solid rgba(var(--v-theme-background-bg-brand-solid), 1)

□ Escape key closes modals:
  - Modal.vue handles this automatically
  - Custom overlays must add @keydown.esc handler

□ Tab order follows visual order:
  - Use natural DOM order — avoid tabindex > 0
  - tabindex="0" for custom interactive elements
  - tabindex="-1" for programmatically focused elements
```

### Screen Readers

```
□ Images have alt text:
  - Decorative images: alt=""
  - Informational images: alt="descriptive text" (translated with $t())

□ Icons have labels:
  - SvgIcon with meaning: add aria-label or adjacent text
  - Decorative icons: aria-hidden="true" (SvgIcon handles this)
  - Button with icon only: Button's title prop provides the label

□ Form fields have labels:
  - GenericForm handles labels automatically via schema label prop
  - Custom inputs: <label :for="fieldId">{{ $t('label') }}</label>

□ Dynamic content announced:
  - Status changes: use aria-live="polite"
  - Error messages: use role="alert"
  - Loading states: use aria-busy="true"

□ Headings hierarchy:
  - One h1 per page (page title)
  - Logical nesting: h2 > h3 > h4
  - No skipped levels
```

### Color & Contrast

```
□ Text contrast ratio ≥ 4.5:1 (AA):
  - text-primary-900 on bg-primary: ✅ project tokens meet this
  - text-secondary-700 on bg-primary: ✅ tested

□ Interactive element contrast ≥ 3:1:
  - Button types (primary, secondary, outline): ✅ pre-tested
  - Badge types: ✅ pre-tested

□ Color is not the only indicator:
  - Status Badge uses isDot + label text (not just color)
  - Error states show text message (not just red border)
  - Required fields show asterisk (not just red label)

□ Dark mode contrast maintained:
  - Design tokens auto-switch — if tokens are used, dark mode works
  - Hardcoded colors will break in dark mode
```

### RTL-Specific Accessibility

```
□ Reading direction set:
  - HTML dir="rtl" is set globally for Arabic
  - lang="ar" on HTML element

□ Directional content adapts:
  - Arrows/chevrons flip: ← means "forward" in RTL
  - Progress bars fill from right
  - Breadcrumbs flow from right

□ Layout mirrors correctly:
  - Sidebar on the right in RTL
  - Form labels align to the right
  - Table columns maintain logical order

□ Bidirectional text handled:
  - Mix of Arabic + English: use dir="auto" on user-content containers
  - Numbers in Arabic text: browsers handle automatically
```

## UX Validation

### Loading States

```
□ Every async operation shows loading:
  - Table: isLoading prop from STORE.loading.fetching
  - Button: isLoading prop during form submission
  - Page: Loading.vue or PageSkeleton for initial load

□ Skeleton screens for initial page load:
  - Use PageSkeleton with config matching the actual layout
  - Never show a blank page while loading
```

### Error States

```
□ Form validation shows inline errors:
  - GenericForm handles 422 errors via handleErrors()
  - Errors appear under the specific field

□ Empty states have messaging:
  - Table shows emptyMessage when no data
  - Use meaningful messages: $t('no_data') not just "—"

□ Network errors show user-friendly messages:
  - jervis-connect interceptors handle 401/403/404/500
  - Custom errors use useAlert() with translated text
```

### Responsive Behavior

```
□ Mobile-first responsive design:
  - Use v-col with md/lg breakpoints: cols="12" md="6" lg="4"
  - Stack on mobile, side-by-side on desktop

□ Touch targets ≥ 44x44px on mobile:
  - Button sizes: sm=32px, md=40px, lg=48px
  - Use md or lg size for primary actions

□ No horizontal scroll on mobile:
  - Tables scroll internally (Table component handles this)
  - Forms stack fields: cols="12" on mobile
```
