
# Filament Widgets Tutorial Part 1: Stat Widgets

Welcome to the first tutorial in our series on Filament widgets! In this video, we'll explore what Stat widgets are, how to create them, and the different ways to register them in your Filament admin panel.

---

## What are Widgets?

In Filament, widgets are components that you can add to your dashboard to display information. They are a great way to provide users with a quick overview of important data.

One of the simplest and most useful types of widgets is the **Stat Widget**. These are used to display key metrics, like the total number of users, new orders, or in our case, blog posts.

>in case that you are in resturant dashboard you can use it to show total orders, pending orders, completed orders, and ingredients in stock.

---

## Creating a Stat Widget

Filament's command-line tools make it incredibly easy to create a new widget. To create a Stat widget, you can run the following Artisan command:

```bash
php artisan make:filament-widget PostWidget --stats-overview
```

This command will generate a new file at `app/Filament/Widgets/PostWidget.php`.

---

## Implementing the Widget

Now, let's look at the code for our `PostWidget`. We'll display the total number of posts, as well as counts for published and draft posts.

Here is the complete code for `app/Filament/Widgets/PostWidget.php`:

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
            Stat::make('All Posts', Post::count())
                ->description('Total number of posts')
                ->descriptionIcon('heroicon-o-newspaper', IconPosition::Before)
                ->chart([1, 100, 20, 105, 40, 190, 50])
                ->color('success'),
            Stat::make('Published Posts', Post::where('published', true)->count()),
            Stat::make('Draft Posts', Post::where('published', false)->count()),
        ];
    }
}
```

### Code Breakdown:

-   `getStats()`: This method returns an array of `Stat` objects. Each `Stat` object represents one card in our overview.
-   `Stat::make('Label', 'Value')`: This is the basic syntax for creating a stat. The first argument is the label, and the second is the value.
-   **Chained Methods**: You can chain several helpful methods to customize the appearance:
    -   `->description()`: Adds a small text description below the main value.
    -   `->descriptionIcon()`: Adds an icon next to the description.
    -   `->chart()`: Adds a small sparkline chart.
    -   `->color()`: Sets the color of the stat card.

---

## Registering the Widget

Filament is smart enough to automatically discover widgets in the `app/Filament/Widgets` directory. As long as your widget is in the correct location, it will appear on your main dashboard page without any extra work!

However, if you need more control or want to register widgets from a different directory, you can do it manually. You would do this in your Panel Provider file, which is usually `app/Providers/Filament/AdminPanelProvider.php`.

Here’s how you would register the `PostWidget` manually:

```php
// app/Providers/Filament/AdminPanelProvider.php

use App\Filament\Widgets\PostWidget;

public function panel(Panel $panel): Panel
{
    return $panel
        // ... other panel configurations
        ->widgets([
            PostWidget::class,
        ]);
}
```

---

## Conclusion

And that's it! You've learned how to create and customize a Stat widget to display important metrics on your dashboard.

In the next tutorial, we'll look at how to create widgets that are specific to a resource and display them on resource pages instead of the main dashboard. Stay tuned!
