# Laravel Database & Eloquent ORM - Complete Guide

## Table of Contents
1. [Database Configuration](#database-configuration)
2. [Migrations](#migrations)
3. [Eloquent Models](#eloquent-models)
4. [Eloquent Relationships](#eloquent-relationships)
5. [Query Builder](#query-builder)
6. [Database Seeding & Factories](#database-seeding--factories)
7. [Advanced Eloquent](#advanced-eloquent)

---

## Database Configuration

### Environment Setup

Configure your `.env` file:

```env
# MySQL
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=root
DB_PASSWORD=secret

# PostgreSQL
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_db
DB_USERNAME=root
DB_PASSWORD=secret

# SQLite
DB_CONNECTION=sqlite
DB_DATABASE=/path/to/database.sqlite

# SQL Server
DB_CONNECTION=sqlsrv
DB_HOST=localhost
DB_PORT=1433
DB_DATABASE=laravel_db
DB_USERNAME=root
DB_PASSWORD=secret
```

### Multiple Database Connections

In `config/database.php`:

```php
'connections' => [
    'mysql' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST', '127.0.0.1'),
        'database' => env('DB_DATABASE', 'laravel'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
        // ...
    ],
    
    'forge' => [
        'driver' => 'mysql',
        'host' => env('FORGE_DB_HOST', '127.0.0.1'),
        'database' => env('FORGE_DB_DATABASE', 'forge'),
        'username' => env('FORGE_DB_USERNAME', 'forge'),
        'password' => env('FORGE_DB_PASSWORD', ''),
        // ...
    ],
],

'default' => env('DB_CONNECTION', 'mysql'),
```

Using different connections:

```php
// Query builder
$users = DB::connection('forge')->table('users')->get();

// Eloquent model
class User extends Model
{
    protected $connection = 'forge';
}
```

---

## Migrations

### Creating Migrations

```bash
# Create table
php artisan make:migration create_users_table

# Add column to existing table
php artisan make:migration add_email_to_users_table --table=users

# Create pivot table
php artisan make:migration create_role_user_table

# All in one command with model
php artisan make:model Post -m
```

### Migration Structure

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id(); // Auto-incrementing primary key
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('content');
            $table->text('excerpt')->nullable();
            $table->string('featured_image')->nullable();
            $table->boolean('is_published')->default(false);
            $table->timestamp('published_at')->nullable();
            $table->integer('views_count')->default(0);
            $table->json('metadata')->nullable(); // JSON column
            $table->timestamps(); // created_at & updated_at
            $table->softDeletes(); // deleted_at for soft deletes
            $table->index(['is_published', 'published_at']); // Index
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

### Column Types

```php
Schema::create('example', function (Blueprint $table) {
    // ID columns
    $table->id();
    $table->bigIncrements('custom_id');
    
    // Numeric
    $table->tinyInteger('age');
    $table->smallInteger('points');
    $table->integer('votes');
    $table->bigInteger('account_number');
    $table->decimal('price', 8, 2);
    $table->float('rating', 3, 2);
    $table->double('latitude', 10, 8);
    
    // String
    $table->char('code', 10);
    $table->string('name');
    $table->string('email', 100)->unique();
    $table->text('description');
    $table->mediumText('content');
    $table->longText('full_article');
    
    // Boolean
    $table->boolean('active')->default(true);
    
    // Date & Time
    $table->date('birth_date');
    $table->time('start_time');
    $table->dateTime('event_datetime');
    $table->timestamp('posted_at')->useCurrent();
    $table->timestampTz('scheduled_at');
    $table->timestamps(); // created_at & updated_at
    $table->softDeletes(); // deleted_at
    
    // JSON
    $table->json('settings');
    $table->jsonb('preferences'); // PostgreSQL only
    
    // Other
    $table->binary('avatar');
    $table->uuid('uuid')->unique();
    $table->ulid('ulid')->unique();
    $table->enum('status', ['draft', 'published', 'archived']);
    $table->ipAddress('ip_address');
    $table->macAddress('mac_address');
    $table->geometry('location');
});
```

### Modifying Columns

```bash
php artisan make:migration update_users_table --table=users
```

```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        // Add column
        $table->string('phone')->nullable()->after('email');
        
        // Rename column
        $table->renameColumn('name', 'full_name');
        
        // Change column type
        $table->string('email', 255)->change();
        
        // Make nullable
        $table->string('nickname')->nullable()->change();
        
        // Drop column
        $table->dropColumn('old_field');
        
        // Add index
        $table->index('email');
        
        // Drop index
        $table->dropIndex('users_email_index');
    });
}
```

### Running Migrations

```bash
# Run all pending migrations
php artisan migrate

# Run migrations for specific path
php artisan migrate --path=database/migrations/2024

# Force migration (production)
php artisan migrate --force

# Rollback last batch
php artisan migrate:rollback

# Rollback all migrations
php artisan migrate:reset

# Rollback and re-run
php artisan migrate:refresh

# Fresh migration (delete all tables and migrate)
php artisan migrate:fresh

# Fresh with seeding
php artisan migrate:fresh --seed

# Show migration status
php artisan migrate:status
```

---

## Eloquent Models

### Basic Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Post extends Model
{
    use HasFactory, SoftDeletes;
    
    // Custom table name (optional if follows convention)
    protected $table = 'blog_posts';
    
    // Primary key (default is 'id')
    protected $primaryKey = 'post_id';
    
    // Key type (for UUIDs)
    public $keyType = 'string';
    public $incrementing = false;
    
    // Timestamps column names
    const CREATED_AT = 'created';
    const UPDATED_AT = 'updated';
    
    // Disable timestamps
    public $timestamps = false;
    
    // Fillable attributes (mass assignment)
    protected $fillable = [
        'title',
        'slug',
        'content',
        'user_id',
        'is_published',
    ];
    
    // Guarded attributes (opposite of fillable)
    protected $guarded = ['id', 'password'];
    
    // Empty guarded = allow all
    protected $guarded = [];
    
    // Hidden attributes (not serialized)
    protected $hidden = [
        'password',
        'remember_token',
    ];
    
    // Visible attributes (inverse of hidden)
    protected $visible = ['title', 'content'];
    
    // Attribute casting
    protected $casts = [
        'is_published' => 'boolean',
        'views_count' => 'integer',
        'metadata' => 'array',
        'published_at' => 'datetime',
        'price' => 'decimal:2',
    ];
    
    // Append attributes to array/json
    protected $appends = ['url', 'formatted_date'];
}
```

### Creating & Retrieving Models

```php
// Create new instance
$post = new Post();
$post->title = 'My First Post';
$post->content = 'Content here...';
$post->save();

// Create with array
Post::create([
    'title' => 'New Post',
    'content' => 'Content...',
    'user_id' => 1,
]);

// Find by ID
$post = Post::find(1);
$post = Post::findOrFail(1); // Throws 404 if not found

// Find by column
$post = Post::where('slug', 'my-post')->first();
$post = Post::where('slug', 'my-post')->firstOrFail();

// Get all
$posts = Post::all();

// Get with conditions
$posts = Post::where('is_published', true)
    ->whereDate('created_at', '>=', now()->subDays(7))
    ->orderBy('created_at', 'desc')
    ->get();

// Chunk results
Post::where('is_published', true)->chunk(100, function ($posts) {
    foreach ($posts as $post) {
        // Process
    }
});

// Cursor for memory efficiency
foreach (Post::where('is_published', true)->cursor() as $post) {
    // Process one at a time
}

// Aggregate functions
$count = Post::count();
$max = Post::max('views_count');
$sum = Post::sum('views_count');
$avg = Post::avg('rating');
```

### Updating Models

```php
// Find and update
$post = Post::find(1);
$post->title = 'Updated Title';
$post->save();

// Update with array
Post::where('id', 1)->update([
    'title' => 'Updated',
    'views_count' => DB::raw('views_count + 1'),
]);

// Mass update
Post::where('is_published', false)
    ->update(['is_published' => true]);

// Increment/Decrement
Post::where('id', 1)->increment('views_count');
Post::where('id', 1)->increment('views_count', 5);
Post::where('id', 1)->decrement('views_count');
Post::where('id', 1)->decrement('views_count', 3, ['updated_at' => now()]);
```

### Deleting Models

```php
// Soft delete (requires SoftDeletes trait)
$post->delete();

// Force delete (permanent)
$post->forceDelete();

// Restore soft deleted
Post::withTrashed()->where('id', 1)->restore();

// Only trashed
$trashed = Post::onlyTrashed()->get();

// Hard delete multiple
Post::where('is_published', false)->forceDelete();
```

---

## Eloquent Relationships

### One to One

```php
// User model
class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
    
    public function phone()
    {
        return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
    }
}

// Profile model
class Profile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}

// Usage
$user = User::find(1);
$profile = $user->profile;

$profile = Profile::find(1);
$user = $profile->user;
```

### One to Many

```php
// Post model
class Post extends Model
{
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
    
    public function recentComments()
    {
        return $this->hasMany(Comment::class)->limit(5);
    }
}

// Comment model
class Comment extends Model
{
    public function post()
    {
        return $this->belongsTo(Post::class);
    }
}

// Usage
$post = Post::find(1);
$comments = $post->comments;

// Create related
$comment = $post->comments()->create([
    'content' => 'Great post!',
    'user_id' => 1,
]);

// Save related
$comment = new Comment(['content' => 'Nice!']);
$post->comments()->save($comment);

// Save many
$post->comments()->saveMany([
    new Comment(['content' => 'First']),
    new Comment(['content' => 'Second']),
]);
```

### Many to Many

```php
// User model
class User extends Model
{
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
    
    // With custom pivot table
    public function roles()
    {
        return $this->belongsToMany(Role::class, 'user_roles');
    }
    
    // With pivot columns
    public function roles()
    {
        return $this->belongsToMany(Role::class)
            ->withPivot('granted_by', 'expires_at')
            ->withTimestamps();
    }
}

// Role model
class Role extends Model
{
    public function users()
    {
        return $this->belongsToMany(User::class);
    }
}

// Usage
$user = User::find(1);

// Attach
$user->roles()->attach($roleId);
$user->roles()->attach([1, 2, 3]);
$user->roles()->attach($roleId, ['granted_by' => auth()->id()]);

// Detach
$user->roles()->detach($roleId);
$user->roles()->detach([1, 2]);
$user->roles()->detach(); // Detach all

// Sync (replace all)
$user->roles()->sync([1, 2, 3]);
$user->roles()->sync([1 => ['granted_by' => 1], 2]);

// Sync without detaching
$user->roles()->syncWithoutDetaching([4, 5]);

// Toggle
$user->roles()->toggle([1, 2, 3]);

// Access pivot
foreach ($user->roles as $role) {
    echo $role->pivot->granted_by;
}
```

### Has Many Through

```php
// Country -> Users -> Posts
class Country extends Model
{
    public function posts()
    {
        return $this->hasManyThrough(Post::class, User::class);
    }
}

// Usage
$country = Country::find(1);
$posts = $country->posts; // All posts from users in this country
```

### Polymorphic Relationships

```php
// Post and Video models can have comments
class Post extends Model
{
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

class Video extends Model
{
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

class Comment extends Model
{
    public function commentable()
    {
        return $this->morphTo();
    }
}

// Migration
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->morphs('commentable'); // Creates commentable_type & commentable_id
    $table->text('content');
    $table->timestamps();
});

// Usage
$post = Post::find(1);
$post->comments()->create(['content' => 'Nice post!']);

$video = Video::find(1);
$video->comments()->create(['content' => 'Great video!']);

$comment = Comment::find(1);
echo get_class($comment->commentable); // Returns Post or Video
```

### Eager Loading

```php
// N+1 problem (BAD)
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name; // Query per post!
}

// Eager loading (GOOD)
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name; // No additional queries
}

// Multiple relationships
$posts = Post::with(['user', 'comments', 'tags'])->get();

// Nested eager loading
$posts = Post::with('comments.user')->get();

// Constrained eager loading
$posts = Post::with(['comments' => function ($query) {
    $query->where('is_approved', true)->limit(5);
}])->get();

// Lazy eager loading
$posts = Post::all();
if ($condition) {
    $posts->load('user', 'comments');
}

// Eager load missing relationships
$posts->loadMissing('user');
```

---

## Query Builder

```php
// Select
$users = DB::table('users')
    ->select('name', 'email')
    ->get();

$users = DB::table('users')
    ->selectRaw('COUNT(*) as count, role')
    ->groupBy('role')
    ->get();

// Where clauses
$users = DB::table('users')
    ->where('active', 1)
    ->where('age', '>=', 18)
    ->whereBetween('age', [18, 65])
    ->whereIn('role', ['admin', 'moderator'])
    ->whereNull('deleted_at')
    ->whereExists(function ($query) {
        $query->select(DB::raw(1))
              ->from('posts')
              ->whereColumn('posts.user_id', 'users.id');
    })
    ->get();

// Or where
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere('is_vip', true)
    ->get();

// Joins
$users = DB::table('users')
    ->join('posts', 'users.id', '=', 'posts.user_id')
    ->leftJoin('profiles', 'users.id', '=', 'profiles.user_id')
    ->select('users.*', 'posts.title', 'profiles.bio')
    ->get();

// Grouping
$results = DB::table('orders')
    ->selectRaw('SUM(price) as total_sales, COUNT(*) as order_count')
    ->groupBy('customer_id')
    ->having('total_sales', '>', 1000)
    ->get();

// Ordering
$users = DB::table('users')
    ->orderBy('name')
    ->orderByDesc('created_at')
    ->inRandomOrder() // Random
    ->get();

// Pagination
$users = DB::table('users')->paginate(15);
$users = DB::table('users')->simplePaginate(15); // Previous/Next only

// Limit & Offset
$users = DB::table('users')
    ->skip(10)
    ->take(5)
    ->get();

// Union
$first = DB::table('users')->whereNull('last_name');
$second = DB::table('users')->whereNull('first_name');
$users = $first->union($second)->get();

// Locks
$users = DB::table('users')
    ->where('id', 1)
    ->lockForUpdate()
    ->get();

$users = DB::table('users')
    ->sharedLock()
    ->get();
```

---

## Database Seeding & Factories

### Seeders

```bash
php artisan make:seeder DatabaseSeeder
php artisan make:seeder UserSeeder
```

```php
// database/seeders/DatabaseSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            RoleSeeder::class,
        ]);
    }
}

// database/seeders/UserSeeder.php
namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => bcrypt('password'),
            'role' => 'admin',
        ]);
        
        User::factory()->count(50)->create();
    }
}
```

```bash
# Run seeders
php artisan db:seed
php artisan db:seed --class=UserSeeder
php artisan migrate:fresh --seed # Fresh migration with seeding
```

### Model Factories

```bash
php artisan make:factory UserFactory
php artisan make:factory PostFactory --model=Post
```

```php
// database/factories/UserFactory.php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;
    
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => bcrypt('password'),
            'remember_token' => Str::random(10),
            'role' => fake()->randomElement(['user', 'moderator', 'admin']),
            'active' => fake()->boolean(90), // 90% chance true
        ];
    }
    
    // States
    public function admin(): static
    {
        return $this->state(fn (array $attributes) => [
            'role' => 'admin',
        ]);
    }
    
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}

// Usage
User::factory()->create();
User::factory()->admin()->create();
User::factory()->count(10)->create();
User::factory()->unverified()->admin()->count(5)->create();

// With relationships
User::factory()
    ->hasPosts(5)
    ->create();

User::factory()
    ->has(
        Post::factory()->count(3)->state(['is_published' => true]),
        'posts'
    )
    ->create();
```

---

## Advanced Eloquent

### Scopes

```php
class Post extends Model
{
    // Local scope
    public function scopePublished($query)
    {
        return $query->where('is_published', true)
                     ->whereNotNull('published_at');
    }
    
    public function scopeRecent($query, $days = 7)
    {
        return $query->whereDate('created_at', '>=', now()->subDays($days));
    }
    
    public function scopePopular($query)
    {
        return $query->where('views_count', '>', 100)
                     ->orderByDesc('views_count');
    }
    
    // Global scope
    protected static function booted()
    {
        static::addGlobalScope('active', function ($query) {
            $query->where('active', true);
        });
    }
}

// Usage
Post::published()->recent(30)->popular()->get();
```

### Accessors & Mutators

```php
class User extends Model
{
    // Accessor (Laravel 9+)
    public function getFirstNameAttribute(): string
    {
        return explode(' ', $this->attributes['name'])[0];
    }
    
    // Mutator (Laravel 9+)
    public function setFirstNameAttribute(string $value): void
    {
        $this->attributes['name'] = "$value {$this->attributes['last_name']}";
    }
    
    // New accessor/mutator syntax (Laravel 9+)
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn ($value) => ucfirst($value),
            set: fn ($value) => strtolower($value),
        );
    }
}

// Usage
$user = User::find(1);
echo $user->first_name; // Uses accessor

$user->first_name = 'john'; // Uses mutator
$user->save();
```

### Events

```php
class Post extends Model
{
    protected $dispatchesEvents = [
        'creating' => \App\Events\PostCreating::class,
        'created' => \App\Events\PostCreated::class,
    ];
    
    protected static function booted()
    {
        static::creating(function ($post) {
            $post->slug = str()->slug($post->title);
        });
        
        static::created(function ($post) {
            // Send notification
        });
        
        static::updating(function ($post) {
            // Check if title changed
            if ($post->isDirty('title')) {
                // Handle title change
            }
        });
    }
}
```

### Collections

```php
$users = User::all();

// Filter
$admins = $users->filter(fn ($user) => $user->role === 'admin');

// Map
$names = $users->map(fn ($user) => $user->name);

// Pluck
$emails = $users->pluck('email');
$emailNamePairs = $users->pluck('email', 'name');

// Group by
$groupedByRole = $users->groupBy('role');

// Unique
$unique = $users->unique('email');

// Sort
$sorted = $users->sortByDesc('created_at');

// Search
$found = $users->firstWhere('email', 'test@example.com');

// Transform
$users->transform(fn ($user) => $user->only(['id', 'name']));
```

---

## Practice Exercises

1. Create a blog system with posts, comments, tags, and categories
2. Build an e-commerce product catalog with variants
3. Implement a social media follow system
4. Create a project management tool with tasks and assignments
5. Build a booking system with availability checking

---

## Next Steps

Continue to Authentication & Authorization for secure user management!
