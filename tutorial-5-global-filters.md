# Filament Widgets Tutorial Part 5: Global Dashboard Filters

Welcome to Part 5 of our Filament widgets series! So far, we've built various widgets, but they've all operated independently. In this tutorial, we'll explore one of Filament's most powerful features: **Global Filters**.

Global filters allow you to add filter controls (like date pickers or select dropdowns) to your dashboard and have all your widgets react to them in real-time. This creates a cohesive, interactive experience for your users.

---

## Why Use Global Filters?

Imagine a dashboard with multiple widgets showing user sign-ups, post counts, and sales data. Instead of adding separate date filters to each widget, you can have one master date range filter at the top of the page. When the user changes the date range, all widgets update simultaneously.

This is what we'll build today: a custom dashboard with global filters for date range and post status.

---

## Step 1: Create a Custom Dashboard Page

To add a global filter form, we first need a custom dashboard page. The default dashboard doesn't support this out of the box.

Create a new dashboard page using this Artisan command:

```bash
php artisan make:filament-page Dashboard --type=dashboard
```

This command generates a new file at `app/Filament/Pages/Dashboard.php`.

Next, we need to tell Filament to use this as our main dashboard. We do this by removing the default `Dashboard::class` from the `discoverPages()` method in our panel provider, which is usually `app/Providers/Filament/AdminPanelProvider.php`.

```php
// app/Providers/Filament/AdminPanelProvider.php

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->pages([
            // No need to register our new Dashboard, it will be discovered.
        ])
        ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages');
        // ...
}
```
By default, Filament discovers pages, so creating `app/Filament/Pages/Dashboard.php` effectively overrides the default.

---

## Step 2: Add the Filter Form to the Dashboard

Now, let's add the filter form. We'll use the `HasFiltersForm` trait, which provides all the necessary logic.

Update `app/Filament/Pages/Dashboard.php` with the following code:

```php
<?php

namespace App\Filament\Pages;

use Filament\Forms\Components\DatePicker;
use Filament\Forms\Components\Section;
use Filament\Forms\Components\Select;
use Filament\Forms\Form;
use Filament\Pages\Dashboard as BaseDashboard;
use Filament\Pages\Dashboard\Concerns\HasFiltersForm;

class Dashboard extends BaseDashboard
{
    use HasFiltersForm;

    public function filtersForm(Form $form): Form
    {
        return $form
            ->schema([
                Section::make()
                    ->schema([
                        Select::make('published_status')
                            ->options([
                                '1' => 'Published',
                                '0' => 'Unpublished',
                            ])->boolean(),
                        DatePicker::make('start_date'),
                        DatePicker::make('end_date'),
                    ])
                    ->columns(3),
            ]);
    }
}
```

### Code Breakdown:
- **`use HasFiltersForm;`**: This trait adds the filtering functionality to our dashboard page.
- **`filtersForm(Form $form)`**: This method is where you define your filter inputs using the Form Builder.
- We've added a `Select` for the published status and two `DatePicker` inputs for a date range.

---


## Step 3: Modify a Stat Widget to Use Global Filters

Any widget registered on a page with `HasFiltersForm` can access the current filter values via the public `$this->filters` property.

Let's update `app/Filament/Widgets/PostWidget.php` to use these filters.

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Filament\Widgets\StatsOverviewWidget as BaseWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;
use Illuminate\Database\Eloquent\Builder;

class PostWidget extends BaseWidget
{
    use interactWithPageFilters;
    protected function getStats(): array
    {
        // Access filters
        $filters = $this->filters;

        $baseQuery = Post::query()
            ->when($filters['start_date'], fn (Builder $query, $date) => $query->whereDate('created_at', '>=', $date))
            ->when($filters['end_date'], fn (Builder $query, $date) => $query->whereDate('created_at', '<=', $date));

        $allPostsQuery = (clone $baseQuery)
            ->when($filters['published_status'] !== null, fn (Builder $query, $status) => $query->where('published', $status));

        return [
            Stat::make('All Posts', $allPostsQuery->count()),
            Stat::make('Published Posts', (clone $baseQuery)->where('published', true)->count()),
            Stat::make('Draft Posts', (clone $baseQuery)->where('published', false)->count()),
        ];
    }
}
```

### Code Breakdown:
- We retrieve all filters into a `$filters` array.
- We build a `$baseQuery` that includes the date range filters.
- For "All Posts", we clone the base query and add the `published_status` filter if it's set.
- For "Published" and "Draft" posts, we clone the base query and apply the respective hardcoded `where('published', ...)` clause. This ensures all stats respect the date range.

---

## Step 4: Modify a Chart Widget to Use Global Filters

The same principle applies to Chart widgets. Let's update `app/Filament/Widgets/PostChartWidget.php` to use the date range from our global filters.

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Carbon\Carbon;
use Filament\Widgets\ChartWidget;
use Flowframe\Trend\Trend;
use Flowframe\Trend\TrendValue;

class PostChartWidget extends ChartWidget
{
    protected static ?string $heading = 'Posts Created Over Time';
    protected static string $color = 'info';
    protected static bool $isLazy = true;

    protected function getData(): array
    {
        // Access filters
        $filters = $this->filters;
        $startDate = $filters['start_date'] ? Carbon::parse($filters['start_date']) : now()->subMonths(6);
        $endDate = $filters['end_date'] ? Carbon::parse($filters['end_date']) : now();

        $data = Trend::model(Post::class)
            ->between(start: $startDate, end: $endDate)
            ->when($filters['published_status'] !== null, fn ($query) => $query->where('published', $filters['published_status']))
            ->perMonth()
            ->count();

        return [
            'datasets' => [
                [
                    'label' => 'Posts',
                    'data' => $data->map(fn (TrendValue $value) => $value->aggregate),
                ],
            ],
            'labels' => $data->map(fn (TrendValue $value) => (new \DateTime($value->date))->format('M')),
        ];
    }

    protected function getType(): string
    {
        return 'line';
    }
}
```

### Code Breakdown:
- We get the `start_date`, `end_date`, and `published_status` from `$this->filters`.
- We provide a default date range if the filters are not set.
- We chain `->when()` to the `Trend` query to conditionally apply the `published_status` filter.
- The chart will now dynamically update based on both the date range and the published status from the global filters.

---

## Conclusion

Congratulations! You've now created a fully interactive dashboard with global filters. This powerful feature allows users to drill down into the data across multiple widgets at once, providing a much richer user experience.

You've learned how to:
- Create a custom dashboard page.
- Add a filter form using the `HasFiltersForm` trait.
- Register widgets on your custom dashboard.
- Access and apply global filter data in both Stat and Chart widgets.

This concludes our tutorial on global filters. With these skills, you can build highly interactive and data-rich dashboards tailored to your users' needs.
