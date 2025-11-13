
# Filament Widgets Tutorial Part 3: Chart Widgets

Welcome to the final tutorial in our series on Filament widgets! We've covered Stat widgets and resource-specific widgets. Now, it's time to visualize our data with **Chart Widgets**.

In this video, we'll learn how to create chart widgets, add filters to them, and switch between different chart types like 'bar' and 'doughnut'.

---

## Why Chart Widgets?

While Stat widgets are great for showing single numbers, Chart widgets allow you to display trends and comparisons over time. They are perfect for visualizing things like posts per month, user sign-ups, or sales data.

---

## Creating a Chart Widget

Just like other widgets, you can create a Chart widget with a simple Artisan command:

```bash
php artisan make:filament-widget PostChartWidget --chart
```

This command will create a new file at `app/Filament/Widgets/PostChartWidget.php`.

---

## Implementing the Chart Widget

Let's build a chart that shows the number of blog posts created. We'll also add a filter to change the time range.

Here is the complete code for `app/Filament/Widgets/PostChartWidget.php`:

```php
<?php

namespace App\Filament\Widgets;

use Filament\Widgets\ChartWidget;

class PostChartWidget extends ChartWidget
{
    protected static ?string $heading = 'Name of Chart'; // Title of the widget
    protected static string $color = 'info'; // Color of the widget
    public ?string $filter = 'today'; // Default filter value
    protected static bool $isLazy = true; // Enable lazy loading of the widget


    // Define available filters
    protected function getFilters(): ?array
    {
        // set the filter options
        return [
            'today' => 'Today',
            'week' => 'Last week',
            'month' => 'Last month',
            'year' => 'This year',
        ];
    }

    // data for the chart
    protected function getData(): array
    {
        // get the active filter value
        $activeFilter = $this->filter;

        // In a real application, you would fetch data from the database
        // based on the $activeFilter. For this example, we'll use static data.
        $data = match ($activeFilter) {
            'today' => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
            'week' => [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110, 120],
            'month' => [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200],
            'year' => [1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 11000, 12000],
            default => [0, 10, 5, 2, 21, 32, 45, 74, 65, 45, 77, 89],
        };

        return [
            'datasets' => [
                [
                    'label' => 'Blog posts created',
                    'data' => $data,
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
        ];
    }

    protected function getType(): string
    {
        return 'doughnut'; // Type of chart (e.g., 'line', 'bar', 'doughnut', etc.)
    }
}
```

### Code Breakdown:

-   `$heading`: Sets the title that appears at the top of the widget.
-   `getFilters()`: This method returns an array of available filters that will be displayed as a dropdown on the widget.
-   `getData()`: This is where the magic happens. It should return an array containing the `datasets` and `labels` for the chart.
    -   `datasets`: An array of datasets. Each dataset has a `label` and a `data` array.
    -   `labels`: An array of labels for the X-axis.
-   `getType()`: This method determines the type of chart to render.

---

## Changing the Chart Type

The `getType()` method is all you need to change the visualization. Let's see how to switch between a `doughnut` chart and a `bar` chart.

### Doughnut Chart

To display a doughnut chart, simply return `'doughnut'` from the `getType()` method, as we did in the example above.

```php
protected function getType(): string
{
    return 'doughnut';
}
```

This is great for showing proportions, like the percentage of posts in different categories.

### Bar Chart

To change to a bar chart, just update the return value:

```php
protected function getType(): string
{
    return 'bar';
}
```

Bar charts are excellent for comparing values across different groups, like the number of posts created each month.

Filament supports many other chart types, including `line`, `pie`, `polarArea`, and `radar`.

---

## Conclusion

Congratulations! You've completed our series on Filament widgets. You now know how to:

-   Create **Stat widgets** for high-level metrics.
-   Assign widgets to **specific resources**.
-   Build dynamic **Chart widgets** with filters and different chart types.

With these skills, you can build rich, informative dashboards that will make your Filament applications more powerful and user-friendly. Happy coding!
