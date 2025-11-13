# EP5: Filament 3 Validation

## What You'll Learn
In this episode, we'll cover all the essential validation methods in Filament 3 using simple, practical examples with our CMS project.

---

## 1. Required Field Validation

### Basic Required
```php
TextInput::make('name')
    ->required()
```

### Conditional Required
```php
TextInput::make('password')
    ->required(fn (string $context): bool => $context === 'create')
```
*Only required when creating new records*

---

## 2. Length Validation

### Maximum Length
```php
TextInput::make('name')
    ->maxLength(50)
```

### Minimum Length  
```php
TextInput::make('name')
    ->minLength(3)
```

### Combined Length Rules
```php
TextInput::make('title')
    ->required()
    ->maxLength(255)
    ->minLength(5)
```

---

## 3. Email Validation

### Basic Email
```php
TextInput::make('email')
    ->email()
    ->required()
```

### Email with Uniqueness
```php
TextInput::make('email')
    ->email()
    ->required()
    ->unique(User::class, 'email', ignoreRecord: true)
```

---

## 4. Unique Field Validation

### Basic Unique
```php
TextInput::make('slug')
    ->unique(Category::class, 'slug')
```

### Unique with Ignore Current Record (for editing)
```php
TextInput::make('slug')
    ->unique(Category::class, 'slug', ignoreRecord: true)
```

---

## 5. URL and Slug Validation

### Alpha Dash (letters, numbers, dashes, underscores)
```php
TextInput::make('slug')
    ->alpha_dash()
```

### URL Validation
```php
TextInput::make('website')
    ->url()
```
#### Valid Examples
These are strings that would pass the alpha_dash validation because they only contain letters, numbers, dashes (-), and underscores (_).

- user_name
- my-profile-123
- alpha_dash_example
- another-example

#### Invalid Examples
These strings would fail the alpha_dash validation because they contain characters that aren't allowed, such as spaces, periods (.), or other special symbols.

- user name (contains a space)
- my.profile (contains a period)
- email@example.com (contains @ and .)
- #profile_name (contains #)
- another example! (contains a space and an exclamation mark)

---


## 6. File Upload Validation

### Basic Image Upload
```php
FileUpload::make('thumbnail')
    ->image()
    ->required()
```

### File Size Limits
```php
FileUpload::make('thumbnail')
    ->image()
    ->maxSize(1024) // 1MB
```

### Multiple Files
```php
FileUpload::make('attachments')
    ->multiple()
    ->maxFiles(5)
```

### Accepted File Types
```php
FileUpload::make('document')
    ->acceptedFileTypes(['application/pdf'])
```

---

## 7. Select and Relationship Validation

### Required Select
```php
Select::make('category_id')
    ->relationship('category', 'name')
    ->required()
```

### Select with Options
```php
Select::make('status')
    ->options([
        'draft' => 'Draft',
        'published' => 'Published',
    ])
    ->required()
```

---

## 8. Boolean Validation

### Toggle Required
```php
Toggle::make('published')
    ->required()
```

### Checkbox
```php
Checkbox::make('terms_accepted')
    ->accepted() // Must be checked
```

---

## 9. Array Validation (Tags)

### Basic Tags Input
```php
TagsInput::make('tags')
```

### Tags with Suggestions
```php
TagsInput::make('tags')
    ->suggestions([
        'laravel',
        'php',
        'javascript',
    ])
```

---

## 10. Password Validation

### Basic Password
```php
TextInput::make('password')
    ->password()
    ->required()
    ->minLength(8)
```

### Password with Confirmation
```php
TextInput::make('password')
    ->password()
    ->required()
    ->confirmed(), // Looks for password_confirmation field

TextInput::make('password_confirmation')
    ->password()
    ->required()
    ->dehydrated(false) // Don't save to database
```

---

## 11. Live Validation (Real-time)

### Validate on Blur
```php
TextInput::make('title')
    ->live(onBlur: true)
    ->afterStateUpdated(fn ($state, Forms\Set $set) => $set('slug', Str::slug($state)))
```

### Validate on Every Change
```php
TextInput::make('search')
    ->live()
```

### Validate with Debounce
```php
TextInput::make('search')
    ->live(debounce: 500) // Wait 500ms after typing stops
```

---


---

## Complete Example: Contact Form

```php
public static function form(Form $form): Form
{
    return $form->schema([
        TextInput::make('name')
            ->required()
            ->maxLength(100)
            ->minLength(2),
            
        TextInput::make('email')
            ->email()
            ->required()
            ->maxLength(255),
            
        TextInput::make('phone')
            ->tel()
            ->rules(['regex:/^[0-9\-\+\(\)\s]+$/']),
            
        Select::make('subject')
            ->options([
                'general' => 'General Inquiry',
                'support' => 'Support',
                'billing' => 'Billing',
            ])
            ->required(),
            
        Textarea::make('message')
            ->required()
            ->maxLength(1000)
            ->rows(5),
            
        Toggle::make('newsletter')
            ->label('Subscribe to newsletter'),
            
        Checkbox::make('terms')
            ->accepted()
            ->label('I agree to the terms'),
    ]);
}
```

---

## Quick Reference: Most Used Validations

| Method | Purpose | Example |
|--------|---------|---------|
| `required()` | Field must be filled | `->required()` |
| `maxLength()` | Maximum characters | `->maxLength(50)` |
| `minLength()` | Minimum characters | `->minLength(3)` |
| `email()` | Valid email format | `->email()` |
| `unique()` | Unique in database | `->unique(User::class, 'email')` |
| `alpha_dash()` | Letters, numbers, dash, underscore | `->alpha_dash()` |
| `image()` | Image files only | `->image()` |
| `maxSize()` | File size limit (KB) | `->maxSize(1024)` |
| `live()` | Real-time validation | `->live(onBlur: true)` |

---

## Tips for Beginners

1. **Start Simple**: Use basic validation first, then add complexity
2. **Always Use Required**: Mark important fields as required
3. **Length Limits**: Always set maxLength to prevent database errors  
4. **Live Validation**: Use `live(onBlur: true)` for better UX
5. **Helper Text**: Guide users with helpful text
6. **Test Everything**: Try invalid data to see how validation works

---

## Common Mistakes to Avoid

1. ❌ Forgetting `ignoreRecord: true` on unique validation for edit forms
2. ❌ Not setting maxLength causing database errors
3. ❌ Using `live()` without debounce on complex validation
4. ❌ Forgetting to make password confirmation `dehydrated(false)`
5. ❌ Not providing helpful validation messages

---

## Next Episode Preview

In EP6, we'll explore:
- Search functionality in tables
- Column sorting
- Custom filters
- Advanced table features

# homework

## 12. Custom Validation Messages

### Simple Custom Messages
```php
TextInput::make('name')
    ->required()
    ->maxLength(50)
    ->validationMessages([
        'required' => 'Please enter a name.',
        'max' => 'Name is too long!',
    ])
```

---

## 13. Helper Text

### Guidance Text
```php
TextInput::make('slug')
    ->helperText('URL-friendly version of the title.')
```

### Dynamic Helper Text
```php
TextInput::make('title')
    ->helperText(fn ($state) => 'Characters: ' . strlen($state ?? ''))
    ->live()
```

---

## 14. Placeholder Text

### Input Placeholder
```php
TextInput::make('title')
    ->placeholder('Enter post title...')
```

---

## 15. Regex Validation

### Using Rules Method
```php
TextInput::make('phone')
    ->rules(['regex:/^[0-9\-\+\(\)\s]+$/'])
```

---

## 16. In and Not In Validation

### Allowed Values
```php
Select::make('status')
    ->in(['draft', 'published', 'archived'])
```
## 17. Numeric Validation

### Basic Numeric
```php
TextInput::make('price')
    ->numeric()
```

### Integer Only
```php
TextInput::make('quantity')
    ->integer()
```

### Decimal with Steps
```php
TextInput::make('price')
    ->numeric()
    ->step(0.01)
```

---

## 18. Date and Time Validation

### Date Picker
```php
DatePicker::make('birth_date')
    ->required()
```

### DateTime Picker
```php
DateTimePicker::make('published_at')
    ->native(false)
```

### Date Rules
```php
DatePicker::make('event_date')
    ->after('today')
    ->before('2025-12-31')
```

---
