# Skill: State Management Intelligence

Smart Pinia store generation and optimization using the Wakeb factory patterns.
Handles store creation, caching strategy, hooks, and custom extensions.

## When to Use

- Creating a new Pinia store for a module
- Optimizing store performance (caching, deduplication)
- Adding custom actions beyond standard CRUD
- Debugging store-related issues (stale data, race conditions)

## Store Creation

### Standard CRUD Store

```js
import { useCrudFactory } from "@/Factory/BaseCrudFactory";

export const useProductsStore = () => {
  return useCrudFactory("products", {
    cache: { enabled: true, ttl: 300000 },
  });
};
```

### Store with Hooks

```js
export const useOrdersStore = () => {
  return useCrudFactory("orders", {
    cache: { enabled: true },
    hooks: {
      beforeCreate: (data) => {
        // Transform data before sending to API
        data.total = calculateTotal(data.items);
        return data;
      },
      afterCreate: (response) => {
        // Side effects after creation
        useAlert().showAlert({ title: t("order_created"), type: "success" });
      },
      beforeGetAll: (params) => {
        // Add default filters
        if (!params.status) params.status = "active";
        return params;
      },
      afterGetAll: (response) => {
        // Post-process list data
      },
    },
  });
};
```

### Store with Custom Actions

```js
export const useReportsStore = () => {
  return useCrudFactory("reports", {
    cache: { enabled: false }, // Reports are always fresh
    extend: {
      exportPdf: async (id) => {
        const response = await httpRequest(`/reports/${id}/export`, {
          method: "GET",
          responseType: "blob",
        });
        return response;
      },
      getStatistics: async (filters) => {
        return await httpRequest("/reports/statistics", {
          method: "GET",
          params: filters,
        });
      },
    },
  });
};
```

## Caching Strategy

| Data Type        | Cache? | TTL   | Reason                            |
| ---------------- | ------ | ----- | --------------------------------- |
| Lookup/Enum data | ✅ Yes | 5 min | Rarely changes, fetched often     |
| CRUD list        | ✅ Yes | 5 min | Reduces API calls on navigation   |
| Detail view      | ✅ Yes | 2 min | May be stale after edit           |
| Reports          | ❌ No  | —     | Must always be fresh              |
| Real-time data   | ❌ No  | —     | Stale data is worse than no cache |
| Search results   | ❌ No  | —     | Depends on query, can't cache     |

## Anti-Patterns

```
❌ Creating stores without useCrudFactory:
   const items = ref([])
   const loading = ref(false)
   const getItems = async () => { loading.value = true; ... }
   → This duplicates what the factory provides

❌ Calling API directly from components:
   const response = await axios.get('/api/products')
   → Use the store: STORE.getAll(params)

❌ Mutating store state directly:
   STORE.items.value.push(newItem)
   → Use store methods: STORE.createRow(data)

❌ Multiple stores for the same endpoint:
   useProductsStore() in IndexView AND useProductsListStore() in AddEditView
   → Share the same store instance
```

## Debugging

| Symptom                  | Cause                 | Fix                                  |
| ------------------------ | --------------------- | ------------------------------------ |
| Data stale after edit    | Cache not invalidated | Call getItems() after submitForm()   |
| Loading never stops      | Promise not resolved  | Check API endpoint returns correctly |
| Items duplicated         | Multiple getAll calls | Check useLookupPage wiring           |
| Store empty on page load | Async timing issue    | Use await useLookupPage()            |
