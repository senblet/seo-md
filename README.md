# SEO.md

**Google's SEO rules in one file for your coding agent.**

Your agent can build a page in seconds. It will also give every route the same
`<title>`, put hash routing in your SPA, block CSS in `robots.txt`, and make up
a publish date for a blog post. None of that breaks the build, and all of it
costs you in search.

`SEO.md` is a single Markdown file you drop into a repository so Claude Code,
Codex, Cursor, Copilot or any other agent follows Google's actual rules while
it works, instead of half-remembered SEO folklore.

```sh
curl -O https://raw.githubusercontent.com/senblet/seo-md/main/SEO.md
```

## Why this exists

Google's [SEO Starter Guide](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)
is short and readable, but it's high-level on purpose: almost every section
ends in "learn more" and sends you to another page for the concrete rules.
Those rules are spread across more than 30 pages of documentation.

This file is the starter guide **plus the 32 pages it points to**, all read in
full and condensed into one document an agent can act on. It keeps the parts
that change what you build and drops the rest. Where Google says something is
optional or not worth your time, the file says that too, so your agent doesn't
turn a "nice to have" into a rule.

It also covers AI search: Google's AI Overviews and AI Mode, and the crawler
tokens for Google and OpenAI, taken from their own documentation. An agent
asked to "block AI bots" can otherwise take a site out of ChatGPT search
without anyone noticing.

## What's inside

Every rule is tagged so an agent can tell a requirement from a preference:

| Tag | Meaning |
|---|---|
| **MUST** | Required to be eligible for Search, or a spam-policy line |
| **SHOULD** | Best practice with a real effect |
| **DON'T** | Actively harmful |
| **Skip** | Effort Google says is wasted |

| # | Section | Covers |
|---|---|---|
| 0 | Ground truth | How crawl → index → serve works, what's a hint and what's a rule, how long changes take |
| 1 | Eligibility | The three technical minimums: not blocked, HTTP `200`, indexable content |
| 2 | Crawling and indexing | robots.txt vs `noindex`, sitemaps, canonicals, redirects, status codes |
| 3 | JavaScript and SPAs | `<a href>` links, History-API routing, soft 404s, canonicals in JS, rendering pitfalls |
| 4 | URLs and structure | Readable URLs, hyphens, case sensitivity, parameters, infinite URL spaces |
| 5 | Content | Helpful content, E-E-A-T, YMYL, Who/How/Why, AI-generated content, honest dates |
| 6 | Search appearance | `<title>`, meta descriptions, snippet controls, "Read more" deep links |
| 7 | Links | Anchor text, internal and external links, `sponsored` / `ugc` / `nofollow` |
| 8 | Images | `<img>` vs CSS backgrounds, alt text, formats, preview images |
| 9 | Video | Watch pages, thumbnails, metadata |
| 10 | Structured data | JSON-LD, visible-content rule, required properties, breadcrumbs, organization |
| 11 | Spam policies | The full list of things that get pages demoted or removed |
| 12 | Page experience | Core Web Vitals, HTTPS, mobile, interstitials |
| 13 | International | `hreflang`, language detection, geotargeting |
| 14 | Promotion | Getting found without manufacturing links |
| 15 | Don't bother | Meta keywords, word counts, heading order, the "duplicate content penalty" myth, `llms.txt` |
| 16 | Monitoring | Search Console, the tools, how to measure a change, AI traffic |
| 17 | AI search | AI Overviews and AI Mode, `Google-Extended`, `OAI-SearchBot` vs `GPTBot` |

It ends with a **pre-ship checklist** an agent can run before any page or
template change, and two rules written for agents in particular: never
invent figures, and never scrape Google's results (that's against Google's
spam policies).

## How to use it

Copy `SEO.md` to the root of your repository, then point your agent at it.

**Any agent that reads `AGENTS.md`** (Codex, Cursor, Gemini, Copilot and
others):

```markdown
<!-- AGENTS.md -->
Read [SEO.md](SEO.md) before changing anything that renders a public page:
routes, <head>, content, links, images or redirects.
```

**Claude Code** reads `CLAUDE.md` and can import files into it, so the rules
load into every session:

```markdown
<!-- CLAUDE.md -->
@AGENTS.md
@SEO.md
```

**Other tools** (Cursor rules, `.github/copilot-instructions.md`, and so on):
add the same one-line pointer to whichever instructions file your tool reads.

Keep `SEO.md` generic. Put project-specific facts (your domain, your routes,
your analytics setup) in `AGENTS.md` next to it, so you can update this file
from upstream without merge conflicts.

If your site is only an app behind a login, you probably don't need this file.
It's for pages you want people to find.

## Keeping it current

Google updates its documentation. This version was read on **2026-09-22**; the
AI search additions on **2026-10-05**. The SEO Starter Guide itself was last
updated 2025-12-10. Every page it draws on is
listed at the bottom of `SEO.md`. If the file is more than a year old, re-read
the starter guide before relying on it.

## Credits

The source material is Google Search Central's
[documentation](https://developers.google.com/search/docs), licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), plus OpenAI's
[crawler documentation](https://developers.openai.com/api/docs/bots) for the
ChatGPT search rules. This is an independent summary. It isn't affiliated
with or endorsed by Google or OpenAI, and when they disagree with it, their
documentation wins.

Made by [@senblet](https://github.com/senblet).
