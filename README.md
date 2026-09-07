# brave-goggles

Personal [Brave Search Goggles](https://search.brave.com/goggles) — custom
search re-ranking rule files.

## What are Goggles?

Goggles are plain-text rule files that re-rank Brave Search results. Each
rule targets a URL pattern (optionally scoped to a `site=`) and says whether
matching results should be boosted, downranked, or discarded entirely.
Applying one is like swapping in a custom ranking function on top of Brave's
normal index — useful for cutting SEO spam, favoring primary sources, or
building a narrow topic-specific search experience.

See [`SYNTAX-NOTES.md`](./SYNTAX-NOTES.md) for a syntax cheat sheet, and the
upstream [brave/goggles-quickstart](https://github.com/brave/goggles-quickstart)
repo for the full spec and more examples.

## How to apply a Goggle

Two ways:

1. **Goggles creator UI** — go to https://search.brave.com/goggles/create,
   paste in the raw URL of a hosted `.goggle` file, and submit. Brave fetches,
   validates, and caches it. From then on you can select it from the Goggles
   picker in Brave Search.
2. **Direct query URL** — append it to any search:
   `https://search.brave.com/goggles?goggles_id={URL_ENCODED_GOGGLE_URL}`
   (the Goggle must have already been submitted once via the creator UI
   above so Brave has fetched/validated it).

## Hosting requirement

Brave Search fetches Goggles over HTTP — a file only sitting in this local
repo isn't usable yet. Each `.goggle` file needs a **public, raw-text URL**
Brave can fetch and validate. Options:

- Push this repo to a **public** GitHub repo and use the raw URL, e.g.:
  `https://raw.githubusercontent.com/<user>/brave-goggles/main/goggles/tech-research.goggle`
- Or host individual files as GitHub Gists or GitLab snippets/files.

Whenever a `.goggle` file is edited, re-submit its URL at
`https://search.brave.com/goggles/create` to force Brave to re-fetch — there's
no auto-refresh and no version history on Brave's side.

## Goggles in this repo

| File | Purpose |
|---|---|
| [`goggles/daily-driver.goggle`](./goggles/daily-driver.goggle) | Light-touch, general-purpose search. Downranks known SEO/content-farm domains without hiding anything outright. Safe to use as a default. |
| [`goggles/tech-research.goggle`](./goggles/tech-research.goggle) | Boosts developer/technical sources — official docs, GitHub, Stack Overflow, Hacker News-adjacent sites, relevant subreddits/forums — and downranks or discards tutorial-mill and copycat/scraper sites. Use for programming, sysadmin, and technical research queries. |
| [`goggles/news.goggle`](./goggles/news.goggle) | Stub only — no rules yet. Placeholder for a future news-source curation Goggle (see comments in the file for the intended direction). |

All three currently have `! public: false` in their metadata — usable by
anyone with the direct URL, but not listed on the
[discovery page](https://search.brave.com/goggles/discover). Flip to `true`
once a Goggle is polished enough to share.
