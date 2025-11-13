# Laravel Filament Roles Setup Tutorial

This tutorial focuses on setting up **user roles** in a Laravel Filament project using Enums, migrations, seeding, and Filament resource configuration.

---

## Overview

In this tutorial, we'll implement a three-tier role system:

- **Subscriber**: Basic user with limited access  
- **Editor**: Can manage content but not users  
- **Admin**: Full system access  

This will establish the foundation for a role-based access control system.

---

## Creating the Role Enum

Create a `RoleEnum` to define roles:

```php
<?php

namespace App;

enum RoleEnum: string
{
    case Subscriber = 'subscriber';
    case Editor = 'editor';
    case Admin = 'admin';

    public static function values(): array
    {
        return array_column(self::cases(), 'value');
    }
}
```

### Why Enums?
- Type safety for roles  
- Centralized role management  
- Easy to use in validation and Filament forms  

---

## Database Migration

Add a `role` column to the users table:

```php
php artisan make:migration add_role_to_users_table --table=users
```

Inside the migration:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('role')->default('subscriber');
});
```

Run the migration:

```bash
php artisan migrate
```

---

## User Model Setup

Update the `User` model:

```php
protected function casts(): array
{
    return [

        'role' => RoleEnum::class, // Cast to enum
    ];
}

public function isSubscriber(): bool { return $this->role === RoleEnum::Subscriber; }
public function isEditor(): bool { return $this->role === RoleEnum::Editor; }
public function isAdmin(): bool { return $this->role === RoleEnum::Admin; }
```

✔ Enum casting ensures type safety  
✔ Helper methods simplify policy checks  

---

## User Resource Configuration (Filament)

Edit `UserResource`:

- **Form**: add role dropdown, password confirmation, validations  
- **Table**: role badges, filters, searchable columns  

```php
Select::make('role')
    ->required()
    ->options(RoleEnum::class);
```

Color-coded role display:

```php

TextColumn::make('role')
        ->badge()
        ->color(fn (Role $state): string => $state->getColor())

```

---

## Database Seeding

Add test users:

```php
 User::factory()->create([
            'name' => 'subsciber',
            'email' => 'subsciber@example.com',
            'password' => 'password',
            'role' => 'subscriber',
        ]);

        User::factory()->create([
            'name' => 'Editor',
            'email' => 'editor@example.com',
            'password' => 'password',
            'role' => 'editor',
        ]);

        User::factory()->create([
            'name' => 'Admin',
            'email' => 'admin@example.com',
            'password' => 'password',
            'role' => 'admin',
        ]);

```

Run:

```bash
php artisan db:seed
```

---

## Testing Basic Setup

1. Log in as different seeded users.  
2. Verify role badges in the Filament UI.  
3. Confirm role dropdown works when editing/creating users.  

---

## Conclusion

By the end of this tutorial, you have:

- Enum-based roles in DB  
- User model helpers  
- Filament UserResource configured  
- Seeded accounts for testing  

This provides the foundation to apply **policies and permissions**, which we’ll cover in the next tutorial.
