# Laravel Filament Policies & Permissions Tutorial

This tutorial builds on the **Roles Setup** tutorial and shows how to protect resources with **Laravel Policies** using the role helpers.

---

## Why Policies?

Policies let you define granular access control for models.  
Benefits:
- Centralized authorization logic  
- Granular permissions (per action)  
- Works seamlessly with Filament  

---

## Creating Policies

Example: `CategoryPolicy`

```php
php artisan make:policy CategoryPolicy --model=Category
```

```php
public function viewAny(User $user): bool
{
    return $user->isAdmin() || $user->isEditor();
}

public function create(User $user): bool
{
    return $user->isAdmin();
}
```

✔ Editors can view  
✔ Admins can create/update/delete  
✔ Subscribers have no access  

---


---

## Testing Permissions

1. Log in as **Subscriber** → cannot view/manage categories.  
2. Log in as **Editor** → can view but not create.  
3. Log in as **Admin** → full access.  

---

