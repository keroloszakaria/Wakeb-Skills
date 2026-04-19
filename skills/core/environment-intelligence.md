# Skill: Environment Intelligence

Detects and adapts to the development environment — local, staging, production.
Ensures environment-specific code is handled correctly and no secrets or
dev-only code leaks into production.

## When to Use

- When generating environment-dependent code (API URLs, feature flags)
- When debugging environment-related issues
- When setting up new environments or deployment configs

## Environment Detection

### Front-End

```
Environment Variables (Vite):
  VITE_SERVER_URL     → API base URL
  VITE_CLIENT         → Client branding (wakeb, aware, sar)
  VITE_FONT           → Font family (ibm-plex-sans-arabic)
  VITE_LOCALE         → Default locale (ar)

Access pattern:
  import.meta.env.VITE_SERVER_URL    ✅
  process.env.VUE_APP_*              ❌ (not Vite)
```

### Back-End

```
Environment Files:
  .env                → Active environment
  .env.example        → Template (commit this)
  .env.testing        → Test environment

Key Variables:
  APP_ENV             → local | staging | production
  APP_DEBUG           → true (local) | false (production)
  APP_URL             → Backend URL
  FRONTEND_URL        → Dashboard URL (CORS)
  DB_CONNECTION       → mysql | pgsql
  SANCTUM_STATEFUL_DOMAINS → Cookie auth domains
```

## Rules

### Never Do

```
□ Never hardcode API URLs — use environment variables
□ Never commit .env files — only .env.example
□ Never use APP_DEBUG=true in production
□ Never expose stack traces in API responses (failResponse handles this)
□ Never hardcode database credentials
□ Never use dd() or dump() in committed code
□ Never disable CSRF or CORS in production
```

### Environment-Specific Patterns

| Pattern          | Local             | Staging            | Production         |
| ---------------- | ----------------- | ------------------ | ------------------ |
| API Error Detail | Full stack trace  | Error message only | Generic message    |
| Logging Level    | debug             | info               | error              |
| Cache            | Disabled or short | Enabled            | Enabled + long TTL |
| Queue Driver     | sync              | redis              | redis              |
| Mail Driver      | log               | mailtrap           | smtp               |
| Debug Bar        | Enabled           | Disabled           | Disabled           |

### Config Patterns

**FE — API base URL:**

```js
// ✅ Correct — uses env variable
const api = httpRequest(`${import.meta.env.VITE_SERVER_URL}/endpoint`);

// ❌ Wrong — hardcoded URL
const api = httpRequest("https://api.example.com/endpoint");
```

**BE — Feature flags:**

```php
// ✅ Correct — uses config
if (config('project.features.notifications')) { ... }

// ❌ Wrong — hardcoded check
if (app()->environment('production')) { ... }
```

**BE — Sensitive operations:**

```php
// ✅ Correct — env-aware logging
Log::channel(app()->isProduction() ? 'stack' : 'single')->error($e->getMessage())

// ❌ Wrong — always logs everything
Log::debug($request->all()) // leaks data in production
```
