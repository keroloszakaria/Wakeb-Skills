# Frontend Codebase Index

The agent MUST consult this index before generating any code. If a component,
composable, or utility already exists here, USE IT — do not create a new one.

## Common Components

| Component         | Import Path                               | Purpose                                                                                                         |
| ----------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `Accordion`       | `@/components/common/Accordion.vue`       | Collapsible panel — props: `title`, `modelValue`, `classList`                                                   |
| `AddEditModal`    | `@/components/common/AddEditModal.vue`    | Form modal/drawer — props: `mode`, `title`, `schema`, `isView`, `isDataLoaded`                                  |
| `Badge`           | `@/components/common/Badge.vue`           | Status tag — props: `label`, `type` (primary/success/danger/warning/info/outline), `icon`, `isDot`              |
| `Button`          | `@/components/common/Button.vue`          | Action button — props: `title`, `icon`, `type` (primary/outline/danger/icon), `isLoading`, `disabled`, `action` |
| `Card`            | `@/components/common/Card.vue`            | Content card — props: `item`, `headers`, `isActive`, `showHeader`                                               |
| `CardView`        | `@/components/common/CardView.vue`        | Grid/list card container — props: `items`, `viewMode`, `columns`                                                |
| `Carousel`        | `@/components/common/Carousel.vue`        | Image slider — props: `slides`, `interval`, `cycle`                                                             |
| `CurrentDataTime` | `@/components/common/CurrentDataTime.vue` | Live date/time display                                                                                          |
| `Dropdown`        | `@/components/common/Dropdown.vue`        | Menu dropdown — props: `menuItems`, `title`, `icon`, `showSelected`                                             |
| `Kanban`          | `@/components/common/Kanban.vue`          | Kanban board — props: `board` (v-model), `canAddColumn`, `canAddCard`                                           |
| `Loading`         | `@/components/common/Loading.vue`         | Spinner overlay                                                                                                 |
| `Modal`           | `@/components/common/Modal.vue`           | Dialog wrapper — props: `dialog` (v-model), `width`, `title`, `persistent`, `isLoading`                         |
| `PageSkeleton`    | `@/components/common/PageSkeleton.vue`    | Loading skeleton — props: `config`, `direction`                                                                 |
| `Tabs`            | `@/components/common/Tabs.vue`            | Tab navigation — props: `tabs`, `selectedTab`, `type` (outlined/filled), `isVertical`                           |
| `ThemeToggle`     | `@/components/common/ThemeToggle.vue`     | Light/dark toggle                                                                                               |
| `TooltipBtn`      | `@/components/common/TooltipBtn.vue`      | Button with tooltip — props: `icon`, `path` (i18n key)                                                          |
| `Tree`            | `@/components/common/Tree.vue`            | Hierarchical tree — props: `nodes`, `itemKey`, `itemLabel`, `activeNode`                                        |
| `VerticalSteps`   | `@/components/common/VerticalSteps.vue`   | Progress steps — props: `steps` (with `is_active`, `title`, `time`)                                             |
| `TableAction`     | `@/components/common/TableAction.vue`     | Row action menu — props: `actions`, `item`, `isDots`, `isIconButtons`                                           |
| `SteamingHandle`  | `@/components/common/SteamingHandle.vue`  | HLS video player — props: `type`, `videoSrc`                                                                    |

### SvgIcon (Global — from jervis-icons plugin)

| Component | Import Path           | Purpose                                                                          |
| --------- | --------------------- | -------------------------------------------------------------------------------- |
| `SvgIcon` | (globally registered) | SVG icon — props: `name`, `size` (xs/sm/md/lg/xl), `class`. Use for inline icons |

> **Note:** SvgIcon is globally registered by the jervis-icons plugin. No import needed.
> For button icons, use Button's `icon` prop instead of SvgIcon directly.

## Table System

| Component             | Import Path                                         | Purpose                                                                               |
| --------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `Table`               | `@/components/common/Table/index.vue`               | Data table — props: `items`, `headers`, `selectedRows`, `showTrashed`, `globalSearch` |
| `TableFilter`         | `@/components/common/Table/TableFilter.vue`         | Filter form — props: `schema`, `emitOnChange`                                         |
| `AdvancedFilter`      | `@/components/common/Table/AdvancedFilter.vue`      | Complex filter builder                                                                |
| `HeaderCustomization` | `@/components/common/Table/HeaderCustomization.vue` | Column visibility editor                                                              |

## GenericForm System

| Component       | Import Path                                         | Purpose                                                                      |
| --------------- | --------------------------------------------------- | ---------------------------------------------------------------------------- |
| `GenericForm`   | `@/components/common/GenericForm/index.vue`         | Schema-driven form renderer — props: `schema`, `id`                          |
| `FieldRenderer` | `@/components/common/GenericForm/FieldRenderer.vue` | Individual field dispatcher — props: `field`, `modelValue`, `mode`, `isView` |

## Field Type Components

Used internally by GenericForm — do NOT import these directly.

`textInput`, `selectBox`, `numberInput`, `checkBox`, `switchbox`, `textArea`,
`dateTimeInput`, `passwordInput`, `phoneInput`, `colorPicker`, `radioButton`,
`combobox`, `autoComplete`, `imageUploader`, `fileInput`, `editor`, `otp`,
`customInput`, `customArea`

## Composables

| Composable             | Import Path                          | Returns                                                                                                                                                                                  |
| ---------------------- | ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useLookupPage`        | `@/composables/useLookupPage`        | Complete CRUD page: `STORE`, `headers`, `getItems`, `addRow`, `editRow`, `viewRow`, `deleteRow`, `deleteRows`, `schema`, `submitForm`, `isShowModal`, `isView`, `isCreate`, `modalTitle` |
| `useAlert`             | `@/composables/useAlert`             | `showAlert({title, text, type, position})` — SweetAlert2 toast                                                                                                                           |
| `useAppSettings`       | `@/composables/useAppSettings`       | `updateFavicon()`, `updateTitle()`                                                                                                                                                       |
| `useAutoTranslate`     | `@/composables/useAutoTranslate`     | `translate(text, from, to)` — Google Translate API                                                                                                                                       |
| `useCookies`           | `@/composables/useCookies`           | `getCookie`, `setCookie`, `removeCookie`, `getAllCookies`                                                                                                                                |
| `useDateTimeFormatter` | `@/composables/useDateTimeFormatter` | Locale-aware date/time formatting                                                                                                                                                        |
| `useFileUpload`        | `@/composables/useFileUpload`        | `errorMessage`, `uploadedFiles`, `previewUrls`, `validateFileSize`                                                                                                                       |
| `useIsDark`            | `@/composables/useIsDark`            | `Computed<boolean>` — current dark mode state                                                                                                                                            |
| `useLanguageSwitcher`  | `@/composables/useLanguageSwitcher`  | `locales`, `switchLanguage`, `loadLocales`                                                                                                                                               |
| `useLogo`              | `@/composables/useLogo`              | `appLogo`, `appLogoSmall` — theme-aware logos                                                                                                                                            |
| `useNumberConverter`   | `@/composables/useNumberConverter`   | Arabic digit converter (٠-٩)                                                                                                                                                             |
| `useResizableSidebar`  | `@/composables/useResizableSidebar`  | Draggable sidebar width resize                                                                                                                                                           |
| `useStorage`           | `@/composables/useStorage`           | `get(...keys)`, `set(key, value)`, `remove(...keys)` — localStorage wrapper                                                                                                              |
| `useTableActions`      | `@/composables/useTableActions`      | `baseActions`, `extraActions`, `filteredActions` — row action config                                                                                                                     |
| `useTeleport`          | `@/composables/useTeleport`          | Teleport mounting state                                                                                                                                                                  |
| `useTextTruncator`     | `@/composables/useTextTruncator`     | Text truncation with ellipsis                                                                                                                                                            |
| `useVariableHandler`   | `@/composables/useVariableHandler`   | Insert variables into text/editor fields                                                                                                                                                 |
| `useVoiceRecorder`     | `@/composables/useVoiceRecorder`     | Browser voice recording with waveform                                                                                                                                                    |

## Factories

| Factory               | Import Path                     | API                                                                                                                                           |
| --------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `BaseCrudFactory`     | `@/Factory/BaseCrudFactory`     | `useCrudFactory(endpoint, {cache, hooks})` → `{getAll, getRow, create, update, delete, toggleActive, forceDelete, rows, pagination, loading}` |
| `ModuleConfigFactory` | `@/Factory/ModuleConfigFactory` | `createModuleConfig({module, routeType, crudConfig, ...})` → module config object                                                             |
| `TableFactory`        | `@/Factory/TableFactory`        | `TableFactory({store, schema, moduleNameSingular, ...})` → modal/form CRUD management (wrapped by `useLookupPage`)                            |

## FieldUtils Creators

Import from `@/utils/FieldUtils`:

| Creator                  | Key Props                                                                    |
| ------------------------ | ---------------------------------------------------------------------------- |
| `createTextField`        | `key`, `label`, `translatable`, `isEmail`, `maxLength`, `rules`              |
| `createTextAreaField`    | `key`, `label`, `maxLength`, `allowVariables`                                |
| `createNumberField`      | `key`, `label`, `min`, `max`, `isIntegerOnly`, `stepSize`                    |
| `createPasswordField`    | `key`, `label`, `minLength`, `isHint`, `username`                            |
| `createSelectField`      | `key`, `label`, `options`, `itemTitle`, `itemValue`, `isMultiple`, `useType` |
| `createComboBoxField`    | `key`, `label`, `options`, `multiple`, `chips`                               |
| `createRadioButtonField` | `key`, `label`, `options`, `itemTitle`, `itemValue`                          |
| `createCheckBoxField`    | `key`, `label`, `trueValue`, `falseValue`, `description`                     |
| `createPhoneField`       | `key`, `phone_label`, `country_label`, `options`                             |
| `createDateTimeField`    | `key`, `label`, `type` (date/time/datetime), `viewFormat`                    |
| `createImageInput`       | `key`, `label`, `maxSize`, `isMultiple`, `max`, `uploadProgress`             |
| `createFileInput`        | `key`, `label`, `maxSize`, `isMultiple`, `accept`                            |
| `createEditorField`      | `key`, `label`, `direction`, `allowVariables`                                |
| `createColorField`       | `key`, `label`, `colors`                                                     |
| `createOtpInput`         | `key`, `label`, `length`                                                     |
| `createCaptchaField`     | `key`, `label`                                                               |
| `createMapField`         | `key`, `label`, `isMultiple`, `zoom`, `center`, `allowSearch`                |
| `createButton`           | `label`, `icon`, `click`, `disabled`                                         |

Helpers: `makeLabel(label)`, `makePlaceholder(label, placeholder, type)`, `makeRules(options)`, `createBaseField({...})`

## Utility Files

| Utility               | Import Path                   | Key Exports                                                                                                              |
| --------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `formDataHandler`     | `@/utils/formDataHandler`     | `handleFormData()`, `transformSchemaToObject()`, `updateSchemaValues()`, `handleErrors()`                                |
| `validationRules`     | `@/utils/validationRules`     | `generateValidationRules({required, minLength, maxLength, isEmail, ...})`                                                |
| `cloneDeep`           | `@/utils/cloneDeep`           | `cloneDeep()`, `cloneDeepReactive()`, `cloneDeepPreserveRefs()`                                                          |
| `bytesCalculator`     | `@/utils/bytesCalculator`     | `bytesToSize()`, `sizeToBytes()`, `BytesToText()`, `hexToRgb()`                                                          |
| `wordHelper`          | `@/utils/wordHelper`          | `toSingular()`, `capitalize()`, `toCamelCase()`, `toSnakeCase()`, `toKebabCase()`                                        |
| `sidebarHelper`       | `@/utils/sidebarHelper`       | `createSidebarLink()`, `createSidebarGroup()`, `createSidebarDivider()`, `createSidebarTitle()`, `createSidebarButton()` |
| `crypto`              | `@/utils/crypto`              | `encrypt()`, `decrypt()`, `decryptOpenSSL()`                                                                             |
| `encoders`            | `@/utils/encoders`            | `encodePassword(password, type)` — base64, hex, url                                                                      |
| `dynamicFieldFactory` | `@/utils/dynamicFieldFactory` | `buildSingleField()`, `buildFieldsArray()`                                                                               |
| `responseHandler`     | `@/utils/responseHandler`     | `responseToGenericForm(response)`                                                                                        |
| `getDisplayName`      | `@/utils/getDisplayName`      | `getDisplayName(item)`                                                                                                   |

## Global Stores

| Store        | Import Path           | Key State/Actions                                                  |
| ------------ | --------------------- | ------------------------------------------------------------------ |
| `app`        | `@/stores/app`        | `mode`, `drawer`, `drawerRail`, `openDrawer()`, `setActiveGroup()` |
| `breadCrumb` | `@/stores/breadCrumb` | `currentPage`, `breadCrumb`, `setBreadcrumbs()`                    |
| `enums`      | `@/stores/enums`      | Cached enum values via `BaseCrudFactory`                           |

## Themes

| Theme   | Location            | Key Files                                                                                                    |
| ------- | ------------------- | ------------------------------------------------------------------------------------------------------------ |
| `aware` | `src/themes/aware/` | `IndexPage.vue`, `AddEditPage.vue`, `Sidebar/`, `Table/`, `tokens.json`, `primitives.json`, `semantics.json` |
| `sar`   | `src/themes/sar/`   | Same structure as aware — SAR-specific overrides                                                             |

## Existing Modules

`auth`, `users`, `profile`, `roles`, `permissions`, `drones`, `flights`,
`events`, `tickets`, `notifications`, `statistics`, `reports`, `settings`,
`dataEntry`, `exportLog`

Each module follows: `config.js`, `router/`, `stores/`, `views/`, `schema/`, `locales/`, `components/`

## Plugins

| Plugin           | Purpose                                                 |
| ---------------- | ------------------------------------------------------- |
| `jervis-connect` | HTTP client with JWT auth, interceptors, locale headers |
| `jervis-icons`   | SVG icon registration                                   |
| `vuetify`        | Material Design UI framework + theme config             |
| `wakeb-clients`  | Client branding (logos, colors, contact info)           |
| `wakeb-font`     | Custom font loading                                     |
