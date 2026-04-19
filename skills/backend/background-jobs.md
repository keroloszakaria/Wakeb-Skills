# Skill: Background Jobs + Scheduler

Generates Laravel queued jobs, notifications, and scheduled tasks following
Wakeb Starter conventions.

## When to Use

- When operations take > 2 seconds (move to queue)
- When sending notifications (email, SMS, push)
- When the user needs scheduled/cron tasks
- When processing file imports/exports

## Queued Job Pattern

```php
<?php

namespace Modules\{ModuleName}\app\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

class Process{Action}Job implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $backoff = 60;  // seconds between retries
    public int $timeout = 120; // max execution time

    public function __construct(
        public readonly int $modelId,
        public readonly array $data = [],
    ) {}

    public function handle(): void
    {
        $model = Model::findOrFail($this->modelId);

        try {
            // Process logic here
            Log::info("Job completed", ['model_id' => $this->modelId]);
        } catch (\Exception $e) {
            Log::error("Job failed", [
                'model_id' => $this->modelId,
                'error' => $e->getMessage(),
            ]);
            throw $e; // Re-throw to trigger retry
        }
    }

    public function failed(\Throwable $exception): void
    {
        Log::critical("Job permanently failed", [
            'model_id' => $this->modelId,
            'error' => $exception->getMessage(),
        ]);
        // Notify admin or update status
    }
}
```

### Dispatching Jobs

```php
// From controller
Process{Action}Job::dispatch($model->id);

// With delay
Process{Action}Job::dispatch($model->id)->delay(now()->addMinutes(5));

// On specific queue
Process{Action}Job::dispatch($model->id)->onQueue('exports');

// Chain jobs
Bus::chain([
    new ValidateDataJob($importId),
    new ProcessDataJob($importId),
    new NotifyCompleteJob($importId),
])->dispatch();
```

## Notification Pattern

```php
<?php

namespace Modules\{ModuleName}\app\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;

class {Action}Notification extends Notification implements ShouldQueue
{
    use Queueable;

    public function __construct(
        public readonly Model $model,
    ) {}

    public function via($notifiable): array
    {
        return ['mail', 'database'];
    }

    public function toMail($notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject(__('notifications.{action}.subject'))
            ->greeting(__('notifications.{action}.greeting', ['name' => $notifiable->name]))
            ->line(__('notifications.{action}.body'))
            ->action(__('notifications.{action}.action'), url('/path'))
            ->line(__('notifications.{action}.thanks'));
    }

    public function toArray($notifiable): array
    {
        return [
            'model_id' => $this->model->id,
            'message' => __('notifications.{action}.body'),
            'type' => '{action}',
        ];
    }
}
```

### Sending Notifications

```php
// To single user
$user->notify(new ActionNotification($model));

// To multiple users
Notification::send($users, new ActionNotification($model));
```

## Scheduler Pattern

```php
// In app/Console/Kernel.php or routes/console.php (Laravel 11+)

use Illuminate\Support\Facades\Schedule;

// Daily cleanup
Schedule::command('model:cleanup')->dailyAt('02:00');

// Every 5 minutes
Schedule::command('queue:retry all')->everyFiveMinutes();

// Custom job on schedule
Schedule::job(new GenerateReportJob)->weeklyOn(1, '08:00');

// With output logging
Schedule::command('reports:generate')
    ->daily()
    ->appendOutputTo(storage_path('logs/reports.log'))
    ->emailOutputOnFailure('admin@example.com');
```

### Artisan Command for Scheduled Tasks

```php
<?php

namespace Modules\{ModuleName}\app\Console;

use Illuminate\Console\Command;

class {Action}Command extends Command
{
    protected $signature = '{module}:{action} {--force}';
    protected $description = 'Description of what this command does';

    public function handle(): int
    {
        $this->info('Starting...');

        // Logic here

        $this->info('Completed.');
        return Command::SUCCESS;
    }
}
```

## Queue Configuration

```
□ Use database driver for development: QUEUE_CONNECTION=database
□ Use redis driver for production: QUEUE_CONNECTION=redis
□ Separate queues by priority:
  - 'default' for standard jobs
  - 'high' for urgent processing
  - 'low' for reports/exports
□ Worker command: php artisan queue:work --queue=high,default,low
□ Supervisor config for production workers
```

## Rules

```
□ Jobs implement ShouldQueue — never process synchronously in controllers
□ Jobs are idempotent — safe to retry without side effects
□ Jobs serialize only IDs — reload models inside handle()
□ Notifications implement ShouldQueue for email/SMS
□ Scheduled tasks have unique signatures to prevent overlap
□ Failed jobs logged with context for debugging
□ Timeouts and retries configured on every job
□ Use readonly constructor properties for job data
```
