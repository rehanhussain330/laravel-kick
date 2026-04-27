# Laravel Learning Notes - Complete Guide

## Overview

This is a comprehensive Laravel learning resource covering everything from basics to advanced topics.

## Table of Contents

### 1. Basics (01-basics)
- Introduction to Laravel
- Installation & Setup
- Directory Structure
- Routing
- Controllers
- Views & Blade Templates
- Middleware
- Request & Response Handling

### 2. Architecture (02-architecture)
- MVC Architecture
- Service Container
- Service Providers
- Facades
- Request Lifecycle
- Best Practices

### 3. Database & Eloquent ORM (03-database-eloquent)
- Database Configuration
- Migrations
- Eloquent Models
- Relationships
- Query Builder
- Seeding & Factories
- Advanced Eloquent Features

### 4. Authentication & Authorization (04-authentication)
- Manual Authentication
- Laravel Breeze
- Gates & Policies
- Role-Based Access Control (RBAC)
- API Authentication (Sanctum)
- OAuth (Passport)
- Email Verification
- Password Reset

### 5. Starter Kits & Frontend Integration (05-starter-kits)
- Laravel Breeze (Blade, React, Vue)
- Laravel Jetstream
- React.js with Inertia
- Vue.js with Inertia
- Livewire
- Vite Configuration

### 6. React.js Integration (06-react-integration)
- Setting up React with Laravel
- Inertia.js Fundamentals
- Building SPA Components
- Form Handling
- State Management

### 7. Vue.js Integration (07-vue-integration)
- Setting up Vue with Laravel
- Composition API
- Vue Router with Inertia
- Vuex/Pinia State Management
- Component Libraries

### 8. Filament Admin Panel (08-filament-admin)
- Installation & Setup
- Resources (CRUD)
- Relation Managers
- Forms & Tables
- Widgets
- Customization
- Advanced Features

### 9. Custom Admin Panel (09-custom-admin)
- Project Structure
- Admin Authentication
- Dashboard Creation
- CRUD Operations
- Layout Components
- Middleware & Authorization

### 10. Tailwind CSS (10-tailwind-css)
- Installation & Configuration
- Utility Classes
- Custom Components
- Responsive Design
- Dark Mode
- Forms & UI Elements

### 11. Node.js & Vite (11-nodejs-vite)
- NPM/YARN Package Management
- Vite Configuration
- Asset Compilation
- Hot Module Replacement
- Production Builds

### 12. Packages (12-packages)
- Installing Composer Packages
- Popular Laravel Packages
- Creating Custom Packages
- Package Development Best Practices

### 13. Layouts & Components (13-layouts-components)
- Blade Layouts
- Partial Views
- Blade Components
- Reusable Components
- Component Libraries

### 14. Advanced Topics (14-advanced-topics)
- Queues & Jobs
- Events & Listeners
- Broadcasting
- Caching
- Testing (PHPUnit, Pest)
- API Development
- Performance Optimization
- Security Best Practices

---

## Quick Start Guide

### Prerequisites
- PHP 8.1 or higher
- Composer
- Node.js & NPM
- Database (MySQL, PostgreSQL, SQLite)

### Installation

```bash
# Create new Laravel project
composer create-project laravel/laravel my-app
cd my-app

# Configure environment
cp .env.example .env
php artisan key:generate

# Install frontend dependencies
npm install && npm run dev

# Run migrations
php artisan migrate

# Start development server
php artisan serve
```

### Learning Path

1. **Start with Basics** - Understand routing, controllers, views
2. **Database & Eloquent** - Learn data modeling and relationships
3. **Authentication** - Implement user management
4. **Frontend Integration** - Choose React, Vue, or Livewire
5. **Admin Panel** - Use Filament or build custom
6. **Advanced Topics** - Explore queues, events, testing

---

## File Structure

```
laravel-learning-notes/
├── 01-basics/
│   └── 01-laravel-basics.md
├── 02-architecture/
│   └── 02-architecture-mvc.md
├── 03-database-eloquent/
│   └── 03-database-eloquent.md
├── 04-authentication/
│   └── 04-authentication-authorization.md
├── 05-starter-kits/
│   └── 05-starter-kits-frontend.md
├── 06-react-integration/
│   └── (coming soon)
├── 07-vue-integration/
│   └── (coming soon)
├── 08-filament-admin/
│   └── 08-filament-admin.md
├── 09-custom-admin/
│   └── 09-custom-admin-panel.md
├── 10-tailwind-css/
│   └── (coming soon)
├── 11-nodejs-vite/
│   └── (coming soon)
├── 12-packages/
│   └── (coming soon)
├── 13-layouts-components/
│   └── (coming soon)
└── 14-advanced-topics/
    └── (coming soon)
```

---

## Practice Projects

### Beginner
1. Blog with CRUD operations
2. Contact form with validation
3. User profile management
4. Simple todo list

### Intermediate
1. E-commerce product catalog
2. Social media follow system
3. Booking/reservation system
4. Content management system

### Advanced
1. Full e-commerce platform
2. Real-time chat application
3. Project management tool
4. Multi-tenant SaaS application

---

## Resources

### Official Documentation
- [Laravel Documentation](https://laravel.com/docs)
- [Laracasts](https://laracasts.com)
- [PHP The Right Way](https://phptherightway.com)

### Community
- [Laravel News](https://laravel-news.com)
- [r/laravel](https://reddit.com/r/laravel)
- [Laravel Discord](https://discord.gg/laravel)

### Packages
- [Packagist](https://packagist.org)
- [Spatie](https://spatie.be/en/opensource/laravel)
- [Laravel Collective](https://laravelcollective.com)

---

## Contributing

Feel free to contribute by:
- Adding examples
- Improving explanations
- Fixing errors
- Adding new topics

---

## License

This guide is open source and available for educational purposes.
