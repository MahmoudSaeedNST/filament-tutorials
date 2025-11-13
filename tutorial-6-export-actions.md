# Filament Tutorial Part 6: Export Actions

Welcome to the final tutorial in our Filament series! We've covered widgets, charts, and global filters. Now, we'll explore a crucial feature for any data-driven application: **Export Actions**.

In this tutorial, you'll learn how to add functionality to export your resource data into CSV or XLSX files, customize the exported data, and add bulk export actions.

---

## Why Use Export Actions?

Export Actions provide a simple way for users to download data from your application for offline analysis, reporting, or use in other systems. Filament's prebuilt Export Action makes this incredibly easy to implement.

---

## Step 1: Adding a Basic Export Action

Let's add an export button to the header of our `CategoryResource` table. This will allow users to export all categories.

You need to add the `ExportAction` to the `headerActions` array in your resource's `table()` method.

Update `app/Filament/Resources/CategoryResource.php` with the following code:

```php
<?php

namespace App\Filament\Resources;

use App\Filament\Resources\CategoryResource\Pages;
use App\Filament\Resources\CategoryResource\RelationManagers;
use App\Models\Category;
use Filament\Forms;
use Filament\Forms\Components\ColorPicker;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Form;
use Filament\Resources\Resource;
use Filament\Tables;
use Filament\Tables\Actions\ExportAction;
use Filament\Tables\Columns\ColorColumn;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Table;
use Illuminate\Support\Facades\Auth;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

class CategoryResource extends Resource
{
    // ... existing code

    public static function table(Table $table): Table
    {
        return $table
            ->columns([
                TextColumn::make('id'),
                TextColumn::make('name')
                    ->sortable(),
                TextColumn::make('slug')
                    ->sortable(),
                TextColumn::make('description'),
                ColorColumn::make('color')
                    ->sortable(),
            ])
            ->filters([
                //
            ])
            ->headerActions([
                ExportAction::make()
            ])
            ->actions([
                Tables\Actions\ViewAction::make(),
                Tables\Actions\EditAction::make(),
            ])
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                ]),
            ]);
    }

    // ... existing code
}
```

### Code Breakdown:
- **`use Filament\Tables\Actions\ExportAction;`**: We import the necessary class.
- **`->headerActions([ ExportAction::make() ])`**: We add the `ExportAction` to the table's header.

With just this change, a new "Export" button will appear on your categories list page. Clicking it will open a modal allowing the user to select columns and export the data.

---

## Step 2: Adding a Bulk Export Action

In addition to a global export, you can allow users to export only selected rows. This is done by adding a `ExportBulkAction` to the `bulkActions` array.

Let's update `app/Filament/Resources/CategoryResource.php` again:

```php
<?php
// ...
use Filament\Tables\Actions\ExportBulkAction;

class CategoryResource extends Resource
{
    // ...

    public static function table(Table $table): Table
    {
        return $table
            // ...
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                    ExportBulkAction::make()
                ]),
            ]);
    }

    // ...
}
```

Now, when users select one or more categories, an "Export selected" option will be available in the bulk actions dropdown.

---

## Step 3: Creating a Custom Exporter

By default, Filament exports the columns visible in the table. However, you often need more control over the exported data, such as adding columns that are not in the table, formatting values, or changing column headers.

You can achieve this by creating a custom exporter class. Let's create one for our categories.

Run the following Artisan command:

```bash
php artisan make:filament-exporter CategoryExporter --resource=CategoryResource
```

This will create a new file at `app/Filament/Resources/CategoryResource/Exporters/CategoryExporter.php`.

Now, let's customize this exporter to include the user who created the category.

Here is the code for `app/Filament/Resources/CategoryResource/Exporters/CategoryExporter.php`:

```php
<?php

namespace App\Filament\Resources\CategoryResource\Exporters;

use App\Models\Category;
use Filament\Actions\Exports\ExportColumn;
use Filament\Actions\Exports\Exporter;
use Filament\Actions\Exports\Models\Export;

class CategoryExporter extends Exporter
{
    protected static ?string $model = Category::class;

    public static function getColumns(): array
    {
        return [
            ExportColumn::make('id')
                ->label('ID'),
            ExportColumn::make('name'),
            ExportColumn::make('slug'),
            ExportColumn::make('description'),
            ExportColumn::make('color'),
            ExportColumn::make('user.name')->label('Author'),
            ExportColumn::make('created_at'),
            ExportColumn::make('updated_at'),
        ];
    }

    public static function getCompletedNotificationBody(Export $export): string
    {
        $body = 'Your category export has completed and ' . number_format($export->successful_rows) . ' ' . str('row')->plural($export->successful_rows) . ' have been exported.';

        if ($failedRowsCount = $export->getFailedRowsCount()) {
            $body .= ' ' . number_format($failedRowsCount) . ' ' . str('row')->plural($failedRowsCount) . ' failed to export.';
        }

        return $body;
    }
}
```

### Code Breakdown:
- **`getColumns()`**: This static method defines the columns for the export.
- **`ExportColumn::make('user.name')->label('Author')`**: We're exporting the `name` from the `user` relationship and giving it a custom label "Author". This assumes you have a `user` relationship on your `Category` model.
- **`getCompletedNotificationBody()`**: This method customizes the notification message shown when the export is complete.

---

## Step 4: Using the Custom Exporter

To use our new `CategoryExporter`, we need to associate it with our export actions in `CategoryResource.php`.

Update the `ExportAction` and `ExportBulkAction` like this:

```php
<?php
// ...
use App\Filament\Resources\CategoryResource\Exporters\CategoryExporter;
use Filament\Tables\Actions\ExportAction;
use Filament\Tables\Actions\ExportBulkAction;

class CategoryResource extends Resource
{
    // ...

    public static function table(Table $table): Table
    {
        return $table
            // ...
            ->headerActions([
                ExportAction::make()->exporter(CategoryExporter::class)
            ])
            // ...
            ->bulkActions([
                Tables\Actions\BulkActionGroup::make([
                    Tables\Actions\DeleteBulkAction::make(),
                    ExportBulkAction::make()->exporter(CategoryExporter::class)
                ]),
            ]);
    }

    // ...
}
```

Now, when a user exports categories, it will use our custom exporter, and the "Author" column will be included in the exported file.

---

## Conclusion

Congratulations on completing the Filament tutorial series! You've learned how to add powerful export functionality to your resources, giving you full control over the data that users can download.

You now know how to:
- Add header and bulk export actions.
- Create custom exporters to define specific columns and formats.
- Associate custom exporters with your actions.

With the skills you've gained throughout this series, you are well-equipped to build sophisticated and user-friendly admin panels with Filament. Happy coding!
