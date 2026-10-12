# Contributing

Thanks for considering a contribution. This repository is a documentation project, so the thing that matters
most is **accuracy**: a confidently wrong statement about an API is worse than no statement at all.

## The one rule that overrides everything

**Every API claim must be verified against the actual Filament source, not against documentation prose and
not from memory.**

Filament's own documentation lags its source in several places, and its `@deprecated` aliases keep old
method names working silently. That combination is exactly how incorrect guidance gets written. Before you
assert that a method exists, that a class lives in a namespace, or that something was removed:

1. Clone the version you are targeting and check it out:

   ```bash
   git clone --filter=blob:none --no-checkout https://github.com/filamentphp/filament.git
   cd filament
   git fetch --depth 1 origin "refs/tags/v5.10.1:refs/tags/v5.10.1"
   git sparse-checkout init --cone
   git sparse-checkout set packages docs tests
   git checkout v5.10.1
   ```

2. Read the source, not the docs. Grep for the declaration:

   ```bash
   grep -rn "public function toolbarActions(" packages/tables/src
   ```

3. Cite the file and line in your reasoning. If you cannot point at a file, it is a guess.

Negative claims matter just as much as positive ones. If you write "this does not exist in v5", search the
whole `packages/` tree for it first, and say which tree you searched.

## What is welcome

- **Corrections** — an API name, signature, namespace, or default that is wrong. Please include the file and
  line from the Filament source that proves it.
- **New verified guidance** — a pattern, pitfall, or deprecation that the skill does not cover yet.
- **Re-verification against a newer Filament release** — see below.
- **Translations** — see the language policy below.

## What is not welcome

- Unverified claims, or claims sourced only from the Filament docs without a source check.
- Style preferences presented as standards. If Filament does not enforce it and the community has not
  converged on it, it does not belong in a "standards" document.
- Large rewrites without a prior issue. Open an issue describing the change first, so we do not both waste
  effort on divergent edits.

## Language policy

`SKILL.md` and `REFERENCE.md` are kept in **English**, deliberately:

- they are a technical reference of roughly 160 KB in which nearly every line is a method name, an FQCN, or
  a code example;
- a translation of that volume drifts from the original on every update, and drift in an API reference is
  worse than having no translation at all;
- agents consume `SKILL.md` as a machine instruction, where English is the lowest-risk option.

The **README is translated** (see [`README.ru.md`](README.ru.md)), because it carries the overview and
requirements, which is what a human reads first.

If you want to add a full translation of the skill, open an issue first. It should be a complete
translation carrying a "current as of `<Filament version>`" marker, kept in sync with the English source —
not a partial one that silently goes stale.

## Re-verifying against a newer Filament release

Guidance is versioned by the Filament release it was checked against. To bump it:

1. Check out the new tag as shown above.
2. Re-check every claim in `SKILL.md` §18 (Gotchas), §20 (Strict Rules), and §16 of `REFERENCE.md`
   (Deprecated names that still compile) — these are the sections most likely to change.
3. Move anything newly deprecated into the deprecated list rather than deleting it from the narrative, so
   readers migrating old code can still find it.
4. Update the tag in the front matter, the intro of both files, and every README mention.
5. Add a `CHANGELOG.md` entry naming the new version.

## Verification tooling

The checks used at release time are simple enough to rerun by hand. For each of these, every referenced
symbol should resolve in the checkout:

- every class, enum, and interface name;
- every method name and artisan command;
- every negative claim — the names the skill says do **not** exist.

A quick shell check for a batch of names:

```bash
hay=$(find packages -name '*.php' -not -path '*/docs/*' -exec cat {} +)
for n in recordActions toolbarActions assertSchemaStateSet scopedUnique; do
  grep -q "$n" <<<"$hay" && echo "OK   $n" || echo "MISS $n"
done
```

## Pull requests

- One logical change per pull request.
- Say **how you verified it** — the command you ran, or the file and line you read.
- Update `CHANGELOG.md` under `[Unreleased]`.
- Keep the existing tone: direct, second person, no marketing language.

Before opening the pull request, confirm:

- [ ] Every API claim is checked against source, with the Filament version stated
- [ ] Negative claims ("X does not exist") were searched across `packages/`
- [ ] Links still resolve, including relative links from inside `.github/skills/`
- [ ] `CHANGELOG.md` updated
- [ ] Code fences are balanced and no tab characters were introduced

## Reporting a problem with Filament itself

Bugs in the framework do not belong here. Report them at
[filamentphp/filament/issues](https://github.com/filamentphp/filament/issues). See
[SECURITY.md](SECURITY.md) for vulnerabilities.
