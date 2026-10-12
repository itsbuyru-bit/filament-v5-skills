# Filament v5 — Reference

<div align="right">

**English** · [Русский README](../../../README.ru.md)

</div>

Detailed, copy-ready code for the standards in [SKILL.md](SKILL.md). Every API here is verified against
`filamentphp/filament` at tag **v5.10.1**.

## Contents

1. [Platform requirements](#1-platform-requirements)
2. [Resource class — full example](#2-resource-class--full-example)
3. [Schema and form class — full example](#3-schema-and-form-class--full-example)
4. [Table class — full example](#4-table-class--full-example)
5. [Page classes](#5-page-classes)
6. [Relation managers](#6-relation-managers)
7. [Actions](#7-actions)
8. [Infolists](#8-infolists)
9. [Widgets](#9-widgets)
10. [Enums](#10-enums)
11. [Multi-tenancy](#11-multi-tenancy)
12. [Authorization and security](#12-authorization-and-security)
13. [Panel provider](#13-panel-provider)
14. [Testing](#14-testing)
15. [v3/v4 → v5 rename map](#15-v3v4--v5-rename-map)
16. [Deprecated names that still compile](#16-deprecated-names-that-still-compile)

---

## 1. Platform requirements

| Requirement | Constraint |
|---|---|
| PHP | `^8.2` |
| `ext-intl` | required |
| Laravel | `^11.28 \| ^12.0 \| ^13.0` |
| Livewire | `^4.4.7` |
| Tailwind CSS | v4.0+ (docs recommend v4.1+) |
| Symfony console / html-sanitizer | `^7.0 \| ^8.0` |

Only `packages/support/composer.json` declares the Laravel and Livewire constraints.

---

## 2. Resource class — full example

Generated layout:

```
app/Filament/Resources/Customers/
├── CustomerResource.php
├── Pages/
│   ├── CreateCustomer.php
│   ├── EditCustomer.php
│   ├── ListCustomers.php
│   └── ViewCustomer.php          # only with --view
├── Schemas/
│   ├── CustomerForm.php          # singular
│   └── CustomerInfolist.php      # singular
└── Tables/
    └── CustomersTable.php        # plural
```

```php
<?php

namespace App\Filament\Resources\Customers;

use App\Filament\Resources\Customers\Pages\CreateCustomer;
use App\Filament\Resources\Customers\Pages\EditCustomer;
use App\Filament\Resources\Customers\Pages\ListCustomers;
use App\Filament\Resources\Customers\Pages\ViewCustomer;
use App\Filament\Resources\Customers\Schemas\CustomerForm;
use App\Filament\Resources\Customers\Schemas\CustomerInfolist;
use App\Filament\Resources\Customers\Tables\CustomersTable;
use App\Models\Customer;
use Filament\Resources\Resource;
use Filament\Schemas\Schema;
use Filament\Support\Icons\Heroicon;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

class CustomerResource extends Resource
{
    protected static ?string $model = Customer::class;

    protected static string | BackedEnum | null $navigationIcon = Heroicon::OutlinedUsers;

    protected static string | UnitEnum | null $navigationGroup = 'Shop';

    protected static ?int $navigationSort = 1;

    protected static ?string $recordTitleAttribute = 'name';

    protected static ?string $slug = 'customers';

    public static function form(Schema $schema): Schema
    {
        return CustomerForm::configure($schema);
    }

    public static function infolist(Schema $schema): Schema
    {
        return CustomerInfolist::configure($schema);
    }

    public static function table(Table $table): Table
    {
        return CustomersTable::configure($table);
    }

    public static function getRelations(): array
    {
        return [
            RelationManagers\OrdersRelationManager::class,
        ];
    }

    public static function getWidgets(): array
    {
        return [
            Widgets\CustomerStats::class,
        ];
    }

    public static function getPages(): array
    {
        return [
            'index' => ListCustomers::route('/'),
            'create' => CreateCustomer::route('/create'),
            'view' => ViewCustomer::route('/{record}'),
            'edit' => EditCustomer::route('/{record}/edit'),
        ];
    }

    public static function getNavigationBadge(): ?string
    {
        return (string) static::getModel()::query()->where('is_active', false)->count() ?: null;
    }

    public static function getNavigationBadgeColor(): string | array | null
    {
        return 'warning';
    }

    public static function getEloquentQuery(): Builder
    {
        return parent::getEloquentQuery()->withCount('orders');
    }

    public static function getGloballySearchableAttributes(): array
    {
        return ['name', 'email', 'company.name'];
    }

    public static function getGlobalSearchResultDetails(Model $record): array
    {
        return [
            __('customers.email') => $record->email,
        ];
    }
}
```

### Exact static property types

```php
// Model & routes
protected static ?string $model = null;
protected static ?string $slug = null;
protected static ?string $recordRouteKeyName = null;
protected static string | array $routeMiddleware = [];
protected static string | array $withoutRouteMiddleware = [];

// Navigation
protected static string | BackedEnum | null $navigationIcon = null;
protected static string | BackedEnum | null $activeNavigationIcon = null;
protected static string | UnitEnum | null $navigationGroup = null;
protected static ?string $navigationLabel = null;
protected static ?int $navigationSort = null;
protected static ?string $navigationParentItem = null;
protected static bool $shouldRegisterNavigation = true;
protected static string | Htmlable | null $navigationBadgeTooltip = null;
protected static ?SubNavigationPosition $subNavigationPosition = null;

// Labels & global search
protected static ?string $modelLabel = null;
protected static ?string $pluralModelLabel = null;
protected static ?string $recordTitleAttribute = null;
protected static bool $hasTitleCaseModelLabel = true;
protected static bool $isGloballySearchable = true;
protected static ?int $globalSearchSort = null;
protected static int $globalSearchResultsLimit = 50;

// Authorization & discovery
protected static bool $shouldSkipAuthorization = false;
protected static bool $shouldCheckPolicyExistence = true;
protected static bool $isDiscovered = true;

// Tenancy & grouping
protected static bool $isScopedToTenant = true;
protected static ?string $tenantOwnershipRelationshipName = null;
protected static ?string $tenantRelationshipName = null;
protected static ?string $cluster = null;
protected static ?string $parentResource = null;
```

> `$label` and `$pluralLabel` still exist but are **deprecated** in favour of `$modelLabel` /
> `$pluralModelLabel`.
>
> `$navigationGroup` accepts a `UnitEnum`, not just a string — `NavigationGroup` enum cases work.
> `$navigationIcon` accepts a `BackedEnum` — `Heroicon` cases work.

### Record sub-navigation

```php
use Filament\Resources\Pages\Page;

public static function getRecordSubNavigation(Page $page): array
{
    return $page->generateNavigationItems([
        ViewCustomer::class,
        EditCustomer::class,
        ManageCustomerAddresses::class,
    ]);
}
```

This applies to `EditRecord`, `ViewRecord`, and `ManageRelatedRecords`. It does **not** apply to
`ListRecords`, which returns only cluster sub-navigation.

---

## 3. Schema and form class — full example

```php
<?php

namespace App\Filament\Resources\Customers\Schemas;

use Filament\Forms\Components\DatePicker;
use Filament\Forms\Components\FileUpload;
use Filament\Forms\Components\Repeater;
use Filament\Forms\Components\Select;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\TextInput;
use Filament\Forms\Components\Toggle;
use Filament\Schemas\Components\Grid;
use Filament\Schemas\Components\Section;
use Filament\Schemas\Components\Utilities\Get;
use Filament\Schemas\Components\Utilities\Set;
use Filament\Schemas\Components\Wizard;
use Filament\Schemas\Components\Wizard\Step;
use Filament\Schemas\Schema;
use Filament\Support\Enums\Operation;
use Illuminate\Support\Str;

class CustomerForm
{
    public static function configure(Schema $schema): Schema
    {
        return $schema
            ->components([
                Wizard::make([
                    Step::make(__('customers.steps.identity'))
                        ->icon(Heroicon::OutlinedUser)
                        ->schema([
                            TextInput::make('name')
                                ->required()
                                ->maxLength(255)
                                ->live(onBlur: true)
                                ->afterStateUpdated(fn (Set $set, ?string $state) => $set(
                                    'slug',
                                    Str::slug($state ?? ''),
                                )),
                            TextInput::make('slug')
                                ->required()
                                ->unique(ignoreRecord: true)
                                ->hiddenOn(Operation::Create),
                            TextInput::make('email')
                                ->email()
                                ->required()
                                ->scopedUnique(),
                        ])
                        ->columns(2),

                    Step::make(__('customers.steps.details'))
                        ->schema([
                            Section::make(__('customers.sections.company'))
                                ->schema([
                                    Select::make('company_id')
                                        ->relationship('company', 'name')
                                        ->searchable()
                                        ->preload()
                                        ->live()
                                        ->afterStateUpdated(fn (Set $set) => $set('branch_id', null)),
                                    Select::make('branch_id')
                                        ->options(fn (Get $get): array => Branch::query()
                                            ->whereBelongsTo(Company::find($get('company_id')))
                                            ->pluck('name', 'id')
                                            ->all())
                                        ->searchable()
                                        ->visible(fn (Get $get): bool => filled($get('company_id'))),
                                    DatePicker::make('onboarded_at')
                                        ->native(false)
                                        ->maxDate(now()),
                                    Toggle::make('is_active')
                                        ->default(true),
                                ])
                                ->columns(2),

                            Section::make(__('customers.sections.notes'))
                                ->schema([
                                    Textarea::make('notes')
                                        ->rows(4)
                                        ->columnSpanFull()
                                        ->belowContent(__('customers.notes_help')),
                                    FileUpload::make('avatar')
                                        ->image()
                                        ->disk('public')
                                        ->directory('customer-avatars')
                                        ->visibility('public')
                                        ->maxSize(2048),
                                ]),

                            Repeater::make('contacts')
                                ->relationship()
                                ->schema([
                                    TextInput::make('label')->required(),
                                    TextInput::make('phone')->tel()->required(),
                                ])
                                ->columns(2)
                                ->addActionLabel(__('customers.actions.add_contact'))
                                ->reorderable()
                                ->collapsible()
                                ->itemLabel(fn (array $state): ?string => $state['label'] ?? null)
                                ->defaultItems(1),
                        ]),
                ])
                    ->skippable()
                    ->persistStepInQueryString()
                    ->columnSpanFull(),
            ]);
    }
}
```

### Component API reference (verified)

**Layout components — `Filament\Schemas\Components\*`**

`Section`, `Grid`, `Flex`, `Fieldset`, `Group`, `FusedGroup`, `Tabs`, `Wizard`, `Callout`, `EmptyState`,
`Actions`, `Form`, `EmbeddedSchema`, `EmbeddedTable`, `Html`, `Icon`, `Image`, `Livewire`, `RenderHook`,
`Text`, `UnorderedList`, `View`.
Tab class: `Filament\Schemas\Components\Tabs\Tab`. Wizard step: `Filament\Schemas\Components\Wizard\Step`.

**Field components — `Filament\Forms\Components\*`**

`TextInput`, `Textarea`, `Select`, `MultiSelect`, `Checkbox`, `CheckboxList`, `Toggle`, `ToggleButtons`,
`Radio`, `DatePicker`, `DateTimePicker`, `TimePicker`, `FileUpload`, `RichEditor`, `MarkdownEditor`,
`Repeater`, `RelationshipRepeater`, `Builder`, `TagsInput`, `KeyValue`, `ColorPicker`, `Slider`,
`CodeEditor`, `Hidden`, `Placeholder`, `OneTimeCodeInput`, `MorphToSelect`, `TableSelect`,
`ModalTableSelect`, `ViewField`, `LivewireField`.

**Utilities — `Filament\Schemas\Components\Utilities\*`**

`Get`, `Set`.

```php
// Get
public function __invoke(string | Component $path = '', bool $isAbsolute = false): mixed

// Set
public function __invoke(
    string | Component $path,
    mixed $state,
    bool $isAbsolute = false,
    bool $shouldCallUpdatedHooks = false,
): mixed
```

`Get` also offers typed accessors: `string()`, `integer()`, `float()`, `boolean()`, `array()`, `date()`,
`enum($key, $enumClass)`, `filled()`, `blank()`.

Doc example with the exact imports:

```php
use Filament\Schemas\Components\Utilities\Get;
use Filament\Schemas\Components\Utilities\Set;
use Livewire\Component as Livewire;

function (Livewire $livewire, Get $get, Set $set) {
    // ...
}
```

Field-level specifics — content slots, validation, `Select`, `FileUpload`, `Repeater`, `Wizard`, and the
`Operation` conditional API — follow below.

**Select (verified)**

```php
Select::make('author_id')
    ->relationship(
        name: 'author',
        titleAttribute: 'name',
        modifyQueryUsing: fn (Builder $query) => $query->where('is_active', true),
    )
    ->searchable()
    ->preload()
    ->multiple()
    ->native(false)          // Filament uses the native HTML5 select by default
    ->allowHtml()            // only with trusted option labels
    ->getOptionLabelFromRecordUsing(fn (Model $record): string => $record->full_name);
```

Exact signature:

```php
public function relationship(
    string | Closure | null $name = null,
    string | Closure | null $titleAttribute = null,
    ?Closure $modifyQueryUsing = null,
    bool $ignoreRecord = false,
): static
```

> `modifyQueryUsing()` is **not** a chainable method on `Select` — it is only a named argument of
> `relationship()`. This is a very common mistake.
>
> When combining `disabled()` with `multiple()` and `relationship()`, call `disabled()` **before**
> `relationship()`.
>
> Filtering inside `modifyQueryUsing()` is a presentational convenience, not a security boundary.

**FileUpload (verified)**

```php
FileUpload::make('attachment')
    ->disk('public')
    ->directory('attachments')
    ->visibility('public')          // default is private — set it explicitly
    ->image()
    ->imageEditor()
    ->multiple()
    ->maxFiles(5)
    ->maxSize(5120)                 // kilobytes
    ->acceptedFileTypes(['application/pdf', 'image/*'])
    ->storeFileNamesIn('attachment_names')
    ->deletable(false);             // hides the remove button
```

Other verified methods: `minSize()`, `downloadable()`, `openable()`, `previewable()`, `reorderable()`,
`pasteable()`, `moveFiles()`, `fetchFileInformation()`, `getUploadedFileNameForStorageUsing()`,
`avatar()`, `circleCropper()`, `imageAspectRatio()`, `imageCropAspectRatio()`,
`automaticallyResizeImagesToWidth()`, `automaticallyResizeImagesToHeight()`, `maxParallelUploads()`,
`panelLayout()`, `appendFiles()`, `idleLabel()`.

> **There is no `deletableFileUploads()`** — the method is `deletable(false)`.
> Never enable `->preserveFilenames()` on user-supplied names. A disk literally named `public` is treated as
> public; every other disk name defaults to private visibility.

**Repeater (verified)**

```php
Repeater::make('qualifications')
    ->relationship()
    ->schema([
        TextInput::make('institution')->required(),
        TextInput::make('qualification')->required(),
    ])
    ->columns(2)          // lays out the FIELDS INSIDE EACH ITEM
    ->grid(2)             // lays out the ITEMS THEMSELVES
    ->addActionLabel(__('forms.actions.add_qualification'))
    ->defaultItems(1)
    ->minItems(1)
    ->maxItems(10)
    ->reorderable()
    ->collapsible()
    ->cloneable()
    ->itemLabel(fn (array $state): ?string => $state['institution'] ?? null);
```

Exact signature:

```php
public function relationship(
    string | Closure | null $name = null,
    ?Closure $modifyQueryUsing = null,
    ?Closure $modifyRecordsUsing = null,
): static
```

> **`columns()` and `grid()` are different scopes.** `columns()` is inherited from the schema package and
> lays out the fields *within each item*; `grid()` (from `HasContainerGridLayout`) lays out the *items*.
> `Builder` has **no** `grid()`.
>
> Deprecated: `->createItemButtonLabel()` → `->addActionLabel()`.
>
> `->itemLabel()` needs the source field to be `live()` to update as the user types.

**Wizard (verified)**

```php
public static function make(array | Closure $steps = []): static
public function steps(array | Closure $steps): static
public function skippable(bool | Closure $condition = true): static
public function startOnStep(int | Closure $startStep): static
public function persistStepInQueryString(string | Closure | null $key = 'step'): static
public function nextAction(?Closure $callback): static
public function previousAction(?Closure $callback): static
public function submitAction(string | Htmlable | null $action): static
public function hiddenHeader(bool | Closure $condition = true): static
```

> `submitAction()` and `cancelAction()` take **rendered HTML** (`string | Htmlable | null`), not an
> `Action`. Only `nextAction()` / `previousAction()` accept action-modifier closures.

```php
Wizard::make([
    Step::make(__('orders.steps.order'))
        ->icon(Heroicon::OutlinedShoppingCart)
        ->schema([
            // ...
        ]),
    Step::make(__('orders.steps.delivery'))
        ->description(__('orders.steps.delivery_help'))
        ->schema([
            // ...
        ]),
])
    ->skippable()
    ->persistStepInQueryString()
    ->columnSpanFull();
```

`Step` methods: `->description()`, `->icon()`, `->completedIcon()`, `->beforeValidation()`,
`->afterValidation()`, `->afterValidated()`, `->formWrapper()`, `->columns()`.

Throw `new Halt()` inside a step's `afterValidation()` callback to stop the wizard advancing.

**Content slots on `Field` (exactly ten)**

`aboveLabel()`, `belowLabel()`, `beforeLabel()`, `afterLabel()`, `aboveContent()`, `belowContent()`,
`beforeContent()`, `afterContent()`, `aboveErrorMessage()`, `belowErrorMessage()`.

Each accepts a component, schema, action, action group, string, `Htmlable`, or `Closure`. `afterLabel` and
`afterContent` child schemas are right-aligned by default. The same slot API exists on `Entry`, `FusedGroup`,
`Actions`, and (`aboveContent`/`belowContent` only) `Section`.

```php
TextInput::make('email')
    ->label(__('fields.email'))
    ->helperText(__('fields.email_help'))       // sugar over the below_content slot
    ->hint(__('fields.optional'))               // renders AFTER the label
    ->hintIcon(Heroicon::OutlinedQuestionMark, tooltip: __('fields.optional_tooltip'))
    ->aboveLabel(__('fields.email_above'))
    ->belowContent(Action::make('verify')->link())
    ->inlineLabel()
    ->columnSpanFull();
```

There is **no `aboveField()` / `belowField()`**.

The v3 form-field methods `->descriptionAbove()` and `->descriptionBelow()` were **removed** and have no
replacement on fields — use `helperText()` or the content slots. (Do not confuse this with
`TextColumn::description()`, which does accept `position: 'above'|'below'`; those two names survive in the
tables package as internal properties of `Columns\Concerns\HasDescription`, not as field methods.)

> **`description()` is not a method on form fields.** It exists only on layout components (`Section`,
> `Callout`, `EmptyState`, `Wizard\Step`). Use `helperText()` — which is sugar over the `below_content`
> slot, and is automatically associated with the input for accessibility.

**Validation (verified signatures)**

```php
->required(bool | Closure $condition = true)
->nullable(bool | Closure $condition = true)
->rules(string | array | Closure $rules, bool | Closure $condition = true)
->rule(mixed $rule, bool | Closure $condition = true)
->unique(string | Closure | null $table = null, string | Closure | null $column = null, Model | Closure | null $ignorable = null, ?bool $ignoreRecord = null, ?Closure $modifyRuleUsing = null)
->scopedUnique(string | Closure | null $model = null, string | Closure | null $column = null, Model | Closure | null $ignorable = null, ?bool $ignoreRecord = null, ?Closure $modifyQueryUsing = null)
->exists(string | Closure | null $table = null, string | Closure | null $column = null, ?Closure $modifyRuleUsing = null)
->scopedExists(string | Closure | null $model = null, string | Closure | null $column = null, ?Closure $modifyQueryUsing = null)
->validationAttribute(string | Closure | null $label)
->validationMessages(array $messages)
->distinct(bool | Closure $condition = true)
->markAsRequired(bool | Closure $condition = true)   // visual asterisk only, not validation
```

> Note the different callback parameter types: `exists(modifyRuleUsing:)` receives an
> `Illuminate\Validation\Rules\Exists` object, while `scopedExists(modifyQueryUsing:)` receives an Eloquent
> `Builder`.

```php
use Filament\Schemas\Components\Utilities\Get;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;

TextInput::make('email')
    ->required(fn (Get $get): bool => filled($get('company_name')))
    ->scopedUnique()
    ->scopedExists(
        modifyQueryUsing: fn (Builder $query) => $query->withoutGlobalScope(SoftDeletingScope::class),
    )
    ->validationMessages(['unique' => __('validation.custom.email.unique')])
    ->validationAttribute(__('fields.email'));
```

**Injectable closure parameters on a schema component**

Resolved by name: `$state`, `$get`, `$set`, `$rawState`, `$record`, `$livewire`, `$model`, `$operation`
(alias `$context`), `$parentRepeaterItemIndex`. Plus `$component` (or a parameter typed as the component
class), and typed injections of `Get`, `Set`, `Model`, or the concrete model class.

Context-specific additions:

| Context | Extra parameters |
|---|---|
| `afterStateUpdated()` | `$old`, `$oldRaw` |
| `Repeater::itemLabel()` | `$item`, `$key`, `$index`, `$state` |
| `relationship(modifyQueryUsing:)` | `$query`, plus `$search` when searchable |
| `unique()` / `exists()` modifiers | `$rule` |
| Repeater `modifyRecordsUsing` | `$records` |
| Action modifier callbacks | `$action` |

> A nullable-typed unknown parameter resolves silently to `null`; a non-nullable one with no default throws
> `BindingResolutionException`. Typos in parameter names can therefore fail silently.
>
> `$get` / `$set` read and write hidden components too.

**`Operation` and conditional visibility (verified)**

```php
public function hidden(bool | Closure $condition = true): static
public function hiddenOn(string | Operation | array $operations): static
public function visible(bool | Closure $condition = true): static
public function visibleOn(string | Operation | array $operations): static
public function hiddenJs(string | Closure | null $condition): static
public function visibleJs(string | Closure | null $condition): static
```

```php
use Filament\Forms\Components\TextInput;
use Filament\Support\Enums\Operation;

TextInput::make('password')
    ->password()
    ->required()
    ->hiddenOn(Operation::Edit);
```

> `hiddenOn()` / `visibleOn()` **overwrite** previous `hidden()` / `visible()` calls and vice versa.
> They also accept a Livewire page class string (e.g. `->hiddenOn(ManagePosts::class)`), not only
> `Operation` cases.
>
> `hiddenJs()` / `visibleJs()` avoid a server round-trip but evaluate on the client — do not put untrusted
> input in them.

---

## 4. Table class — full example

```php
<?php

namespace App\Filament\Resources\Customers\Tables;

use App\Enums\OrderStatus;
use Filament\Actions\ActionGroup;
use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\EditAction;
use Filament\Actions\ViewAction;
use Filament\Forms\Components\DatePicker;
use Filament\Tables\Columns\IconColumn;
use Filament\Tables\Columns\ImageColumn;
use Filament\Tables\Columns\Summarizers\Sum;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Enums\FiltersLayout;
use Filament\Tables\Filters\Filter;
use Filament\Tables\Filters\SelectFilter;
use Filament\Tables\Filters\TernaryFilter;
use Filament\Tables\Filters\TrashedFilter;
use Filament\Tables\Table;
use Illuminate\Database\Eloquent\Builder;

class CustomersTable
{
    public static function configure(Table $table): Table
    {
        return $table
            ->columns([
                ImageColumn::make('avatar')
                    ->circular()
                    ->imageSize(40),
                TextColumn::make('name')
                    ->searchable()
                    ->sortable()
                    ->description(fn ($record): ?string => $record->company?->name),
                TextColumn::make('email')
                    ->searchable()
                    ->copyable()
                    ->copyMessage(__('tables.copied')),
                TextColumn::make('status')
                    ->badge()
                    ->color(fn (OrderStatus $state): string => $state->getColor())
                    ->icon(fn (OrderStatus $state): Heroicon => $state->getIcon()),
                IconColumn::make('is_active')
                    ->boolean()
                    ->sortable(),
                TextColumn::make('total_spent')
                    ->money('USD')
                    ->summarize(Sum::make()->money('USD'))
                    ->sortable(),
                TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
            ])
            ->filters([
                SelectFilter::make('status')
                    ->options(OrderStatus::class)
                    ->multiple(),
                SelectFilter::make('company')
                    ->relationship('company', 'name')
                    ->searchable()
                    ->preload(),
                TernaryFilter::make('is_active')
                    ->placeholder(__('tables.all_customers')),
                Filter::make('created_at')
                    ->schema([
                        DatePicker::make('created_from'),
                        DatePicker::make('created_until'),
                    ])
                    ->query(fn (Builder $query, array $data): Builder => $query
                        ->when($data['created_from'], fn (Builder $q, $date): Builder => $q->whereDate('created_at', '>=', $date))
                        ->when($data['created_until'], fn (Builder $q, $date): Builder => $q->whereDate('created_at', '<=', $date))),
                TrashedFilter::make(),
            ], layout: FiltersLayout::AboveContentCollapsible)
            ->deferFilters(false)
            ->recordActions([
                ActionGroup::make([
                    ViewAction::make(),
                    EditAction::make(),
                    DeleteAction::make(),
                ]),
            ])
            ->toolbarActions([
                BulkActionGroup::make([
                    DeleteBulkAction::make(),
                ]),
            ])
            ->defaultSort('created_at', 'desc')
            ->persistSearchInSession()
            ->persistSortInSession()
            ->striped()
            ->emptyStateHeading(__('tables.no_customers'))
            ->emptyStateDescription(__('tables.no_customers_help'))
            ->emptyStateIcon(Heroicon::OutlinedUsers);
    }
}
```

### Verified table configuration methods

**Wiring:** `columns()`, `pushColumns()`, `filters()`, `pushFilters()`, `recordActions()`, `pushRecordActions()`,
`toolbarActions()`, `pushToolbarActions()`, `groupedBulkActions()`, `headerActions()`, `emptyStateActions()`.

**Query:** `query()`, `modifyQueryUsing()`, `relationship()`, `inverseRelationship()`, `records()`.

**Sorting & paging:** `defaultSort()`, `defaultSortOptionLabel()`, `defaultKeySort()`, `paginated()`,
`paginationPageOptions()`, `defaultPaginationPageOption()`, `paginationMode()`, `extremePaginationLinks()`,
`paginatedWhileReordering()`.

**Search:** `searchable()`, `searchPlaceholder()`, `searchDebounce()` (default `500ms`), `searchOnBlur()`,
`splitSearchTerms()`, `searchUsing()`.

**Persistence:** `persistSearchInSession()`, `persistColumnSearchesInSession()`, `persistSortInSession()`,
`persistFiltersInSession()`, `persistGroupInSession()`, `persistColumnsInSession()`, `persistInSession()`,
`persistRecordsPerPageInSession()`.

**Grouping:** `groups()`, `defaultGroup($group, direction: 'desc')`, `groupingSettingsInDropdownOnDesktop()`,
`groupingSettingsHidden()`, `groupingDirectionSettingHidden()`, `collapsedGroupsByDefault()`, `groupsOnly()`,
`groupRecordTriggerAction()`. Group class: `Filament\Tables\Grouping\Group`.

**Display:** `striped()`, `deferLoading()`, `loadingSkeleton()`, `poll()`, `stackedOnMobile()`,
`contentGrid()`, `content()`, `contentFooter()`, `heading()`, `description()`, `header()`, `recordUrl()`,
`openRecordUrlInNewTab()`, `recordAction()`, `recordClasses()`, `queryStringIdentifier()`,
`identifier()`, `summaries()`, `emptyState()`, `emptyStateHeading()`, `emptyStateDescription()`,
`emptyStateIcon()`.

**Selection:** `selectable()`, `checkIfRecordIsSelectableUsing()`, `selectCurrentPageOnly()`,
`selectGroupsOnly()`, `maxSelectableRecords()`, `trackDeselectedRecords()`, `recordCheckboxPosition()`,
`disabledSelection()`, `currentSelectionLivewireProperty()`.

**Column manager:** `columnManager()`, `deferColumnManager()`, `reorderableColumns()`,
`columnManagerLayout()`, `columnManagerTriggerAction()`, `columnManagerColumns()`,
`columnManagerResetActionPosition()`.

> `->groupingDirection()` does **not** exist — pass direction to `defaultGroup()`.

### Enums used by tables

| Enum | Cases |
|---|---|
| `Filament\Tables\Enums\FiltersLayout` | `AboveContent`, `AboveContentCollapsible`, `BelowContent`, `BeforeContent`, `AfterContent`, `BeforeContentCollapsible`, `AfterContentCollapsible`, `Dropdown`, `Modal`, `Hidden` |
| `Filament\Tables\Enums\PaginationMode` | `Default`, `Simple`, `Cursor` |
| `Filament\Tables\Enums\RecordActionsPosition` | `AfterCells`, `AfterColumns`, `AfterContent`, `BeforeCells`, `BeforeColumns` |
| `Filament\Tables\Enums\RecordCheckboxPosition` | `BeforeCells`, `AfterCells` |
| `Filament\Tables\Enums\ColumnManagerLayout` | `Dropdown`, `Modal` |
| `Filament\Tables\Actions\HeaderActionsPosition` | `Adaptive`, `Bottom` |
| `Filament\Tables\Enums\FiltersResetActionPosition` | `Header`, `Footer` |

---

## 5. Page classes

| FQCN | Notes |
|---|---|
| `Filament\Resources\Pages\Page` | abstract base; requires `protected static string $resource` |
| `Filament\Resources\Pages\ListRecords` | table page |
| `Filament\Resources\Pages\CreateRecord` | create page |
| `Filament\Resources\Pages\EditRecord` | edit page |
| `Filament\Resources\Pages\ViewRecord` | view page |
| `Filament\Resources\Pages\ManageRecords` | simple (modal) resource page; extends `ListRecords` |
| `Filament\Resources\Pages\ManageRelatedRecords` | relationship table page |

```php
namespace App\Filament\Resources\Customers\Pages;

use App\Filament\Resources\Customers\CustomerResource;
use Filament\Actions\CreateAction;
use Filament\Resources\Pages\ListRecords;
use Filament\Schemas\Components\Tabs\Tab;
use Illuminate\Database\Eloquent\Builder;

class ListCustomers extends ListRecords
{
    protected static string $resource = CustomerResource::class;

    protected function getHeaderActions(): array
    {
        return [
            CreateAction::make(),
        ];
    }

    public function getTabs(): array
    {
        return [
            'all' => Tab::make(__('customers.tabs.all')),
            'active' => Tab::make(__('customers.tabs.active'))
                ->modifyQueryUsing(fn (Builder $query) => $query->where('is_active', true)),
        ];
    }
}
```

### Lifecycle hooks

| Page | Hooks |
|---|---|
| `CreateRecord` | `beforeFill`, `afterFill`, `beforeValidate`, `afterValidate`, `beforeCreate`, `afterCreate` |
| `EditRecord` | `beforeFill`, `afterFill`, `beforeValidate`, `afterValidate`, `beforeSave`, `afterSave` |
| `ViewRecord` | **only** `beforeFill`, `afterFill` — and they do not run when an infolist is defined |

### Redirect signatures — they differ

```php
// CreateRecord
public function getRedirectUrl(): string        // non-nullable; always redirects

// EditRecord
public function getRedirectUrl(): ?string       // nullable; default null = no redirect
```

### Widgets on pages

```php
protected function getHeaderWidgets(): array
{
    return [Widgets\CustomerStats::class];
}

protected function getFooterWidgets(): array
{
    return [Widgets\RecentOrders::class];
}

public function getHeaderWidgetsColumns(): int | array
{
    return 3;
}
```

Both `getHeaderWidgets()` and `getFooterWidgets()` are `protected`. There is **no** `getFooterActions()`.
`getHeaderActions()` and `getFormActions()` are also `protected`.

---

## 6. Relation managers

```php
<?php

namespace App\Filament\Resources\Customers\RelationManagers;

use Filament\Actions\AssociateAction;
use Filament\Actions\BulkActionGroup;
use Filament\Actions\CreateAction;
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\DissociateAction;
use Filament\Actions\DissociateBulkAction;
use Filament\Actions\EditAction;
use Filament\Forms\Components\TextInput;
use Filament\Resources\RelationManagers\RelationManager;
use Filament\Schemas\Schema;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Table;

class OrdersRelationManager extends RelationManager
{
    protected static string $relationship = 'orders';

    protected static ?string $title = 'Orders';

    public function form(Schema $schema): Schema          // INSTANCE method
    {
        return $schema
            ->components([
                TextInput::make('reference')->required(),
            ]);
    }

    public function table(Table $table): Table            // INSTANCE method
    {
        return $table
            ->columns([
                TextColumn::make('reference')->searchable(),
                TextColumn::make('total')->money('USD'),
            ])
            ->headerActions([
                CreateAction::make(),
                AssociateAction::make(),
            ])
            ->recordActions([
                EditAction::make(),
                DissociateAction::make(),
                DeleteAction::make(),
            ])
            ->toolbarActions([
                BulkActionGroup::make([
                    DissociateBulkAction::make(),
                    DeleteBulkAction::make(),
                ]),
            ])
            ->defaultSort('created_at', 'desc');
    }
}
```

Register on the resource:

```php
public static function getRelations(): array
{
    return [
        RelationManagers\OrdersRelationManager::class,
        'orders' => RelationManagers\OrdersRelationManager::class,   // forces ?relation=orders
    ];
}
```

Key facts:

- A relation manager is a **Livewire component**. `$ownerRecord` and `$pageClass` are `#[Locked]` public
  properties injected as `['ownerRecord' => ..., 'pageClass' => ...]`.
- Inject `RelationManager $livewire` into closures, or call `$this->getOwnerRecord()`.
- Authorization gate is `canViewForRecord(Model $ownerRecord, string $pageClass)`.
- `AssociateAction`, `AttachAction`, `DetachAction`, and `DissociateAction` consult **only**
  `isReadOnly()` — they do **not** check a policy method.
- Configure the table in `table()`, not via static properties. `$recordTitleAttribute`, `$inverseRelationship`,
  `$label`, `$pluralLabel`, `$modelLabel`, `$pluralModelLabel` are all `@deprecated` on `RelationManager`.
- For complex relations, `ManageRelatedRecords` is the alternative page with `$relatedResource`.

---

## 7. Actions

### Full custom action

```php
<?php

namespace App\Filament\Resources\Invoices\Actions;

use App\Models\Invoice;
use Filament\Actions\Action;
use Filament\Forms\Components\Textarea;
use Filament\Forms\Components\TextInput;
use Filament\Notifications\Notification;
use Filament\Support\Icons\Heroicon;

class SendInvoiceAction
{
    public static function make(): Action
    {
        return Action::make('send')
            ->label(__('invoices.actions.send'))
            ->icon(Heroicon::OutlinedPaperAirplane)
            ->color('primary')
            ->requiresConfirmation()
            ->modalHeading(__('invoices.modal.send_heading'))
            ->modalDescription(__('invoices.modal.send_description'))
            ->modalSubmitActionLabel(__('invoices.actions.send_now'))
            ->schema([
                TextInput::make('subject')
                    ->required()
                    ->maxLength(255),
                Textarea::make('body')
                    ->required()
                    ->autosize(),
            ])
            ->authorize('update')
            ->visible(fn (Invoice $record): bool => $record->status->canSend())
            ->action(function (Invoice $record, array $data): void {
                $record->send($data['subject'], $data['body']);
            })
            ->successNotificationTitle(__('invoices.notifications.sent'))
            ->failureNotificationTitle(__('invoices.notifications.send_failed'))
            ->successRedirectUrl(fn (Invoice $record): string => InvoiceResource::getUrl('edit', ['record' => $record]));
    }
}
```

Use it on a page and on a table row:

```php
protected function getHeaderActions(): array
{
    return [SendInvoiceAction::make()];
}

// in a table class
->recordActions([
    SendInvoiceAction::make(),
])
```

### Action lifecycle hooks

```php
->beforeFormFilled(fn (Invoice $record) => ...)    // fires before the modal schema is filled
->afterFormFilled(fn () => ...)
->beforeFormValidated(fn () => ...)
->afterFormValidated(fn () => ...)
->before(fn () => ...)                              // before the ->action() closure
->after(fn () => ...)                               // after the ->action() closure
```

Order: mount → `beforeFormFilled` → `mountUsing` (fills the schema) → `afterFormFilled`.
Submit → `beforeFormValidated` → validate → `afterFormValidated` → `before` → `->action()` → `after`.

### Injectables in an action closure

```php
->action(function (
    array $data,
    Invoice $record,
    Action $action,
    Component $component,
    Livewire $livewire,
) {
    // ...
})
```

By name: `$data`, `$record`, `$livewire`, `$model`, `$arguments`, `$schema`, `$component`
(`$schemaComponent`), `$state` (`$schemaComponentState`), `$schemaState`, `$get` (`$schemaGet`),
`$set` (`$schemaSet`), `$operation` (`$context`, `$schemaOperation`), `$table`, `$action`,
`$parentAction`, `$mountedActions`, `$records` (`$selectedRecords`), `$recordsQuery`
(`$selectedRecordsQuery`).

`ActionGroup` closures expose a smaller set — no `$set`, `$data`, or `$arguments`.

### Bulk actions

```php
use Filament\Actions\BulkAction;
use Filament\Support\Authorization\DenyResponse;
use Illuminate\Support\Collection;

BulkAction::make('archive')
    ->requiresConfirmation()
    ->authorizeIndividualRecords('update')
    ->deselectRecordsAfterCompletion()
    ->chunkSelectedRecords(250)
    ->action(fn (Collection $records) => $records->each->archive())
    ->successNotificationTitle(__('invoices.notifications.archived'))
    ->failureNotificationTitle(fn (int $successCount, int $totalCount): string => __(
        'invoices.notifications.partial',
        ['success' => $successCount, 'total' => $totalCount],
    ));
```

Deny a specific record during bulk processing:

```php
->action(function (Collection $records, BulkAction $action) {
    foreach ($records as $record) {
        if (! auth()->user()->can('archive', $record)) {
            $action->reportBulkProcessingFailure(
                'unauthorized',
                message: fn (): string => __('invoices.errors.unauthorized', ['record' => $record->id]),
            );

            continue;
        }

        $record->archive();
    }
})
```

### Import and export classes

```php
namespace App\Filament\Imports;

use App\Models\Product;
use Filament\Actions\Imports\ImportColumn;
use Filament\Actions\Imports\Importer;
use Filament\Actions\Imports\Models\Import;

class ProductImporter extends Importer          // abstract class, not an interface
{
    protected static ?string $model = Product::class;

    public static function getColumns(): array
    {
        return [
            ImportColumn::make('sku')
                ->requiredMapping()
                ->rules(['required', 'max:255']),
            ImportColumn::make('name')
                ->requiredMapping()
                ->rules(['required', 'max:255']),
            ImportColumn::make('price')
                ->numeric(decimalPlaces: 2)
                ->rules(['required', 'numeric']),
            ImportColumn::make('category')
                ->relationship('category', 'name')
                ->guess(['category_name']),
        ];
    }

    public function resolveRecord(): ?Product
    {
        return Product::firstOrNew(['sku' => $this->data['sku']]);
    }

    protected function beforeSave(): void
    {
        // ...
    }

    public static function getCompletedNotificationBody(Import $import): string
    {
        return __('imports.completed', [
            'successful' => number_format($import->successful_rows),
            'failed' => number_format($import->failed_rows),
        ]);
    }
}
```

```php
namespace App\Filament\Exports;

use App\Models\Product;
use Filament\Actions\Exports\ExportColumn;
use Filament\Actions\Exports\Exporter;
use Filament\Actions\Exports\Models\Export;

class ProductExporter extends Exporter          // abstract class, not an interface
{
    protected static ?string $model = Product::class;

    public static function getColumns(): array
    {
        return [
            ExportColumn::make('sku'),
            ExportColumn::make('name'),
            ExportColumn::make('price')
                ->formatStateUsing(fn (float $state): string => number_format($state, 2)),
            ExportColumn::make('orders_count')
                ->counts('orders'),
        ];
    }

    public static function getCompletedNotificationBody(Export $export): string
    {
        return __('exports.completed', ['count' => number_format($export->successful_rows)]);
    }
}
```

Wire them up:

```php
->headerActions([
    ImportAction::make()
        ->importer(ProductImporter::class)
        ->chunkSize(250)
        ->csvDelimiter(','),
    ExportAction::make()
        ->exporter(ProductExporter::class)
        ->formats([ExportFormat::Csv, ExportFormat::Xlsx])
        ->columnMapping(),
])
```

Remember the migration publish step:
`php artisan vendor:publish --tag=filament-actions-migrations`.

---

## 8. Infolists

```php
namespace App\Filament\Resources\Customers\Schemas;

use Filament\Infolists\Components\ImageEntry;
use Filament\Infolists\Components\KeyValueEntry;
use Filament\Infolists\Components\RepeatableEntry;
use Filament\Infolists\Components\TextEntry;
use Filament\Schemas\Components\Section;
use Filament\Schemas\Schema;

class CustomerInfolist
{
    public static function configure(Schema $schema): Schema
    {
        return $schema
            ->components([
                Section::make(__('customers.sections.overview'))
                    ->schema([
                        ImageEntry::make('avatar')
                            ->circular()
                            ->hiddenLabel(),
                        TextEntry::make('name')
                            ->weight(FontWeight::Bold)
                            ->size(TextSize::Large),
                        TextEntry::make('email')
                            ->copyable()
                            ->icon(Heroicon::OutlinedEnvelope),
                        TextEntry::make('status')
                            ->badge()
                            ->color(fn (OrderStatus $state): string => $state->getColor()),
                        TextEntry::make('notes')
                            ->prose()
                            ->markdown()
                            ->columnSpanFull(),
                    ])
                    ->columns(2),

                Section::make(__('customers.sections.metadata'))
                    ->schema([
                        KeyValueEntry::make('meta'),
                    ]),

                RepeatableEntry::make('contacts')
                    ->schema([
                        TextEntry::make('label'),
                        TextEntry::make('phone'),
                    ])
                    ->columns(2),
            ]);
    }
}
```

`infolist(Schema $schema): Schema` — it returns `Filament\Schemas\Schema` in v5.

### Entry inventory — `Filament\Infolists\Components\*`

`TextEntry`, `IconEntry`, `ImageEntry`, `ColorEntry`, `CodeEntry`, `KeyValueEntry`, `RepeatableEntry`,
`ViewEntry`.

> **There is no `BadgeEntry`.** Use `TextEntry::make('status')->badge()`.

### Entry API (verified)

`->label()`, `->hiddenLabel()`, `->inlineLabel()`, `->helperText()`, `->hint()`, `->hintIcon()`,
`->aboveLabel()`, `->belowLabel()`, `->aboveContent()`, `->belowContent()`, `->state()`,
`->getStateUsing()`, `->formatStateUsing()`, `->badge()`, `->color()`, `->icon()`, `->size()`,
`->weight()`, `->fontFamily()`, `->copyable()`, `->prose()`, `->markdown()`, `->html()`, `->url()`,
`->openUrlInNewTab()`, `->tooltip()`, `->placeholder()`, `->columnSpanFull()`, `->listWithLineBreaks()`,
`->bulleted()`, `->limitList()`, `->separator()`, `->visible()`, `->hidden()`.

`->html()` and `->markdown()` output is sanitized by Symfony's HtmlSanitizer.

---

## 9. Widgets

### Class inventory — `Filament\Widgets\*`

`Widget` (base, abstract), `StatsOverviewWidget` (concrete), `ChartWidget` (abstract), `TableWidget`
(concrete), and the **deprecated** per-type chart classes `LineChartWidget`, `BarChartWidget`,
`PieChartWidget`, `DoughnutChartWidget`, `PolarAreaChartWidget`, `RadarChartWidget`, `ScatterChartWidget`,
`BubbleChartWidget`.

`Widget` declares:

```php
protected static bool $isDiscovered = true;
protected static ?int $sort = null;
protected int | string | array $columnSpan = 1;
protected int | string | array $columnStart = [];

/**
 * @var view-string
 */
protected string $view;          // NO DEFAULT — you must declare it on a custom widget

public static function canView(): bool { return true; }
public static function getSort(): int { return static::$sort ?? -1; }
public static function make(array $properties = []): WidgetConfiguration
```

`$sort` and `$isLazy` are **static**; `$pollingInterval`, `$heading`, `$color`, `$maxHeight`, `$options`
are instance properties. `Widget::make([...])` returns a `WidgetConfiguration`, valid only inside
`widgets()` / `getHeaderWidgets()` / `getFooterWidgets()` arrays.

### Stats overview

```php
namespace App\Filament\Widgets;

use Filament\Support\Icons\Heroicon;
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class RevenueWidget extends StatsOverviewWidget
{
    protected static ?int $sort = 1;

    protected ?string $heading = 'Revenue';

    protected int | array | null $columns = 3;

    protected function getStats(): array
    {
        return [
            Stat::make(__('widgets.revenue'), '$192.00')
                ->description(__('widgets.revenue_change'))
                ->descriptionIcon(Heroicon::OutlinedArrowTrendingUp)
                ->descriptionColor('success')
                ->chart([7, 2, 10, 3, 15, 4, 17])
                ->chartColor('success')
                ->color('success'),

            Stat::make(__('widgets.orders'), '1,204')
                ->icon(Heroicon::OutlinedShoppingCart),
        ];
    }
}
```

`Stat` is a **schema component** (`extends Filament\Schemas\Components\Component`) with a `final`
constructor — not a value object:

```php
Stat::make(string | Htmlable $label, $value)   // BOTH arguments required
    ->value($value)
    ->description(string | Htmlable | Closure | null)
    ->descriptionIcon(string | BackedEnum | null, IconPosition | string | null $position = null)
    ->descriptionColor(string | array | null)
    ->chart(array | Arrayable | null)
    ->chartColor(string | array | null)
    ->icon(string | BackedEnum | null)
    ->color(string | array | Closure | null)
    ->placeholder(...)
    ->url(..., $shouldOpenInNewTab = null)
    ->key(...)
    ->extraAttributes(...)
```

There is **no** `size()`, `weight()`, or `badge()` on `Stat`.

`StatsOverviewWidget` also declares `protected int|string|array $columnSpan = 'full'` (full width by
default) and `protected ?string $heading`.

### Chart widget

```php
namespace App\Filament\Widgets;

use Filament\Widgets\ChartWidget;

class OrdersChart extends ChartWidget
{
    protected ?string $heading = 'Orders';          // INSTANCE property in v5

    protected static ?int $sort = 2;

    protected int | string | array $columnSpan = 'full';

    protected ?string $maxHeight = '300px';

    protected ?string $pollingInterval = null;      // default is '5s' — null disables polling

    protected ?string $emptyStateHeading = null;

    protected function getType(): string
    {
        return 'line';
    }

    protected function getData(): array
    {
        return [
            'datasets' => [
                [
                    'label' => 'Orders',
                    'data' => [10, 20, 15, 30],
                ],
            ],
            'labels' => ['Jan', 'Feb', 'Mar', 'Apr'],
        ];
    }

    protected function getOptions(): array
    {
        return ['scales' => ['y' => ['beginAtZero' => true]]];
    }
}
```

**Chart type is a bare string:** `'bar'`, `'bubble'`, `'doughnut'`, `'line'`, `'pie'`, `'polarArea'`,
`'radar'`, `'scatter'`. There are **no `TYPE_*` constants** on `ChartWidget`.

`ChartWidget` property/visibility map — getting these wrong is the most common override error:

| Member | Visibility | Default |
|---|---|---|
| `getType()` | `protected`, **abstract** | — |
| `getData()`, `getOptions()`, `getMaxHeight()`, `getFilters()` | `protected` | `[]`, `null`, `null`, `null` |
| `getHeading()`, `getDescription()`, `getColor()`, `isCollapsible()`, `isEmpty()` | `public` | — |
| `$heading`, `$description`, `$color`, `$maxHeight`, `$options`, `$isCollapsible` | `protected` instance | `null`, `null`, `'primary'`, … |
| `$filter` | **`public`** (Livewire property, user-controllable) | `null` |
| `$dataChecksum` | `public`, `#[Locked]` | — |

Polling comes from `Filament\Widgets\Concerns\CanPoll`:

```php
protected ?string $pollingInterval = '5s';
```

**Widgets poll every 5 seconds by default.** There is **no `->poll()` method on widgets** — `poll()` exists
only on schema components and tables.

Lazy loading comes from `Filament\Support\Concerns\CanBeLazy`:

```php
protected static bool $isLazy = true;          // STATIC
```

Writing a non-static `protected bool $isLazy = false;` creates a shadow property and has no effect.

### Chart filters

Use the `HasFiltersSchema` concern — there is **no** `HasFilters` / `HasFiltersForm` in the widgets package:

```php
use Filament\Schemas\Schema;
use Filament\Widgets\ChartWidget;
use Filament\Widgets\ChartWidget\Concerns\HasFiltersSchema;

class RevenueChart extends ChartWidget
{
    use HasFiltersSchema;

    protected bool $hasDeferredFilters = true;

    public function filtersSchema(Schema $schema): Schema
    {
        return $schema->components([
            Select::make('period')->options([...]),
        ]);
    }

    protected function getData(): array
    {
        return match ($this->filters['period'] ?? null) {
            // ...
        };
    }
}
```

The simpler legacy mechanism is `protected function getFilters(): ?array` plus `public ?string $filter`.

### Dashboard

**There is no `Panel::dashboard()` method.** The dashboard is the page class `Filament\Pages\Dashboard`,
registered via `->pages([Dashboard::class])` and customised by extending it:

```php
namespace App\Filament\Pages;

use Filament\Pages\Dashboard as BaseDashboard;

class Dashboard extends BaseDashboard
{
    protected static string $routePath = '/';

    protected static ?int $navigationSort = -2;

    public function getColumns(): int | array
    {
        return ['md' => 4, 'xl' => 5];
    }
}
```

A second dashboard:

```php
class FinanceDashboard extends BaseDashboard
{
    protected static string $routePath = 'finance';

    protected static ?string $title = 'Finance dashboard';

    protected static ?int $navigationSort = 15;
}
```

Dashboard filters use `Filament\Pages\Dashboard\Concerns\HasFiltersForm` (or `HasFiltersAction`); widgets
consume them through `Filament\Widgets\Concerns\InteractsWithPageFilters`, which provides
`#[Reactive] public ?array $pageFilters = null`.

### Table widget

```php
namespace App\Filament\Widgets;

use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Table;
use Filament\Widgets\TableWidget;

class LatestOrders extends TableWidget
{
    protected function table(Table $table): Table
    {
        return $table
            ->query(Order::query()->latest()->limit(5))
            ->columns([
                TextColumn::make('reference'),
                TextColumn::make('total')->money('USD'),
            ])
            ->paginated(false);
    }
}
```

`TableWidget` forces `PaginationMode::Simple` and derives a heading from the class name unless you set
`->heading()` in `table()`. `protected static ?string $heading` is `@deprecated`.

### Sharing table state with widgets

| Purpose | Trait | Applied to |
|---|---|---|
| Page exposes table state | `Filament\Pages\Concerns\ExposesTableToWidgets` | the **page** class |
| Widget consumes table state | `Filament\Widgets\Concerns\InteractsWithPageTable` | the **widget** class |

`Filament\Widgets\ExposesTableToWidgets` (the v3 name) **does not exist in v5** — there is no `class_alias`
shim. `InteractsWithPageTable` physically lives in the **panels** package
(`packages/panels/src/Widgets/Concerns/InteractsWithPageTable.php`).

```php
use Filament\Pages\Concerns\ExposesTableToWidgets;
use Filament\Resources\Pages\ListRecords;

class ListProducts extends ListRecords
{
    use ExposesTableToWidgets;
}
```

```php
use Filament\Widgets\Concerns\InteractsWithPageTable;
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class ProductStats extends StatsOverviewWidget
{
    use InteractsWithPageTable;

    protected function getTablePage(): string
    {
        return ListProducts::class;
    }

    protected function getStats(): array
    {
        return [
            Stat::make(__('widgets.total_products'), (string) $this->tableRecordsCount),
        ];
    }
}
```

`ExposesTableToWidgets` supplies `getWidgetData()` with `activeTab`, `paginators`, `parentRecord`,
`tableColumnSearches`, `tableFilters`, `tableGrouping`, `tableRecordsCount`, `tableRecordsPerPage`,
`tableSearch`, `tableSort`.

`InteractsWithPageTable` requires `getTablePage()` (throws `LogicException` if missing) and provides
`getTablePageMountParameters()`, `getTablePageInstance()`, `getPageTableQuery()`, `getPageTableRecords()`.

> `$record` and `$parentRecord` on widgets use **Livewire's native restoration**. Filament does not
> re-apply global scopes or policies. Authorize them yourself.

### Deprecated widget APIs

`getVisibleWidgets()`, `getVisibleHeaderWidgets()`, `getVisibleFooterWidgets()`, and
`filterVisibleWidgets()` are all `@deprecated` in favour of
`getWidgetsSchemaComponents(array $widgets, array $data = [])`, which filters `canView()`, wraps each widget
in a `Livewire::make(...)`, merges `getWidgetData()`, and injects `pageFilters`.

### Registration

```php
$panel
    ->widgets([
        AccountWidget::class,
        FilamentInfoWidget::class,
    ])
    ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets');
```

`->widgets()` **appends** to the existing list rather than replacing it, so combining it with
`->discoverWidgets()` merges both sources. Use `->widgets([])` to remove the two defaults.

---

## 10. Enums

```php
<?php

namespace App\Enums;

use Filament\Support\Contracts\HasColor;
use Filament\Support\Contracts\HasIcon;
use Filament\Support\Contracts\HasLabel;
use Filament\Support\Icons\Heroicon;

enum OrderStatus: string implements HasLabel, HasColor, HasIcon
{
    case Draft = 'draft';
    case Sent = 'sent';
    case Paid = 'paid';
    case Cancelled = 'cancelled';

    public function getLabel(): string
    {
        return __("enums.order-status.{$this->value}");
    }

    public function getColor(): string
    {
        return match ($this) {
            self::Draft => 'gray',
            self::Sent => 'info',
            self::Paid => 'success',
            self::Cancelled => 'danger',
        };
    }

    public function getIcon(): Heroicon
    {
        return match ($this) {
            self::Draft => Heroicon::OutlinedPencil,
            self::Sent => Heroicon::OutlinedPaperAirplane,
            self::Paid => Heroicon::OutlinedCheckCircle,
            self::Cancelled => Heroicon::OutlinedXCircle,
        };
    }
}
```

Cast it on the model (Laravel 11+ style):

```php
protected function casts(): array
{
    return [
        'status' => OrderStatus::class,
    ];
}
```

Use it in Filament:

```php
// Select options from an enum
Select::make('status')->options(OrderStatus::class)

// Table filter
SelectFilter::make('status')->options(OrderStatus::class)

// Column badge, driven by the enum's own colour
TextColumn::make('status')
    ->badge()
    ->color(fn (OrderStatus $state): string => $state->getColor())
```

### Contracts

| Contract | Method |
|---|---|
| `Filament\Support\Contracts\HasLabel` | `getLabel(): string` |
| `Filament\Support\Contracts\HasColor` | `getColor(): string \| array` |
| `Filament\Support\Contracts\HasIcon` | `getIcon(): string \| BackedEnum \| null` |
| `Filament\Support\Contracts\HasDescription` | `getDescription(): string` |
| `Filament\Support\Contracts\ScalableIcon` | `getIconForSize(IconSize $size): string` |

> `HasColor` and `HasIcon` exist as **both** interfaces (`Filament\Support\Contracts`) and traits
> (`Filament\Support\Concerns`). Import deliberately.

### Colors

`Filament\Support\Colors\Color` is a **plain class of array constants**, not an enum:

```php
use Filament\Support\Colors\Color;

$panel->colors([
    'primary' => Color::Amber,
    'danger' => Color::Rose,
]);
```

Available palettes: `Slate, Gray, Zinc, Neutral, Stone, Mauve, Olive, Mist, Taupe, Red, Orange, Amber,
Yellow, Lime, Green, Emerald, Teal, Cyan, Sky, Blue, Indigo, Violet, Purple, Fuchsia, Pink, Rose`.

Semantic color names for `->color()`: `primary`, `secondary`, `success`, `warning`, `danger`, `info`,
`gray`.

### Icons

`Filament\Support\Icons\Heroicon` — 647 cases in two families:

```php
Heroicon::ShoppingBag           // 'shopping-bag'     323 solid/default cases
Heroicon::OutlinedShoppingBag   // 'o-shopping-bag'   324 outlined cases
```

There are **no** `Solid*`, `Mini*`, or `Micro*` cases. For un-prefixed cases the rendered size variant is
chosen by `getIconForSize()`: `heroicon-c-*` (xs/sm), `heroicon-m-*` (md, default), `heroicon-s-*`
(lg/xl/2xl). `Outlined*` always renders `heroicon-o-*`.

```php
protected static string | BackedEnum | null $navigationIcon = Heroicon::OutlinedRectangleStack;
```

Icon aliases: `FilamentIcon::resolve(SomeIconAlias::CONST) ?? Heroicon::OutlinedFoo`.

---

## 11. Multi-tenancy

### Panel

```php
use App\Models\Team;

$panel
    ->tenant(Team::class)
    ->tenant(Team::class, slugAttribute: 'slug')      // path-based with a slug
    ->tenantDomain('{tenant:slug}.example.com')        // domain-based
    ->tenantMiddleware([ApplyTenantScopes::class], isPersistent: true)
    ->tenantRoutePrefix('t')
    ->tenantMenu(false)
    ->tenantProfile(EditTeamProfile::class)
    ->tenantRegistration(RegisterTeam::class);
```

### User model

```php
use Filament\Models\Contracts\FilamentUser;
use Filament\Models\Contracts\HasTenants;
use Filament\Panel;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Collection;

class User extends Authenticatable implements FilamentUser, HasTenants
{
    public function teams(): BelongsToMany
    {
        return $this->belongsToMany(Team::class);
    }

    public function getTenants(Panel $panel): Collection
    {
        return $this->teams;
    }

    public function canAccessTenant(Model $tenant): bool
    {
        return $this->teams()->whereKey($tenant)->exists();
    }

    public function canAccessPanel(Panel $panel): bool
    {
        return match ($panel->getId()) {
            'admin' => $this->is_admin,
            default => true,
        };
    }
}
```

### Scoping a resource

```php
protected static bool $isScopedToTenant = true;                 // default
protected static ?string $tenantOwnershipRelationshipName = null;
protected static ?string $tenantRelationshipName = null;
```

Disable per resource:

```php
protected static bool $isScopedToTenant = false;
```

Invert the default globally, then opt in per resource:

```php
use Filament\Resources\Resource;

// service provider boot() or middleware
Resource::scopeToTenant(false);
```

### Scope form fields yourself — the global scope will not

```php
use Filament\Facades\Filament;
use Illuminate\Database\Eloquent\Builder;

Select::make('team_id')
    ->relationship(
        name: 'team',
        titleAttribute: 'name',
        modifyQueryUsing: fn (Builder $query) => $query->whereBelongsTo(Filament::getTenant()),
    )
    ->searchable()
    ->preload();
```

### Tenant-aware validation

```php
TextInput::make('email')
    ->scopedUnique();      // applies the model's global scopes

TextInput::make('code')
    ->scopedExists();
```

### Extra global scopes for models without a resource

```php
namespace App\Http\Middleware;

use App\Models\Author;
use Closure;
use Filament\Facades\Filament;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Http\Request;

class ApplyTenantScopes
{
    public function handle(Request $request, Closure $next)
    {
        Author::addGlobalScope(
            'tenant',
            fn (Builder $query) => $query->whereBelongsTo(Filament::getTenant()),
        );

        return $next($request);
    }
}
```

Register it as persistent tenant middleware so it also runs on Livewire AJAX requests.

### Tenancy security boundaries — verified

- Global scoping applies **only** to models that have a tenant-aware resource in a tenancy-enabled panel.
- Scopes are applied **after** the tenant is resolved from the request. Queries in early middleware or
  service providers are not scoped.
- Queries made **outside** the panel are not scoped.
- New records are associated with the current tenant automatically on `creating`/`created` — again only for
  models with a tenant-aware resource.
- On each Livewire request, resource pages re-resolve their record through the resource binding query
  (including tenant scoping). **This does not extend to relation manager `$ownerRecord`, widget
  `$record`/`$parentRecord`, or modal-table-select `$record`.**
- Escape hatch: `withoutGlobalScope(filament()->getTenancyScopeName())`. Never disable all global scopes
  blindly — pass an explicit list to `withoutGlobalScopes([...])`.

---

## 12. Authorization and security

### Policy abilities

| Operation | Ability |
|---|---|
| List / view any | `viewAny` |
| View one | `view` |
| Create | `create` |
| Edit / update | `update` |
| Delete | `delete` |
| Delete bulk | `deleteAny` |
| Restore | `restore` |
| Restore bulk | `restoreAny` |
| Force delete | `forceDelete` |
| Force delete bulk | `forceDeleteAny` |
| Replicate | `replicate` |
| Reorder | `reorder` |

### Skip authorization

```php
protected static bool $shouldSkipAuthorization = true;
```

### Strict authorization mode

```php
$panel->strictAuthorization();
```

By default Filament **grants** access when the policy or policy method does not exist. Strict mode throws
instead. This is controlled by `protected static bool $shouldCheckPolicyExistence = true;` on the resource.

### Hide vs authorize

`$shouldRegisterNavigation = false` and `->hidden()` only hide UI — they do **not** restrict URL access.
Authorization must come from policies and `canAccess*()` methods.

### `canAccessPanel()` is mandatory in production

When `APP_ENV` is not `local`, Filament requires the `FilamentUser` contract. Without it, **no one can sign
in** in production. Panel access is checked on every request, including Livewire requests.

### Authorizing custom actions

```php
Action::make('refund')
    ->authorize('refund')                       // Gate::check('refund', [$record])
    ->action(fn (Order $order) => $order->refund());

BulkAction::make('archive')
    ->authorizeIndividualRecords('archive');
```

Custom actions have **no** automatic policy lookup — authorization defaults to allowed. `->visible()` and
`->hidden()` are re-checked at call time and also drive `disabled()`.

### Sanitizing untrusted values

```php
use Illuminate\Support\Str;

TextColumn::make('website')
    ->url(fn (string $state): ?string => Str::sanitizeUrl($state));

TextColumn::make('contact')
    ->url(fn (string $state): ?string => Str::sanitizeUrl(
        $state,
        allowedSchemes: ['http', 'https', 'mailto', 'tel'],
    ));
```

- `Str::sanitizeUrl()` returns the URL when schemeless or `http`/`https`, and `null` otherwise. It decodes
  HTML entities and rejects control characters and raw whitespace. It does **not** provide open-redirect or
  SSRF protection, and it does not escape — Filament escapes when rendering into an HTML attribute.
- `Str::sanitizeCssColor()` allows hex colours, bare CSS keywords, and the `rgb()`/`hsl()`/`lab()`/`oklch()`
  family; everything else is rejected.
- `extraAttributes()`, `extraInputAttributes()`, `extraCellAttributes()` and friends render **without
  escaping** by design. Never pass user-controlled data.
- `->icon()` with user input should be validated against an allowlist — an invalid icon name is a rendering
  error.

### Sanitizing rich text in your own views

```blade
{!! str($record->content)->sanitizeHtml() !!}
{!! str($record->content)->markdown()->sanitizeHtml() !!}
```

Never `{!! $content !!}` with unsanitized user content.

### Restricting Livewire uploads to schema components

```php
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Concerns\RestrictsFileUploadsToSchemaComponents;
use Filament\Schemas\Contracts\HasSchemas;
use Livewire\Component;

class ViewProduct extends Component implements HasSchemas
{
    use InteractsWithSchemas;
    use RestrictsFileUploadsToSchemaComponents;
}
```

Every component using `InteractsWithSchemas` exposes `_startUpload` / `_finishUpload` for **any** property
name. Add the restriction trait on any component reachable by users who should not upload files.

### Model attribute exposure

Filament exposes all non-`$hidden` model attributes to JavaScript via Livewire's model binding. Add
sensitive columns to `$hidden`, or strip them in the Edit/View page:

```php
protected function mutateFormDataBeforeFill(array $data): array
{
    unset($data['internal_api_key']);

    return $data;
}
```

### Protecting your own model properties

```php
use Illuminate\Support\Facades\Gate;
use Livewire\Attributes\Computed;
use Livewire\Attributes\Locked;

#[Locked]
public int $postId;

#[Computed]
public function post(): Post
{
    $post = Post::query()->findOrFail($this->postId);

    Gate::authorize('view', $post);

    return $post;
}
```

Livewire restores public properties **before** authorization runs. A page's `boot()`, trait initializers,
early `mount()`/`hydrate()` code, and a widget's or relation manager's `boot()` can all run first. Never
perform writes, events, audit logging, or external calls in those hooks without authorizing first.

Do not enable Livewire lazy loading on a Filament page that relies on page authorization hooks — pages
have no authorization check while a lazy component waits to mount (widgets and relation managers do).

### CSP

```php
use Illuminate\Support\Facades\Vite;

Vite::useCspNonce($nonce);      // must run per request
```

Register CSP middleware on the **panel** with `->middleware([...])` — panels build their own middleware
stack and never resolve Laravel's `web` group. Filament adds the nonce to scripts it renders itself; you
must add it to your own render hooks, `scripts` stack, `RawJs`, and `Js::make()->html()` output via
`Vite::cspNonce()`. Alpine and Livewire still need `'unsafe-eval'`. File upload previews need
`worker-src 'self' blob:`.

---

## 13. Panel provider

```php
<?php

namespace App\Providers\Filament;

use App\Http\Middleware\ApplyTenantScopes;
use Filament\Http\Middleware\Authenticate;
use Filament\Http\Middleware\AuthenticateSession;
use Filament\Http\Middleware\DisableBladeIconComponents;
use Filament\Http\Middleware\DispatchServingFilamentEvent;
use Filament\Pages\Dashboard;
use Filament\Panel;
use Filament\PanelProvider;
use Filament\Support\Colors\Color;
use Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse;
use Illuminate\Cookie\Middleware\EncryptCookies;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Routing\Middleware\SubstituteBindings;
use Illuminate\Session\Middleware\StartSession;
use Illuminate\View\Middleware\ShareErrorsFromSession;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->default()
            ->id('admin')
            ->path('admin')
            ->domain('admin.example.com')
            ->login()
            ->registration()
            ->passwordReset()
            ->emailVerification()
            ->profile()
            ->brandName('Acme')
            ->brandLogo(asset('images/logo.svg'))
            ->favicon(asset('images/favicon.png'))
            ->colors([
                'primary' => Color::Emerald,
                'danger' => Color::Rose,
            ])
            ->darkMode()
            ->viteTheme('resources/css/filament/admin/theme.css')
            ->spa()
            ->databaseTransactions()
            ->unsavedChangesAlerts()
            ->strictAuthorization()
            ->sidebarCollapsibleOnDesktop()
            ->collapsibleNavigationGroups()
            ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
            ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
            ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets')
            ->discoverClusters(in: app_path('Filament/Clusters'), for: 'App\\Filament\\Clusters')
            ->pages([
                Dashboard::class,
            ])
            ->widgets([])
            ->middleware([
                EncryptCookies::class,
                AddQueuedCookiesToResponse::class,
                StartSession::class,
                AuthenticateSession::class,
                ShareErrorsFromSession::class,
                VerifyCsrfToken::class,
                SubstituteBindings::class,
                DisableBladeIconComponents::class,
                DispatchServingFilamentEvent::class,
            ])
            ->authMiddleware([
                Authenticate::class,
            ]);
    }
}
```

### Selecting a panel in tests

```php
use Filament\Facades\Filament;

Filament::setCurrentPanel('admin');
```

`setCurrentPanel(Panel|string|null $panel)`, `setTenant(?Model $tenant, bool $isQuiet = false)`,
`getCurrentPanel()`, `getCurrentOrDefaultPanel()`, `getPanel(?string $id)`, `bootCurrentPanel()`.

---

## 14. Testing

### Pest skeleton

```php
<?php

use App\Filament\Resources\Customers\Pages\CreateCustomer;
use App\Filament\Resources\Customers\Pages\EditCustomer;
use App\Filament\Resources\Customers\Pages\ListCustomers;
use App\Models\Customer;
use App\Models\User;
use Filament\Actions\Testing\TestAction;
use function Pest\Livewire\livewire;

beforeEach(function () {
    $this->actingAs(User::factory()->create());
});

it('lists customers', function () {
    $customers = Customer::factory()->count(5)->create();

    livewire(ListCustomers::class)
        ->assertOk()
        ->assertCanSeeTableRecords($customers);
});

it('searches customers', function () {
    $customers = Customer::factory()->count(5)->create();
    $target = $customers->first();

    livewire(ListCustomers::class)
        ->searchTable($target->name)
        ->assertCanSeeTableRecords([$target])
        ->assertCanNotSeeTableRecords($customers->skip(1));
});

it('creates a customer', function () {
    livewire(CreateCustomer::class)
        ->fillForm([
            'name' => 'Nuno Maduro',
            'email' => 'nuno@example.com',
        ])
        ->call('create')
        ->assertHasNoFormErrors()
        ->assertNotified()
        ->assertRedirect();

    $this->assertDatabaseHas(Customer::class, [
        'email' => 'nuno@example.com',
    ]);
});

it('validates the name', function () {
    livewire(CreateCustomer::class)
        ->fillForm(['name' => null, 'email' => 'not-an-email'])
        ->call('create')
        ->assertHasFormErrors(['name' => 'required', 'email' => 'email'])
        ->assertNotNotified();
});

it('edits a customer', function () {
    $customer = Customer::factory()->create();

    livewire(EditCustomer::class, ['record' => $customer->id])
        ->assertSchemaStateSet(['name' => $customer->name])
        ->fillForm(['name' => 'Updated'])
        ->call('save')
        ->assertHasNoFormErrors()
        ->assertNotified();

    $this->assertDatabaseHas(Customer::class, [
        'id' => $customer->id,
        'name' => 'Updated',
    ]);
});

it('sends an invoice from a row action', function () {
    $invoice = Invoice::factory()->create();

    livewire(ListInvoices::class)
        ->callAction(TestAction::make('send')->table($invoice), [
            'subject' => 'Your invoice',
        ])
        ->assertNotified();
});

it('bulk deletes selected customers', function () {
    $customers = Customer::factory()->count(3)->create();

    livewire(ListCustomers::class)
        ->selectTableRecords($customers)
        ->callAction(TestAction::make('delete')->table()->bulk())
        ->assertNotified()
        ->assertCanNotSeeTableRecords($customers);
});
```

### Multi-tenant / multi-panel

```php
use Filament\Facades\Filament;

beforeEach(function () {
    Filament::setCurrentPanel('admin');
    Filament::setTenant($this->team);
    Filament::bootCurrentPanel();
});
```

### Relation manager tests

```php
use App\Filament\Resources\Customers\Pages\EditCustomer;
use App\Filament\Resources\Customers\RelationManagers\OrdersRelationManager;

livewire(OrdersRelationManager::class, [
    'ownerRecord' => $customer,
    'pageClass' => EditCustomer::class,
])->assertCanSeeTableRecords($customer->orders);
```

Or assert nesting on the page: `->assertSeeLivewire(OrdersRelationManager::class)`.

### Testing authorization through the request lifecycle

```php
it('rejects access after the tenant changes', function () {
    $order = Order::factory()->for($teamA)->create();

    $component = livewire(EditOrder::class, ['record' => $order->id])
        ->assertOk();

    Filament::setTenant($teamB);

    $component->call('save')->assertStatus(404);
});
```

---

## 15. v3/v4 → v5 rename map

| Old | v5 |
|---|---|
| `Table::actions()` | **`Table::recordActions()`** |
| `Table::pushActions()` | `Table::pushRecordActions()` |
| `Table::actionsColumnLabel()` / `actionsAlignment()` / `actionsPosition()` | `recordActionsColumnLabel()` / `recordActionsAlignment()` / `recordActionsPosition()` |
| `Table::getActions()` | `Table::getRecordActions()` |
| `Table::bulkActions()` | **`Table::toolbarActions()`** |
| `Table::pushBulkActions()` | `Table::pushToolbarActions()` |
| `Table::getBulkActions()` | `Table::getToolbarActions()` |
| `Action::form()` | **`Action::schema()`** |
| `Action::getForm()` | `Action::getSchema()` |
| `Action::disableForm()` / `disabledForm()` | `Action::disabledSchema()` |
| `Action::isFormDisabled()` | `Action::isSchemaDisabled()` |
| `Filter::form()` | **`Filter::schema()`** |
| `Filter::getFormSchema()` | `Filter::getSchemaComponents()` |
| `Action::mutateFormDataUsing()` | `Action::mutateDataUsing()` |
| `Action::hold()` | `Action::halt()` |
| `Action::afterReplicaSaved()` | `Action::after()` |
| `Action::modalSubheading()` | `Action::modalDescription()` |
| `Action::modalFooter()` | `Action::modalContentFooter()` |
| `Action::modalButton()` | `Action::modalSubmitActionLabel()` |
| `Action::centerModal()` | `Action::modalAlignment(Alignment::Center)` |
| `Page::getActions()` (header buttons) | `Page::getHeaderActions()` |
| `CreateRecord::getCreatedNotificationMessage()` | `getCreatedNotificationTitle()` |
| `EditRecord::getSavedNotificationMessage()` | `getSavedNotificationTitle()` |
| `Resource::$label` / `$pluralLabel` | `$modelLabel` / `$pluralModelLabel` |
| `Resource::getLabel()` / `getPluralLabel()` | `getModelLabel()` / `getPluralModelLabel()` |
| `Filament\Tables\Actions\*` (action classes) | **`Filament\Actions\*`** |
| `Filament\Forms\Get` / `Filament\Forms\Set` | **`Filament\Schemas\Components\Utilities\Get` / `Set`** |
| `Filament\Tables\Actions\ActionGroup` | `Filament\Actions\ActionGroup` |
| `Filament\Forms\Components\Actions` | `Filament\Schemas\Components\Actions` |
| `Filament\Forms\Components\Section` / `Grid` / `Tabs` / `Wizard` / `Fieldset` | `Filament\Schemas\Components\*` |
| `Filament\Forms\Form` | `Filament\Schemas\Schema` |
| `Filament\Infolists\Infolist` | `Filament\Schemas\Schema` |
| `MultiselectFilter` | `SelectFilter::make()->multiple()` |
| `BadgeColumn` | `TextColumn::make()->badge()` |
| `BooleanColumn` | `IconColumn::make()->boolean()` |
| `TagsColumn` | `TextColumn::make()->badge()` |
| `Filament\Widgets\ExposesTableToWidgets` | `Filament\Pages\Concerns\ExposesTableToWidgets` |
| `Filament\Tables\Filters\QueryBuilder\Constraints\*` | `Filament\QueryBuilder\Constraints\*` |
| `SubNavigationPosition` under `Filament\Navigation` | `Filament\Pages\Enums\SubNavigationPosition` |

### Test-method renames (not automated by the upgrade script)

| Old | v5 |
|---|---|
| `assertFormSet()` | `assertSchemaStateSet()` |
| `assertFormExists()` | `assertSchemaExists()` |
| `assertFormFieldHidden()` / `assertFormFieldVisible()` | `assertSchemaComponentHidden()` / `assertSchemaComponentVisible()` |
| `assertFormComponentExists()` / `...DoesNotExist()` | `assertSchemaComponentExists()` / `...DoesNotExist()` |
| `assertTableActionExists()` / `assertTableBulkActionExists()` | `assertActionExists()` |
| `assertTableActionVisible()` / `Hidden()` / `Enabled()` / `Disabled()` | `assertActionVisible()` / `Hidden()` / `Enabled()` / `Disabled()` |
| `callTableAction()` / `callTableBulkAction()` / `callFormComponentAction()` | `callAction()` |
| `mountTableAction()` | `mountAction()` |
| `mountTableBulkAction()` | `selectTableRecords()` + `mountAction()` |
| `setActionData()` / `setTableActionData()` / `setTableBulkActionData()` | `fillForm()` |
| `assertActionDataSet()` / `assertTableActionDataSet()` | `assertSchemaStateSet()` |
| `assertHasActionErrors()` / `assertHasTableActionErrors()` / `assertHasTableBulkActionErrors()` | `assertHasFormErrors()` |
| `assertHasNoActionErrors()` / `assertHasNoTableActionErrors()` | `assertHasNoFormErrors()` |
| `assertTableBulkActionsExistInOrder()` | `assertActionListInOrder()` |
| `assertActionHeld()` | `assertActionHalted()` |

Filament ships an automated upgrade tool:

```bash
composer require filament/upgrade:"^5.0" -W --dev
vendor/bin/filament-v5
```

---

## 16. Deprecated names that still compile

These still exist in v5.10.1 and **silently proxy** to the modern method. Linters and static analysis will
not flag them, so they are the most likely way to ship v3/v4 style code into a v5 app.

**Tables:** `actions()`, `pushActions()`, `actionsColumnLabel()`, `actionsAlignment()`, `actionsPosition()`,
`getActions()`, `bulkActions()`, `pushBulkActions()`, `getBulkActions()`.

Not deprecated, and easy to mislabel: **`groupedBulkActions()` is current** — it is a shorthand for
`toolbarActions([BulkActionGroup::make([...])])`, not the rename target of `bulkActions()`.

**Columns:** `BadgeColumn`, `BooleanColumn`, `TagsColumn`,
`Filament\Tables\Columns\Concerns\CanBeCopied`,
`TagsColumn::limit()`, `ImageColumn::height()`, `ImageColumn::icons()`.

**Filters:** `form()`, `getFormSchema()`, `MultiSelectFilter`.

**Actions:** `form()`, `getForm()`, `disableForm()`, `disabledForm()`, `isFormDisabled()`,
`mutateFormDataUsing()`, `hold()`, `afterReplicaSaved()`, `modalButton()`, `modalSubheading()`,
`modalActions()`, `extraModalActions()`, `makeExtraModalAction()`, `modalFooter()`, `centerModal()`,
`successNotificationMessage()`, `failureNotificationMessage()`, `ButtonAction`, `IconButtonAction`.

**Pages & resources:** `getActions()` (header), `getCreatedNotificationMessage()`,
`getSavedNotificationMessage()`, `getVisibleHeaderWidgets()`, `getVisibleFooterWidgets()`,
`filterVisibleWidgets()`, `ListRecords::getPluralModelLabel()`, `ListRecords::getTableQuery()`,
`$label`, `$pluralLabel`, `getLabel()`, `getPluralLabel()`.

**Relation managers:** `$recordTitleAttribute`, `$inverseRelationship`, `$label`, `$pluralLabel`,
`$modelLabel`, `$pluralModelLabel`, and their getters.

**Groups:** `Group::getDescriptionUsing()` → `getDescriptionFromRecordUsing()`.

**InteractsWithTable:** `getTableQuery()`.

---

## Further reading

- Official docs in the repository: `docs/` (panel-level) and `packages/{package}/docs/` (component-level).
- Filament's own agent skill: `packages/panels/resources/boost/skills/filament-development/SKILL.md`,
  installed with `php artisan boost:install` → **Agent Skills** → `filament/filament (skills)`.
- Component CSS hooks and `fi-` prefixes: `docs/08-styling/02-css-hooks.md`.
- Security model and its boundaries: `docs/09-advanced/06-security.md`.
