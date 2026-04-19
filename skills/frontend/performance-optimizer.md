# Skill: Performance Optimizer

Identifies and fixes performance bottlenecks in Vue 3 dashboard code.
Applies project-specific optimizations using the Wakeb Starter patterns.

## When to Use

- When generating new pages or components (preventive)
- When the user reports slow pages or laggy UI
- When reviewing code that handles large datasets
- When optimizing bundle size or load times

## Optimization Checklist

### Route-Level

```
□ Lazy load ALL route components:
  component: () => import('../views/IndexView.vue')   ✅
  component: IndexView                                 ❌

□ Prefetch critical routes:
  /* webpackPrefetch: true */ in dynamic imports for frequently accessed pages

□ Route-level code splitting — each module is a separate chunk
```

### Component-Level

```
□ Heavy components use defineAsyncComponent:
  const Chart = defineAsyncComponent(() => import('./Chart.vue'))

□ Conditional rendering with v-if (not v-show) for heavy sections:
  v-if removes from DOM = no render cost
  v-show hides with CSS = still renders

□ Key attribute on v-for items:
  <li v-for="item in items" :key="item.id">   ✅
  <li v-for="item in items">                   ❌

□ Avoid inline object/array creation in templates:
  :style="{ color: tokenVar }"                 ✅ (simple)
  :style="computedStyle"                       ✅ (complex — use computed)
  :style="{ ...baseStyle, ...overrides }"      ❌ (creates new object every render)
```

### Reactivity

```
□ Use computed for derived state (cached):
  const fullName = computed(() => `${first.value} ${last.value}`)    ✅
  const getFullName = () => `${first.value} ${last.value}`           ❌

□ Use shallowRef for large objects that replace entirely:
  const items = shallowRef([])    ✅ for large lists from API
  const items = ref([])           OK for small reactive lists

□ Avoid deep watchers on large objects:
  watch(store.items, handler, { deep: true })   ❌ expensive
  watch(() => store.items.length, handler)      ✅ targeted
```

### Data Fetching

```
□ useCrudFactory cache for repeated fetches:
  useCrudFactory('endpoint', { cache: { enabled: true, ttl: 300000 } })

□ Debounce search inputs (300ms minimum):
  GenericForm with emitOnChange handles this automatically

□ Paginate ALL list views — never load unbounded data:
  useLookupPage handles pagination automatically

□ Cancel previous requests on new search:
  useCrudFactory handles abort signals internally
```

### Rendering Large Lists

```
□ Virtual scrolling for lists > 100 items:
  Use Vuetify's virtual-scroller component

□ Pagination for tables > 50 rows:
  Table component handles this via pagination prop

□ Infinite scroll for feeds:
  Use intersection observer + paginated API calls
```

### Bundle Size

```
□ Tree-shake icon imports:
  import { specificIcon } from 'jervis-icons'    ✅
  import * as icons from 'jervis-icons'          ❌

□ Dynamic imports for heavy libraries:
  const lib = await import('heavy-library')      ✅
  import lib from 'heavy-library'                ❌ (if used conditionally)

□ Analyze bundle:
  npx vite-bundle-visualizer
```

## Performance Patterns

### Optimized Table Page

```vue
<script setup>
// ✅ useLookupPage handles pagination, caching, debounced search
const { STORE, headers, getItems, ... } = await useLookupPage({
  itemKey: config.module,
  structureType: 'module',
  headersConfig: config.headers,
})
// Pagination, search debouncing, and cache invalidation are automatic
</script>
```

### Optimized Heavy Component

```vue
<script setup>
// ✅ Async component — loaded only when needed
const HeavyChart = defineAsyncComponent(
  () => import("@/components/common/Chart.vue"),
);

// ✅ Computed instead of method
const chartData = computed(() =>
  STORE.items.value.map((item) => ({ x: item.date, y: item.value })),
);
</script>

<template>
  <!-- ✅ Conditional render — chart only loaded when data exists -->
  <Suspense v-if="chartData.length">
    <HeavyChart :data="chartData" />
  </Suspense>
</template>
```
