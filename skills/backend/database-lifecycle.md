# Skill: Database Lifecycle Manager

Manages migrations, seeders, model relationships, and database patterns
following Wakeb Starter conventions.

## When to Use

- When creating or modifying database tables
- When defining model relationships
- When creating seeders for test/demo data
- When the user asks about database design

## Migration Patterns

### Column Types

```php
// Standard columns
$table->id();
$table->string('name');
$table->string('email')->unique();
$table->text('description')->nullable();
$table->integer('count')->default(0);
$table->decimal('price', 10, 2);
$table->boolean('is_active')->default(true);
$table->enum('status', ['draft', 'active', 'archived'])->default('draft');
$table->json('metadata')->nullable();
$table->date('birth_date')->nullable();
$table->timestamp('verified_at')->nullable();
$table->timestamps();
$table->softDeletes();

// Foreign keys
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
$table->foreignId('category_id')->nullable()->constrained()->nullOnDelete();

// Polymorphic
$table->morphs('commentable');  // creates commentable_type + commentable_id
```

### Migration Naming

```
create_{table}_table         → New table
add_{column}_to_{table}      → Add column
rename_{old}_to_{new}_in_{table} → Rename column
drop_{column}_from_{table}   → Remove column
create_{table1}_{table2}_table → Pivot table (alphabetical order)
```

### Modify Existing Table

```php
public function up(): void
{
    Schema::table('employees', function (Blueprint $table) {
        $table->string('phone')->nullable()->after('email');
    });
}

public function down(): void
{
    Schema::table('employees', function (Blueprint $table) {
        $table->dropColumn('phone');
    });
}
```

## Model Patterns

### Relationships

```php
// One to Many
public function posts()
{
    return $this->hasMany(Post::class);
}

public function author()
{
    return $this->belongsTo(User::class, 'user_id');
}

// Many to Many
public function roles()
{
    return $this->belongsToMany(Role::class);
}

// Has One
public function profile()
{
    return $this->hasOne(Profile::class);
}

// Polymorphic
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

### Scopes

```php
// Local scope
public function scopeActive($query)
{
    return $query->where('is_active', true);
}

public function scopeOfStatus($query, string $status)
{
    return $query->where('status', $status);
}

// Usage: ModuleName::active()->ofStatus('draft')->get();
```

### Accessors & Mutators

```php
// Accessor
protected function fullName(): Attribute
{
    return Attribute::make(
        get: fn () => "{$this->first_name} {$this->last_name}",
    );
}

// Mutator
protected function email(): Attribute
{
    return Attribute::make(
        set: fn (string $value) => strtolower($value),
    );
}
```

### Casts

```php
protected $casts = [
    'is_active' => 'boolean',
    'metadata' => 'array',
    'birth_date' => 'date',
    'verified_at' => 'datetime',
    'price' => 'decimal:2',
    'status' => StatusEnum::class,  // if using enum
];
```

## Seeder Patterns

```php
<?php

namespace Modules\{ModuleName}\database\seeders;

use Illuminate\Database\Seeder;
use Modules\{ModuleName}\app\Models\{ModuleName};

class {ModuleName}Seeder extends Seeder
{
    public function run(): void
    {
        $items = [
            ['name' => 'Item 1', 'status' => 'active'],
            ['name' => 'Item 2', 'status' => 'draft'],
        ];

        foreach ($items as $item) {
            {ModuleName}::updateOrCreate(
                ['name' => $item['name']],  // unique key
                $item                        // data
            );
        }
    }
}
```

### Using Factories

```php
// Factory
class {ModuleName}Factory extends Factory
{
    protected $model = {ModuleName}::class;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'status' => fake()->randomElement(['active', 'draft']),
        ];
    }
}

// In seeder
{ModuleName}::factory()->count(50)->create();
```

## Rules

```
□ Always include down() method in migrations
□ Always use softDeletes() on main tables
□ Foreign keys use constrained() with explicit cascade behavior
□ Seeders use updateOrCreate to be idempotent (re-runnable)
□ Model fillable includes ALL mass-assignable fields
□ Casts defined for non-string fields
□ Relationships return the relation (not ->get())
□ Nullable foreign keys use nullOnDelete()
□ Required foreign keys use cascadeOnDelete()
```
