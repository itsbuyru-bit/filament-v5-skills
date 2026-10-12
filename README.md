<div align="right">

**English** · [Русский](README.ru.md)

</div>

# Filament v5 Skills

[![GitHub stars](https://img.shields.io/github/stars/itsbuyru-bit/filament-v5-skills?style=flat-square)](https://github.com/itsbuyru-bit/filament-v5-skills/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

An agent skill documenting coding standards, architecture patterns, and best practices for building with
[Filament](https://github.com/filamentphp/filament) **v5**.

Every rule in this skill was verified against the `filamentphp/filament` source at tag **`v5.10.1`** — by
reading the actual package source, not by relying on documentation prose or recall. Where the framework's
own documentation lags the source, the source wins, and the discrepancy is called out.

## What's here

| File | Purpose |
|---|---|
| [`SKILL.md`](.github/skills/filament-v5-development/SKILL.md) | The skill: `name` + `description` front matter, 20 structured sections, a gotchas section, 20 strict rules, and a 32-item summary checklist. |
| [`REFERENCE.md`](.github/skills/filament-v5-development/REFERENCE.md) | Companion reference: full copy-ready code examples, the complete v3/v4 → v5 rename map, and a list of deprecated names that still compile. |

> Both skill files are written in English — a deliberate choice, explained under
> [A note on language](#a-note-on-language) below.

## Platform requirements (v5)

| Requirement | Constraint |
|---|---|
| PHP | `^8.2` |
| `ext-intl` | required |
| Laravel | `^11.28 \| ^12.0 \| ^13.0` |
| Livewire | `^4.4.7` |
| Tailwind CSS | v4.0+ (docs recommend v4.1+) |

## What this skill covers

Resources, pages, relation managers, clusters, schemas, forms, tables, columns, filters, actions, bulk
actions, notifications, infolists, widgets, dashboards, panels, multi-tenancy, authorization, security,
testing, plugins, custom themes, and imports/exports.

## Using it

**With an agent that reads repository skills** — point it at
`.github/skills/filament-v5-development/SKILL.md`.

**Anywhere else** — copy the skill directory into wherever your tool expects skills, or paste `SKILL.md`
into context directly. The two files are self-contained and only reference each other.

**Filament also ships its own agent skill** at
`packages/panels/resources/boost/skills/filament-development/SKILL.md`, installed via
`php artisan boost:install` → **Agent Skills** → `filament/filament (skills)`. This skill is a deeper
companion to it; §19 of `SKILL.md` reconciles the two where they overlap.

## A note on v5 migrations

Filament v5 kept most v3/v4 method names working as `@deprecated` aliases that silently proxy to the new
method. Static analysis and linters will **not** flag them, which makes them the most likely way to ship
outdated code into a v5 app. Two examples that trip people up:

- `Table::bulkActions()` is deprecated in favour of **`toolbarActions()`** — *not* `groupedBulkActions()`,
  which is a separate, current convenience.
- The namespace `Filament\Tables\Actions` still exists but contains a single enum
  (`HeaderActionsPosition`) and **no action classes**. All actions live in `Filament\Actions\*`.

See §16 of `REFERENCE.md` for the full list.

## Verification

The API surface referenced here was machine-checked against the v5.10.1 checkout:

- every referenced class, enum, and interface name
- ~250 referenced method names and artisan commands
- every **negative** claim — the ~24 names the skill asserts do *not* exist in v5

## A note on language

Only this README is translated (see [Русский](README.ru.md)) — it carries the repository overview and the
platform requirements.

`SKILL.md` and `REFERENCE.md` are kept in English on purpose:

- they are a technical reference of roughly 160 KB in which nearly every line is a method name, FQCN, or
  code example;
- a translation of that volume drifts from the original on every update, and drift in an API reference is
  worse than having no translation at all;
- agents read `SKILL.md` as a machine instruction, where English is the lowest-risk option.

If a translated skill is needed, treat it as a separate task: a complete translation carrying a
"current as of" marker, not a partial one.

## License

MIT. Filament itself is MIT-licensed and is the work of the Filament team; this repository only documents
its public API.
