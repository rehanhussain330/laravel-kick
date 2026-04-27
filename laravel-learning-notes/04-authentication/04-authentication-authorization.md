# Laravel Authentication & Authorization

## Table of Contents
1. [Authentication Basics](#authentication-basics)
2. [Manual Authentication](#manual-authentication)
3. [Authentication Scaffolding](#authentication-scaffolding)
4. [Authorization - Gates & Policies](#authorization--gates--policies)
5. [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
6. [API Authentication with Sanctum](#api-authentication-with-sanctum)
7. [OAuth with Passport](#oauth-with-passport)
8. [Email Verification](#email-verification)
9. [Password Reset](#password-reset)
10. [Two-Factor Authentication](#two-factor-authentication)

---

## Authentication Basics

### User Model

Laravel includes a `User` model out of the box:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed', // Laravel 11+
    ];
}
```

### Authentication Guards

Guards determine how users are authenticated:

```php
// config/auth.php

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'sanctum' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],

    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],

    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
],
```

---

## Manual Authentication

### Login

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials, $request->boolean('remember'))) {
        $request->session()->regenerate();

        return redirect()->intended('/dashboard');
    }

    throw ValidationException::withMessages([
        'email' => 'The provided credentials do not match our records.',
    ]);
}
```

### Logout

```php
public function logout(Request $request)
{
    Auth::logout();

    $request->session()->invalidate();
    $request->session()->regenerateToken();

    return redirect('/');
}
```

### Check Authentication Status

```php
// In controller
if (Auth::check()) {
    // User is logged in
    $user = Auth::user();
}

// In view
@auth
    <p>Welcome, {{ Auth::user()->name }}!</p>
@endauth

@guest
    <a href="{{ route('login') }}">Login</a>
@endguest
```

### Remember Me

```php
// Login with remember me
Auth::attempt(['email' => $email, 'password' => $password], true);

// Check if user was remembered
if (Auth::viaRemember()) {
    // User authenticated via remember cookie
}
```

### Multiple Guards

```php
// Admin login
if (Auth::guard('admin')->attempt($credentials)) {
    return redirect()->route('admin.dashboard');
}

// Get admin user
$admin = Auth::guard('admin')->user();

// Check if admin is logged in
if (Auth::guard('admin')->check()) {
    // ...
}
```

---

## Authentication Scaffolding

### Laravel Breeze (Simple Starter Kit)

```bash
# Install Breeze
composer require laravel/breeze --dev

# Install with Blade
php artisan breeze:install blade

# Or install with React
php artisan breeze:install react

# Or install with Vue
php artisan breeze:install vue

# Run migrations
php artisan migrate

# Start development server
php artisan serve

# In another terminal, compile assets
npm install && npm run dev
```

Breeze provides:
- Login/Register pages
- Password reset
- Email verification
- Profile management
- Dark mode support

### Laravel UI (Legacy)

```bash
composer require laravel/ui

php artisan ui bootstrap --auth
php artisan ui vue --auth
php artisan ui react --auth

npm install && npm run dev
```

---

## Authorization - Gates & Policies

### Gates

Gates are simple closures for authorization:

```php
// app/Providers/AuthServiceProvider.php

public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });

    Gate::define('delete-post', function (User $user, Post $post) {
        return $user->id === $post->user_id || $user->is_admin;
    });

    Gate::define('admin-access', function (User $user) {
        return $user->role === 'admin';
    });
}
```

Using gates:

```php
// In controller
public function update(Request $request, Post $post)
{
    if (!Gate::allows('update-post', $post)) {
        abort(403);
    }

    // Or using helper
    if (!Gate::forUser($user)->allows('update-post', $post)) {
        abort(403);
    }

    // Update post...
}

// In view
@can('update-post', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

@cannot('delete-post', $post)
    <p>You cannot delete this post</p>
@endcannot
```

### Policies

Policies organize authorization logic around models:

```bash
php artisan make:policy PostPolicy --model=Post
```

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if user can view any posts.
     */
    public function viewAny(?User $user): bool
    {
        return true; // All posts are public
    }

    /**
     * Determine if user can view the post.
     */
    public function view(?User $user, Post $post): bool
    {
        return $post->is_published || ($user && $user->id === $post->user_id);
    }

    /**
     * Determine if user can create posts.
     */
    public function create(User $user): bool
    {
        return true;
    }

    /**
     * Determine if user can update the post.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    /**
     * Determine if user can delete the post.
     */
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    /**
     * Determine if user can restore soft-deleted post.
     */
    public function restore(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    /**
     * Determine if user can permanently delete the post.
     */
    public function forceDelete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

Using policies:

```php
// In controller
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);

    // If authorized, continues...
}

// With different method
$this->authorize('delete', $post);

// Create new resource
$this->authorize('create', Post::class);

// In view
@can('update', $post)
    <button>Edit Post</button>
@endcan

// Check without throwing exception
if (Gate::allows('update', $post)) {
    // ...
}
```

### Auto-Discovery

Laravel auto-discovers policies if:
- Model and policy follow naming convention
- Policy is in `App\Policies` namespace
- No need to register in `AuthServiceProvider`

Manual registration (if needed):

```php
protected $policies = [
    Post::class => PostPolicy::class,
];
```

---

## Role-Based Access Control (RBAC)

### Simple RBAC Implementation

**Migration:**

```bash
php artisan make:migration create_roles_and_permissions_tables
```

```php
Schema::create('roles', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();
    $table->string('description')->nullable();
    $table->timestamps();
});

Schema::create('permissions', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();
    $table->string('description')->nullable();
    $table->timestamps();
});

Schema::create('role_user', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->foreignId('role_id')->constrained()->onDelete('cascade');
    $table->unique(['user_id', 'role_id']);
});

Schema::create('permission_role', function (Blueprint $table) {
    $table->id();
    $table->foreignId('role_id')->constrained()->onDelete('cascade');
    $table->foreignId('permission_id')->constrained()->onDelete('cascade');
    $table->unique(['role_id', 'permission_id']);
});
```

**Models:**

```php
// Role Model
class Role extends Model
{
    protected $fillable = ['name', 'description'];

    public function users()
    {
        return $this->belongsToMany(User::class);
    }

    public function permissions()
    {
        return $this->belongsToMany(Permission::class);
    }

    public function hasPermission(string $permission): bool
    {
        return $this->permissions->contains('name', $permission);
    }
}

// Permission Model
class Permission extends Model
{
    protected $fillable = ['name', 'description'];

    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}

// User Model extension
class User extends Authenticatable
{
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }

    public function hasRole(string $role): bool
    {
        return $this->roles->contains('name', $role);
    }

    public function assignRole(string $role): self
    {
        $roleId = Role::where('name', $role)->firstOrFail()->id;
        $this->roles()->syncWithoutDetaching($roleId);
        return $this;
    }

    public function removeRole(string $role): self
    {
        $roleId = Role::where('name', $role)->firstOrFail()->id;
        $this->roles()->detach($roleId);
        return $this;
    }

    public function hasPermission(string $permission): bool
    {
        foreach ($this->roles as $role) {
            if ($role->hasPermission($permission)) {
                return true;
            }
        }
        return false;
    }

    public function hasAnyRole(array $roles): bool
    {
        return $this->roles->whereIn('name', $roles)->isNotEmpty();
    }
}
```

**Middleware:**

```bash
php artisan make:middleware RoleMiddleware
```

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class RoleMiddleware
{
    public function handle(Request $request, Closure $next, string $role)
    {
        if (!$request->user() || !$request->user()->hasRole($role)) {
            abort(403, 'Unauthorized action.');
        }

        return $next($request);
    }
}
```

Register middleware in `app/Http/Kernel.php`:

```php
protected $routeMiddleware = [
    // ...
    'role' => \App\Http\Middleware\RoleMiddleware::class,
];
```

Usage:

```php
// Routes
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'dashboard']);
    Route::resource('/admin/users', AdminUserController::class);
});

// In controller
public function __construct()
{
    $this->middleware(['auth', 'role:admin']);
}
```

### Using Spatie Laravel Permission Package

```bash
composer require spatie/laravel-permission
```

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

Usage:

```php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
}

// Assign role
$user->assignRole('admin');

// Remove role
$user->removeRole('admin');

// Check role
$user->hasRole('admin');
$user->hasAnyRole(['admin', 'moderator']);
$user->hasAllRoles(['admin', 'moderator']);

// Give permission
$user->givePermissionTo('edit articles');
$user->revokePermissionTo('edit articles');

// Check permission
$user->can('edit articles');
$user->hasPermissionTo('edit articles');

// Role has permissions
$role->givePermissionTo('edit articles');
$role->hasPermissionTo('edit articles');

// Middleware
Route::middleware(['role:admin'])->group(function () {
    // ...
});

Route::middleware(['permission:edit articles'])->group(function () {
    // ...
});

// In blade
@role('admin')
    <p>Admin content</p>
@endrole

@hasrole('admin')
    <p>Admin content</p>
@endhasrole

@can('edit articles')
    <button>Edit</button>
@endcan
```

---

## API Authentication with Sanctum

### Installation

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

Add trait to User model:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

### Token-Based Authentication

```php
// Issue token on login
public function login(Request $request)
{
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (!$user || !Hash::check($request->password, $user->password)) {
        return response(['message' => 'Invalid credentials'], 401);
    }

    $token = $user->createToken('auth-token')->plainTextToken;

    return response(['token' => $token]);
}

// Protected route
public function profile(Request $request)
{
    return $request->user();
}

// Revoke token
public function logout(Request $request)
{
    $request->user()->currentAccessToken()->delete();
    
    return response(['message' => 'Logged out']);
}

// Revoke all tokens
$request->user()->tokens()->delete();

// Revoke specific token
$user->tokens()->where('id', $tokenId)->delete();
```

### Token Abilities (Scopes)

```php
// Create token with abilities
$token = $user->createToken('token-name', ['posts:edit', 'posts:delete'])->plainTextToken;

// Check ability
if ($request->user()->tokenCan('posts:edit')) {
    // User can edit posts
}

// Middleware protection
Route::middleware(['auth:sanctum', 'ability:posts:edit'])->group(function () {
    // ...
});
```

### SPA Authentication

Configure `config/sanctum.php`:

```php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort()
))),
```

In `config/cors.php`:

```php
'supports_credentials' => true,
```

Make sure your API routes use `auth:sanctum` middleware and web routes include `web` middleware.

---

## OAuth with Passport

```bash
composer require laravel/passport
php artisan migrate
php artisan passport:install
```

Add trait to User model:

```php
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Register routes in `AuthServiceProvider`:

```php
use Laravel\Passport\Passport;

public function boot(): void
{
    Passport::routes();
}
```

Change auth guard in `config/auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

---

## Email Verification

### Setup

Make User model implement `MustVerifyEmail`:

```php
use Illuminate\Contracts\Auth\MustVerifyEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

Add middleware to routes:

```php
Route::get('/profile', ProfileController::class)
    ->middleware(['auth', 'verified']);
```

Send verification email:

```php
use Illuminate\Support\Facades\Auth;

Auth::user()->sendEmailVerificationNotification();
```

Resend verification:

```php
public function resend(Request $request)
{
    if ($request->user()->hasVerifiedEmail()) {
        return redirect()->intended('/dashboard');
    }

    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
}
```

---

## Password Reset

### Configuration

Ensure `users` table has `remember_token` column.

Create reset token:

```php
use Illuminate\Support\Facades\Password;

$status = Password::sendResetLink(
    $request->only('email')
);

return $status === Password::RESET_LINK_SENT
    ? back()->with(['status' => __($status)])
    : back()->withErrors(['email' => __($status)]);
```

Reset password:

```php
$status = Password::reset(
    $request->only('email', 'password', 'password_confirmation', 'token'),
    function (User $user, string $password) {
        $user->forceFill([
            'password' => Hash::make($password)
        ])->save();
    }
);

return $status === Password::PASSWORD_RESET
    ? redirect()->route('login')->with('status', __($status))
    : back()->withErrors(['email' => [__($status)]]);
```

---

## Two-Factor Authentication

### Using Laravel Fortify

```bash
composer require laravel/fortify
php artisan fortify:install
php artisan migrate
```

Enable 2FA in `config/fortify.php`:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

### Manual TOTP Implementation

```bash
composer require pragmarx/google2fa-laravel
```

```php
use PragmaRX\Google2FALaravel\Facade as Google2FA;

// Generate secret
$secret = Google2FA::generateSecretKey();

// Verify code
$valid = Google2FA::verifyKey($secret, $request->input('code'));
```

---

## Practice Exercises

1. Build complete authentication system with Breeze
2. Implement RBAC with roles and permissions
3. Create API with Sanctum token authentication
4. Build password reset flow
5. Add email verification to registration
6. Implement two-factor authentication
7. Create multi-guard authentication (user + admin)

---

## Next Steps

Continue to Starter Kits & Frontend Integration!
