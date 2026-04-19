# Factory Patterns Reference

## Contents

- CRUD Factory (BaseCrudFactory)
- Table Factory (TableFactory)

---

## CRUD Factory

```js
import { useCrudFactory } from "@/Factory/BaseCrudFactory";

const store = useCrudFactory("endpoint-name", {
  cache: { enabled: true, ttl: 300000 },
  hooks: {
    beforeCreate: (data) => {
      /* transform */ return data;
    },
    afterCreate: (response) => {
      /* post-create */
    },
    beforeGetAll: (params) => {
      /* modify params */
    },
    afterGetAll: (response) => {
      /* process response */
    },
  },
  extend: {
    customAction: async (id) => {
      /* custom endpoint */
    },
  },
  isFile: false, // true for multipart/form-data
  queryParams: {}, // default query parameters
});
```

### Store return value (BaseCrudReturn)

```
Refs: items, selectedItem, pagination, loading, error
Methods: getAll(params), getRow(id), createRow(data), updateRow(data), deleteRow(id|[ids])
```

---

## Table Factory

```js
import { TableFactory } from "@/Factory/TableFactory";

const {
  isView,
  isShowModal,
  isCreate,
  schema,
  modalTitle,
  addEditLoading,
  getItems,
  handleModal,
  addRow,
  editRow,
  viewRow,
  deleteRow,
  deleteRows,
  toggleActiveRow,
  submitForm,
} = TableFactory({
  store,
  schema: useModuleSchema,
  moduleNameSingular: "item",
  fetchDetailedItem: true,
  steps: null, // StepDefinition[] for stepper forms
});
```

> **Note**: Prefer `useLookupPage` over manual `TableFactory` usage. `useLookupPage` wraps TableFactory and adds store creation, enum/model prefetching, breadcrumbs, and schema loading automatically.
