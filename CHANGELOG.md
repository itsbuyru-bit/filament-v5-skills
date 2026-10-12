# Changelog

All notable changes to this repository are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Because this repository contains a skill document rather than code, the version reflects the state of the
guidance: a **major** bump means existing advice changed or was removed, a **minor** bump means new guidance
was added, and a **patch** bump means corrections that do not change the substance.

The Filament version the guidance was verified against is stated explicitly in every release, since that is
what determines whether the advice is still current.

## [Unreleased]

Nothing yet.

## [1.0.0] — 2026-10-12

Verified against **Filament v5.10.1**.

### Added

- `SKILL.md` — Filament v5 coding standards: `name` and `description` front matter, 20 structured sections,
  a gotchas section, 20 strict rules, and a 32-item summary checklist.
- `REFERENCE.md` — copy-ready code examples, the complete v3/v4 → v5 rename map, and a list of deprecated
  names that still compile.
- `README.md` — repository overview, platform requirements, usage, and the verification method.
- `README.ru.md` — Russian translation of the overview, with a language switcher in both READMEs.
- `LICENSE.md` — MIT, with a note that no Filament source is redistributed.
- `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and `SECURITY.md`.
- GitHub issue and pull request templates.

### Notes

- The skill files are kept in English deliberately; the rationale is documented in both READMEs and in
  `CONTRIBUTING.md`.
- API claims were machine-checked against the v5.10.1 checkout: every referenced class, enum, and interface
  name; roughly 250 method names and artisan commands; and every negative claim (the ~24 names the skill
  asserts do not exist in v5).

[Unreleased]: https://github.com/itsbuyru-bit/filament-v5-skills/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/itsbuyru-bit/filament-v5-skills/releases/tag/v1.0.0
