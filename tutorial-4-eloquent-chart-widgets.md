# Filament Widgets Tutorial Part 4: Generating Chart Data from Eloquent Models

In our previous tutorial, we created a Chart widget with static data. While useful for demonstration, real-world applications require dynamic data directly from your database.

In this tutorial, we'll learn how to generate chart data from an Eloquent model using the powerful `flowframe/laravel-trend` package, which is recommended by Filament for this purpose.

---

## Why Use a Package for Trends?

Aggregating data over time (e.g., daily, monthly, yearly) can be complex. You need to write database queries that group by different time intervals, which can be tedious and error-prone.

The `flowframe/laravel-trend` package simplifies this process by providing a fluent, easy-to-use API for generating trend data from your Eloquent models.

---

## Installing the Package

First, you need to install the package into your Laravel project using Composer:

```bash
composer require flowframe/laravel-trend
```

---

## Basic Usage: A Simple Monthly Trend

Before diving into dynamic filters, let's start with a basic example. We'll create a chart that shows the number of posts created each month for the current year.

Update `app/Filament/Widgets/PostChartWidget.php` with the following code:

```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Filament\Widgets\ChartWidget;
use Flowframe\Trend\Trend;
use Flowframe\Trend\TrendValue;

class PostChartWidget extends ChartWidget
{
    protected static ?string $heading = 'Post Creations - Monthly';
    protected static string $color = 'info';
    protected static bool $isLazy = true;

    protected function getData(): array
    {
        $data = Trend::model(Post::class)
            ->between(
                start: now()->startOfYear(),
                end: now()->endOfYear(),
            )
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

-   **`Trend::model(Post::class)`**: We start by telling the package which Eloquent model to analyze.
-   **`->between(...)`**: We define the time range. In this case, from the beginning of the current year to the end of the year.
-   **`->perMonth()`**: This is the magic! It tells the package to group the results by month. You can also use `perDay()`, `perHour()`, etc.
-   **`->count()`**: This is the aggregate function. We're counting the number of posts.
-   **`labels` and `datasets`**: The `$data` variable now holds a collection of `TrendValue` objects. We map over this collection to get the month name for the labels and the aggregated count for the chart data.

---

## Advanced Usage: Generating Chart Data with Filters

Now that we understand the basics, let's bring back the filters to create a more interactive chart. This allows the user to change the time period they want to see.

Here is the full code for a widget that filters data by day, week, month, and year.


```php
<?php

namespace App\Filament\Widgets;

use App\Models\Post;
use Filament\Widgets\ChartWidget;
use Flowframe\Trend\Trend;
use Flowframe\Trend\TrendValue;

class PostChartWidget extends ChartWidget
{
    protected static ?string $heading = 'Posts Created Over Time';
    protected static string $color = 'info';
    public ?string $filter = 'today'; // Default filter
    protected static bool $isLazy = true;

    protected function getFilters(): ?array
    {
        return [
            'today' => 'Today',
            'week' => 'Last week',
            'month' => 'Last month',
            'year' => 'This year',
        ];
    }

    protected function getData(): array
    {
        $activeFilter = $this->filter;

        $data = Trend::model(Post::class)
            ->between(
                start: match ($activeFilter) {
                    'today' => now()->startOfDay(),
                    'week' => now()->subWeek()->startOfDay(),
                    'month' => now()->subMonth()->startOfDay(),
                    'year' => now()->startOfYear(),
                    default => now()->startOfYear(),
                },
                end: now()->endOfDay(),
            )
            ->per(
                match ($activeFilter) {
                    'today' => 'hour',
                    'week' => 'day',
                    'month' => 'day',
                    'year' => 'month',
                    default => 'month',
                }
            )
            ->count();

        return [
            'datasets' => [
                [
                    'label' => 'Posts',
                    'data' => $data->map(fn (TrendValue $value) => $value->aggregate),
                ],
            ],
            'labels' => $data->map(fn (TrendValue $value) => match($this->filter) {
                'today' => (new \DateTime($value->date))->format('H:i'),
                'week' => (new \DateTime($value->date))->format('D'),
                'month' => (new \DateTime($value->date))->format('d'),
                'year' => (new \DateTime($value->date))->format('M'),
                default => $value->date,
            }),
        ];
    }

    protected function getType(): string
    {
        return 'line';
    }
}
```

### Code Breakdown:

-   **`getFilters()`**: We define the same time-based filters as in the previous tutorial.
-   **`getData()`**: This is where we use the `laravel-trend` package.
    -   `Trend::model(Post::class)`: We specify that we want to analyze the `Post` model.
    -   `->between(start: ..., end: ...)`: We set the date range for our analysis. The `start` date is dynamically set based on the `$activeFilter`.
    -   `->per(...)`: This method sets the interval for the data points. For the 'year' filter, we get data `perMonth()`. For 'week' and 'month', we get it `perDay()`, and for 'today', `perHour()`.
    -   `->count()`: This is the aggregate function. We are counting the number of records in each interval. You could also use `sum('column')`, `average('column')`, etc.
-   **`datasets` and `labels`**: The `Trend` object returns a collection of `TrendValue` objects.
    -   We use `map()` to extract the `aggregate` (the count) for our chart's `data`.
    -   We use `map()` again to extract the `date` for our chart's `labels`, formatting it appropriately for the selected filter.

---

## Conclusion

You've now learned how to create dynamic, database-driven charts in Filament. By combining Chart widgets with the `flowframe/laravel-trend` package, you can easily create powerful and insightful data visualizations for your admin panel.

This concludes our series on Filament widgets. You now have the skills to build a wide variety of widgets to make your applications more informative and user-friendly.
