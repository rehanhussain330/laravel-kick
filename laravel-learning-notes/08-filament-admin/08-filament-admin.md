# Laravel Filament Admin Panel - Complete Guide

## Table of Contents
1. [Introduction to Filament](#introduction-to-filament)
2. [Installation & Setup](#installation--setup)
3. [Resources (CRUD)](#resources-crud)
4. [Relations Managers](#relation-managers)
5. [Pages](#pages)
6. [Widgets](#widgets)
7. [Forms](#forms)
8. [Tables](#tables)
9. [Customization](#customization)
10. [Advanced Features](#advanced-features)

---

## Introduction to Filament

Filament is a full-stack framework for building Laravel admin panels, customer-facing apps, and more.

### Key Features:
- Beautiful UI out of the box
- CRUD operations (Resources)
- Forms with validation
- Tables with sorting/filtering
- Dashboard widgets
- Notifications
- Dark mode support
- Mobile responsive
- Extensible architecture

### Versions:
- **Filament v3**: Latest version (recommended)
- Built on TALL stack (Tailwind, Alpine, Laravel, Livewire)

---

## Installation & Setup

### Step 1: Install Filament

```bash
# In your Laravel project
composer require filament/filament:"^3.2" -W

# Create admin user
php artisan make:filament-user

# Start development server
php artisan serve
```

Visit: `http://localhost:8000/admin`

### Step 2: Create Panel Provider (Optional)

```bash
php artisan make:filament-panel admin
```

This creates `app/Providers/Filament/AdminPanelProvider.php`:

```php
<?php

namespace App\Providers\Filament;

use Filament\Http\Middleware\Authenticate;
use Filament\Http\Middleware\DisableBladeIconComponents;
use Filament\Http\Middleware\DispatchServingFilamentEvent;
use Filament\Pages;
use Filament\Panel;
use Filament\PanelProvider;
use Filament\Support\Colors\Color;
use Filament\Widgets;
use Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse;
use Illuminate\Cookie\Middleware\EncryptCookies;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Routing\Middleware\SubstituteBindings;
use Illuminate\Session\Middleware\AuthenticateSession;
use Illuminate\Session\Middleware\StartSession;
use Illuminate\View\Middleware\ShareErrorsFromSession;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->default()
            ->id('admin')
            ->path('admin')
            ->login()
            ->colors([
                'primary' => Color::Amber,
            ])
            ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
            ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
            ->pages([
                Pages\Dashboard::class,
            ])
            ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets')
            ->widgets([
                Widgets\AccountWidget::class,
                Widgets\FilamentInfoWidget::class,
            ])
            ->middleware([
                EncryptCookies::class,
                AddQueuedCookiesToResponse::class,
                StartSession::class,
                AuthenticateSession::class,
                ShareErrorsFromSession::class,
                VerifyCsrfToken::class,
                SubstituteBindings::class,
                DisableBladeIconComponents::class,
                DispatchServingFilamentEvent::class,
            ])
            ->authMiddleware([
                Authenticate::class,
            ]);
    }
}
```

---

## Resources (CRUD)

### Create Resource

```bash
php artisan make:filament-resource User --generate
php artisan make:filament-resource Post
```

### Resource Structure

```php
<?php

namespace App\Filament\Resources;

use App\Filament\Resources\UserResource\Pages;
use App\Filament\Resources\UserResource\RelationManagers;
use App\Models\User;
use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

class UserResource extends Resource
{
    protected static ?string $model = User::class;
    
    protected static ?string $navigationIcon = 'heroicon-o-users';
    
    protected static ?string $navigationGroup = 'User Management';
    
    protected static ?int $navigationSort = 1;
    
    public static function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\Section::make('User Information')
                    ->schema([
                        Forms\Components\TextInput::make('name')
                            ->required()
                            ->maxLength(255),
                        
                        Forms\Components\TextInput::make('email')
                            ->email()
                            ->required()
                            ->maxLength(255)
                            ->unique(ignoreRecord: true),
                        
                        Forms\Components\DateTimePicker::make('email_verified_at'),
                        
                        Forms\Components\TextInput::make('password')
                            ->password()
                            ->dehydrateStateUsing(fn ($state) => 
                                filled($state) ? bcrypt($state) : null)
                            ->dehydrated(fn ($state) => filled($state))
                            ->required(fn (string $context): bool => 
                                $context === 'create')
                            ->minLength(8),
                    ])->columns(2),
                
                Forms\Components\Select::make('roles')
                    ->multiple()
                    ->relationship('roles', 'name')
                    ->preload(),
                
                Forms\Components\Toggle::make('is_active')
                    ->default(true)
                    ->columnSpanFull(),
            ]);
    }
    
    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                Tables\Columns\TextColumn::make('name')
                    ->searchable()
                    ->sortable(),
                
                Tables\Columns\TextColumn::make('email')
                    ->searchable()
                    ->copyable(),
                
                Tables\Columns\TextColumn::make('email_verified_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
                
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
                
                Tables\Columns\IconColumn::make('is_active')
                    ->boolean(),
            ])
            ->filters([
                Tables\Filters\Filter::make('verified')
                    ->query(fn (Builder $query): Builder => 
                        $query->whereNotNull('email_verified_at')),
                
                Tables\Filters\SelectFilter::make('role')
                    ->relationship('roles', 'name'),
            ])
            ->actions([
                Tables\Actions\ViewAction::make(),
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }
    
    public static function getRelations(): array
    {
        return [
            // Relation managers
        ];
    }
    
    public static function getPages(): array
    {
        return [
            'index' => Pages\ListUsers::route('/'),
            'create' => Pages\CreateUser::route('/create'),
            'view' => Pages\ViewUser::route('/{record}'),
            'edit' => Pages\EditUser::route('/{record}/edit'),
        ];
    }
    
    public static function getNavigationBadge(): ?string
    {
        return static::getModel()::count();
    }
}
```

---

## Relation Managers

Manage related records within a resource:

```bash
php artisan make:filament-relation-manager UserResource posts title
```

```php
<?php

namespace App\Filament\Resources\UserResource\RelationManagers;

use Filament\Forms;
use Filament\Forms\Form;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Tables;
use Filament\Tables\Table;

class PostsRelationManager extends RelationManager
{
    protected static string $relationship = 'posts';
    
    protected static ?string $title = 'Blog Posts';
    
    public function form(Form $form): Form
    {
        return $form
            ->schema([
                Forms\Components\TextInput::make('title')
                    ->required()
                    ->maxLength(255),
                
                Forms\Components\MarkdownEditor::make('content')
                    ->required()
                    ->columnSpanFull(),
            ]);
    }
    
    public function table(Table $table): Table
    {
        return $table
            ->recordTitleAttribute('title')
            ->columns([
                Tables\Columns\TextColumn::make('title')
                    ->searchable(),
                
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime(),
            ])
            ->filters([
                //
            ])
            ->headerActions([
                Tables\Actions\CreateAction::make(),
            ])
            ->actions([
                Tables\Actions\EditAction::make(),
                Tables\Actions\DeleteAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }
}
```

---

## Pages

Custom pages in Filament:

```bash
php artisan make:filament-page Settings
```

```php
<?php

namespace App\Filament\Pages;

use Filament\Pages\Page;

class Settings extends Page
{
    protected static ?string $navigationIcon = 'heroicon-o-cog-6-tooth';
    
    protected static string $view = 'filament.pages.settings';
    
    protected static ?int $navigationSort = 100;
}
```

```blade
<!-- resources/views/filament/pages/settings.blade.php -->
<x-filament-panels::page>
    <div class="grid gap-6">
        <x-filament::section>
            <h3>General Settings</h3>
            <!-- Your settings form -->
        </x-filament::section>
    </div>
</x-filament-panels::page>
```

### Settings Page with Form

```bash
php artisan make:filament-settings-page
```

---

## Widgets

Dashboard widgets:

```bash
php artisan make:filament-widget StatsOverview --stats-overview
php artisan make:filament-widget OrdersChart --chart
```

### Stats Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use App\Models\User;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class StatsOverview extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('Total Users', User::count())
                ->description('All time')
                ->descriptionIcon('heroicon-m-arrow-trending-up')
                ->color('success'),
            
            Stat::make('Total Orders', Order::count())
                ->description('Lifetime orders')
                ->color('primary'),
            
            Stat::make('Revenue', '$' . number_format(Order::sum('total'), 2))
                ->description('Total revenue')
                ->color('success'),
        ];
    }
}
```

### Chart Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Widgets\ChartWidget;

class OrdersChart extends ChartWidget
{
    protected static ?int $sort = 2;
    
    protected function getType(): string
    {
        return 'line';
    }
    
    protected function getData(): array
    {
        return [
            'datasets' => [
                [
                    'label' => 'Orders',
                    'data' => Order::selectRaw('COUNT(*) as count')
                        ->selectRaw('MONTH(created_at) as month')
                        ->groupBy('month')
                        ->pluck('count', 'month'),
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 
                        'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        ];
    }
}
```

### Table Widget

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Order;
use Filament\Tables;
use Filament\Tables\Table;
use Filament\Widgets\TableWidget as BaseWidget;

class RecentOrders extends BaseWidget
{
    protected int | string | array $columnSpan = 'full';
    
    public function table(Table $table): Table
    {
        return $table
            ->query(Order::query()->latest()->limit(10))
            ->columns([
                Tables\Columns\TextColumn::make('id'),
                Tables\Columns\TextColumn::make('customer.name'),
                Tables\Columns\TextColumn::make('total')
                    ->money('USD'),
                Tables\Columns\TextColumn::make('status')
                    ->badge(),
                Tables\Columns\TextColumn::make('created_at')
                    ->dateTime(),
            ]);
    }
}
```

---

## Forms

### Form Components

```php
use Filament\Forms\Components\*;

// Text Input
TextInput::make('name')
    ->required()
    ->maxLength(255)
    ->prefixIcon('heroicon-m-user')
    ->helperText('Enter your full name');

// Select
Select::make('status')
    ->options([
        'draft' => 'Draft',
        'published' => 'Published',
        'archived' => 'Archived',
    ])
    ->default('draft')
    ->searchable()
    ->preload();

// Relationship Select
Select::make('category_id')
    ->relationship('category', 'name')
    ->searchable()
    ->preload()
    ->createOptionForm([
        TextInput::make('name')->required(),
    ]);

// Date Picker
DatePicker::make('birth_date')
    ->native(false)
    ->displayFormat('d/m/Y')
    ->maxDate(now());

// File Upload
FileUpload::make('avatar')
    ->image()
    ->disk('public')
    ->directory('avatars')
    ->maxSize(1024) // KB
    ->imageResizeMode('cover')
    ->imageCropAspectRatio('1:1');

// Rich Text Editor
RichTextEditor::make('content')
    ->fileAttachmentsDisk('public')
    ->fileAttachmentsDirectory('attachments');

// Markdown Editor
MarkdownEditor::make('content')
    ->fileAttachmentsDisk('public');

// Toggle
Toggle::make('is_active')
    ->default(true)
    ->inline(false);

// Checkbox
Checkbox::make('agree_to_terms')
    ->label('I agree to the terms');

// Radio
Radio::make('contact_method')
    ->options([
        'email' => 'Email',
        'phone' => 'Phone',
        'sms' => 'SMS',
    ]);

// Repeater
Repeater::make('skills')
    ->schema([
        TextInput::make('name')->required(),
        TextInput::make('level')->numeric(),
    ])
    ->columns(2)
    ->collapsible()
    ->itemLabel(fn (array $state): ?string => $state['name'] ?? null);

// Key-Value
KeyValue::make('meta')
    ->keyLabel('Property')
    ->valueLabel('Value');

// Tags Input
TagsInput::make('tags')
    ->separator(',')
    ->suggestions(['laravel', 'php', 'javascript']);
```

### Form Layout

```php
use Filament\Forms\Components\Section;
use Filament\Forms\Components\Tabs;
use Filament\Forms\Components\Grid;

// Section
Section::make('User Details')
    ->schema([...])
    ->description('Basic information about the user')
    ->icon('heroicon-o-user')
    ->collapsed()
    ->columns(2);

// Tabs
Tabs::make('Content')
    ->tabs([
        Tabs\Tab::make('Main')
            ->schema([...]),
        Tabs\Tab::make('SEO')
            ->schema([...]),
        Tabs\Tab::make('Settings')
            ->schema([...]),
    ]);

// Grid
Grid::make()
    ->schema([
        TextInput::make('first_name')->columnSpan(1),
        TextInput::make('last_name')->columnSpan(1),
        TextInput::make('email')->columnSpanFull(),
    ]);
```

---

## Tables

### Table Columns

```php
use Filament\Tables\Columns\*;

// Text Column
TextColumn::make('name')
    ->searchable()
    ->sortable()
    ->toggleable()
    ->copyable()
    ->limit(50)
    ->tooltip(fn ($record) => $record->name);

// Icon Column
IconColumn::make('is_active')
    ->boolean()
    ->trueIcon('heroicon-o-check-circle')
    ->falseIcon('heroicon-o-x-circle');

// Badge Column
BadgeColumn::make('status')
    ->colors([
        'danger' => 'draft',
        'warning' => 'review',
        'success' => 'published',
    ]);

// Image Column
ImageColumn::make('avatar')
    ->circular()
    ->size(40);

// Color Column
ColorColumn::make('color')
    ->circular();

// Text Column with Formatting
TextColumn::make('price')
    ->money('USD')
    ->summarize([
        Summarizers\Sum::make()->money('USD'),
        Summarizers\Average::make()->money('USD'),
    ]);

// Date Column
TextColumn::make('created_at')
    ->dateTime()
    ->since()
    ->sortable();
```

### Table Filters

```php
use Filament\Tables\Filters\*;

Filters\SelectFilter::make('status')
    ->options([
        'draft' => 'Draft',
        'published' => 'Published',
    ]);

Filters\TernaryFilter::make('is_active')
    ->placeholder('All')
    ->trueLabel('Active')
    ->falseLabel('Inactive');

Filters\Filter::make('verified')
    ->query(fn ($query) => $query->whereNotNull('email_verified_at'));

Filters\DatePickerFilter::make('created_from')
    ->column('created_at');

Filters\Filter::make('date_range')
    ->form([
        DatePicker::make('from'),
        DatePicker::make('until'),
    ])
    ->query(function ($query, array $data): Builder {
        return $query
            ->when(
                $data['from'],
                fn (Builder $query, $date): Builder => 
                    $query->whereDate('created_at', '>=', $date),
            )
            ->when(
                $data['until'],
                fn (Builder $query, $date): Builder => 
                    $query->whereDate('created_at', '<=', $date),
            );
    });
```

---

## Customization

### Custom Theme

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init
```

```javascript
// tailwind.config.js
module.exports = {
    content: [
        './resources/**/*.blade.php',
        './vendor/filament/**/*.blade.php',
    ],
    theme: {
        extend: {},
    },
    plugins: [],
}
```

### Custom Colors

```php
// In Panel Provider
->colors([
    'primary' => Color::hex('#3B82F6'),
    'secondary' => Color::Gray,
    'success' => Color::Green,
    'danger' => Color::Red,
    'warning' => Color::Orange,
])
```

### Branding

```php
// In Panel Provider
->brandName('My Admin')
->brandLogo(asset('images/logo.svg'))
->darkModeBrandLogo(asset('images/logo-dark.svg'))
->brandLogoHeight('2rem')
->favicon(asset('images/favicon.ico'))
```

### Custom CSS

```php
// In Panel Provider
->viteTheme('resources/css/filament/admin/theme.css')
```

```css
/* resources/css/filament/admin/theme.css */
@import '/vendor/filament/filament/resources/css/theme.css';

@config '../../../../tailwind.config.js';

.fi-brand {
    font-weight: bold;
}
```

---

## Advanced Features

### Actions

```php
use Filament\Actions\Action;

// Simple Action
Action::make('publish')
    ->requiresConfirmation()
    ->action(fn ($record) => $record->update(['is_published' => true]))
    ->successNotificationTitle('Post published!');

// Modal Action
Action::make('delete')
    ->requiresConfirmation()
    ->modalHeading('Delete Post')
    ->modalDescription('Are you sure?')
    ->modalSubmitActionLabel('Yes, delete it')
    ->color('danger')
    ->action(fn ($record) => $record->delete());

// Form Action
Action::make('sendEmail')
    ->form([
        TextInput::make('subject')->required(),
        Textarea::make('message')->required(),
    ])
    ->action(function (array $data, $record) {
        Mail::to($record->email)->send(new CustomMail($data));
    });
```

### Authorization

```php
// In Resource
public static function canViewAny(): bool
{
    return auth()->user()->can('view', static::$model);
}

public static function canCreate(): bool
{
    return auth()->user()->can('create', static::$model);
}

public static function canEdit(Model $record): bool
{
    return auth()->user()->can('update', $record);
}

public static function canDelete(Model $record): bool
{
    return auth()->user()->can('delete', $record);
}
```

### Global Search

Filament includes global search by default. Customize searchable models:

```php
// In Model
use Filament\Models\Contracts\FilamentModel;
use Filament\Panel;

class User extends Authenticatable implements FilamentModel
{
    public function getFilamentSearchResults(): array
    {
        return static::all();
    }
}
```

### Notifications

```php
use Filament\Notifications\Notification;

// Success notification
Notification::make()
    ->title('Post created successfully')
    ->success()
    ->send();

// Warning notification
Notification::make()
    ->title('Low stock warning')
    ->body('Product XYZ is running low')
    ->warning()
    ->send();

// With actions
Notification::make()
    ->title('New order received')
    ->actions([
        \Filament\Notifications\Actions\Action::make('view')
            ->url(route('orders.show', $order))
            ->markAsRead(),
    ])
    ->send();
```

---

## Practice Exercises

1. Create a complete blog admin with posts, categories, tags
2. Build an e-commerce admin with products, orders, customers
3. Create a user management system with roles
4. Build a dashboard with custom widgets
5. Implement file uploads with multiple images
6. Create nested resources with relation managers

---

## Next Steps

Continue to Custom Admin Panel without Filament!
