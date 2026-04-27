# Laravel Architecture - MVC, Service Container, Providers

## Table of Contents
1. [MVC Architecture](#mvc-architecture)
2. [Service Container](#service-container)
3. [Service Providers](#service-providers)
4. [Facades](#facades)
5. [Request Lifecycle](#request-lifecycle)
6. [Application Structure Best Practices](#application-structure-best-practices)

---

## MVC Architecture

Laravel follows the Model-View-Controller pattern to separate business logic, UI, and control flow.

### Model
Models represent your data and business logic. They interact with the database using Eloquent ORM.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    // Table name (optional if follows convention)
    protected $table = 'posts';
    
    // Fillable attributes (mass assignment)
    protected $fillable = ['title', 'content', 'user_id'];
    
    // Hidden attributes (not serialized)
    protected $hidden = ['password'];
    
    // Casts
    protected $casts = [
        'published_at' => 'datetime',
        'is_published' => 'boolean',
    ];
    
    // Relationships
    public function user(): HasMany
    {
        return $this->belongsTo(User::class);
    }
    
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
    
    // Accessors
    public function getSlugAttribute(): string
    {
        return strtolower(str_replace(' ', '-', $this->title));
    }
    
    // Mutators
    public function setTitleAttribute(string $value): void
    {
        $this->attributes['title'] = ucfirst($value);
    }
    
    // Scopes
    public function scopePublished($query)
    {
        return $query->where('is_published', true);
    }
    
    public function scopeRecent($query, $days = 7)
    {
        return $query->whereDate('created_at', '>=', now()->subDays($days));
    }
}
```

### View
Views handle the presentation layer using Blade templating.

```blade
{{-- resources/views/posts/show.blade.php --}}
@extends('layouts.app')

@section('title', $post->title)

@section('content')
<article class="post">
    <header>
        <h1>{{ $post->title }}</h1>
        <p class="meta">
            By {{ $post->user->name }} on {{ $post->published_at->format('M d, Y') }}
        </p>
    </header>
    
    <div class="content">
        {!! $post->content !!}
    </div>
    
    <footer>
        @if($post->tags->count())
            <div class="tags">
                @foreach($post->tags as $tag)
                    <span class="badge">{{ $tag->name }}</span>
                @endforeach
            </div>
        @endif
    </footer>
    
    {{-- Include comments section --}}
    @include('posts.comments', ['comments' => $post->comments])
</article>
@endsection
```

### Controller
Controllers handle user requests, process data, and return responses.

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use App\Http\Requests\StorePostRequest;
use App\Http\Requests\UpdatePostRequest;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth')->except(['index', 'show']);
    }
    
    public function index()
    {
        $posts = Post::with(['user', 'comments'])
            ->published()
            ->recent(30)
            ->paginate(10);
            
        return view('posts.index', compact('posts'));
    }
    
    public function create()
    {
        return view('posts.create');
    }
    
    public function store(StorePostRequest $request)
    {
        $post = auth()->user()->posts()->create($request->validated());
        
        return redirect()->route('posts.show', $post)
            ->with('success', 'Post created successfully!');
    }
    
    public function show(Post $post)
    {
        $post->load(['user', 'comments.user']);
        
        return view('posts.show', compact('post'));
    }
    
    public function edit(Post $post)
    {
        $this->authorize('update', $post);
        
        return view('posts.edit', compact('post'));
    }
    
    public function update(UpdatePostRequest $request, Post $post)
    {
        $this->authorize('update', $post);
        
        $post->update($request->validated());
        
        return redirect()->route('posts.show', $post)
            ->with('success', 'Post updated successfully!');
    }
    
    public function destroy(Post $post)
    {
        $this->authorize('delete', $post);
        
        $post->delete();
        
        return redirect()->route('posts.index')
            ->with('success', 'Post deleted successfully!');
    }
}
```

---

## Service Container

The service container is a powerful tool for managing class dependencies and performing dependency injection.

### Basic Binding

```php
// In a Service Provider's register() method

// Bind interface to implementation
$this->app->bind(PaymentGatewayInterface::class, StripeGateway::class);

// Singleton binding (single instance shared)
$this->app->singleton(CacheService::class, function ($app) {
    return new CacheService(config('cache.default'));
});

// Instance binding
$instance = new ConfigService();
$this->app->instance(ConfigService::class, $instance);
```

### Constructor Injection

```php
<?php

namespace App\Services;

use App\Repositories\UserRepository;
use App\Notifications\WelcomeNotification;

class UserService
{
    protected UserRepository $userRepository;
    protected WelcomeNotification $notification;
    
    // Automatic resolution via type-hinting
    public function __construct(
        UserRepository $userRepository,
        WelcomeNotification $notification
    ) {
        $this->userRepository = $userRepository;
        $this->notification = $notification;
    }
    
    public function createUser(array $data): User
    {
        $user = $this->userRepository->create($data);
        $this->notification->send($user);
        
        return $user;
    }
}
```

### Method Injection

```php
<?php

namespace App\Http\Controllers;

use App\Services\ImageProcessor;
use Illuminate\Http\Request;

class PhotoController extends Controller
{
    public function store(Request $request, ImageProcessor $processor)
    {
        $image = $request->file('photo');
        $processed = $processor->process($image);
        
        return response()->json(['path' => $processed]);
    }
}
```

### Resolving from Container

```php
// Resolve automatically
$service = app()->make(MyService::class);

// Or using helper
$service = resolve(MyService::class);

// With parameters
$service = app()->makeWith(MyService::class, ['config' => $config]);
```

---

## Service Providers

Service providers are the central place to configure your application.

### Creating a Provider

```bash
php artisan make:provider RepositoryServiceProvider
```

### Provider Structure

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Repositories\Contracts\UserRepositoryInterface;
use App\Repositories\EloquentUserRepository;

class RepositoryServiceProvider extends ServiceProvider
{
    /**
     * Register services.
     */
    public function register(): void
    {
        // Bind interfaces to implementations
        $this->app->bind(
            UserRepositoryInterface::class,
            EloquentUserRepository::class
        );
        
        // Singleton for expensive services
        $this->app->singleton(
            \App\Services\AnalyticsService::class,
            function ($app) {
                return new \App\Services\AnalyticsService(
                    config('analytics.api_key')
                );
            }
        );
    }

    /**
     * Bootstrap services.
     */
    public function boot(): void
    {
        // Register view composers
        view()->composer('partials.header', function ($view) {
            $view->with('categories', \App\Models\Category::all());
        });
        
        // Register macro
        \Illuminate\Http\Request::macro('isAjax', function () {
            return $this->ajax() || $this->wantsJson();
        });
    }
}
```

### Registering Providers

In `config/app.php`:

```php
'providers' => [
    // Package Service Providers...
    
    // Application Service Providers...
    App\Providers\AppServiceProvider::class,
    App\Providers\AuthServiceProvider::class,
    App\Providers\EventServiceProvider::class,
    App\Providers\RouteServiceProvider::class,
    App\Providers\RepositoryServiceProvider::class, // Your custom provider
],
```

---

## Facades

Facades provide a static interface to classes in the service container.

### Using Built-in Facades

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Log;

// Cache example
Cache::remember('users', 3600, function () {
    return DB::table('users')->get();
});

// Mail example
Mail::to($user)->send(new WelcomeMail($user));

// Log example
Log::info('User logged in', ['user_id' => $user->id]);
```

### Creating Custom Facades

```php
<?php

namespace App\Facades;

use Illuminate\Support\Facades\Facade;

class Payment extends Facade
{
    protected static function getFacadeAccessor()
    {
        return 'payment'; // Must match binding in service provider
    }
}
```

```php
// In service provider
$this->app->singleton('payment', function ($app) {
    return new \App\Services\PaymentService();
});
```

```php
// Usage
use App\Facades\Payment;

Payment::process($amount, $card);
```

---

## Request Lifecycle

### 1. Entry Point (`public/index.php`)

```php
<?php

use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

// Register autoloader
require __DIR__.'/../vendor/autoload.php';

// Bootstrap application
$app = require_once __DIR__.'/../bootstrap/app.php';

// Handle request
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Request::capture()
)->send();

$kernel->terminate($request, $response);
```

### 2. HTTP Kernel

The kernel handles middleware and routes:

```php
// app/Http/Kernel.php

protected $middleware = [
    // Global middleware (runs on every request)
    \App\Http\Middleware\TrustProxies::class,
    \Illuminate\Http\Middleware\HandleCors::class,
    \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
];

protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
    
    'api' => [
        \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];

protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'verified' => \App\Http\Middleware\EnsureEmailIsVerified::class,
];
```

### 3. Middleware Pipeline

Request flows through middleware stack before reaching controller.

### 4. Router

Routes matched and controller dispatched.

### 5. Controller Execution

Controller method executed, returns response.

### 6. Response

Response flows back through middleware, sent to browser.

---

## Application Structure Best Practices

### Domain-Driven Design (DDD) Approach

```
app/
├── Actions/              # Single-purpose action classes
│   └── CreateUserAction.php
├── Models/               # Eloquent models
│   └── User.php
├── Repositories/         # Data access layer
│   ├── Contracts/
│   │   └── UserRepositoryInterface.php
│   └── EloquentUserRepository.php
├── Services/             # Business logic
│   └── UserService.php
├── DTOs/                 # Data Transfer Objects
│   └── UserData.php
├── Enums/                # PHP 8.1+ enums
│   └── UserRole.php
├── Events/               # Event classes
│   └── UserRegistered.php
├── Listeners/            # Event listeners
│   └── SendWelcomeEmail.php
├── Jobs/                 # Queue jobs
│   └── ProcessImage.php
├── Notifications/        # Notification classes
│   └── WelcomeNotification.php
├── Rules/                # Custom validation rules
│   └── ValidPhoneNumber.php
└── Policies/             # Authorization policies
    └── PostPolicy.php
```

### Example: Action Class

```php
<?php

namespace App\Actions;

use App\Models\User;
use App\DTOs\UserData;
use App\Events\UserRegistered;
use Illuminate\Support\Facades\Hash;

class CreateUserAction
{
    public function execute(UserData $data): User
    {
        $user = User::create([
            'name' => $data->name,
            'email' => $data->email,
            'password' => Hash::make($data->password),
            'role' => $data->role ?? UserRole::USER,
        ]);
        
        event(new UserRegistered($user));
        
        return $user;
    }
}
```

### Example: DTO

```php
<?php

namespace App\DTOs;

use App\Enums\UserRole;

class UserData
{
    public function __construct(
        public string $name,
        public string $email,
        public string $password,
        public ?UserRole $role = null,
    ) {}
    
    public static function fromRequest(array $data): self
    {
        return new self(
            name: $data['name'],
            email: $data['email'],
            password: $data['password'],
            role: isset($data['role']) ? UserRole::from($data['role']) : null,
        );
    }
}
```

---

## Practice Exercises

1. Create a repository pattern implementation for products
2. Build a custom service provider for payment gateways
3. Implement action classes for user registration flow
4. Create custom facade for logging system
5. Refactor fat controllers using service classes

---

## Next Steps

Continue to Database & Eloquent ORM for advanced data modeling!
