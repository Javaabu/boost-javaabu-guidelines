# Javaabu Laravel & PHP Guidelines (Reference)

## Table of Contents
- [Core Laravel Principle](#core-laravel-principle)
- [PHP Standards](#php-standards)
- [Class Structure](#class-structure)
- [Type Declarations and Docblocks](#type-declarations-and-docblocks)
- [Control Flow](#control-flow)
- [Laravel Conventions](#laravel-conventions)
- [Strings and Formatting](#strings-and-formatting)
- [Enums](#enums)
- [Comments](#comments)
- [Whitespace](#whitespace)
- [Validation](#validation)
- [Blade Templates](#blade-templates)
- [Authorization](#authorization)
- [Translations](#translations)
- [API Routing](#api-routing)
- [Testing](#testing)
- [Quick Reference](#quick-reference)

## Core Laravel Principle
**Follow Laravel conventions first.** If Laravel has a documented way to do something, use it. Only deviate when you have a clear justification.

## PHP Standards
- Follow PSR-1, PSR-2, and PSR-12
- Use snake_case for class properties and variables
- Use camelCase for methods and function names
- Use short nullable notation: `?string` not `string|null`
- Always specify `void` return types when methods return nothing

## Class Structure
- Use typed properties, not docblocks
- Use constructor property promotion when all properties can be promoted
- Use one trait per line

## Type Declarations and Docblocks
- Use typed properties over docblocks
- Specify return types including `void`
- Use short nullable syntax: `?Type` not `Type|null`
- Document iterables with generics:

```php
/** @return Collection<int, User> */
public function getUsers(): Collection
```

### Docblock Rules
- Don't use docblocks for fully type-hinted methods (unless description needed)
- Always import classnames in docblocks; never use fully qualified names:

```php
use Javaabu\Url\Url;
/** @return Url */
```

- Use one-line docblocks when possible: `/** @var string */`
- Most common type should be first in multi-type docblocks:

```php
/** @var Collection|SomeWeirdVendor\Collection */
```

- If one parameter needs docblock, add docblocks for all parameters
- For iterables, always specify key and value types:

```php
/**
 * @param array<int, MyObject> $my_array
 * @param int $typed_argument
 */
function someFunction(array $my_array, int $typed_argument) {}
```

- Use array shape notation for fixed keys, put each key on its own line:

```php
/** @return array{
   first: SomeClass,
   second: SomeClass
} */
```

## Control Flow
- Happy path last: handle error conditions first, success case last
- Avoid `else`: use early returns instead of nested conditions
- Separate conditions: prefer multiple if statements over compound conditions
- Always use curly brackets even for single statements
- Ternary operators: each part on its own line unless very short

```php
// Happy path last
if (! $user) {
    return null;
}

if (! $user->isActive()) {
    return null;
}

// Process active user...

// Short ternary
$name = $is_foo ? 'foo' : 'bar';

// Multi-line ternary
$result = $object instanceof Model
    ? $object->name
    : 'A default value';

// Ternary instead of else
$condition
    ? $this->doSomething()
    : $this->doSomethingElse();
```

## Laravel Conventions

### Routes
- URLs: kebab-case (`/open-source`)
- Route names: kebab-case (`->name('open-source')`)
- Parameters: snake_case (`{user_id}`)
- Use tuple notation: `[Controller::class, 'method']`

### Controllers
- Use plural resource names (`PostsController`)
- Stick to CRUD methods (`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`)
- Add additional controller methods for non-CRUD actions

### Configuration
- Files: kebab-case (`pdf-generator.php`)
- Keys: snake_case (`chrome_path`)
- Add service configs to `config/services.php`, don't create new files
- Use `config()` helper, avoid `env()` outside config files

### Artisan Commands
- Names: snake_case (`delete_old_records`)
- Always provide feedback (`$this->comment('All ok!')`)
- Show progress for loops, summary at end
- Put output before processing an item (easier debugging):

```php
$items->each(function (Item $item) {
    $this->info("Processing item id `{$item->id}`...");
    $this->processItem($item);
});

$this->comment("Processed {$items->count()} items.");
```

## Strings and Formatting
- Use string interpolation over concatenation
- Use `__()` localization function for any user facing strings

## Enums
- Use UPPER_SNAKE_CASE for enum values
- Use snake_case for backed enum strings
- For backed string enums, implement `Javaabu\Helpers\Enums\IsEnum` interface and use the `Javaabu\Helpers\Enums\NativeEnumsTrait` trait

## Comments
- Avoid comments; write expressive code instead
- When needed, use proper formatting:

```php
// Single line with space after //

/*
 * Multi-line blocks start with single *
 */
```

- Refactor comments into descriptive function names

## Whitespace
- Add blank lines between statements for readability
- Exception: sequences of equivalent single-line operations
- No extra empty lines between `{}` brackets
- Let code breathe; avoid cramped formatting

## Validation
- Use array notation for multiple rules (easier for custom rule classes):

```php
public function rules()
{
    return [
        'email' => ['required', 'email'],
    ];
}
```

- Custom validation rules use snake_case:

```php
Validator::extend('organisation_type', function ($attribute, $value) {
    return OrganisationType::isValid($value);
});
```

## Blade Templates
- Indent with 4 spaces
- No spaces after control structures:

```blade
@if($condition)
    Something
@endif
```

## Authorization
- Policies use camelCase: `Gate::define('editPost', ...)`
- Use CRUD words, but `view` instead of `show`

## Translations
- Use `__()` function over `@lang`

## API Routing
- Use plural resource names: `/errors`
- Use kebab-case: `/error-occurrences`
- Limit deep nesting for simplicity:

```
/error-occurrences/1
/errors/1/occurrences
```

## Testing
- Use descriptive test method names
- Follow the arrange-act-assert pattern
- Use the `#[Test]` attribute for PHPUnit tests
- Use snake_case for test methods
- Use camelCase for test support methods
- Start test methods with `it_`
- Use unique names for each test method, referencing the test class if needed
- Do not mock model methods unless necessary. Instead use factories to create model records.
- Extract reusable long test factory setups or mocks into test support methods
- For admin controller tests, user `adminGet`, `adminPost`, etc, instead of normal `get`, `post` methods
- Use `$this->setFakeSetting('setting_name', 'setting_value')` to mock fake settings where `get_setting` method is used
- Prefer database assertions over model assertions
- Prefer `assertEquals` over `assertSame`

```php
<?php

namespace Tests\Feature\Controllers;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use PHPUnit\Framework\Attributes\Test;
use Propaganistas\LaravelDisposableEmail\DisposableDomains;
use Tests\TestCase;

class HomeControllerTest extends TestCase
{
    use RefreshDatabase;


    #[Test]
    public function it_redirects_the_home_page_to_the_dashboard()
    {
        $user = User::factory()->create();
        $this->actingAs($user);
    
        $this->get('/')
            ->assertRedirect("/dashboard/{$user->id}");
    }
    
    protected function mockDisposableDomains()
    {
        // Create a mock of the Random Interface
        $mock = $this->mock(DisposableDomains::class);

        // Set our expectation for the methods that should be called
        // and what is supposed to be returned
        $mock->shouldReceive('isNotDisposable')
            ->andReturnTrue();

        $this->app->instance(DisposableDomains::class, $mock);
        $this->app->instance('disposable_email.domains', $mock);
    }
}
```

## Quick Reference

### Naming Conventions
- Classes: PascalCase (`UsersController`, `OrderStatus`)
- Methods: camelCase (`getUserName`)
- Variables: snake_case (`$first_name`)
- Routes: kebab-case (`/open-source`, `/user-profile`)
- Config files: kebab-case (`pdf-generator.php`)
- Config keys: snake_case (`chrome_path`)
- Artisan commands: snake_case (`php artisan delete_old_records`)

### File Structure
- Controllers: plural resource name + `Controller` (`PostsController`)
- Views: kebab-case (`open-source.blade.php`)
- Jobs: action-based (`CreateUser`, `SendEmailNotification`)
- Events: tense-based (`UserRegistering`, `UserRegistered`)
- Listeners: action + `Listener` suffix (`SendInvitationMailListener`)
- Commands: action + `Command` suffix (`PublishScheduledPostsCommand`)
- Mailables: purpose + `Mail` suffix (`AccountActivatedMail`)
- Resources and Transformers: plural + `Resource` or `Transformer` (`UsersResource`)
- Enums: plural descriptive name, no prefix (`OrderStatuses`, `BookingTypes`)

### Migrations
- Write down methods in migrations

### Code Quality Reminders

#### PHP
- Use typed properties over docblocks
- Prefer early returns over nested if/else
- Use constructor property promotion when all properties can be promoted
- Avoid `else` statements when possible
- Use string interpolation over concatenation
- Always use curly braces for control structures
