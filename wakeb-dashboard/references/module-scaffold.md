# Module Scaffolding — Complete Templates

## Contents

- Module Config (config.js)
- Router (router/index.js)
- Store (stores/{moduleName}.js)
- Schema (schema/index.js)
- Index View (views/index.vue)
- Add/Edit View (views/AddEditView.vue)
- Locales (locales/ar.json, en.json)

---

## Module Config (`config.js`)

```js
import { createModuleConfig } from "@/Factory/ModuleConfigFactory.ts";

export default createModuleConfig({
  module: "{moduleName}",
  enums: [
    /* { name: 'enum.key' } */
  ],
  models: [
    /* { name: 'modelName', extra: ['field1'] } */
  ],
  sidebar: { order: { n }, icon: "{icon-name}" },
  headers: [
    { title: "index", key: "id" },
    // ... column definitions
    { title: "actions", key: "actions", align: "center" },
  ],
  breadcrumbs: [
    { title: "dashboard", disabled: false, to: "/" },
    { title: "{moduleName}", disabled: true, to: "" },
  ],
  crud: { mode: "modal", viewMode: "table" },
  isDots: true,
});
```

---

## Router (`router/index.js`)

```js
import config from "../config";

export default [
  {
    path: "/{kebab-module-name}",
    name: "{moduleName}",
    component: () => import("../views/IndexView.vue"),
    meta: {
      title: "{moduleName}",
      permissions: ["{moduleName}.index"],
      ...config,
    },
  },
  {
    path: "/{kebab-module-name}/create",
    name: "{moduleName}.create",
    component: () => import("../views/AddEditView.vue"),
    meta: {
      title: "{moduleName}.create",
      permissions: ["{moduleName}.create"],
      ...config,
    },
  },
  {
    path: "/{kebab-module-name}/:id/edit",
    name: "{moduleName}.edit",
    component: () => import("../views/AddEditView.vue"),
    meta: {
      title: "{moduleName}.edit",
      permissions: ["{moduleName}.update"],
      ...config,
    },
  },
];
```

---

## Store (`stores/{moduleName}.js`)

```js
import { useCrudFactory } from '@/Factory/BaseCrudFactory'

export const use{ModuleName}Store = () => {
  return useCrudFactory('{kebab-module-name}', {
    cache: { enabled: true },
    // hooks, extend, etc.
  })
}
```

---

## Schema (`schema/index.js`)

```js
import {
  createTextField,
  createSelectField,
  createTextAreaField
} from '@/utils/FieldUtils'
import i18n from '@/utils/i18n'
import { ref } from 'vue'

export const use{ModuleName}Schema = ({ isCreate = false, isView = false }) => {
  const t = i18n.global.t
  const inputStyle = isView ? 'viewMode' : 'flat'

  const schema = ref([
    createTextField({
      key: 'name',
      label: 'name',
      inputStyle,
      cols: { md: 6, lg: 6 },
      value: ''
    }),
    // ... more fields
  ])

  return schema
}
```

---

## Index View (`views/index.vue`)

ALWAYS use `useLookupPage` — it wires up store, schema, table factory, enums, models, breadcrumbs, and all CRUD actions automatically.

```vue
<template>
  <IndexPage
    v-model="isShowModal"
    :moduleName="moduleName"
    :moduleNameSingular="moduleNameSingular"
    :headers="headers"
    :items="STORE.items"
    :pagination="STORE.pagination"
    :isLoading="STORE.loading.fetching"
    :schema="schema"
    :showSelect="config.showSelect"
    :showTrashed="config.showTrashed"
    :showInactive="config.showInactive"
    :switchType="config.switchType"
    :modalTitle="modalTitle"
    :isView="isView"
    :skeleton="config.skeleton"
    :isDots="config.isDots"
    :isCreate="isCreate"
    :modalWidth="config.crud.modalWidth"
    :addMode="config.crud.mode"
    :viewMode="config.crud.viewMode"
    @searchValue="getItems"
    @openModal="addRow"
    @isTrashed="getItems"
    @onUpdateOptions="getItems"
    @editRow="editRow"
    @viewRow="viewRow"
    @deleteRow="deleteRow"
    @deleteRows="deleteRows"
    @addEditRow="submitForm"
  />
</template>

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
  fetchDetailedItem: config.fetchDetailedItems,
  enums: config.enums,
  models: config.models,
  viewMode: config.crud.viewMode,
  headersConfig: config.headers,
});
</script>
```

---

## Add/Edit View (`views/AddEditView.vue`)

```vue
<template>
  <AddEditPage
    :title="pageTitle"
    :isCreate="isCreate"
    :isView="isView"
    @back="goBack"
  >
    <GenericForm ref="formRef" :schema="schema" @submit="submitForm" />
  </AddEditPage>
</template>

<script setup>
import { useLookupPage } from "@/composables/useLookupPage";
import config from "../config";

const AddEditPage = useThemedComponent("AddEditPage");

const { STORE, schema, isCreate, isView, submitForm } = await useLookupPage({
  itemKey: config.module,
  structureType: "module",
  pageType: route.params.id ? "edit" : "add",
  fetchDetailedItem: true,
});

const goBack = () => router.back();
</script>
```

---

## Locales

**`locales/ar.json`:**

```json
{
  "{moduleName}": {
    "title": "...",
    "fields": { "name": "...", "status": "..." },
    "messages": { "created": "...", "updated": "...", "deleted": "..." }
  }
}
```

**`locales/en.json`:** Same structure, English values.
