# Skill: CI/CD Integration

Provides patterns for CI/CD pipelines, deployment checklists, and environment
configuration for Wakeb Starter projects.

## When to Use

- When setting up CI/CD for a new project
- When the user asks about deployment
- When configuring environment variables
- When creating GitHub Actions or similar workflows

## GitHub Actions Workflow

### Laravel Backend

```yaml
name: Backend CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_DATABASE: testing
          MYSQL_ROOT_PASSWORD: password
        ports: ["3306:3306"]
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: "8.2"
          extensions: mbstring, pdo_mysql
          coverage: xdebug

      - name: Install dependencies
        run: composer install --no-interaction --prefer-dist

      - name: Copy env
        run: cp .env.testing .env

      - name: Generate key
        run: php artisan key:generate

      - name: Run migrations
        run: php artisan migrate --force

      - name: Run tests
        run: php artisan test --coverage --min=80
```

### Vue Frontend

```yaml
name: Frontend CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run build
```

## Deployment Checklist

### Pre-Deploy (Backend)

```
□ All tests pass: php artisan test
□ No pending migrations: php artisan migrate:status
□ Environment variables set on server
□ Queue workers configured (Supervisor)
□ Scheduler cron entry added
□ Storage link created: php artisan storage:link
□ Cache cleared: php artisan config:clear; php artisan cache:clear
□ Config cached: php artisan config:cache
□ Routes cached: php artisan route:cache
```

### Pre-Deploy (Frontend)

```
□ Build succeeds: npm run build
□ No lint errors: npm run lint
□ Environment variables set (.env.production)
□ API base URL correct: VITE_API_BASE_URL
□ Source maps disabled in production
□ Bundle size checked: npx vite-bundle-visualizer
```

### Post-Deploy

```
□ Run migrations: php artisan migrate --force
□ Clear caches: php artisan optimize:clear
□ Rebuild caches: php artisan optimize
□ Restart queue workers: php artisan queue:restart
□ Verify health endpoint responds
□ Check error logs for first 15 minutes
```

## Environment Variables

### Backend (.env)

```env
# Required
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.domain.com

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=app_production
DB_USERNAME=app_user
DB_PASSWORD=  # from secrets manager

# Sanctum
SANCTUM_STATEFUL_DOMAINS=dashboard.domain.com
SESSION_DOMAIN=.domain.com

# Queue
QUEUE_CONNECTION=redis
REDIS_HOST=127.0.0.1

# Mail
MAIL_MAILER=smtp
MAIL_HOST=smtp.provider.com
```

### Frontend (.env.production)

```env
VITE_API_BASE_URL=https://api.domain.com/api
VITE_APP_NAME=Dashboard
VITE_APP_ENV=production
```

## Rules

```
□ Never commit .env files — use .env.example as template
□ APP_DEBUG=false in production — ALWAYS
□ Secrets from environment variables or secrets manager — never hardcoded
□ Run migrations with --force flag in production
□ Cache config/routes in production for performance
□ Queue workers supervised by Supervisor or systemd
□ HTTPS enforced — APP_URL starts with https://
□ CORS configured for specific domains — never wildcard in production
```
