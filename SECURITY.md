# Security Policy

## Scope

This repository contains **documentation only** — a skill document describing Filament's public API. There is
no executable code, no dependencies, and nothing is deployed from here.

That narrows the threat model considerably, but it is not zero. The realistic issues are:

| In scope | Why it matters |
|---|---|
| Guidance that would lead a reader to write insecure code | An agent reading `SKILL.md` will follow it literally. Advice that omits an authorization check, recommends `->preserveFilenames()`, or suggests passing unescaped user input to `extra*Attributes()` is a security bug *in this repository*. |
| A claim that overstates a protection Filament provides | The skill is explicit about where Filament's guarantees stop. Softening or losing those boundaries is a regression. |
| Malicious content in a contribution | Including prompt-injection text aimed at agents that consume `SKILL.md`, or links to compromised resources. |
| A dependency or CI workflow that could be subverted | Currently there are none; see below. |

Out of scope: typos, broken links, style disagreements, and anything that only affects rendering.

## Reporting a vulnerability in Filament itself

**Do not report framework vulnerabilities here.** Filament's own security policy applies:

<https://github.com/filamentphp/filament/security/policy>

Report through GitHub's private advisory flow on the upstream repository:

<https://github.com/filamentphp/filament/security/advisories/new>

If you believe this repository's *guidance* about a Filament security boundary is wrong, that **is** in scope
here — see below.

## Reporting an issue with this repository

Use GitHub's private vulnerability reporting:

**<https://github.com/itsbuyru-bit/filament-v5-skills/security/advisories/new>**

This opens a private advisory visible only to you and the maintainer. Please do not open a public issue for
anything in the "In scope" table above, since a public report of "this guidance tells people to do X
insecurely" is itself a usable hint for as long as the guidance is unfixed.

For anything that is clearly not security-sensitive — a wrong method signature, a missing deprecation — a
normal [issue](https://github.com/itsbuyru-bit/filament-v5-skills/issues) is fine and faster.

### What to include

- **Which file and section** — for example `SKILL.md` §13, or `REFERENCE.md` §12.
- **The guidance as written**, quoted.
- **Why it is unsafe**, ideally with the file and line in the Filament source that shows the real behaviour,
  or a concrete attack sketch.
- **Suggested wording**, if you have it.

## Response expectations

This is a single-maintainer documentation project, so please calibrate accordingly:

- **Acknowledgement** — best effort within a few days.
- **Assessment** — whether the guidance is wrong, and whether it needs an immediate correction or can wait
  for the next verification pass.
- **Fix** — documentation fixes ship as soon as they are verified, and the `CHANGELOG.md` entry will
  reference the advisory where appropriate.
- **Credit** — you will be credited in the advisory and the changelog unless you ask not to be.

## Note on the threat model that actually matters here

Because `SKILL.md` is written to be consumed by AI agents, the highest-impact failure mode is not a
compromised artifact — it is **plausible-sounding but wrong security guidance**. An agent that follows
"Filament scopes this for you" when it does not will generate an application with a data leak.

Every security claim in `SKILL.md` §13 and §12 is therefore written as an explicit boundary: what Filament
does, where that stops, and what the application developer must implement. Preserving those boundaries is a
hard requirement for contributions — see [CONTRIBUTING.md](CONTRIBUTING.md).

## Automated tooling

There are no dependencies, no build step, and no CI workflows in this repository. If that changes, this
section must be updated to describe the supply-chain surface, and dependency updates should be reviewed with
the same care as the guidance itself.

## Supported versions

Only the latest release of the skill document is supported. Guidance is versioned against the Filament release
it was verified against, stated in the front matter of `SKILL.md` and in every `CHANGELOG.md` entry.
