---
name: filament-v5-development
description: Use when writing, reviewing, refactoring, or debugging Filament PHP v5 code — resources, pages, schemas, forms, tables, columns, filters, actions, bulk actions, infolists, widgets, panels, relation managers, clusters, multi-tenancy, authorization, testing, plugins, custom themes, or imports and exports. Always apply these standards to all Filament v5 work.
---

# Filament PHP v5 Coding Standards

<div align="right">

**English** · [Русский README](../../../README.ru.md)

</div>

Standards for building production-grade Filament v5 applications. Every rule below is verified against the
`filamentphp/filament` source at tag **v5.10.1**.

> Filament ships its own agent skill at `packages/panels/resources/boost/skills/filament-development/SKILL.md`
> (installed via `php artisan boost:install` → **Agent Skills** → `filament/filament (skills)`). This document
> is a complementary, deeper standard. Where the two overlap, they agree — see §19 for the reconciliations.

## Platform Requirements

| Requirement | Constraint | Where it is declared |
|---|---|---|
| PHP | `^8.2` | every `packages/*/composer.json` |
| `ext-intl` | hard requirement | `packages/support/composer.json` |
| Laravel | `^11.28 \| ^12.0 \| ^13.0` (via `illuminate/contracts`) | `packages/support/composer.json` — the **only** place Laravel is constrained |
| Livewire | `^4.4.7` | `packages/support/composer.json` |
| Tailwind CSS | v4.0+ (docs recommend v4.1+) | `docs/01-introduction/02-installation.md` |

Note: reading `packages/panels/composer.json` alone tells you nothing about Laravel or Livewire versions —
only `filament/support` carries that envelope.


For longer code examples, see [REFERENCE.md](REFERENCE.md).

## When to Apply

Apply to ALL Filament v5 work: resources, pages, relation managers, clusters, schemas, forms, infolists,
tables, columns, filters, actions, bulk actions, notifications, widgets, dashboards, panels, multi-tenancy,
authorization, global search, imports/exports, custom themes, plugins, and tests.

---

## 1. The v5 Mental Model

Filament v5 reorganised the framework around **one unifying abstraction: `Schema`**.

- `Filament\Schemas\Schema` now backs **forms, infolists, resource forms, resource infolists, action modals, and filter forms**.
- Therefore `->form()` is **deprecated** everywhere it used to exist. Use `->schema()`.
- All **actions** — page, table, and bulk — now live in the single namespace `Filament\Actions\*`.
  `Filament\Tables\Actions\*` no longer contains action classes (only the `HeaderActionsPosition` enum).
- Layout components (`Section`, `Grid`, `Tabs`, `Wizard`, `Fieldset`, `Flex`, `Callout`, …) live in
  `Filament\Schemas\Components\*`. Field inputs (`TextInput`, `Select`, `Repeater`, …) still live in
  `Filament\Forms\Components\*`. Entries (`TextEntry`, `IconEntry`, …) live in `Filament\Infolists\Components\*`.
- **A new content-slot API replaced the old single-slot conveniences.** Fields, entries, and layout
  components now expose ten named slots — `aboveLabel()`, `belowLabel()`, `beforeLabel()`, `afterLabel()`,
  `aboveContent()`, `belowContent()`, `beforeContent()`, `afterContent()`, `aboveErrorMessage()`,
  `belowErrorMessage()` — each accepting a component, `Schema`, action, string, `Htmlable`, or `Closure`.
  `helperText()`, `hint()`, and `hintIcon()` still exist and are sugar over these slots (or, for hints,
  rendered after the label).
- **`description()` is no longer a form-field method** — it survives only on layout components such as
  `Section`, `Callout`, `EmptyState`, and `Wizard\Step`.

---

## 2. Project Structure

### Directory Layout

- Resources are grouped in **plural, domain-oriented directories**:
  `app/Filament/Resources/Customers/`, `app/Filament/Resources/Shop/Products/`.
- The v5 generated layout per resource is:

```
app/Filament/Resources/Customers/
├── CustomerResource.php
├── Pages/
│   ├── CreateCustomer.php
│   ├── EditCustomer.php
│   └── ListCustomers.php
├── Schemas/
│   └── CustomerForm.php
└── Tables/
    └── CustomersTable.php
```

- Schemas extracted to `<Resource>/Schemas/`, tables to `<Resource>/Tables/`.
- Panels: one `PanelProvider` per panel under `app/Providers/Filament/`
  (`AdminPanelProvider.php`, `AppPanelProvider.php`, …).

### Naming Rules

- Plural resource directories: `Customers/`, never `Customer/`.
- File name matches class name: `CustomerResource.php`.
- Slugs are kebab-case: `order-items`.
- Namespace must match the directory path exactly.

### Resource Complexity Tiers

| Tier | Pages | Use when |
|---|---|---|
| Simple | `ManageRecords` | Modal CRUD, no separate pages |
| Standard | `ListRecords` + `CreateRecord` + `EditRecord` | Normal CRUD |
| Full | Standard + `ViewRecord` | Read-only detail page, infolists, relation managers |

---

## 3. Resource Architecture

### Delegate to Schema and Table Classes

`configure()` classes have **no parent class and no interface** by design, so you can pass your own
configuration parameters and reuse the class in several places with slight tweaks.

```php
namespace App\Filament\Resources\Customers;

use App\Filament\Resources\Customers\Schemas\CustomerForm;
use App\Filament\Resources\Customers\Tables\CustomersTable;
use Filament\Resources\Resource;
use Filament\Schemas\Schema;
use Filament\Tables\Table;

class CustomerResource extends Resource
{
    protected static ?string $model = Customer::class;

    protected static ?string $recordTitleAttribute = 'name';

    public static function form(Schema $schema): Schema
    {
        return CustomerForm::configure($schema);
    }

    public static function table(Table $table): Table
    {
        return CustomersTable::configure($table);
    }
}
```

`Resource` exposes these signatures in v5.10.1:

```php
public static function form(Schema $schema): Schema
public static function infolist(Schema $schema): Schema
public static function table(Table $table): Table
public static function configureTable(Table $table): void   // new in v5
public static function getEloquentQuery(): Builder
public static function getRelations(): array
public static function getWidgets(): array
```

### Key Rules

- **Always set `$recordTitleAttribute`** — it powers global search result titles and record labels.
- Use the `Heroicon` enum, never raw strings: `use Filament\Support\Icons\Heroicon;`.
- **All actions import from `Filament\Actions\*`** — never `Filament\Tables\Actions\*`.
- Break long `configure()` methods into per-component classes (`CustomerNameInput`, `CustomerCountryColumn`,
  `CustomerStatusFilter`, `EmailCustomerAction`).

---

## 4. Resource Properties (exact v5 types)

These are the real declarations from `packages/panels/src/Resources/Resource/Concerns/*` in v5.10.1.
Matching the exact type matters — `$navigationGroup` accepts a `UnitEnum` (a `NavigationGroup` enum case),
and `$navigationIcon` accepts a `BackedEnum` (a `Heroicon` case).

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

> `$label` and `$pluralLabel` still exist but are **deprecated** — use `$modelLabel` / `$pluralModelLabel`.

Add a navigation badge or tooltip through the method API, not properties:

```php
public static function getNavigationBadge(): ?string
{
    return static::getModel()::where('is_active', false)->count() ?: null;
}

public static function getNavigationBadgeColor(): string | array | null
{
    return 'warning';
}
```

---

## 5. Form & Schema Design

### Structure

- The top-level method is `$schema->components([...])`. `$schema->schema([...])` is a **valid alias** on the
  `Schema` class itself — but on every *component*, `->schema([...])` means "nest these children", so use
  `components()` at the top level for clarity and never confuse the two.
- Wrap related fields in `Section::make()` from `Filament\Schemas\Components\Section`.
- Layout components come from `Filament\Schemas\Components\*`; inputs from `Filament\Forms\Components\*`.
- Use `Operation` for create/edit conditional logic — never string comparisons.

```php
use Filament\Forms\Components\TextInput;
use Filament\Schemas\Components\Section;
use Filament\Schemas\Components\Utilities\Get;
use Filament\Schemas\Schema;
use Filament\Support\Enums\Operation;

public static function configure(Schema $schema): Schema
{
    return $schema
        ->components([
            Section::make('Details')
                ->schema([
                    TextInput::make('name')
                        ->required()
                        ->maxLength(255)
                        ->unique(ignoreRecord: true),
                    TextInput::make('vat_number')
                        ->required(fn (Get $get): bool => filled($get('company_name'))),
                    TextInput::make('slug')
                        ->required()
                        ->live(onBlur: true)
                        ->hiddenOn(Operation::Create)
                        ->disabled()
                        ->dehydrated(),
                ])
                ->columns(2),
        ]);
}
```

### Rules

- **Reactive fields**: `->live()` (with `onBlur: true` to avoid a request per keystroke).
  Read sibling state in the closure via the injected `Get` utility. A field only re-renders if it is
  `live()` — `Closure`s alone do not make a field reactive.
- **`Get` / `Set` moved namespace in v5**:
  `Filament\Schemas\Components\Utilities\Get` and `Filament\Schemas\Components\Utilities\Set`.
  The v3 paths `Filament\Forms\Get` / `Filament\Forms\Set` no longer exist.
- **Field help text is `helperText()`, not `description()`.** `description()` is **not** a method on form
  fields in v5 — it exists only on layout components (`Section`, `Callout`, `EmptyState`, `Wizard\Step`).
  `helperText()` is sugar over the `belowContent()` slot; calling `belowContent()` afterwards replaces it.
- **The ten content slots** on `Field` are `aboveLabel()`, `belowLabel()`, `beforeLabel()`, `afterLabel()`,
  `aboveContent()`, `belowContent()`, `beforeContent()`, `afterContent()`, `aboveErrorMessage()`,
  `belowErrorMessage()`. There is **no `aboveField()` / `belowField()`**, and the v3 field methods
  `->descriptionAbove()` / `->descriptionBelow()` were removed with no field-level replacement.
  (The names survive only inside `Filament\Tables\Columns\Concerns\HasDescription`, where
  `TextColumn::description()` accepts `position: 'above'|'below'` — a different API.)
- `hint()`, `hintIcon()`, `hintColor()`, and `hintAction()` exist but render **after the label**, not below
  the field. They are implemented but undocumented in v5.10.1 — prefer `helperText()`.
- **Selects with relationships** must be `->searchable()` and, for small option sets, `->preload()`.
  Scope them yourself (see §12). Note `->modifyQueryUsing()` is **not** a chainable method on `Select` —
  it is a named argument of `relationship()`.
- **File uploads**: always set `->disk()`, `->directory()`, and `->visibility()` explicitly
  (visibility defaults to `private`). Never enable `->preserveFilenames()` on user-supplied names.
  The method for hiding the remove button is `->deletable(false)` — there is no `deletableFileUploads()`.
- **Repeaters**: `->columns()` lays out the fields **inside each item**; `->grid()` lays out the
  **items themselves**. Use `Wizard::make()->steps([...])` for multi-step forms. (`Builder` has no `grid()`.)
- **Wizard**: `->nextAction()` / `->previousAction()` take action-modifier closures, but
  `->submitAction()` / `->cancelAction()` take `string|Htmlable|null` — rendered HTML, **not** an `Action`.
- **Every user-facing string goes through a language file.** Never hardcode copy.

```php
// Reactive sibling lookup with the v5 Get utility
use Filament\Schemas\Components\Utilities\Get;

Select::make('country_id')
    ->relationship('country', 'name')
    ->searchable()
    ->preload()
    ->live()
    ->afterStateUpdated(fn (Set $set) => $set('state_id', null)),

Select::make('state_id')
    ->options(fn (Get $get): array => State::where('country_id', $get('country_id'))->pluck('name', 'id')->all())
    ->searchable(),
```

---

## 6. Table Design

### Canonical `configure()`

```php
namespace App\Filament\Resources\Customers\Tables;

use Filament\Actions\BulkActionGroup;
use Filament\Actions\DeleteAction;
use Filament\Actions\DeleteBulkAction;
use Filament\Actions\EditAction;
use Filament\Tables\Columns\IconColumn;
use Filament\Tables\Columns\TextColumn;
use Filament\Tables\Filters\SelectFilter;
use Filament\Tables\Table;

class CustomersTable
{
    public static function configure(Table $table): Table
    {
        return $table
            ->columns([
                TextColumn::make('name')
                    ->searchable()
                    ->sortable(),
                TextColumn::make('status')
                    ->badge()
                    ->color(fn (OrderStatus $state): string => $state->getColor()),
                IconColumn::make('is_active')
                    ->boolean(),
                TextColumn::make('created_at')
                    ->dateTime()
                    ->sortable()
                    ->toggleable(isToggledHiddenByDefault: true),
            ])
            ->filters([
                SelectFilter::make('status')
                    ->options(OrderStatus::class),
            ])
            ->recordActions([
                EditAction::make(),
                DeleteAction::make(),
            ])
            ->toolbarActions([
                BulkActionGroup::make([
                    DeleteBulkAction::make(),
                ]),
            ])
            ->defaultSort('created_at', 'desc');
    }
}
```

### The v5 action method names — get these right

This is the single most common source of broken v5 code:

| Old (v3 / v4) | v5 method | Notes |
|---|---|---|
| `->actions([...])` | **`->recordActions([...])`** | `actions()` still exists but is `@deprecated` |
| `->bulkActions([...])` | **`->toolbarActions([...])`** | `bulkActions()` still exists but is `@deprecated` |
| `->pushActions()` | `->pushRecordActions()` | deprecated alias |
| `->actionsColumnLabel()` | `->recordActionsColumnLabel()` | deprecated alias |
| `->actionsAlignment()` | `->recordActionsAlignment()` | deprecated alias |
| — | `->groupedBulkActions([...])` | Convenience: wraps your array in a `BulkActionGroup` |
| — | `->headerActions([...])` | Table header actions |

`groupedBulkActions()` is **not** the rename target of `bulkActions()` — it is a separate convenience.
Prefer the explicit canonical form: `->toolbarActions([BulkActionGroup::make([...])])`.

> **Do not** "fix" a v5 codebase by renaming `bulkActions()` to `groupedBulkActions()`. The documented
> rename is to `toolbarActions()`.

### Columns

Columns live in `Filament\Tables\Columns\*`: `TextColumn`, `IconColumn`, `ImageColumn`, `ColorColumn`,
`ToggleColumn`, `CheckboxColumn`, `SelectColumn`, `TextInputColumn`, `ViewColumn`, `ColumnGroup`.

**Three column classes are deprecated — use the modern equivalents:**

| Deprecated class | Use instead |
|---|---|
| `BadgeColumn` | `TextColumn::make('status')->badge()` |
| `BooleanColumn` | `IconColumn::make('is_active')->boolean()` |
| `TagsColumn` | `TextColumn::make('tags')->badge()` |

**Column API methods are NOT universal — they are scoped per subclass.** Getting this wrong is a common
error:

| Method | Available on |
|---|---|
| `->badge()`, `->description()`, `->money()`, `->numeric()`, `->dateTime()`, `->since()`, `->limit()`, `->words()`, `->prefix()`, `->suffix()`, `->html()`, `->markdown()`, `->formatStateUsing()`, `->bulleted()`, `->listWithLineBreaks()`, `->limitList()` | **`TextColumn` only** |
| `->color()` | **`TextColumn` and `IconColumn` only** — `ColorColumn` has no `color()` |
| `->icon()`, `->iconPosition()`, `->icons()` | **`TextColumn` and `IconColumn` only** |
| `->copyable()`, `->copyMessage()`, `->copyMessageDuration()` | `TextColumn`, `ColorColumn` |
| `->wrap()` | `TextColumn`, `IconColumn`, `ImageColumn`, `ColorColumn` |
| `->alignment()`, `->alignStart()`, `->alignCenter()`, `->alignEnd()`, `->grow()`, `->width()`, `->state()`, `->getStateUsing()`, `->default()`, `->placeholder()`, `->extraAttributes()`, `->tooltip()`, `->summarize()`, `->hidden()`, `->visible()` | the base `Column`, so all columns |
| `->searchable()`, `->sortable()`, `->toggleable()`, `->url()`, `->openUrlInNewTab()`, `->action()`, `->visibleFrom()` | the base `Column`, so all columns |

Mark less important columns `->toggleable(isToggledHiddenByDefault: true)`.
Inline-editable column methods: `->rules()` (`TextInputColumn`), `->beforeStateUpdated()`,
`->afterStateUpdated()`.

### Filters

Filters live in `Filament\Tables\Filters\*`: `SelectFilter`, `TernaryFilter`, `TrashedFilter`, `Filter`,
`QueryBuilder`, `Indicator`. `MultiSelectFilter` is **deprecated** — use `SelectFilter::make()->multiple()`.

- A custom filter's schema method is **`->schema()`**, not `->form()` (`form()` is `@deprecated`, and
  `getFormSchema()` is deprecated in favour of `getSchemaComponents()`).
- A bare `Filter::make('name')` renders a **checkbox**; `->toggle()` swaps it for a toggle.
- `->query()` is wrapped in a scoped `where()` clause; `->baseQuery()` modifies the root query instead.
- Layout: `->filtersLayout(FiltersLayout::AboveContent)`, or pass `layout:` to `->filters()`.
- **Filters are deferred by default** — an "Apply" button is expected. `->deferFilters(false)` makes them
  live.
- Indicators: `->indicateUsing(fn (array $data) => ...)` returning strings or
  `Indicator::make(...)->removeField('from')->removable(false)`.
- **`Filament\Tables\Filters\QueryBuilder\Constraints\*` are empty backward-compatibility aliases.** The
  real constraint classes are `Filament\QueryBuilder\Constraints\*` (`TextConstraint`, `NumberConstraint`,
  `BooleanConstraint`, `SelectConstraint`, `DateConstraint`, `RelationshipConstraint`). The *filter* itself
  is still `Filament\Tables\Filters\QueryBuilder`.

### Summaries

Summarizers live in **`Filament\Tables\Columns\Summarizers\*`**: `Summarizer`, `Sum`, `Average`, `Count`,
`Range`, `Values`.

**There is no `Max` and no `Min` summarizer.** Attach with
`TextColumn::make('price')->summarize(Sum::make())`, and enable table summaries with
`->summaries()`. The **first column in a table may not use summarizers**.

### Inline-editable columns do not authorize

`ToggleColumn`, `TextInputColumn`, `SelectColumn`, and `CheckboxColumn` **do not check Model Policies**
when saving. They only respect `->disabled()`. Gate them yourself:

```php
TextInputColumn::make('price')
    ->disabled(fn (Product $record): bool => ! auth()->user()->can('update', $record)),
```

### Other table rules

- Searchable columns: `->searchable()` on the columns users actually search.
- Record links: `->recordUrl(...)` on the table, or per-row `->url(...)` on a column.
- Bulk actions that should be authorized per record: `->authorizeIndividualRecords('delete')`.
- Bulk-action helpers: `->chunkSelectedRecords(250)`, `->deselectRecordsAfterCompletion()`,
  `->fetchSelectedRecords(false)`.
- Row actions that need the selection: add `->selectable()` and `->accessSelectedRecords()`, then take a
  `Collection $selectedRecords` parameter.
- `Table::configureUsing()` in a service provider `boot()` sets framework-wide defaults. This is different
  from the userland `configure(Table $table)` scaffolding method, which is **not** a framework hook.
- Selection helpers: `->checkIfRecordIsSelectableUsing()`, `->maxSelectableRecords()`,
  `->selectCurrentPageOnly()`, `->disabledSelection()`.
- Column layout components are `Filament\Tables\Columns\Layout\*` (`Grid`, `Panel`, `Split`, `Stack`,
  `View`) — **not** `Filament\Schemas\Components\*`. Use `->contentGrid()` to lay records out in a grid.
- `->grow(false)` is required inside an aligned `Stack` for the stack's alignment to take effect.

### Custom (non-Eloquent) tables

```php
$table
    ->records(fn (): array => [
        1 => ['title' => 'First item', 'is_featured' => true],
        2 => ['title' => 'Second item', 'is_featured' => false],
    ])
    ->columns([
        TextColumn::make('title'),
        IconColumn::make('is_featured')->boolean(),
    ]);
```

- **Array keys are the record IDs** and must be unique and consistent.
- The `records()` closure can receive `$sortColumn`, `$sortDirection`, `$search`, `$columnSearches`,
  `$filters`, `$page`, `$recordsPerPage` — search and sort do nothing unless you implement them yourself.
- Return a `LengthAwarePaginator` to paginate; `->paginated(false)` disables pagination.
- Cross-page bulk selection needs `->resolveSelectedRecordsUsing()`.
- The table does not auto-refresh after an action on custom data — call `$this->resetTable()` yourself.
- Action callbacks receive `array $record`, not a `Model`.

### Custom Livewire table component

```php
use Filament\Actions\Concerns\InteractsWithActions;
use Filament\Actions\Contracts\HasActions;
use Filament\Schemas\Concerns\InteractsWithSchemas;
use Filament\Schemas\Concerns\RestrictsFileUploadsToSchemaComponents;
use Filament\Schemas\Contracts\HasSchemas;
use Filament\Tables\Concerns\InteractsWithTable;
use Filament\Tables\Contracts\HasTable;
use Filament\Tables\Table;
use Livewire\Component;

class ListProducts extends Component implements HasActions, HasSchemas, HasTable
{
    use InteractsWithActions;
    use InteractsWithSchemas;
    use InteractsWithTable;
    use RestrictsFileUploadsToSchemaComponents;

    public function table(Table $table): Table
    {
        return $table->query(Product::query())->columns([...]);
    }
}
```

Render with `{{ $this->table }}`. `InteractsWithForms` / `HasForms` still exist but are no longer what the
table docs recommend — use the `Schemas` pair.

---

## 7. Actions

All actions live in `Filament\Actions\*`. The complete v5.10.1 class inventory is:

`Action`, `ActionGroup`, `BulkAction`, `BulkActionGroup`, `CreateAction`, `EditAction`, `ViewAction`,
`DeleteAction`, `DeleteBulkAction`, `ForceDeleteAction`, `ForceDeleteBulkAction`, `RestoreAction`,
`RestoreBulkAction`, `ReplicateAction`, `AttachAction`, `DetachAction`, `DetachBulkAction`,
`AssociateAction`, `DissociateAction`, `DissociateBulkAction`, `ImportAction`, `ExportAction`,
`ExportBulkAction`, `SelectAction`, plus the deprecated shims `ButtonAction` and `IconButtonAction`.

**These names do NOT exist in v5 — never emit them:** `Filament\Actions\MountableAction`,
`Filament\Actions\StaticAction`, `Filament\Actions\ReplicateBulkAction`.
Also, `Filament\Forms\Components\Actions` does **not** exist — the "row of action buttons inside a schema"
component is `Filament\Schemas\Components\Actions`.

### Modal schema

```php
Action::make('sendInvoice')
    ->label(__('invoices.actions.send'))
    ->icon(Heroicon::Envelope)
    ->schema([
        TextInput::make('subject')->required(),
        Textarea::make('body')->required(),
    ])
    ->action(function (Invoice $record, array $data): void {
        // ...
    })
    ->successNotificationTitle(__('invoices.notifications.sent'));
```

**`->schema()` is the current method; `->form()` is deprecated.** The same applies to
`disabledSchema()` (`disabledForm()` / `disableForm()` deprecated), `getSchema()` (`getForm()`
deprecated), and `isSchemaDisabled()` (`isFormDisabled()` deprecated).
Note: `packages/actions/docs/02-modals.md` still shows `->disabledForm()` — the docs lag the source.

### Lifecycle hooks — the real six

```php
public function before(?Closure $callback): static
public function after(?Closure $callback): static
public function beforeFormFilled(?Closure $callback): static
public function afterFormFilled(?Closure $callback): static
public function beforeFormValidated(?Closure $callback): static
public function afterFormValidated(?Closure $callback): static
```

Real execution order: mount → `beforeFormFilled` → `mountUsing` (defaults to filling the schema) →
`afterFormFilled`. Submit → `beforeFormValidated` → validate → `afterFormValidated` → `before` → your
`->action()` closure → `after` → success/failure notification and redirect.

Data mutation method names:

```php
->mutateDataUsing(?Closure $callback)        // current
->mutateFormDataUsing(?Closure $callback)    // @deprecated — use mutateDataUsing()
->mutateRecordDataUsing(?Closure $callback)  // Edit/View/Replicate: before fill
->using(?Closure $callback)                  // replaces the whole default process
```

### Injectable closure parameters

For an action closure, these resolve **by name**: `$data`, `$record`, `$livewire`, `$model`,
`$arguments`, `$schema`, `$schemaComponent` (alias `$component`), `$schemaComponentState` (alias
`$state`), `$schemaState`, `$schemaGet` (alias `$get`), `$schemaSet` (alias `$set`),
`$schemaOperation` (aliases `$context`, `$operation`), `$table`, `$action`, `$parentAction`,
`$mountedActions`, `$records` / `$selectedRecords`, `$recordsQuery` / `$selectedRecordsQuery`.
`ReplicateAction` adds `$replica`.

Resolution order is name → type → `$this` → the Laravel container. A typo'd parameter name that happens
to match a container binding resolves silently; an unresolvable required parameter throws
`BindingResolutionException`.

`$form` and `$schema` are also passed as named injections from `->action()` and `->mountUsing()`, so
`function (Schema $form)` works there — but `$form` is not a general injectable everywhere.

`ActionGroup` has a **smaller** resolver: `$livewire`, `$model`, `$mountedActions`, `$record`, `$schema`,
`$schemaComponent`/`$component`, `$schemaOperation`/`$context`/`$operation`, `$schemaGet`/`$get`.
It exposes **no** `$set`, `$data`, or `$arguments`.

### ActionGroup

```php
ActionGroup::make([
    ViewAction::make(),
    EditAction::make(),
    DeleteAction::make(),
])
```

- Actions are a **constructor argument** to `make()`. `->actions([...])` exists but replaces the list.
- Divider between groups: a nested `ActionGroup::make([...])->dropdown(false)`.
- A group is hidden entirely when all children are hidden.
- Dropdown sizing is `->dropdownMaxHeight()` — **not** `maxHeight()`.
- `ActionGroup` is **not** authorized or disabled as a group (no `CanBeAuthorized` / `CanBeDisabled`).
- Group visibility is not authorization: hide + authorize the children.

### Per-action specifics

| Action | Key methods |
|---|---|
| `CreateAction` | `->model()`, `->schema()`, `->mutateDataUsing()`, `->using()`, `->slideOver()`, `->modalHeading()`, `->successRedirectUrl()`, `->relationship()`, `->createAnother()`, `->steps()` |
| `EditAction` | `->mutateRecordDataUsing()`, `->mutateDataUsing()`, `->using()`, `->successRedirectUrl()`, `->fillForm()` |
| `DeleteAction` | Set up already applies `requiresConfirmation()`, `defaultColor('danger')`, `keyBindings(['mod+d'])`, and hides itself for trashed records |
| `ReplicateAction` | `->excludeAttributes()`, `->mutateRecordDataUsing()`, `->beforeReplicaSaved()`, `->after()` |
| `ImportAction` | `->importer()`, `->job()`, `->chunkSize()`, `->csvDelimiter()`, `->maxRows()`, `->headerOffset()`, `->options()`, `->fileRules()`, `->authGuard()` |
| `ExportAction` | `->exporter()`, `->formats()`, `->columnMapping()`, `->columnMappingColumns()`, `->fileDisk()`, `->fileName()`, `->job()`, `->chunkSize()`, `->modifyQueryUsing()`, `->authGuard()` |

> **`->fillForm()` on `EditAction` replaces the default record hydration.** `EditAction::setUp()` installs
> its own `fillForm()` closure that reads from the record. Chaining your own overrides it.
>
> **`afterReplicaSaved()` is deprecated** — use `->after()`.
>
> **`ImportAction` has no `->file()` and no `->columnMapping()`.** The upload is an internal
> `FileUpload::make('file')`; `columnMapping()` is export-only. The "file" concepts on export are
> `fileDisk()` / `fileName()`.

### Imports & exports

- `Filament\Actions\Imports\Importer` and `Filament\Actions\Exports\Exporter` are **abstract classes, not
  interfaces**. Both require `getColumns()` and `getCompletedNotificationBody()`.
- Columns: `Filament\Actions\Imports\ImportColumn` and `Filament\Actions\Exports\ExportColumn`.
- `ImportColumn`: `->example()`, `->examples()`, `->requiredMapping()`, `->numeric()`, `->integer()`,
  `->boolean()`, `->rules()`, `->array()` / `->multiple()`, `->guess()`, `->castStateUsing()`,
  `->fillRecordUsing()`, `->relationship()`, `->sensitive()`.
- `ExportColumn`: `->enabledByDefault()`, `->formatStateUsing()`, `->limit()`, `->words()`, `->prefix()`,
  `->suffix()`, `->listAsJson()`, `->preventFormulaInjection()`, plus aggregates `->avg()`, `->counts()`,
  `->exists()`, `->max()`, `->min()`, `->sum()`.
- Import record resolution: `resolveRecord()` returns `firstOrNew`, a new model, or `null` to skip the row.
  Throw `RowImportFailedException` to fail a row.
- Importer row hooks are **protected methods you override**: `beforeValidate`, `afterValidate`, `beforeFill`,
  `afterFill`, `beforeSave`, `afterSave`, `beforeCreate`, `afterCreate`, `beforeUpdate`, `afterUpdate`.
- Both features need `php artisan vendor:publish --tag=filament-actions-migrations`.

---

## 8. Notifications

```php
use Filament\Notifications\Notification;

Notification::make()
    ->title(__('notifications.saved'))
    ->success()
    ->send();
```

- Factory: `Notification::make(?string $id = null)`.
- Status helpers: `success()`, `warning()`, `danger()`, `info()` — or `->status()`.
- Content: `->title()`, `->body()`, `->icon()`, `->iconColor()`, `->color()`, `->iconSize()`, `->date()`.
- Timing: `->duration()` (default 6000), `->seconds(float)`, `->persistent()`.
- Structure: `->actions([...])`, `->inline(bool)` — **note `inline()` accepts only `bool`, not a
  `Closure`**, unlike nearly every other setter.
- Delivery: `->send()` (session), `->broadcast($users)`, `->sendToDatabase($users)`.

### Two ways to notify from an action

1. **Declarative (preferred — what the built-ins do):** `->successNotificationTitle()`,
   `->successNotification()`, `->failureNotificationTitle()`, `->failureNotification()`. The framework fires
   them itself based on the action's `ActionStatus`. Pass `->successNotification(null)` to disable.
2. **Imperative:** call `Notification::make()->...->send()` inside the `->action()` closure.

There is **no `HasNotifications` / `Notifies` trait** for Livewire components in v5.10.1. The only
`HasNotifications` is a **panel** concern (`->databaseNotifications()`,
`->databaseNotificationsPolling()`). To render the notification centre, use `@livewire('notifications')`.

### Action authorization

```php
->authorize('update')            // Gate::check('update', [$record])
->authorizeAny(['update', 'delete'])
->authorizeIndividualRecords('delete')   // per-record checks for bulk actions
->authorizationMessage(...)
```

- **Custom actions have no automatic policy lookup.** Authorization defaults to `null`, which means
  *allowed for all users*. The framework's own comment states this explicitly.
- The action's record (or model) is auto-prepended to the gate arguments.
- Built-in actions get their authorization injected by the **host resource page** — `CreateAction` →
  `create`, `EditAction` → `update`, `DeleteAction` → `delete`, `DeleteBulkAction` → `deleteAny`,
  `ForceDeleteAction` → `forceDelete`, `RestoreAction` → `restore`, `ReplicateAction` → `replicate`,
  `ViewAction` → `view`, and the `*BulkAction` variants → `*Any`.
- Delete/restore bulk actions perform **no per-record** check unless you call
  `->authorizeIndividualRecords()`.
- In a bare Livewire component or custom page there is no mapping at all — `->authorize()` is on you.
- `->visible()` / `->hidden()` control rendering **and** are re-checked at call time; they also drive
  `disabled()`.
- `->requiresConfirmation()` does nothing when `->url()` is set instead of `->action()`.
- `getUrl()` returns `null` whenever a modal is configured — `url()` and `modal()` are mutually exclusive.

---

## 9. Enum Design System

Every status / type / category should be a PHP backed string enum implementing Filament's contracts:

| Contract | FQCN | Exact method signature |
|---|---|---|
| `HasLabel` | `Filament\Support\Contracts\HasLabel` | `getLabel(): string \| Htmlable \| null` |
| `HasColor` | `Filament\Support\Contracts\HasColor` | `getColor(): string \| array \| null` |
| `HasIcon` | `Filament\Support\Contracts\HasIcon` | `getIcon(): string \| BackedEnum \| Htmlable \| null` |
| `HasDescription` | `Filament\Support\Contracts\HasDescription` | `getDescription(): string \| Htmlable \| null` |

`getColor()` returns a **colour name string** (or an array / `null`) — never a `Color` case, because there
is no `Color` enum. `getIcon()` may return a `Heroicon` case, any `BackedEnum`, an `Htmlable`, or a string;
Filament normalises it at render time.

```php
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

    public function getLabel(): string | Htmlable | null
    {
        return __("enums.order-status.{$this->value}");
    }

    public function getColor(): string | array | null
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

- **Semantic colors only**: `success` (positive), `danger` (negative), `warning` (caution), `info` (new),
  `primary` (special), `gray` (neutral). Do not invent colors per enum case.
  The **only** built-in semantic names are `danger`, `gray`, `info`, `primary`, `success`, `warning` —
  `secondary` is **not** built in; register it yourself with `FilamentColor::register()`.
- **Naming**: PascalCase case names, snake_case backed values.
- Cast on the model: `protected function casts(): array { return ['status' => OrderStatus::class]; }`.

Filament resolves these automatically — a `TextColumn` with `->badge()` reads `getColor()` and `getIcon()`
straight off the enum state, so you do not need to repeat the `match` in every column.

### Icons

`Filament\Support\Icons\Heroicon` is a backed string enum with **647 cases** in exactly two families:

| Family | Case style | Value style | Example |
|---|---|---|---|
| Solid / default | `Heroicon::ShoppingBag` | `shopping-bag` | 323 cases |
| Outlined | `Heroicon::OutlinedShoppingBag` | `o-shopping-bag` | 324 cases |

- **There are no `Solid*`, `Mini*`, or `Micro*` cases.** For un-prefixed cases the size variant is chosen at
  render time by `getIconForSize()`: `heroicon-c-*` (xs/sm), `heroicon-m-*` (md — the default), and
  `heroicon-s-*` (lg/xl/2xl). `Outlined*` cases always render `heroicon-o-*`.
- Use the enum everywhere. String icon names (e.g. `'heroicon-o-user'`) still work and are **not**
  deprecated, but they are not type-safe and are not the idiom in v5.
- Icon **aliases** are the customisation seam: `FilamentIcon::resolve(SomeIconAlias::CONST) ?? Heroicon::OutlinedFoo`.
- A string containing `/` is treated as an image path and rendered as `<img src="...">`.

### Colors

`Filament\Support\Colors\Color` is a **plain class of `public const` arrays** — it is **not an enum**.
There is no `Color::Amber->value`. The constants hold 11-shade palettes (`50`–`950`) in **oklch**.

```php
use Filament\Support\Colors\Color;

$panel->colors(['primary' => Color::Amber]);   // array constant — correct
```

Color values flow as `string | array | Closure | null` (`'danger'`, `'gray'`, or `Color::Emerald`).
The 26 available palettes are `Slate, Gray, Zinc, Neutral, Stone, Mauve, Olive, Mist, Taupe, Red, Orange,
Amber, Yellow, Lime, Green, Emerald, Teal, Cyan, Sky, Blue, Indigo, Violet, Purple, Fuchsia, Pink, Rose`.

The **built-in semantic colour names** are exactly `danger`, `gray`, `info`, `primary`, `success`,
`warning` (mapped to `Red`, `Zinc`, `Blue`, `Amber`, `Green`, `Amber`). `secondary` is frequently cited
online but is **not** built in — add it with
`FilamentColor::register(['secondary' => Color::Indigo])`.

---

## 10. Widgets & Dashboards

- `Filament\Widgets\Widget` — base class. `protected string $view` has **no default**, so a custom widget
  must declare it or rendering fails. The Blade namespace is `filament-widgets::`.
- `StatsOverviewWidget` — implement `protected function getStats(): array` returning `Stat` objects.
  It is **not abstract** — forgetting `getStats()` silently yields an empty widget.
- `ChartWidget` — extend it and implement `abstract protected function getType(): string`. Chart type is a
  **bare string** (`'line'`, `'bar'`, `'pie'`, `'doughnut'`, `'polarArea'`, `'radar'`, `'scatter'`,
  `'bubble'`). **There are no `TYPE_*` constants.** The per-type classes (`LineChartWidget`,
  `BarChartWidget`, …) exist but are all **deprecated**.
- `TableWidget` — renders a table; pagination is forced to `PaginationMode::Simple`.
- **There is no `Panel::dashboard()`.** The dashboard is the page class `Filament\Pages\Dashboard`,
  registered through `->pages([Dashboard::class])` and customised by extending it. Only `->widgets()` and
  `->discoverWidgets()` exist on the panel.

### Property modifiers matter

| Property | Modifier | Default |
|---|---|---|
| `$sort` | **`static ?int`** | `null` (read as `-1` via `getSort()`) |
| `$isLazy` | **`static bool`** (from `CanBeLazy`) | `true` |
| `$pollingInterval` | instance `?string` (from `CanPoll`) | **`'5s'`** |
| `$heading`, `$description`, `$color`, `$maxHeight`, `$options`, `$isCollapsible` | instance | — |
| `$columnSpan` | instance `int \| string \| array` | `1` (`'full'` on `StatsOverviewWidget`) |

Writing `protected bool $isLazy = false;` (non-static) creates a shadow property and does nothing.
**Widgets poll every 5 seconds by default** — set `protected ?string $pollingInterval = null;` to disable.
There is **no `->poll()` on widgets**; `poll()` belongs to schema components and tables.

```php
use Filament\Widgets\StatsOverviewWidget;
use Filament\Widgets\StatsOverviewWidget\Stat;

class RevenueWidget extends StatsOverviewWidget
{
    protected static ?int $sort = 1;

    protected ?string $heading = 'Revenue';

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
        ];
    }
}
```

`Stat` is a **schema component**, not a value object: `Stat::make(string|Htmlable $label, $value)` then
`->description()`, `->descriptionIcon()`, `->descriptionColor()`, `->color()`, `->chart()`,
`->chartColor()`, `->icon()`, `->value()`, `->placeholder()`, `->url()`, `->key()`, `->extraAttributes()`.
It has **no** `size()`, `weight()`, or `badge()`.

### Chart filters and dashboard filters use different names

- Chart widgets: the `Filament\Widgets\ChartWidget\Concerns\HasFiltersSchema` concern —
  implement `filtersSchema(Schema $schema): Schema`. There is **no** `HasFilters` / `HasFiltersForm` in the
  widgets package.
- Dashboards: `Filament\Pages\Dashboard\Concerns\HasFiltersForm` (or `HasFiltersAction`), consumed by
  widgets through `Filament\Widgets\Concerns\InteractsWithPageFilters`.

### Sharing table state with widgets

| Purpose | Trait | Applied to |
|---|---|---|
| Page exposes table state | `Filament\Pages\Concerns\ExposesTableToWidgets` | the **page** class |
| Widget consumes table state | `Filament\Widgets\Concerns\InteractsWithPageTable` | the **widget** class |

`Filament\Widgets\ExposesTableToWidgets` (the v3 name) **does not exist in v5** and there is no alias.
`InteractsWithPageTable` requires `getTablePage(): string`.

> `$record` and `$parentRecord` on widgets use **Livewire's native restoration** — scopes and policies are
> not re-applied. Authorize them yourself. `Widget::make([...])` returns a `WidgetConfiguration`, valid
> only inside `widgets()` / `getHeaderWidgets()` / `getFooterWidgets()`.

### Deprecated widget APIs

`getVisibleWidgets()`, `getVisibleHeaderWidgets()`, `getVisibleFooterWidgets()`, and
`filterVisibleWidgets()` are all **deprecated** — use `getWidgetsSchemaComponents($widgets, $data = [])`,
which also filters `canView()` and injects page filters.

---

## 11. Infolists

- **There is no `Filament\Infolists\Infolist` class in v5.** An infolist **is** a
  `Filament\Schemas\Schema`.
- `public static function infolist(Schema $schema): Schema` on the resource; `ViewRecord` also declares an
  instance `infolist(Schema $schema): Schema`.
- Entries live in `Filament\Infolists\Components\*`: `TextEntry`, `IconEntry`, `ImageEntry`, `ColorEntry`,
  `CodeEntry`, `KeyValueEntry`, `RepeatableEntry`, `ViewEntry`.
  **There is no `BadgeEntry`** — use `TextEntry::make('status')->badge()`.
- Complex view pages: organise entries with `Tabs`, `Section`, `Grid` from `Filament\Schemas\Components\*`.

### Entry API gotchas

- **`prose()` and `bulleted()` / `limitList()` exist only on `TextEntry`.**
- **`size()` means three different things**: `TextEntry` → `TextSize`, `IconEntry` → `IconSize`,
  `ImageEntry` → a **pixel integer**.
- For entries, `state()` routes to `constantState()`, and `getStateUsing()` is simply an alias of `state()`.
- `CodeEntry` requires the app to install `phiki/phiki` — Filament does not require it.
- `Filament\Support\Concerns\HasBadge::badge()` is a **numeric** badge used by navigation items — a
  completely different thing from the boolean entry `badge()`.

Compact layouts: `->inlineLabel()`. Rich content: `->prose()->markdown()` or `->html()` (sanitized
automatically — see §13).

---

## 12. Multi-Tenancy

### Panel setup

```php
use App\Models\Team;

$panel->tenant(Team::class);
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
    public function getTenants(Panel $panel): Collection
    {
        return $this->teams;
    }

    public function canAccessTenant(Model $tenant): bool
    {
        return $this->teams()->whereKey($tenant)->exists();
    }
}
```

### Scoping rules

- Resources are **scoped automatically** (`$isScopedToTenant = true` by default). Opt out with
  `protected static bool $isScopedToTenant = false;`, or invert the default globally with
  `Resource::scopeToTenant(false);` in a service provider.
- Ownership relationship is customised with `$tenantOwnershipRelationshipName`; the display relationship
  with `$tenantRelationshipName`.

### CRITICAL: form fields are NOT tenant-scoped

Global scopes only apply to **Eloquent model queries**. Form `Select` fields built from `->options()` or
`->relationship()` run their own query and **will leak other tenants' records**. Scope them yourself:

```php
use Filament\Facades\Filament;

Select::make('team_id')
    ->relationship(
        name: 'team',
        titleAttribute: 'name',
        modifyQueryUsing: fn (Builder $query) => $query->whereBelongsTo(Filament::getTenant()),
    )
    ->searchable()
    ->preload();
```

### CRITICAL: `unique()` / `exists()` are not tenant-aware

Laravel's `unique` and `exists` rules do not use Eloquent models, so **model global scopes — including the
tenancy scope — are not applied**. Two tenants cannot hold the same value unless you switch to the
tenant-aware variants:

```php
TextInput::make('email')
    ->scopedUnique();   // instead of ->unique()

TextInput::make('code')
    ->scopedExists();   // instead of ->exists()
```

### Other tenancy facts

- Models are associated with the current tenant automatically on `creating` / `created` — but only for
  models that have a tenant-aware resource in a tenancy-enabled panel.
- Queries made **before** the tenant is resolved (early middleware, service providers) are not scoped.
- Models **without** a resource need explicit scoping via tenant middleware
  (`->tenantMiddleware([ApplyTenantScopes::class], isPersistent: true)`).
- `->tenantDomain('{tenant:slug}.example.com')` for domain-based tenancy;
  `->tenant(Team::class, slugAttribute: 'slug')` for path-based.
- Never disable all global scopes blindly — use `withoutGlobalScopes([...])` with an explicit list, or
  `withoutGlobalScope(filament()->getTenancyScopeName())`.

---

## 13. Authorization & Security

### What Filament does for you

Filament automatically checks **Laravel Model Policies** for the standard CRUD operations on resources:
`viewAny`, `view`, `create`, `update`, `delete`, `deleteAny`, `restore`, `restoreAny`, `forceDelete`,
`forceDeleteAny`, `replicate`, `reorder`.

Authorization is **re-checked on every Livewire request**, not just on mount:

- resource pages re-check resource + parent resource + current record access;
- custom pages check `canAccess()`;
- relation managers check `canViewForRecord()`;
- widgets check `canView()`;
- panel access checks `canAccessPanel()` on every request.

### What you MUST do yourself

Filament's automatic authorization **only** covers built-in resource CRUD. You are responsible for:

- **Custom actions** — `visible()`, `hidden()`, and `authorize()` are available but **not applied
  automatically**.
- **Custom pages, custom Livewire components, and API endpoints.**
- **Inline-editable columns** (see §6).
- **Query scoping.** Resource queries return **all** records by default. Apply `modifyQueryUsing()` or
  override `getEloquentQuery()`.
- **Records passed into relation managers, widgets, and modal table selects.** `$ownerRecord`, widget
  `$record`/`$parentRecord`, and modal-table-select `$record` use **Livewire's native restoration**, which
  does **not** apply global scopes. Enforce tenant/ownership restrictions in `canViewForRecord()` /
  `canView()`, or re-query and authorize before use.
- **Your own model properties.** Store a scalar key and re-query + authorize:

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

### Panel access

```php
use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;

class User extends Authenticatable implements FilamentUser
{
    public function canAccessPanel(Panel $panel): bool
    {
        return match ($panel->getId()) {
            'admin' => $this->is_admin,
            default => true,
        };
    }
}
```

- When `APP_ENV` is **not** `local`, Filament **requires** the `FilamentUser` contract. Without it,
  **no one can sign in** in production.
- Multi-panel apps must branch on `$panel` inside `canAccessPanel()`.
- Enable strict mode with `->strictAuthorization()` so a missing policy or policy method throws instead of
  granting access.

### Validating untrusted values

- **URLs**: `Str::sanitizeUrl($state)` returns the URL only when schemeless or `http`/`https`, otherwise
  `null`. Pass a second argument to replace the allowlist (include `http`/`https` if you still want them).
- **CSS colors**: `Str::sanitizeCssColor($value)` for `ColorColumn` / `ColorEntry` custom output.
- **`extraAttributes()` and friends render without escaping by design.** Never pass user-controlled data.
- **HTML**: `->html()` / `->markdown()` are sanitized with Symfony's HtmlSanitizer.
  In your own Blade views you must sanitize yourself:

```blade
{!! str($record->content)->sanitizeHtml() !!}
{!! str($record->content)->markdown()->sanitizeHtml() !!}
```

### Model attribute exposure

Filament exposes all non-`$hidden` model attributes to JavaScript via Livewire's model binding. Add
sensitive columns to the model's `$hidden`, or strip them in the Edit/View page's
`mutateFormDataBeforeFill()`.

### File uploads

Every Livewire component using `InteractsWithSchemas` exposes `_startUpload` / `_finishUpload` for **any**
property name. On components whose schema has no upload field — or that are publicly reachable — add:

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

### Lifecycle ordering

Livewire restores public properties **before** authorization runs. A page's `boot()`, trait initializers,
and early `mount()`/`hydrate()` code, plus a widget's or relation manager's `boot()`, can run before the
authorization check. **Never perform writes, events, audit logging, or external calls in those hooks
without authorizing first.**

Do not enable Livewire lazy loading on a Filament page that relies on page authorization hooks — pages
have no authorization check while a lazy component waits to mount (widgets and relation managers do).

### CSP

Filament adds a `nonce` to scripts it renders once you call `Vite::useCspNonce($nonce)` **per request**
(not in a service provider's `boot()`, because Octane does not re-boot providers). Register CSP middleware
on the **panel** via `->middleware([...])` — panels build their own stack and never resolve Laravel's `web`
group. Scripts you render yourself (render hooks, `scripts` stack, `RawJs`, `Js::make()->html()`) need the
nonce added manually via `Vite::cspNonce()`.

---

## 14. Panel Configuration

Canonical provider shape:

```php
namespace App\Providers\Filament;

use Filament\Http\Middleware\Authenticate;
use Filament\Http\Middleware\AuthenticateSession;
use Filament\Http\Middleware\DisableBladeIconComponents;
use Filament\Http\Middleware\DispatchServingFilamentEvent;
use Filament\Pages\Dashboard;
use Filament\Panel;
use Filament\PanelProvider;
use Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse;
use Illuminate\Cookie\Middleware\EncryptCookies;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;
use Illuminate\Routing\Middleware\SubstituteBindings;
use Illuminate\Session\Middleware\AuthenticateSession as LaravelAuthenticateSession;
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
            ->login()
            ->colors(['primary' => Color::Amber])
            ->discoverResources(in: app_path('Filament/Resources'), for: 'App\\Filament\\Resources')
            ->discoverPages(in: app_path('Filament/Pages'), for: 'App\\Filament\\Pages')
            ->pages([Dashboard::class])
            ->discoverWidgets(in: app_path('Filament/Widgets'), for: 'App\\Filament\\Widgets')
            ->widgets([])
            ->middleware([...])
            ->authMiddleware([Authenticate::class]);
    }
}
```

Useful, verified panel methods: `default()`, `id()`, `path()`, `domain()`, `login()`, `registration()`,
`passwordReset()`, `emailVerification()`, `profile()`, `spa()`, `databaseTransactions()`,
`unsavedChangesAlerts()`, `sidebarCollapsibleOnDesktop()`, `sidebarFullyCollapsibleOnDesktop()`,
`collapsibleNavigationGroups()`, `brandName()`, `brandLogo()`, `favicon()`, `font()`, `colors()`,
`darkMode()`, `viteTheme()`, `middleware()`, `authMiddleware()`, `resources()`, `pages()`, `widgets()`,
`discoverResources()`, `discoverPages()`, `discoverWidgets()`, `plugins()`, `navigationGroups()`,
`renderHook()`, `bootUsing()`, `broadcasting()`, `strictAuthorization()`, `errorNotifications()`.

### Multi-panel

- One `PanelProvider` per panel, each with its own auth, resources, pages, widgets, middleware and theme.
- `->default()` marks one panel as the fallback.
- Branch on `$panel->getId()` in `canAccessPanel()`.
- In tests, switch panels with `Filament::setCurrentPanel('admin')`.

---

## 15. Testing

- Filament pages and widgets are Livewire components: test them with `livewire(PageClass::class)`.
- Test macros are registered on Livewire's `Testable` by each package's service provider — **nothing to
  import**.
- There is **no `packages/panels/src/Testing`** in v5.10.1; helpers live in `actions`, `tables`, `forms`,
  `schemas`, `infolists`, and `notifications`.

### Skeleton

```php
use App\Filament\Resources\Customers\Pages\CreateCustomer;
use App\Models\Customer;
use App\Models\User;
use function Pest\Livewire\livewire;

beforeEach(function () {
    $this->actingAs(User::factory()->create());
});

it('creates a customer', function () {
    livewire(CreateCustomer::class)
        ->fillForm([
            'name' => 'Nuno Maduro',
            'email' => 'nuno@example.com',
        ])
        ->call('create')
        ->assertHasNoFormErrors();

    expect(Customer::query()->where('email', 'nuno@example.com')->exists())->toBeTrue();
});
```

### The v5 pattern: ONE generic action API

v5 replaced the whole family of context-specific action helpers (`callTableAction`, `callTableBulkAction`,
`callFormComponentAction`, `assertTableActionExists`, …) with **one generic API** driven by
`Filament\Actions\Testing\TestAction`. Those context-specific names still resolve but are `@deprecated`.

```php
use Filament\Actions\Testing\TestAction;

// Row action
livewire(ListInvoices::class)
    ->callAction(TestAction::make('send')->table($invoice));

// Header action (no record)
livewire(ListInvoices::class)
    ->callAction(TestAction::make('create')->table());

// Bulk action
livewire(ListInvoices::class)
    ->selectTableRecords($invoices->pluck('id')->toArray())
    ->callAction(TestAction::make('send')->table()->bulk());

// Action inside a schema component
livewire(EditInvoice::class, ['record' => $invoice->id])
    ->callAction(TestAction::make('send')->schemaComponent('customer_id'));

// Assertions use the same TestAction descriptor
livewire(ListInvoices::class)
    ->assertActionVisible(TestAction::make('send')->table($invoice))
    ->assertActionExists(TestAction::make('send')->table($invoice));
```

`TestAction::make(string $name)` also supports `->arguments([...])` and
`->schemaComponent(string|bool|null $component = true, ?string $schema = null)`.
Nested actions are addressed with an **ordered array** of `TestAction`s.

> `TestAction::make()` takes the action **name**, not the class — unless the class carries
> `#[ActionName('send')]` (`Filament\Actions\ActionName`) or defines `getDefaultName()`. Built-ins like
> `CreateAction`, `EditAction`, and `DeleteAction` work when passed directly.

### Verified assertion names

**It is `assertSchemaStateSet()`, not `assertFormSet()`.** `assertFormSet()`, `assertFormExists()`, and the
`assertFormField*` family are all real but **deprecated**.

**Records**: `assertCanSeeTableRecords()`, `assertCanNotSeeTableRecords()`, `assertCountTableRecords()`,
`loadTable()`

**Search / sort / filter**: `searchTable()`, `searchTableColumns()`, `sortTable()`, `filterTable()`,
`resetTableFilters()`, `removeTableFilter()`, `removeTableFilters()`, `assertTableFilterExists()`,
`assertTableFilterVisible()`, `assertTableFilterHidden()`

**Columns**: `assertTableColumnExists()`, `assertTableColumnDoesNotExist()`, `assertTableColumnVisible()`,
`assertTableColumnHidden()`, `assertTableColumnStateSet()`, `assertTableColumnFormattedStateSet()`,
`assertTableColumnHasDescription()`, `assertCanRenderTableColumn()`, `toggleAllTableColumns()`

**Actions (generic)**: `mountAction()`, `callAction()`, `callMountedAction()`, `unmountAction()`,
`assertActionExists()`, `assertActionDoesNotExist()`, `assertActionVisible()`, `assertActionHidden()`,
`assertActionEnabled()`, `assertActionDisabled()`, `assertActionHasLabel()`, `assertActionHasIcon()`,
`assertActionMounted()`, `assertActionNotMounted()`, `assertActionHalted()`, `assertActionListInOrder()`,
`assertMountedActionModalSee()`, `assertMountedActionModalDontSee()`

**Selection**: `selectTableRecords()` — accepts **Models or keys**

**Schema / form**: `fillForm()`, `assertSchemaExists()`, `assertSchemaStateSet()`,
`assertSchemaComponentExists()`, `assertSchemaComponentDoesNotExist()`, `assertSchemaComponentVisible()`,
`assertSchemaComponentHidden()`, `assertSchemaComponentStateSet()`, `assertHasNoFormErrors()`,
`assertHasFormErrors()`, `assertFormFieldExists()`, `assertFormFieldDisabled()`, `assertFormFieldReadOnly()`

**Wizards**: `assertWizardStepExists()`, `assertWizardCurrentStep()`, `goToWizardStep()`,
`goToNextWizardStep()`, `goToPreviousWizardStep()`

**Summaries**: `assertTableColumnSummarySet()`, `assertTableColumnSummaryNotSet()`,
`assertTableColumnSummarizerExists()`

**Notifications**: `assertNotified()`, `assertNotNotified()` (also available as
`Notification::assertNotified()` and `Filament\Notifications\Testing\assertNotified()`)

**Imports / exports**: `ProductImporter::test()->import([...])->assertImported()->getRecord()`;
`ProductExporter::test(columnMap: [...])->export($record)`;
`ImportAction::fake()`, `ExportAction::fake()` with `assertDispatched()` / `assertDispatchedTimes()`

### Test names that DO NOT exist — never emit them

`sortTableBy`, `deselectTableRecords`, `selectTableRecord` (singular), `assertTableActionCalled`,
`assertTableBulkActionCalled`, `assertTableHeaderActionsExist`,
`assertTableEmptyStateHeading`, `assertTableRecordsReordered`,
`assertNotImported`, `assertExported`, `assertExportedFile`.

Also: `assertTableColumnStateSet()` and `assertTableColumnFormattedStateSet()` **require** the `$record`
third argument — there is no default.

### Renamed test APIs

| Deprecated | v5 replacement |
|---|---|
| `assertFormSet()` | **`assertSchemaStateSet()`** |
| `assertFormExists()` | **`assertSchemaExists()`** |
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

### Multi-tenant & multi-panel tests

```php
use Filament\Facades\Filament;

Filament::setCurrentPanel('admin');   // non-default panel — required when not making a real request
Filament::setTenant($team);           // current tenant
Filament::bootCurrentPanel();         // apply tenant scopes + model event listeners
```

Panel middleware does not run when you test a Livewire component directly, so setting the current panel
manually is mandatory for non-default panels. `Filament::bootCurrentPanel()` is documented and idempotent
but is not exercised in Filament's own test suite; `setCurrentPanel()` and `setTenant()` are.

### Testing caveats the docs call out

- `assertCanSeeTableRecords()` only sees the **first page** — use `call('gotoPage', 2)`.
- `deferLoading()` requires `loadTable()` before asserting records.
- Expected sort order must be built with a **query** `orderBy()`, not a collection `sortBy()` — database
  and PHP sorting differ.
- Only default-enabled toggleable columns render; `toggleAllTableColumns()` enables them all.
- Repeaters and builders generate UUIDs per item — wrap with `Repeater::fake()` / `Builder::fake()` and
  call the returned undo closure afterwards.
- `Resource`, schema component, and `Action` classes are **not** Livewire components — test them with
  direct method calls and expectations, not `livewire()`.
- `assertNotified('Title')` matches the **title only**; passing a `Notification` object compares the whole
  payload except `id`, so it must be spelled out in full.
- `ImportAction::fake()` / `ExportAction::fake()` reset recorded dispatches — call `fake()` once, before
  the actions you want to assert together.

Test authorization **through the request lifecycle**, not just on mount: mount the component, change the
model's tenant/ownership/policy result, then send another request and assert it is rejected.

---

## 16. Plugins

Implement `Filament\Contracts\Plugin` with `getId()`, `register(Panel $panel)`, and `boot(Panel $panel)`.
Register with `->plugin(MyPlugin::make())` on the panel, and configure with static `make()` +
`->configureUsing()` conventions.

---

## 17. Performance & Deployment

### Production deploy script

```bash
php artisan filament:optimize        # caches components + Blade icons
php artisan optimize                 # Laravel config + route cache
php artisan icons:cache              # (included in filament:optimize)
```

- `filament:optimize` is shorthand for `filament:cache-components` + `icons:cache`.
- Clear with `php artisan filament:optimize-clear` or `php artisan filament:clear-cached-components`.

> **Never run `filament:cache-components` in local development.** It prevents new resources, pages,
> widgets, and relation managers from being discovered until the cache is cleared.

### Laravel `composer.json`

Keep `php artisan filament:upgrade` in the `post-autoload-dump` script so assets stay up to date.

### Panel config for robustness

- `->spa()` for SPA navigation.
- `->unsavedChangesAlerts()` to protect unsaved form data.
- `->databaseTransactions()` so each action/page runs in a transaction.

### Performance

- Widgets are lazy-loadable; avoid heavy work in `mount()`.
- Use `->poll()` / `->pollingInterval()` deliberately, and disable polling where it is not needed.
- Cache Blade icons (`php artisan icons:cache`) and use OPcache in production.
- On Windows, excluding the project folder from real-time antivirus scanning is a documented, large win.

---

## 18. Gotchas That Break Code

These are verified traps — each one silently produces wrong behavior rather than an error.

1. **File visibility defaults to `private`.** Always pass `->visibility('public')` when public URLs are
   needed on `FileUpload`.
2. **Nothing is full-width by default.** `Grid`, `Section`, `Fieldset`, and `Repeater` do not span all
   columns — use `->columnSpanFull()`.
3. **`BelongsToSelect` does not exist.** Use `Select::make('author_id')->relationship('author', 'name')`.
4. **`Repeater` uses `->schema()`, not `->fields()`.**
5. **Do not add `->dehydrated(false)` to a field you need saved** — it strips the value from form state
   before the save handler or `->action()` runs. Use it only for helper/UI-only fields. The v5-documented
   alternative is `->saved(false)`, which is clearer about intent.
6. **`Repeater::columns()` and `Repeater::grid()` are different scopes** — `columns()` lays out the fields
   *inside each item*; `grid()` lays out the *items themselves*.
7. **`Wizard::submitAction()` / `cancelAction()` take rendered HTML, not an `Action`.**
   Only `nextAction()` / `previousAction()` accept action-modifier closures.
8. **`hiddenOn()` / `visibleOn()` overwrite `hidden()` / `visible()`** and vice versa — they call through to
   each other. They also accept a Livewire page class string, not just `Operation` cases.
9. **Preserve union types when overriding properties.** `$navigationIcon` is
   `string | BackedEnum | null` (not `?string`); `$navigationGroup` is `string | UnitEnum | null`. Also
   `$view` is a non-static `protected string` on `Page` and `Widget`.
10. **`$shouldRegisterNavigation = false` only hides the sidebar link** — it does **not** restrict access by
    direct URL.
11. **`Resource::form()` / `table()` / `infolist()` are `static`, but a RelationManager's are instance
    methods.**
12. **`CreateRecord::getRedirectUrl()` returns `string` (always redirects); `EditRecord::getRedirectUrl()`
    returns `?string` (default `null` = no redirect).** Do not assume one shape for both.
13. **`ViewRecord` only has `beforeFill` / `afterFill` hooks, and they do not run when an infolist is
    defined.** Create/Edit have the full six-hook set.
14. **`getHeaderActions()` and `getFormActions()` are `protected`.** There is no `getFooterActions()`.
15. **`getRecordSubNavigation()` does not affect `ListRecords`** — the List page returns only cluster
    sub-navigation.
16. **Global search silently returns nothing if the resource has no View or Edit page.**
17. **`getGloballySearchableAttributes()` defaults to `[getRecordTitleAttribute()]`.** Dot notation into
    relationships works (it switches to `whereHas`).
18. **Repeater items get UUID keys.** Numeric-key assumptions in tests fail without `Repeater::fake()`.
19. **`RawJs` and `extra*Attributes()` are not escaped** — never pass user input.
20. **`SubNavigationPosition` lives at `Filament\Pages\Enums\SubNavigationPosition`**, not under
    `Filament\Navigation`.
21. **`packages/panels/src/Enums/` is namespaced `Filament\Enums`** — because `Filament\` maps to
    `packages/panels/src`.
22. **A `Select`'s `modifyQueryUsing()` is a named argument of `relationship()`, not a chainable method.**
23. **`FileUpload`'s `->deletable(false)`** hides the remove button. There is no `deletableFileUploads()`.

---

## 19. Reconciling With Filament's Own Skill

Filament ships its own agent skill at
`packages/panels/resources/boost/skills/filament-development/SKILL.md` (installed with
`php artisan boost:install`). It and this document agree. Three points are worth making explicit because
they are widely gotten wrong:

- **Actions namespace.** Filament's skill says: *"Actions (`DeleteAction`, `CreateAction`, etc.):
  `Filament\Actions\`. Never use `Filament\Tables\Actions\`, `Filament\Forms\Actions\`, or any other
  sub-namespace for actions."* Precisely: the namespace `Filament\Tables\Actions` still exists in v5.10.1
  but contains exactly one file — `HeaderActionsPosition` (an enum). It holds **no action classes**.
- **Icons.** The official skill uses `Heroicon::PencilSquare` (the un-prefixed family). Both families are
  valid; use `Outlined*` for navigation icons and softer UI, un-prefixed for solid emphasis.
- **Property types.** The official skill's "Common mistakes" list on union types is the same list as
  §18.9 above, and both come from the same source traits.

---

## 20. Strict Rules

Do not ship code that breaks any of these:

1. **Never import actions from `Filament\Tables\Actions\*`** — use `Filament\Actions\*`.
2. **Never use `->form()` on actions or filters** — use `->schema()`.
3. **Never use `->actions()` / `->bulkActions()` on a table** — use `->recordActions()` /
   `->toolbarActions()`.
4. **Never use `Filament\Forms\Get` / `Filament\Forms\Set`** — use
   `Filament\Schemas\Components\Utilities\Get` / `Set`.
5. **Always set `$recordTitleAttribute`** on every resource that has a View or Edit page.
6. **Icons only through the `Heroicon` enum**, never string icon names.
7. **Operation checks only through the `Operation` enum**, never string comparisons.
8. **Use `$schema->components([...])` at the top level.** `$schema->schema([...])` is a valid alias on the
   `Schema` class, but `->schema()` means "nest children" on every component — do not blur the two.
9. **Searchable + preloaded** for relationship selects.
10. **`scopedUnique()` / `scopedExists()`** instead of `unique()` / `exists()` in any multi-tenant app.
11. **Never rely on tenant scoping for form field options** — scope them with `modifyQueryUsing()`.
12. **Implement `FilamentUser` with `canAccessPanel()`** before deploying (production requires it).
13. **Authorize every custom action** — `authorize()` is not automatic.
14. **Never pass user input to `extra*Attributes()`** or `url()` without `Str::sanitizeUrl()`.
15. **Never publish Filament's Blade views** — customise with `fi-` CSS hooks.
16. **Never hardcode user-facing strings** — use language files.
17. **Never use `final` or `readonly` on classes you might need to extend**, and never abbreviate variable
    names (`$record`, not `$rec`).
18. **Never add `declare(strict_types=1)`** — the framework does not use it.
19. **Never run `filament:cache-components` locally.**
20. **Never make inline-editable columns without a `disabled()` authorization check.**

---

## Summary Checklist

**Structure & naming**
- [ ] Resource delegates to `Schemas/*Form.php` (singular) and `Tables/*Table.php` (plural) classes
- [ ] `$recordTitleAttribute` set on every resource
- [ ] Resource properties use the exact v5 types (`string | BackedEnum | null $navigationIcon`)
- [ ] Resource `form()`/`table()`/`infolist()` are `static`; relation manager methods are not

**Namespaces & enums**
- [ ] All actions imported from `Filament\Actions\*`
- [ ] `Get` / `Set` imported from `Filament\Schemas\Components\Utilities`
- [ ] Icons use the `Heroicon` enum (`Outlined*` for navigation)
- [ ] `Operation::Create` / `Operation::Edit` used instead of string comparisons
- [ ] Enums implement `HasLabel`, `HasColor`, `HasIcon` with semantic colors
- [ ] Colors passed as strings or `Color::*` array constants — not enum cases

**Schemas & tables**
- [ ] `$schema->components([...])` at the top level of every schema
- [ ] Action and filter modals use `->schema()`, not `->form()`
- [ ] Table uses `recordActions()`, `toolbarActions()`, `headerActions()`
- [ ] Relationship selects are `searchable()` and `preload()`
- [ ] `->columnSpanFull()` used where full width is intended
- [ ] `->visibility('public')` set on uploads that need public URLs
- [ ] No `->dehydrated(false)` on fields that must be saved

**Security**
- [ ] Multi-tenant selects scoped with `modifyQueryUsing()` + `Filament::getTenant()`
- [ ] `scopedUnique()` / `scopedExists()` used in multi-tenant forms
- [ ] Inline-editable columns gated with `disabled()` + a policy check
- [ ] `FilamentUser` implemented with a panel-aware `canAccessPanel()`
- [ ] Policies exist for every resource model; `->strictAuthorization()` considered
- [ ] Custom actions call `authorize()`; bulk actions use `authorizeIndividualRecords()`
- [ ] `Str::sanitizeUrl()` used for any user-supplied URL
- [ ] Custom Blade rich-text output wrapped in `->sanitizeHtml()`
- [ ] Records passed to widgets/relation managers re-queried and authorized
- [ ] Sensitive model attributes hidden via `$hidden` or `mutateFormDataBeforeFill()`

**Testing & deployment**
- [ ] Tests use `livewire()` with `Filament::setTenant()` / `Filament::setCurrentPanel()`
- [ ] Tests use `assertSchemaStateSet()` and the generic `TestAction` / `callAction()` API
- [ ] No hardcoded user-facing text — everything through language files
- [ ] No published Blade views — `fi-` CSS hooks only
- [ ] `filament:optimize` in the production deploy script, never locally
