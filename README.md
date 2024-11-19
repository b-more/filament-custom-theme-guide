# Laravel Filament Custom Theme Setup Guide

A comprehensive guide for setting up a custom theme in Laravel Filament admin panel.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Configuration Files](#configuration-files)
- [Running the Application](#running-the-application)
- [File Structure](#file-structure)

## Prerequisites
- PHP >= 8.1
- Laravel (latest version recommended)
- Node.js & NPM
- Composer

## Installation Steps

### 1. Create New Laravel Project
```bash
laravel new project_name
cd project_name
```

### 2. Install Filament and Required Packages
```bash
composer require filament/filament
composer require filament/notifications
composer require filament/forms
composer require filament/tables
composer require filament/actions
```

### 3. Install NPM Dependencies
```bash
npm install -D tailwindcss @tailwindcss/forms @tailwindcss/typography autoprefixer
npm install
```

### 4. Create Filament Admin Panel
```bash
php artisan filament:install --panels
```

### 5. Create Admin User
```bash
php artisan make:filament-user
```

### 6. Create Theme Directory Structure
```bash
mkdir -p resources/css/filament/admin
touch resources/css/filament/admin/theme.css
touch resources/css/filament/admin/tailwind.config.js
```

## Configuration Files

### 1. Vite Configuration (`vite.config.js`)
```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

const refreshPaths = [
    'resources/**',
    'routes/**',
    'config/**',
];

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css', 
                'resources/js/app.js',
                'resources/css/filament/admin/theme.css'
            ],
            refresh: [
                ...refreshPaths,
                'app/Filament/**',
                'app/Forms/Components/**',
                'app/Livewire/**',
                'app/Infolists/Components/**',
                'app/Providers/Filament/**',
                'app/Tables/Columns/**',
            ],
        }),
    ],
});
```

### 2. Filament Theme CSS (`resources/css/filament/admin/theme.css`)
```css
@import "../../../../vendor/filament/filament/resources/css/theme.css";
@config './tailwind.config.js';

/* Sidebar Styles */
.fi-sidebar {
    @apply bg-transparent !important;
}

.fi-sidebar-header {
    @apply bg-transparent shadow-transparent !important;
}

.fi-sidebar-item-label {
    @apply text-[#209F59] hover:bg-transparent hover:text-[#ffffff] !important;
}

.fi-sidebar-item {
    @apply bg-transparent !important;
}

.fi-sidebar-item-button {
    @apply bg-transparent hover:bg-[#209F59] hover:bg-opacity-20 !important;
}

.fi-sidebar-item .fi-active .fi-sidebar-item-active {
    @apply hover:bg-transparent border-r-8 border-[#885029] text-[#00747B] bg-transparent !important;
}

.fi-sidebar-item-active .fi-sidebar-item-label {
    @apply text-[#00747B] hover:bg-transparent !important;
}

.fi-sidebar-item-active .fi-sidebar-item-button {
    @apply bg-[#209F59] bg-opacity-30 rounded-lg border-r-4 border-[#209F59] text-[#00747B] !important;
}

.fi-sidebar-item-active .fi-sidebar-item-icon {
    @apply p-1 rounded-lg bg-opacity-20 shadow-md border border-[#209F59] group-hover:border-transparent text-[#209F59] !important;
}

.fi-sidebar-item-icon {
    @apply p-1 rounded-lg bg-opacity-20 shadow-md border border-[#00747B] group-hover:border-transparent text-[#885029] !important;
}

.fi-sidebar-group-label {
    @apply text-[#458082] text-sm !important;
}

/* Topbar Styles */
.fi-topbar nav {
    @apply bg-transparent !important;
}

.fi-topbar {
    @apply shadow-[#885029] !important;
}

/* Body Styles */
.fi-body {
    @apply bg-[#C6E0CD] dark:bg-[#2F4858] !important;
}

/* Header Styles */
.fi-header-heading {
    @apply text-2xl text-[#008673] !important;
}

.fi-icon-btn-icon {
    @apply text-[#885029] !important;
}

.fi-header-subheading {
    @apply text-sm !important;
}
```

### 3. Filament Tailwind Config (`resources/css/filament/admin/tailwind.config.js`)
```javascript
import preset from '../../../../vendor/filament/support/tailwind.config.preset'

export default {
    presets: [preset],
    content: [
        './app/Filament/**/*.php',
        './resources/views/filament/**/*.blade.php',
        './vendor/filament/**/*.blade.php',
    ],
    theme: {
        extend: {
            colors: {
                primary: '#209F59',
                secondary: '#00747B',
                accent: '#885029',
                background: '#C6E0CD',
                'background-dark': '#2F4858',
            }
        }
    }
}
```

### 4. Root Tailwind Config (`tailwind.config.js`)
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: [
        './resources/**/*.blade.php',
        './resources/**/*.js',
        './resources/**/*.vue',
        // Exclude Filament admin files to avoid conflicts
        '!./resources/css/filament/**/*.{css,js}',
    ],
    theme: {
        extend: {},
    },
    plugins: [
        require('@tailwindcss/typography'),
        require('@tailwindcss/forms')
    ],
}
```

### 5. Update Admin Panel Provider (`app/Providers/Filament/AdminPanelProvider.php`)
```php
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        ->default()
        ->id('admin')
        ->path('admin')
        ->login()
        ->colors([
            'primary' => Color::hex('#209F59'),
        ])
        ->discoverResources(in: app_path('Filament/Resources'))
        ->discoverPages(in: app_path('Filament/Pages'))
        ->pages([
            Pages\Dashboard::class,
        ])
        ->discoverWidgets(in: app_path('Filament/Widgets'))
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
        ])
        ->viteTheme('resources/css/filament/admin/theme.css');
}
```

## Running the Application

### 1. Clear Cache and Build Assets
```bash
# Clear cached config
php artisan optimize:clear

# Build assets
npm run build

# Rebuild Filament assets
php artisan filament:assets
```

### 2. Start Development Servers
```bash
# In one terminal
npm run dev

# In another terminal
php artisan serve
```

### 3. Access Admin Panel
- Visit: `http://localhost:8000/admin`
- Login with your admin credentials

## File Structure
```
project_root/
├── app/
│   └── Providers/
│       └── Filament/
│           └── AdminPanelProvider.php
├── resources/
│   └── css/
│       └── filament/
│           └── admin/
│               ├── theme.css
│               └── tailwind.config.js
├── tailwind.config.js
└── vite.config.js
```

## Customization

To modify the theme colors:
1. Update the color values in `resources/css/filament/admin/theme.css`
2. Update the color palette in `resources/css/filament/admin/tailwind.config.js`
3. Rebuild assets using the commands in the [Running the Application](#running-the-application) section

## Notes
- Always run `npm run build` and `php artisan filament:assets` after making changes to theme files
- Keep these files as templates for future projects
- The color scheme can be customized by modifying the hex values in the theme files
- Make sure all required dependencies are properly installed before starting

## Troubleshooting
If you encounter any styling issues:
1. Clear all caches: `php artisan optimize:clear`
2. Rebuild all assets: `npm run build`
3. Rebuild Filament assets: `php artisan filament:assets`
4. Restart development servers
