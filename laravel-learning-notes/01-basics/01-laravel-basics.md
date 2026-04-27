# Laravel Basics - Complete Guide

## Table of Contents
1. [Introduction to Laravel](#introduction-to-laravel)
2. [Installation & Setup](#installation--setup)
3. [Directory Structure](#directory-structure)
4. [Routing](#routing)
5. [Controllers](#controllers)
6. [Views & Blade Templates](#views--blade-templates)
7. [Middleware](#middleware)
8. [Request & Response](#request--response)

---

## Introduction to Laravel

Laravel is a free, open-source PHP web framework following the Model-View-Controller (MVC) architectural pattern. It was created by Taylor Otwell in 2011.

### Key Features:
- **Elegant Syntax**: Clean, expressive code
- **MVC Architecture**: Separation of concerns
- **Eloquent ORM**: Beautiful ActiveRecord implementation
- **Blade Templating**: Powerful templating engine
- **Artisan CLI**: Command-line interface for development
- **Database Migrations**: Version control for databases
- **Authentication**: Built-in auth scaffolding
- **Testing Support**: PHPUnit integration
- **Queue System**: Background job processing
- **Event Broadcasting**: Real-time features

---

## Installation & Setup

### Prerequisites:
- PHP 8.1 or higher
- Composer (PHP dependency manager)
- Node.js & NPM (for frontend assets)
- Database (MySQL, PostgreSQL, SQLite, or SQL Server)

### Step 1: Install Laravel via Composer

```bash
composer create-project laravel/laravel my-first-app
cd my-first-app
```

### Step 2: Configure Environment

Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```

Generate application key:
```bash
php artisan key:generate
```

### Step 3: Configure Database

Edit `.env` file:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=root
DB_PASSWORD=secret
```

### Step 4: Run Development Server

```bash
php artisan serve
```

Visit: `http://localhost:8000`

---

## Directory Structure

```
laravel-app/
├── app/
│   ├── Console/          # Custom Artisan commands
│   ├── Exceptions/       # Exception handlers
│   ├── Http/
│   │   ├── Controllers/  # Your controllers
│   │   ├── Middleware/   # Custom middleware
│   │   └── Requests/     # Form request validation
│   ├── Models/           # Eloquent models
│   ├── Providers/        # Service providers
│   └── ...
├── bootstrap/            # Application bootstrapping
├── config/               # Configuration files
├── database/
│   ├── factories/        # Model factories for testing
│   ├── migrations/       # Database migrations
│   └── seeders/          # Database seeders
├── public/               # Entry point & static assets
│   └── index.php         # Main entry point
├── resources/
│   ├── css/              # CSS files
│   ├── js/               # JavaScript files
│   └── views/            # Blade templates
├── routes/
│   ├── web.php           # Web routes
│   ├── api.php           # API routes
│   ├── console.php       # Console routes
│   └── channels.php      # Broadcast channels
├── storage/              # Logs, uploads, cache
├── tests/                # Automated tests
├── vendor/               # Composer dependencies
└── .env                  # Environment configuration
```

---

## Routing

### Basic Routes

All routes are defined in `routes/web.php` for web applications.

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\UserController;

// Simple GET route
Route::get('/', function () {
    return view('welcome');
});

// Route with parameter
Route::get('/user/{id}', function ($id) {
    return "User ID: $id";
});

// Optional parameter
Route::get('/user/{name?}', function ($name = 'Guest') {
    return "Hello, $name";
});

// Route with constraint
Route::get('/user/{id}', function ($id) {
    return "User ID: $id";
})->where('id', '[0-9]+');

// Named routes
Route::get('/profile', function () {
    return view('profile');
})->name('profile');

// Generate URL from named route
$url = route('profile');

// Route groups with prefix
Route::prefix('admin')->group(function () {
    Route::get('/dashboard', function () {
        return "Admin Dashboard";
    });
    
    Route::get('/users', function () {
        return "Admin Users";
    });
});

// Route groups with middleware
Route::middleware(['auth'])->group(function () {
    Route::get('/account', function () {
        return "Account Page";
    });
});
```

### HTTP Verbs

```php
// GET
Route::get('/posts', function () {
    return "List Posts";
});

// POST
Route::post('/posts', function () {
    return "Create Post";
});

// PUT/PATCH
Route::put('/posts/{id}', function ($id) {
    return "Update Post $id";
});

Route::patch('/posts/{id}', function ($id) {
    return "Partial Update Post $id";
});

// DELETE
Route::delete('/posts/{id}', function ($id) {
    return "Delete Post $id";
});

// Any HTTP verb
Route::any('/webhook', function () {
    return "Handle Webhook";
});
```

### Resource Routes

```php
// Full resource controller (7 routes)
Route::resource('posts', PostController::class);

// Partial resource
Route::apiResource('posts', PostController::class); // Excludes create & edit

// Customizing resource routes
Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'edit', 'update', 'destroy'
]);
```

---

## Controllers

### Creating a Controller

```bash
php artisan make:controller UserController
php artisan make:controller UserController --resource
php artisan make:controller UserController --api
php artisan make:controller UserController --model=User
```

### Basic Controller

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Display a listing of users.
     */
    public function index()
    {
        $users = User::all();
        return view('users.index', compact('users'));
    }

    /**
     * Show the form for creating a new user.
     */
    public function create()
    {
        return view('users.create');
    }

    /**
     * Store a newly created user.
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:8',
        ]);

        User::create($validated);

        return redirect()->route('users.index')
            ->with('success', 'User created successfully!');
    }

    /**
     * Display the specified user.
     */
    public function show(User $user)
    {
        return view('users.show', compact('user'));
    }

    /**
     * Show the form for editing the specified user.
     */
    public function edit(User $user)
    {
        return view('users.edit', compact('user'));
    }

    /**
     * Update the specified user.
     */
    public function update(Request $request, User $user)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email,' . $user->id,
            'password' => 'nullable|min:8',
        ]);

        if ($validated['password'] ?? false) {
            $validated['password'] = bcrypt($validated['password']);
        } else {
            unset($validated['password']);
        }

        $user->update($validated);

        return redirect()->route('users.show', $user)
            ->with('success', 'User updated successfully!');
    }

    /**
     * Remove the specified user.
     */
    public function destroy(User $user)
    {
        $user->delete();

        return redirect()->route('users.index')
            ->with('success', 'User deleted successfully!');
    }
}
```

### Single Action Controller

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;

class ShowUserProfile extends Controller
{
    public function __invoke(User $user)
    {
        return view('profile.show', compact('user'));
    }
}

// Route
Route::get('/user/{user}/profile', ShowUserProfile::class);
```

---

## Views & Blade Templates

### Creating Views

Views are stored in `resources/views/` with `.blade.php` extension.

```bash
mkdir resources/views/users
touch resources/views/users/index.blade.php
```

### Basic Blade Syntax

```blade
{{-- resources/views/users/index.blade.php --}}

@extends('layouts.app')

@section('title', 'Users List')

@section('content')
<div class="container">
    <h1>{{ $title ?? 'Users' }}</h1>
    
    {{-- Display variable --}}
    <p>Total Users: {{ count($users) }}</p>
    
    {{-- Escape output (default) --}}
    <p>Name: {{ $userName }}</p>
    
    {{-- Unescaped output (use carefully!) --}}
    <p>HTML: {!! $htmlContent !!}</p>
    
    {{-- Conditional statements --}}
    @if(count($users) > 0)
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                @foreach($users as $user)
                    <tr>
                        <td>{{ $user->id }}</td>
                        <td>{{ $user->name }}</td>
                        <td>{{ $user->email }}</td>
                        <td>
                            <a href="{{ route('users.show', $user) }}">View</a>
                            
                            @if($user->active)
                                <span class="badge bg-success">Active</span>
                            @else
                                <span class="badge bg-danger">Inactive</span>
                            @endif
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    @else
        <div class="alert alert-info">No users found.</div>
    @endif
    
    {{-- Loop with $loop variable --}}
    @foreach($users as $user)
        <div class="{{ $loop->first ? 'first' : '' }} {{ $loop->last ? 'last' : '' }}">
            {{ $loop->iteration }}. {{ $user->name }}
        </div>
    @endforeach
    
    {{-- Forelse (loop with empty state) --}}
    @forelse($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users available.</p>
    @endforelse
</div>
@endsection
```

### Blade Components

```blade
{{-- resources/views/components/alert.blade.php --}}
@props(['type' => 'info', 'message'])

<div class="alert alert-{{ $type }}" role="alert">
    {{ $message }}
</div>

{{-- Usage --}}
<x-alert type="success" message="Operation completed!" />
```

### Layouts with Sections

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'My App')</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    @include('partials.header')
    
    <main class="container">
        @if(session('success'))
            <div class="alert alert-success">
                {{ session('success') }}
            </div>
        @endif
        
        @yield('content')
    </main>
    
    @include('partials.footer')
    
    @stack('scripts')
</body>
</html>
```

---

## Middleware

### What is Middleware?

Middleware acts as a bridge between requests and responses, filtering HTTP requests entering your application.

### Built-in Middleware Examples:
- `auth` - Check if user is authenticated
- `guest` - Check if user is not authenticated
- `verified` - Check if email is verified
- `throttle` - Rate limiting

### Creating Custom Middleware

```bash
php artisan make:middleware CheckAge
```

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckAge
{
    public function handle(Request $request, Closure $next)
    {
        if ($request->age <= 18) {
            return redirect('home');
        }

        return $next($request);
    }
}
```

### Registering Middleware

In `app/Http/Kernel.php`:

```php
protected $routeMiddleware = [
    // ...
    'age' => \App\Http\Middleware\CheckAge::class,
];
```

### Using Middleware

```php
// In routes/web.php
Route::get('/adult-content', function () {
    return view('adult-content');
})->middleware('age:18');

// Multiple middleware
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified']);

// Middleware groups
Route::middleware(['web', 'auth'])->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
});
```

---

## Request & Response

### Accessing Request Data

```php
use Illuminate\Http\Request;

public function store(Request $request)
{
    // Get all input
    $input = $request->all();
    
    // Get specific input
    $name = $request->input('name');
    $email = $request->input('email', 'default@example.com'); // with default
    
    // Get from route
    $id = $request->route('id');
    
    // Check if input exists
    if ($request->has('name')) {
        // ...
    }
    
    // Get only specific inputs
    $data = $request->only(['name', 'email']);
    
    // Get all except some
    $data = $request->except(['password', 'confirm_password']);
    
    // Old input (after validation failure)
    $oldName = $request->old('name');
}
```

### Validation

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8|confirmed',
        'age' => 'nullable|integer|min:18',
        'website' => 'url',
        'avatar' => 'image|mimes:jpeg,png,jpg|max:2048',
    ]);
    
    // If validation fails, redirects back with errors
    // If passes, continues execution
    
    User::create($validated);
}
```

### Custom Validation Messages

```php
$validated = $request->validate([
    'name' => 'required|string|max:255',
], [
    'name.required' => 'Please provide your name.',
    'name.max' => 'Name cannot exceed :max characters.',
]);
```

### Response Methods

```php
// Return view
return view('users.show', compact('user'));

// Return JSON
return response()->json([
    'success' => true,
    'data' => $user
]);

// Return JSON with status code
return response()->json([
    'error' => 'Not found'
], 404);

// Redirect
return redirect()->route('users.index');
return redirect()->back();
return redirect('/home');

// Redirect with flash data
return redirect()->route('users.index')
    ->with('success', 'User created!');

// Download file
return Storage::download('file.pdf');

// Custom response
return response($content, 200)
    ->header('Content-Type', 'text/plain');
```

---

## Practice Exercises

1. Create a blog with CRUD operations for posts
2. Build a user management system with roles
3. Create a product catalog with categories
4. Implement a contact form with validation
5. Build a simple API for a todo list

---

## Next Steps

After mastering basics, move on to:
- Database & Eloquent ORM
- Authentication & Authorization
- Frontend Integration (React/Vue)
- Admin Panels (Filament/Custom)
- Advanced Topics (Queues, Events, Testing)
