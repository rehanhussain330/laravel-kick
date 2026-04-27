# Laravel Starter Kits & Frontend Integration

## Table of Contents
1. [Laravel Breeze](#laravel-breeze)
2. [Laravel Jetstream](#laravel-jetstream)
3. [React.js Integration](#reactjs-integration)
4. [Vue.js Integration](#vuejs-integration)
5. [Inertia.js](#inertiajs)
6. [Livewire](#livewire)
7. [Vite Configuration](#vite-configuration)

---

## Laravel Breeze

Breeze is a minimal, simple authentication scaffolding for Laravel.

### Installation

```bash
# Create new Laravel project
composer create-project laravel/laravel my-app
cd my-app

# Install Breeze
composer require laravel/breeze --dev

# Choose your stack
php artisan breeze:install blade    # Blade templates
php artisan breeze:install react    # React with Inertia
php artisan breeze:install vue      # Vue with Inertia
php artisan breeze:install api      # API only

# Install dependencies
npm install

# Run migrations
php artisan migrate

# Start development
php artisan serve
npm run dev
```

### Breeze Features

- Login / Registration
- Password Reset
- Email Verification
- Password Confirmation
- Profile Management
- Dark Mode Support
- TypeScript Support (optional)

### Customizing Breeze

#### Add Additional Fields to Registration

**Update migration:**

```bash
php artisan make:migration add_phone_to_users_table --table=users
```

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->nullable()->after('email');
});
```

**Update registration request:**

```php
// app/Http/Requests/Auth/RegisterRequest.php
public function rules(): array
{
    return [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'lowercase', 'email', 'max:255', 'unique:'.User::class],
        'password' => ['required', 'confirmed', Rules\Password::defaults()],
        'phone' => ['nullable', 'string', 'max:20'],
    ];
}
```

**Update registration view:**

```blade
<!-- resources/views/auth/register.blade.php -->
<div>
    <x-input-label for="phone" :value="__('Phone')" />
    <x-text-input id="phone" class="block mt-1 w-full" 
                  type="text" name="phone" :value="old('phone')" required />
    <x-input-error :messages="$errors->get('phone')" class="mt-2" />
</div>
```

**Update User creation:**

```php
// app/Http/Controllers/Auth/RegisteredUserController.php
User::create([
    'name' => $request->name,
    'email' => $request->email,
    'password' => Hash::make($request->password),
    'phone' => $request->phone,
]);
```

---

## Laravel Jetstream

Jetstream is a more feature-rich application starter kit.

### Installation

```bash
composer require laravel/jetstream

# Choose stack
php artisan jetstream:install livewire  # Livewire + Alpine
php artisan jetstream:install inertia   # Vue/React + Inertia

npm install && npm run build
php artisan migrate
```

### Jetstream Features

- All Breeze features plus:
- Two-Factor Authentication
- Team Management
- Profile Photos
- Browser Session Management
- Delete Account
- Terms & Privacy Policy
- API Tokens (Sanctum)

### Team Management

Jetstream includes built-in team support:

```php
use Laravel\Jetstream\HasTeams;

class User extends Authenticatable
{
    use HasTeams;
}
```

Access current team:

```php
$team = auth()->user()->currentTeam;
$members = $team->users;
$team->addUser($user);
$team->removeUser($user);
```

---

## React.js Integration

### Using Breeze with React

```bash
php artisan breeze:install react
```

This installs:
- React via Inertia.js
- Tailwind CSS
- Pre-built auth components
- TypeScript support (optional)

### Project Structure

```
resources/js/
├── Components/
│   ├── ApplicationLogo.jsx
│   ├── Button.jsx
│   ├── InputError.jsx
│   └── ...
├── Layouts/
│   ├── AuthenticatedLayout.jsx
│   └── GuestLayout.jsx
├── Pages/
│   ├── Auth/
│   │   ├── Login.jsx
│   │   └── Register.jsx
│   ├── Dashboard.jsx
│   └── Welcome.jsx
├── app.jsx
└── bootstrap.js
```

### Creating React Components

```jsx
// resources/js/Components/UserList.jsx
import { Link } from '@inertiajs/react';

export default function UserList({ users }) {
    return (
        <div className="overflow-hidden shadow-sm sm:rounded-lg">
            <div className="p-6 text-gray-900">
                <h2 className="text-xl font-semibold mb-4">Users</h2>
                
                <table className="min-w-full divide-y divide-gray-200">
                    <thead>
                        <tr>
                            <th>Name</th>
                            <th>Email</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {users.map((user) => (
                            <tr key={user.id}>
                                <td>{user.name}</td>
                                <td>{user.email}</td>
                                <td>
                                    <Link 
                                        href={`/users/${user.id}`}
                                        className="text-blue-600 hover:text-blue-900"
                                    >
                                        View
                                    </Link>
                                </td>
                            </tr>
                        ))}
                    </tbody>
                </table>
            </div>
        </div>
    );
}
```

### Using Components in Pages

```jsx
// resources/js/Pages/Users/Index.jsx
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout';
import UserList from '@/Components/UserList';
import { Head } from '@inertiajs/react';

export default function Index({ auth, users }) {
    return (
        <AuthenticatedLayout
            user={auth.user}
            header={
                <h2 className="font-semibold text-xl text-gray-800 leading-tight">
                    Users
                </h2>
            }
        >
            <Head title="Users" />
            
            <div className="py-12">
                <div className="max-w-7xl mx-auto sm:px-6 lg:px-8">
                    <UserList users={users} />
                </div>
            </div>
        </AuthenticatedLayout>
    );
}
```

### Form Handling with Inertia

```jsx
import { useForm, Head, Link } from '@inertiajs/react';

export default function Create() {
    const { data, setData, post, processing, errors, reset } = useForm({
        name: '',
        email: '',
        password: '',
        password_confirmation: '',
    });

    const submit = (e) => {
        e.preventDefault();
        post(route('users.store'), {
            onSuccess: () => reset('password', 'password_confirmation'),
        });
    };

    return (
        <form onSubmit={submit}>
            <input
                value={data.name}
                onChange={(e) => setData('name', e.target.value)}
                className={errors.name ? 'border-red-500' : ''}
            />
            {errors.name && <div className="text-red-500">{errors.name}</div>}
            
            <button disabled={processing}>Submit</button>
        </form>
    );
}
```

---

## Vue.js Integration

### Using Breeze with Vue

```bash
php artisan breeze:install vue
```

### Creating Vue Components

```vue
<!-- resources/js/Components/UserList.vue -->
<script setup>
import { Link } from '@inertiajs/vue3';

defineProps({
    users: Array,
});
</script>

<template>
    <div class="overflow-hidden shadow-sm sm:rounded-lg">
        <div class="p-6 text-gray-900">
            <h2 class="text-xl font-semibold mb-4">Users</h2>
            
            <table class="min-w-full divide-y divide-gray-200">
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Email</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="user in users" :key="user.id">
                        <td>{{ user.name }}</td>
                        <td>{{ user.email }}</td>
                        <td>
                            <Link 
                                :href="`/users/${user.id}`"
                                class="text-blue-600 hover:text-blue-900"
                            >
                                View
                            </Link>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>
```

### Using Composition API

```vue
<script setup>
import { ref, computed } from 'vue';
import { useForm, usePage } from '@inertiajs/vue3';

const props = defineProps({
    users: Array,
});

const search = ref('');

const filteredUsers = computed(() => {
    if (!search.value) return props.users;
    return props.users.filter(user => 
        user.name.toLowerCase().includes(search.value.toLowerCase())
    );
});

const form = useForm({
    name: '',
    email: '',
});

const submit = () => {
    form.post(route('users.store'));
};
</script>
```

---

## Inertia.js

Inertia.js allows you to build SPAs without building an API.

### How It Works

```
Browser → Laravel Controller → Inertia Response → Vue/React Component
```

### Server-Side Rendering

```php
use Inertia\Inertia;

public function index()
{
    return Inertia::render('Users/Index', [
        'users' => User::all(),
        'filters' => [
            'search' => request('search'),
        ],
    ]);
}
```

### Shared Data

```php
// app/Http/Middleware/HandleInertiaRequests.php
public function share(Request $request): array
{
    return array_merge(parent::share($request), [
        'flash' => [
            'success' => fn () => $request->session()->get('success'),
            'error' => fn () => $request->session()->get('error'),
        ],
        'appName' => config('app.name'),
    ]);
}
```

### Client-Side Navigation

```jsx
import { Link, router } from '@inertiajs/react';

// Link component (like <a> tag but SPA navigation)
<Link href="/users">Users</Link>

// Programmatic navigation
router.get('/users');
router.post('/users', formData);
router.put(`/users/${id}`, formData);
router.delete(`/users/${id}`);

// With callbacks
router.post('/users', formData, {
    onSuccess: (page) => console.log('Success!'),
    onError: (errors) => console.log(errors),
    onFinish: () => console.log('Done!'),
});
```

### Handling Responses

```jsx
import { usePage } from '@inertiajs/react';

const { flash } = usePage().props;

// Show flash message
{flash.success && (
    <div className="alert alert-success">{flash.success}</div>
)}
```

---

## Livewire

Livewire allows you to write dynamic interfaces with PHP.

### Installation

```bash
composer require livewire/livewire
```

### Creating Components

```bash
php artisan make:livewire CreateUser
php artisan make:livewire UserTable --inline
```

### Full Component

```php
<?php

namespace App\Livewire;

use App\Models\User;
use Livewire\Component;

class CreateUser extends Component
{
    public string $name = '';
    public string $email = '';
    public string $password = '';
    
    protected $rules = [
        'name' => 'required|min:3',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8',
    ];
    
    public function save()
    {
        $this->validate();
        
        User::create([
            'name' => $this->name,
            'email' => $this->email,
            'password' => bcrypt($this->password),
        ]);
        
        session()->flash('message', 'User created successfully!');
        
        $this->reset();
        
        $this->dispatch('user-created');
    }
    
    public function render()
    {
        return view('livewire.create-user');
    }
}
```

```blade
<!-- resources/views/livewire/create-user.blade.php -->
<div>
    @if (session()->has('message'))
        <div class="alert alert-success">
            {{ session('message') }}
        </div>
    @endif
    
    <form wire:submit="save">
        <input 
            type="text" 
            wire:model="name"
            placeholder="Name"
            class="@error('name') border-red-500 @enderror"
        >
        @error('name') <span class="error">{{ $message }}</span> @enderror
        
        <input 
            type="email" 
            wire:model="email"
            placeholder="Email"
        >
        @error('email') <span class="error">{{ $message }}</span> @enderror
        
        <input 
            type="password" 
            wire:model="password"
            placeholder="Password"
        >
        @error('password') <span class="error">{{ $message }}</span> @enderror
        
        <button type="submit">Create User</button>
    </form>
</div>
```

### Lifecycle Hooks

```php
class UserTable extends Component
{
    public function mount()
    {
        // Runs once when component is initialized
    }
    
    public function hydrate()
    {
        // Runs on every request
    }
    
    public function updating($name, $value)
    {
        // Before property update
    }
    
    public function updated($name, $value)
    {
        // After property update
    }
    
    public function updatingName($value)
    {
        // Before specific property update
    }
    
    public function updatedName($value)
    {
        // After specific property update
    }
}
```

### Events

```php
// Dispatch event
$this->dispatch('user-created', userId: $user->id);

// Listen in component
protected $listeners = ['user-created' => 'refreshUsers'];

public function refreshUsers()
{
    // Refresh data
}

// Dispatch to parent
$this->dispatch('close-modal')->self();

// Dispatch to all
$this->dispatch('notification', message: 'Saved!');
```

---

## Vite Configuration

### Basic Setup

Laravel uses Vite by default for asset bundling.

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,
        }),
        react(),
    ],
});
```

### Multiple Entry Points

```javascript
export default defineConfig({
    plugins: [
        laravel({
            input: {
                'app': ['resources/css/app.css', 'resources/js/app.js'],
                'admin': ['resources/css/admin.css', 'resources/js/admin.js'],
            },
            refresh: true,
        }),
    ],
});
```

```blade
<!-- In layout -->
@vite(['resources/css/app.css', 'resources/js/app.js'])
@vite(['resources/css/admin.css', 'resources/js/admin.js'])
```

### Environment Variables

```javascript
// .env
VITE_APP_NAME="${APP_NAME}"
VITE_API_URL="${API_URL}"
```

```javascript
// In JS
const appName = import.meta.env.VITE_APP_NAME;
const apiUrl = import.meta.env.VITE_API_URL;
```

### Build Commands

```bash
# Development with hot reload
npm run dev

# Production build
npm run build

# Preview production build
npm run preview
```

---

## Practice Exercises

1. Install Breeze with React and customize the registration form
2. Create a CRUD interface using Livewire
3. Build a dashboard with Vue.js and Inertia
4. Implement real-time notifications with Livewire events
5. Create reusable React/Vue components for your project
6. Set up Vite for multiple entry points (app + admin)

---

## Next Steps

Continue to Tailwind CSS & Frontend Styling!
