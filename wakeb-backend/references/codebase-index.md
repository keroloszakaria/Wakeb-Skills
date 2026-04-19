# Backend Codebase Index

The agent MUST consult this index before generating any code. If a controller,
filter, trait, service, or rule already exists here, USE IT — do not create a new one.

## Controllers

| Controller                  | Location                    | Methods                                                           |
| --------------------------- | --------------------------- | ----------------------------------------------------------------- |
| `BaseController`            | `app/Http/Controllers/API/` | Constructor (guard/userModel setup)                               |
| `LoginController`           | `Auth/`                     | `__invoke(LoginRequest)`                                          |
| `LogoutController`          | `Auth/`                     | `__invoke(Request)`                                               |
| `OTPController`             | `Auth/`                     | `send(SendOtpRequest)`, `verify(VerifyOtpRequest)`                |
| `ResetPasswordController`   | `Auth/`                     | `__invoke(ResetPasswordRequest)`                                  |
| `UserController`            | `User/`                     | `index`, `store`, `show`, `update` + delete/restore/toggle traits |
| `RoleController`            | `User/`                     | `index`, `store`, `show`, `update` + delete/restore/toggle traits |
| `PermissionController`      | `User/`                     | `index`, `show`, `store`, `update`, `destroy`                     |
| `CountryController`         | `DataEntry/`                | `index`, `store`, `show`, `update` + delete/restore/toggle traits |
| `ProfileController`         | `Profile/`                  | `user()`, `updateProfile()`, `destroyAvatar()`                    |
| `SettingController`         | `Global/Setting/`           | `index()`, `update(SettingRequest)`                               |
| `TestCredentialsController` | `Global/Setting/`           | `testEmail(TestCredentialsRequest)`                               |
| `ActivityLogController`     | `Global/ActivityLog/`       | `index(Request)`, `show(Activity)`                                |
| `NotificationController`    | `Global/Notification/`      | `index()`, `update(NotificationRequest)`                          |
| `ExportController`          | `Global/Export/`            | `__invoke(ExportRequest)`                                         |
| `ReportController`          | `Global/Report/`            | `__invoke(ReportRequest)`                                         |
| `HelpController`            | `Global/Help/`              | `models()`, `enums()`, `configs()`                                |
| `CaptchaController`         | `Global/Captcha/`           | `generateCaptcha()`, `verifyCaptcha()`                            |
| `ChunkFileController`       | `Global/Chunk/`             | `__invoke(ChunkFileRequest)`                                      |

## Models

| Model                 | Traits                                                                                                                                                                               | Key Fields                                                          |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| `BaseModel`           | —                                                                                                                                                                                    | Base class for all models                                           |
| `User`                | `SoftDeletes`, `AuthenticatesWithLdap`, `UserScopes`, `ApplyNotification`, `CreatedByObserver`, `Notifiable`, `HasApiTokens`, `HasRoles`, `LogsActivityOptions`, `HasModelRelations` | name, email, phone, avatar, gender, password, is_active, created_by |
| `Role`                | `RoleScopes`, `CreatedByObserver`, `HasTranslations`                                                                                                                                 | name, guard_name, display_name, is_active, created_by               |
| `Permission`          | `HasTranslations`                                                                                                                                                                    | name, guard_name, display_name, group                               |
| `Country`             | `HasTranslations`, `SoftDeletes`                                                                                                                                                     | name, nationality, flag, code, phone_code, is_active                |
| `Setting`             | `HasTranslations`                                                                                                                                                                    | key, value, group, type, label, is_multi_lang, is_env               |
| `Notification`        | —                                                                                                                                                                                    | notifiable_type, notifiable_id, data, open_at, read_at              |
| `PersonalAccessToken` | —                                                                                                                                                                                    | name, token, abilities, expires_at, meta                            |

## Form Requests

| Request                | Location             | Validates                                                                              |
| ---------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| `BaseFormRequest`      | `app/Http/Requests/` | Base: empty→null, boolean normalization, JSON errors                                   |
| `PageRequest`          | `Global/`            | page, per_page, search, sort_column, sort_direction, start, end, is_active, is_trashed |
| `ModelBatchRequest`    | `Global/`            | ids, action (delete/restore/force-delete)                                              |
| `UserRequest`          | `User/`              | name, email, phone, password (StrongPassword), gender, roles, permissions              |
| `RoleRequest`          | `User/`              | name, display_name (translatable), permissions (min:1), guard_name                     |
| `PermissionRequest`    | `User/`              | name, display_name, group                                                              |
| `CountryRequest`       | `DataEntry/`         | name (translatable), code, phone_code, phone_length, flag                              |
| `LoginRequest`         | `Auth/`              | email, password, OTP if enabled, meta                                                  |
| `SendOtpRequest`       | `Auth/`              | email, OTP type                                                                        |
| `VerifyOtpRequest`     | `Auth/`              | email, OTP                                                                             |
| `ResetPasswordRequest` | `Auth/`              | email, password, OTP                                                                   |
| `UpdateProfileRequest` | `Profile/`           | name, email, phone, password, avatar                                                   |
| `SettingRequest`       | `Global/`            | settings (array of key=>value)                                                         |
| `ExportRequest`        | `Global/`            | page, start, end, prefer_chart                                                         |
| `ReportRequest`        | `Global/`            | page, start, end, prefer_chart, related_type                                           |
| `NotificationRequest`  | `Global/`            | action (open/read), ids                                                                |

## API Resources

| Resource               | Transforms                             |
| ---------------------- | -------------------------------------- |
| `LoginResource`        | User + API token                       |
| `SessionResource`      | Active token/session data              |
| `UserResource`         | User with roles, permissions, phone    |
| `RoleResource`         | Role with permissions, user count      |
| `PermissionResource`   | Permission with group                  |
| `CountryResource`      | Country with translated name, flag URL |
| `ActivityLogResource`  | Activity with model info, changes      |
| `NotificationResource` | Notification with open/read status     |
| `SettingResource`      | Setting with value, type, group        |
| `SettingGroupResource` | Settings nested by group               |
| `BasicResource`        | Basic resource transformation          |
| `BasicUserResource`    | Minimal user data                      |

## Pipeline Filters

| Filter                  | Query Param                                                       | Purpose                                    |
| ----------------------- | ----------------------------------------------------------------- | ------------------------------------------ |
| `ActiveFilter`          | `is_active`                                                       | Filter by active status                    |
| `DateFilter`            | `start`, `end`                                                    | Filter by created_at range                 |
| `EmailFilter`           | `email`                                                           | Email search                               |
| `NameFilter`            | `search`                                                          | Plain name search                          |
| `JsonNameFilter`        | `search`                                                          | JSON name field search (multi-lang)        |
| `JsonDisplayNameFilter` | `search`                                                          | JSON display_name search                   |
| `PhoneFilter`           | `phone`                                                           | Phone search                               |
| `OrderByFilter`         | `sort_column`, `sort_direction`                                   | Column ordering                            |
| `TrashedFilter`         | `is_trashed`                                                      | Include soft-deleted                       |
| `UserFilter`            | `search`                                                          | Multi-field user search (name+email+phone) |
| `ActivityLogFilter`     | `search`, `model`, `user_id`, `operation`, `date_from`, `date_to` | Complex activity log filtering             |
| `KeyFilter`             | `key`                                                             | Setting key search                         |
| `GroupFilter`           | `group`                                                           | Setting group search                       |

## Custom Validation Rules

| Rule                   | Validates                                                                    |
| ---------------------- | ---------------------------------------------------------------------------- |
| `StrongPassword`       | Min 8, uppercase, lowercase, digit, special, no repeats/sequences/dictionary |
| `UniqueCheck`          | Model uniqueness — throws `ModelAlreadyExistsException`                      |
| `ValidLength`          | Field length against reference model's length column                         |
| `CheckSamePassword`    | New password differs from current                                            |
| `TranslatableRequired` | Required translations per `config/project.php` languages                     |
| `TranslatableNullable` | Optional translations per config languages                                   |
| `TotalFileSize`        | Total upload size limit in MB                                                |

## Traits

| Trait                    | Methods                                                                                                               | Purpose                                             |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `HasDeleteMethods`       | `setDeleteModel()`, `setDeleteGuards()`, `beforeDelete()`, `afterDelete()`, `destroy()`, `forceDelete()`, `restore()` | Bulk delete/restore with guards and lifecycle hooks |
| `HasToggleActiveMethods` | `setToggleModel()`, `setToggleGuards()`, `beforeToggle()`, `afterToggle()`, `toggleActive()`                          | Bulk toggle is_active with guards                   |
| `ApplyNotification`      | `sendNotification(data, types)`                                                                                       | Send notifications to model                         |
| `CreatedByObserver`      | `bootCreatedByObserver()`                                                                                             | Auto-set created_by from auth user                  |
| `HasOrder`               | `changeOrder()`, `updateOrderField()`                                                                                 | Model ordering/sorting                              |
| `LogsActivityOptions`    | `getActivitylogOptions()`                                                                                             | Configure Spatie activity logging                   |

## Services

| Service                | Methods                                                                               | Purpose                     |
| ---------------------- | ------------------------------------------------------------------------------------- | --------------------------- |
| `LoginService`         | `attempt()`, `attemptDefaultLogin()`, `attemptLdapLogin()`                            | Authentication              |
| `OTPService`           | `send()`, `verify()`, `check()`, `generateOtp()`                                      | OTP generation/validation   |
| `ResetPasswordService` | `reset()`                                                                             | Password reset via OTP      |
| `ThrottleService`      | `ensureIsNotRateLimited()`, `incrementRateLimit()`, `clearRateLimit()`                | Rate limiting               |
| `SettingService`       | `all()`, `get()`, `set()`, `flushCache()`                                             | Settings management         |
| `NotificationService`  | `resolve()`, `sendNotify()`, `sendEmail()`, `sendRealtimeNotification()`, `sendSMS()` | Multi-channel notifications |
| `EncryptionService`    | `encrypt()`, `decrypt()`                                                              | AES-256-CBC encryption      |
| `QueryHelper`          | `applyJsonSearch()`                                                                   | JSON field searching        |

## Helper Functions (App.php)

**Response:** `successResponse(data, msg, code)`, `failResponse(msg, data, code)`, `abort403(condition)`, `unKnownError(message)`

**Resolve:** `resolveTrans(trans, page)`, `resolveBool(item)`, `resolvePhoto(image, type)`, `resolveArray(array)`, `resolveModel(name, module)`, `resolveClass(path)`, `resolveEmptyLang(trans)`, `resolveEmptyToNull(value)`

**Model:** `getModelKey(className)`, `detectModelPath(model)`, `getModelTranslatable(model)`, `shouldVerifyOtp()`, `brandName()`

**Pagination:** `wrapPaginate(query, resource)`

## Enums

| Enum                      | Type   | Cases                                                           |
| ------------------------- | ------ | --------------------------------------------------------------- |
| `ActiveTypeEnum`          | int    | Active=1, InActive=0                                            |
| `OtpTypeEnum`             | string | Login, ResetPassword, VerifyEmail                               |
| `UserGenderEnum`          | string | Male, Female                                                    |
| `SettingTypeEnum`         | string | Text, TextArea, ImageUploader, File, CheckBox, Radio, SwitchBox |
| `ReportChartTypeEnum`     | string | HighChart                                                       |
| `ReportPageTypeEnum`      | string | User                                                            |
| `NotificationChannelEnum` | string | Email, SMS, Realtime, Notify                                    |

## Policies

| Policy       | Methods                                                                        |
| ------------ | ------------------------------------------------------------------------------ |
| `UserPolicy` | `view`, `create`, `update`, `delete`, `restore`, `forceDelete`, `toggleActive` |
| `RolePolicy` | `view`, `create`, `update`, `delete`, `toggleActive`                           |

## Events & Jobs

| Class               | Purpose                     |
| ------------------- | --------------------------- |
| `NotificationEvent` | Broadcasts to user channels |
| `SendEmailJob`      | Queued email notification   |
| `SendSmsJob`        | Queued SMS notification     |

## Scopes

| Scope        | Methods                                                                 |
| ------------ | ----------------------------------------------------------------------- |
| `UserScopes` | `related()`, `excludeLoggedInUser()`, `excludeRoot()`, `withRole(role)` |
| `RoleScopes` | `related()`, `excludeRoot()`, `excludeLoggedInRole()`                   |

## Installed Modules (nwidart)

| Module         | Key Models                                                                                                     |
| -------------- | -------------------------------------------------------------------------------------------------------------- |
| `Form`         | Form, FormField, FormStep, FormSubmission                                                                      |
| `Notification` | Channel, NotificationEvent, NotificationLog, NotificationTemplate, ScheduleEvent, SystemNotification, Variable |
