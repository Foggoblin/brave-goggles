# Goggles Syntax Notes

Quick reference distilled from [brave/goggles-quickstart](https://github.com/brave/goggles-quickstart)
(`getting-started.md`, `faq.md`, and `goggles/quickstart.goggle`). Read the
upstream repo for the full, authoritative spec — this is a cheat sheet, not a
replacement.

## File basics

- A Goggle is a plain-text file, one instruction per line.
- Blank lines and comments are ignored. Comments start with `!`.
- Instructions are only valid within the limits Brave imposes: file ≤ 2MB,
  ≤ 100,000 instructions, each instruction ≤ 500 characters, and at most 2
  `*` (wildcard) or 2 `^` (caret) characters per instruction.

## Required header metadata

Every Goggle must start with a metadata block, one `!`-prefixed key per line:

```
! name: My Goggle
! description: What my Goggle does
! public: false
! author: Me
```

Optional metadata: `homepage`, `issues`, `transferred_to`, `avatar` (a hex
color), `license`.

`public: true` makes the Goggle discoverable on
https://search.brave.com/goggles/discover. `public: false` keeps it usable by
anyone with the URL but unlisted.

## URL pattern matching

- A bare pattern matches anywhere in the URL:
  `/this/is/a/pattern`
- `*` is a wildcard matching zero or more characters (max 2 per instruction):
  `/this/is/*/pattern`
- `^` matches a URL "separator" (end of URL, or any char that isn't a
  letter/digit/`.`/`_`/`%`/`-` — e.g. `/`, `=`, `?`, `:`). Useful to avoid
  accidental substring matches:
  - `|https://example.org^` matches `example.org`, `example.org/`,
    `example.org/path`, but *not* `example.org.ac`.
  - `/foo.js^` matches `/foo.js` and `/foo.js?x=1` but not `/foo.jsx`.
- `|` anchors a pattern to the start or end of the URL:
  - `|https://en.` — prefix match.
  - `/some/path.html|` — suffix match.
  - `|https://brave.com|` — anchored both ends.

## Options (`$...`)

Options follow `$`, comma-separated, and can be combined with a pattern or
used standalone.

- `site=<domain>` — restrict to a domain (with or without a URL pattern):
  `$site=brave.com` or `/blog/$site=brave.com`
- Future/planned match targets: `$inurl`, `$intitle`, `$indescription`,
  `$incontent` (URL matching is what's implemented today).

## Actions (`$boost` / `$downrank` / `$discard`)

Every instruction has an implicit or explicit action. Default action (no
action specified) is `boost`.

- `$boost` / `$boost=N` — increase ranking. `N` is a strength, 1–10 (higher =
  stronger boost). Plain `/path/` with no options at all also implies boost.
- `$downrank` / `$downrank=N` — decrease ranking, same strength scale.
- `$discard` — remove the result entirely. Can stand alone (`$discard,site=example.com`)
  or attached to a pattern (`/this/is/spam/$discard`).

Examples:
```
/r/brave_browser/$boost=3
/r/google/$downrank=2
$discard,site=idontwanttobepartoftheresults.com
```

## Conflict resolution

When multiple instructions match the same URL:

- Precedence: **discard > boost > downrank**, and within boost/downrank,
  higher strength wins over lower.
  - `$discard` (non-generic) beats any `$boost`.
  - `$boost=3` beats `$boost=2`.
  - `$boost=1` beats `$downrank` (any strength) — boosts always win over
    downranks per Goggles' "surface more content" philosophy.
  - `$downrank=3` beats `$downrank=2`.

## Default action / "closed" Goggles

By default, a result not matched by any instruction can still appear if
Brave's core relevance ranking considers it a good match. To make a Goggle
"closed" (only show explicitly matched results), add a generic `$discard` as
the first rule, then boost what you want to allow through:

```
$discard
$boost,site=en.wikipedia.org
$boost,site=de.wikipedia.org
```

This discards everything except pages on the two listed sites.

## Hosting & applying a Goggle

- Host the raw text file publicly (GitHub, GitHub Gist, or GitLab
  file/snippet) — Brave needs a URL it can fetch.
- Submit that URL at https://search.brave.com/goggles/create to validate and
  cache it.
- Apply directly via query string:
  `https://search.brave.com/goggles?goggles_id={URL_ENCODED_GOGGLE_URL}`
- Re-submit the same URL at `/goggles/create` any time you edit the file, to
  force Brave to re-fetch it. There's no version history — you're the sole
  keeper of history via your own repo/git log.
