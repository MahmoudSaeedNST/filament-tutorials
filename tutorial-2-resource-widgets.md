
# Filament Widgets Tutorial Part 2: Resource-Specific Widgets

In our last tutorial, we created a Stat widget that appeared on our main dashboard. But what if you want to display widgets that are relevant only to a specific resource, like on the "Users" or "Posts" page?

In this video, we'll learn how to create widgets for a specific resource and how to place them at the top or bottom of the resource's page.

---

## Why Resource Widgets?

Resource widgets allow you to provide contextual statistics and information. For example, on the "Users" page, you might want to see how many users are admins versus regular users. This information is most useful when you're managing users, not necessarily on the main dashboard.

---

## Creating a Resource Widget

Creating a widget for a resource is similar to creating a regular widget. The key difference is where you create the file. To keep things organized, Filament recommends placing resource-specific widgets in a subdirectory named after the resource.

For our `UserResource`, we can create a widget with this command:

```bash
php artisan make:filament-widget UserWidget --resource=UserResource --stats-overview
```

This will create the file at `app/Filament/Resources/UserResource/Widgets/UserWidget.php`. Because it's associated with `UserResource`, Filament will automatically know where to register it.

---

## Implementing the User Widget

Let's look at the code for our `UserWidget`. This widget will show the total number of users, and a breakdown of users by their role.

Here is the complete code for `app/Filament/Resources/UserResource/Widgets/UserWidget.php`:

```php
<?php

namespace App\Filament\Resources\UserResource\Widgets;

use App\Models\User;
use Filament\Support\Enums\IconPosition;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

// Note: This enum is just for the widget's internal logic.
// In a real app, you would use the Role enum from the User model.
enum Role: string {
    case Admin = 'admin';
    case User = 'user';
}

class UserWidget extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            Stat::make('All Users', User::count())
                ->description('Total number of users')
                ->descriptionIcon('heroicon-o-users', IconPosition::Before)
                ->chart([1, 100, 20, 105, 40, 190, 50])
                ->color('success'),
            Stat::make('Admins', User::where('role', 'admin')->count()),
            Stat::make('Users', User::where('role', 'user')->count()),
        ];
    }
}
```

This is very similar to the `PostWidget` we created before, but it's focused on user data.

---

## Registering Widgets on a Resource Page

To display widgets on a resource's list page, you need to edit the `ManageRecords` page file for that resource. For our `UserResource`, this file is `app/Filament/Resources/UserResource/Pages/ManageUsers.php`.

Inside this file, you have two methods to register widgets:

-   `getHeaderWidgets()`: Displays widgets above the main table.
-   `getFooterWidgets()`: Displays widgets below the main table.

Let's add our `UserWidget` to the header and the `PostWidget` from the previous tutorial to the footer.

Here is the code for `app/Filament/Resources/UserResource/Pages/ManageUsers.php`:

```php
<?php

namespace App\Filament\Resources\UserResource\Pages;

use App\Filament\Resources\UserResource;
use App\Filament\Resources\UserResource\Widgets\UserWidget;
use App\Filament\Widgets\PostWidget;
use Filament\Actions;
use Filament\Resources\Pages\ManageRecords;

class ManageUsers extends ManageRecords
{
    protected static string $resource = UserResource::class;

    protected function getHeaderActions(): array
    {
        return [
            Actions\CreateAction::make(),
        ];
    }

    public function getHeaderWidgets(): array
    {
        return [
            UserWidget::class,
        ];
    }

     public function getFooterWidgets(): array
    {
        return [
            PostWidget::class,
        ];
    }
}
```

And here's the code for `PostWidget.php` for reference:
```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Filament\Support\Enums\IconPosition;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class PostWidget extends BaseWidget
{
    protected function getStats(): array
    {
        return [
            stat::make('All Posts', Post::count())
            ->description('Total number of posts')
            ->descriptionIcon('heroicon-o-newspaper', IconPosition::Before)
            ->chart([1, 100, 20, 105, 40, 190, 50])
            ->color('success'),
            stat::make('Published Posts', Post::where('published', true)->count()),
            stat::make('Draft Posts', Post::where('published', false)->count()),
        ];
    }
}
```

Now, when you visit the "Users" page in your Filament admin panel, you'll see the `UserWidget` at the top and the `PostWidget` at the bottom!

---

## Conclusion

You've now learned how to create widgets that are tied to a specific resource and how to control their placement. This is a powerful way to make your resource pages more informative.

In the final tutorial of this series, we'll dive into Chart widgets to create more complex data visualizations. See you there!
