# SEO.md — search engine optimization rules for agents

Project-agnostic rules for building and editing websites so they are easy for
search engines to crawl, index and understand, and worth a searcher's click.
Reference it from `AGENTS.md` / `CLAUDE.md`; project-specific facts belong there,
not here.

Everything below is drawn from Google Search Central documentation (read in full
on 2026-09-22; the SEO Starter Guide itself was last updated 2025-12-10). §17
adds Google's AI-features and crawler pages and OpenAI's crawler documentation
(read 2026-10-05); keyword research in §5 adds Google's Trends and Search
Console pages (read 2026-10-06). Where Google says something is optional or not worth
worrying about, this file says so too — do not upgrade a "nice to have" into a
rule.

**How to read it.** **MUST** = required for eligibility or a spam-policy line;
breaking it can keep pages out of Search. **SHOULD** = best practice with real
effect. **DON'T** = actively harmful. **Skip** = effort Google says is wasted.

---

## 0. Ground truth

- SEO is helping search engines understand content and helping users decide
  whether to visit. There are no secrets that rank a page first, and
  appearing in Google costs nothing — anyone claiming otherwise is wrong.
- Google works in three stages: **crawl** (discover URLs, fetch, render
  JavaScript in a recent Chrome), **index** (understand the page, cluster
  duplicates, pick a canonical), **serve** (rank for a query). A page can drop
  out at any stage. Meeting every guideline does not guarantee crawling,
  indexing or ranking.
- Canonical preferences and sitemap submissions are **hints**, not commands;
  Google may choose differently.
- Changes take hours to months to show. Wait **a few weeks** before judging an
  SEO change; iterate if needed. Not every change has a visible effect.
- The single biggest lever is **content people find compelling and useful**.
  Technical SEO removes obstacles; it does not create demand.

## 1. Eligibility — the technical minimum (MUST)

A page is only eligible for indexing if **all three** hold:

1. **Googlebot isn't blocked** — not disallowed in robots.txt, not behind a
   login, not otherwise access-controlled.
2. **It returns HTTP `200`.** 4xx/5xx pages are not indexed.
3. **It has indexable content** — text in a supported file type, not in
   violation of the spam policies.

Also:
- Google must see the page **the way a user does**. Never block the CSS,
  JavaScript or images needed to render it.
- Crawling comes from the **US**, usually without `Accept-Language`. If content
  varies by location or language, make sure what that crawler sees is what you
  want indexed.
- Put key information **in text, not in images**.

## 2. Crawling and indexing controls

### Discovery
- Google finds pages mainly **through links** from pages it already knows.
  Every page you care about SHOULD be linked from at least one other page on
  the site.
- Check indexing with the `site:example.com` search operator or Search
  Console's URL Inspection tool before "fixing" anything.

### robots.txt vs `noindex` — they are not interchangeable
- **robots.txt controls crawling, not indexing.** A disallowed URL can still be
  indexed (without a description) if other pages link to it. DON'T use
  robots.txt to hide pages.
- **To keep a page out of Search:** use `<meta name="robots" content="noindex">`
  or the `X-Robots-Tag: noindex` HTTP header (for PDFs, images and other
  non-HTML files) — **and allow crawling**, or Google never sees the rule.
  Password-protect anything that is actually private. Deleting the content
  is the surest removal.
- `noindex` in robots.txt is **not supported**.
- Use robots.txt for: crawl-traffic management, duplicate/unimportant URLs,
  state-changing URLs (add to cart, post comment, create account), infinite
  spaces (search results, calendars, filter combinations), and keeping
  images/video/audio out of results.
- DON'T block resources Google needs to understand the page.
- Combining crawling and indexing rules can cancel them out — a `noindex` on a
  robots.txt-blocked page is never read.
- AI crawlers have their own tokens, and blocking the wrong one removes the
  site from AI search. See §17 before adding any AI-related robots.txt rule.

### Sitemaps
- **Optional for small sites.** Likely not needed if the site is ~500 pages or
  fewer that matter, is fully linked from the home page, and has little
  image/video/news content. Likely needed if the site is large, new with few
  external links, or media-heavy.
- If you have one: **UTF-8**, at the **site root**, **absolute URLs**, only
  **canonical URLs you want in results**, max **50,000 URLs / 50 MB**
  uncompressed per file (split with a sitemap index beyond that). URL order
  doesn't matter.
- Generate it from the same data the site renders from; hand-maintain only
  below a few dozen URLs.
- Google **ignores `<priority>` and `<changefreq>`**. It uses `<lastmod>` only
  when it is **consistently and verifiably accurate** — set it to the last
  *significant* change (main content, structured data, links), not a copyright
  year or build time.
- Submit via Search Console's Sitemaps report or a `Sitemap:` line in
  robots.txt. Submission is a hint, not a guarantee.
- Image, video and news sitemap extensions exist for media-heavy sites.

### Duplicates and canonicalization
- Duplicate content is **not a penalty** or spam; it wastes crawl and confuses
  users. Copying *other sites'* content is a different matter (see §11).
- Aim for **one URL per piece of content**. Strongest to weakest signals:
  **redirect** → **`rel="canonical"`** → **sitemap inclusion**. They stack.
- `rel="canonical"`: in `<head>`, **absolute URL**, a **self-referencing
  canonical on the canonical page**, one method per page (don't mix header and
  element), same answer across every signal (sitemap, canonical, internal
  links). Link internally to the canonical URL.
- DON'T canonicalize with robots.txt, the URL removal tool, `noindex`, or a URL
  fragment.
- Google prefers **HTTPS** as canonical unless undermined by an invalid
  certificate, insecure dependencies, HTTPS→HTTP redirects, a canonical pointing
  to HTTP, or a certificate for the wrong host. Redirect HTTP→HTTPS; HSTS helps.
- Pick one host variant (`www` or apex) and 301 the others to it.

### Redirects
- **Permanent** (`301`, `308`, instant `meta refresh`): the target shows in
  results. **Temporary** (`302`, `303`, `307`, delayed `meta refresh`): the
  source keeps showing.
- Prefer **server-side** redirects. Use JavaScript `location` redirects only if
  nothing else is possible — they only work if rendering succeeds.
- Moving a page: `301` to its new home. Removed page: return a **real `404`**
  (a friendly 404 page is fine), never a `200` "not found" (a *soft 404*).
- Whole-site moves: redirects + updated sitemap + Search Console's Change of
  Address tool.

### HTTP status codes
- Use meaningful codes: `404` for pages that don't exist, `401` for pages
  behind a login, `301`/`308` for moved pages, `5xx` only for real server errors
  (`500`s tell Google to slow down).
- Pages that aren't `200` may skip rendering entirely.

## 3. JavaScript sites and SPAs

Google renders JavaScript, but rendering is queued and can fail, and not every
bot runs JavaScript. **Server-side rendering or prerendering is still
recommended** — it is faster for users and crawlers alike.

- **Links MUST be `<a href="...">`** with a resolvable URL. Google does not
  reliably follow `onclick`, `routerLink` without `href`, `<span href>`, or
  `href="javascript:…"`. Injecting `<a href>` with JavaScript is fine.
- **Routing MUST use the History API**, not `#fragments`. Google ignores
  fragments; `example.com/#/products` is one URL.
- **Titles, meta descriptions and canonicals** may be set by JavaScript, but
  the canonical is best in the initial HTML, and JavaScript **must not change**
  it to a different value. If it can't be in the HTML, set it only in
  JavaScript. Exactly one canonical per page.
- **Don't ship `noindex` in the initial HTML of a page you want indexed** —
  Google may skip rendering on seeing it, so removing it with JavaScript may
  never be seen. Adding `noindex` via JavaScript (e.g. on an error state) works.
- **Soft 404s in SPAs:** for a missing resource, either redirect with
  JavaScript to a URL that returns a real `404`, or inject
  `<meta name="robots" content="noindex">`.
- **Cache-bust assets with content fingerprints** in filenames
  (`main.2bb85551.js`); Google's renderer may ignore caching headers and use
  stale files otherwise.
- **Web components:** Google indexes the flattened, rendered DOM — use `<slot>`
  so light-DOM content appears.
- **Lazy loading** must follow Google's lazy-loading guidelines so content
  appears without user interaction. **Infinite scroll** needs a paginated,
  linkable equivalent. Multi-page articles need crawlable next/previous links.
- JSON-LD may be injected by JavaScript — test it.
- **Verify with URL Inspection / Rich Results Test** that content, links,
  anchor text and structured data are in the *rendered* HTML.

## 4. URLs and site structure

- **Descriptive, readable words**, in the audience's language:
  `/pets/cats` beats `/2/6772756D707920636174`. Parts of the URL can appear as
  breadcrumbs in results.
- **Hyphens** between words, not underscores or run-together words.
- **URLs are case-sensitive** — `/APPLE` and `/apple` are different URLs. If the
  server treats them the same, normalise to one case.
- **Percent-encode** reserved and non-ASCII characters in `href`s (IETF
  STD 66); use UTF-8.
- Parameters: `key=value` joined by `&`; multiple values with `,`. **Use as few
  parameters as possible**; drop ones that don't change content. No session IDs
  in URLs — use cookies.
- Avoid **infinite URL spaces**: additive filters, sort/referral parameters,
  endless calendars (`nofollow` future-date links), and parent-relative links
  that compound (`../../x`) — use **root-relative** links.
- **Group related pages in directories.** It matters most above a few thousand
  URLs (Google learns change frequency per directory). Don't reorganise a
  working small site just for this.
- Organize logically so users and crawlers can see how pages relate. Don't
  drop everything to restructure — Google understands most sites as they are.

## 5. Content

### What Google rewards
Easy to read and well organized (paragraphs, sections, headings, no spelling or
grammar errors); **unique** (not copied or lightly rewritten); **up to date**
(revise, or delete what is no longer relevant); **helpful, reliable and
people-first**.

Self-check before publishing. Does it:
- provide original information, research or analysis beyond the obvious?
- describe the topic substantially and completely?
- add real value if it draws on other sources, rather than rewriting them?
- have a title/main heading that summarises it without exaggeration or shock?
- leave the reader able to reach their goal without searching again?
- contain **no easily verified factual errors**?
- look carefully produced, not hasty or mass-produced?

### Warning signs of search-engine-first content (stop and rethink)
Made mainly to attract search visits; lots of topics hoping some rank; heavy
automation across many topics; mainly summarising others; chasing trends
outside the site's purpose; writing to a word count (**there is no preferred
word count**); entering a niche without expertise; promising answers that don't
exist (e.g. unconfirmed release dates); **changing dates to look fresh without
substantial changes**; adding or deleting content in bulk to seem "fresh"
(it doesn't help).

### E-E-A-T and YMYL
- Experience, Expertise, Authoritativeness, **Trust** (the most important).
  E-E-A-T is **not itself a ranking factor**, but systems favour content that
  shows it — especially on **YMYL** topics (health, finance, safety, societal
  welfare), where the bar is higher.
- Show first-hand experience and clear sourcing; cite and link sources.

### Who / How / Why
- **Who:** make authorship self-evident. Bylines where readers expect them,
  leading to author or About pages with background. Don't invent authors or
  reviewers.
- **How:** explain how content was produced. If automation or AI substantially
  generated it, disclose that where a reader would reasonably wonder, with
  background on how and why it was used.
- **Why:** content must exist primarily to help people. Automation used
  primarily to manipulate rankings is spam.

### AI-generated content
- Allowed, but held to the same standards: **accuracy, quality, relevance** —
  including generated **titles, meta descriptions, structured data and alt
  text**.
- Generating many pages without adding value is **scaled content abuse** (§11).
- **Agent rule:** never invent figures, dates, quotes, studies or credentials.
  Take numbers from a verifiable source (the product's own data, a cited
  study). If something is an estimate, say so in the text.
- E-commerce: AI images need IPTC `DigitalSourceType` =
  `TrainedAlgorithmicMedia`; AI product titles/descriptions must be labelled as
  AI-generated (Merchant Center policy).

### Keywords and keyword research
- Think about the words readers actually search — beginners and experts use
  different terms ("cheese board" vs "charcuterie") — and use them naturally in
  prominent places: title, main heading, alt text, link text.
- Don't chase every variation; Google's language matching understands synonyms
  and related concepts.
- **Research with real data**, in this order:
  1. **Search Console** Performance report, Queries tab: the terms the site
     already appears and gets clicks for. Search Console is the source of
     truth for Search performance.
  2. **Google Trends** Explore (up to 5 terms at once): which terms have high
     or rising interest, where they are popular, and the related topics and
     queries (Top and Rising). A rising term that is still little known may be
     less competitive.
  3. **Filter** to terms that fit the business and where the site has
     first-hand experience. Don't write about something only because it is
     trending (warning signs above).
  4. **Check each market separately.** Interest and seasonality differ by
     country; publish seasonal content a little before searches peak.
- **Agent rule:** never invent search volumes, difficulty scores, rankings or
  trends. Use data the user can see (Search Console, Trends, or an export
  they provide) and label anything else as an estimate. Trends is a sample of
  searches showing interest over time, not exact volume.
- **One page per topic, not per keyword variant.** Substantially similar pages
  aimed at similar queries (one per city, one per synonym) are doorway abuse
  (§11). Cover close variations on one page.

### Ads and interstitials
- Ads must not become distracting or stop users from reading.
- **Avoid intrusive interstitials:** prefer small **banners** over full-page
  overlays; DON'T cover the whole page; DON'T redirect users to a separate page
  for consent or input. Legally mandatory gates (age, consent) are exempt, but
  overlay them on the content rather than redirecting.

## 6. How pages look in results

### Title (`<title>`) — the title link
- **Every page MUST have a `<title>`.** Unique per page, descriptive, concise.
  No hard length limit; results truncate to device width.
- Avoid vague titles ("Home", "Profile"), **keyword stuffing**, repeated or
  boilerplate titles, and titles that vary by one token across many pages.
- Brand concisely: site name at start or end with a delimiter (`-`, `:`, `|`).
  A longer tagline fits the home page; repeated on every page it looks
  repetitive.
- Same **language and script** as the page's main content. Keep dates in titles
  current.
- Make the main title unambiguous: one clearly most prominent heading,
  typically the first visible `<h1>`.
- Google may rewrite the title link from `<h1>`, `og:title`, prominent text,
  anchor text or `WebSite` structured data when the `<title>` is half-empty,
  obsolete, inaccurate or boilerplate.

### Meta description — the snippet
- Snippets come **mainly from page content**; the meta description is used when
  it describes the page better. No length limit.
- **Unique per page**, accurately summarising *this* page: site-level text only
  on the home page or aggregation pages. Prioritise key pages if you can't do
  all of them.
- Can hold structured facts (author, date, price, specs). Programmatic
  generation is fine for large sites if human-readable and varied.
- DON'T write keyword lists, one description for every page, or descriptions
  that don't summarise.
- Controls: `nosnippet`, `max-snippet:[n]`, `data-nosnippet` on elements.
- For "Read more" deep links: keep content visible (not collapsed in
  tabs/accordions), don't force scroll position on load, don't strip the URL
  hash.

### Other result elements
Favicon, site name, visible URL and breadcrumb, byline date, sitelinks,
text-result image, rich attributes. Most are automatic; favicon, site name
(`WebSite` structured data), breadcrumbs and article dates can be provided.

## 7. Links

- **Anchor text** must be descriptive, reasonably concise, and relevant to both
  pages. Read it out of context — if you can't tell where it goes, rewrite it.
  No "click here", "read more", or bare "website".
- No empty anchors. For image links, the `alt` text is the anchor text.
- Give links context (surrounding words matter); don't chain adjacent links.
- Don't cram keywords into anchors — that is keyword stuffing.
- **Internal links:** cross-reference related pages in context. There is no
  ideal number; if it feels like too many, it is.
- **External links:** link out when it helps, especially to cite sources — it
  builds trust. Qualify only when needed:
  - `rel="sponsored"` — paid or advertising links (`nofollow` also accepted);
  - `rel="ugc"` — user-generated links (comments, forums), added automatically;
  - `rel="nofollow"` — sources you don't trust or don't want associated with.
  Don't `nofollow` every external link. For links within your own site, use
  robots.txt `Disallow` instead.

## 8. Images

- Embed with **`<img src>`** (inside `<picture>` is fine). **CSS background
  images are not indexed.** Always give `srcset`/`<picture>` a fallback `src`.
- Formats: BMP, GIF, JPEG, PNG, WebP, SVG, AVIF; the extension should match the
  type.
- **Alt text** on every meaningful image: short, descriptive, in context of the
  page ("Dalmatian puppy playing fetch", not "puppy" and never a keyword list).
  For inline SVG use `<title>`. It serves accessibility first.
- Sharp, high-quality images **near relevant text**, on relevant pages.
- Short descriptive filenames (`black-kitten.jpg`, not `IMG00023.JPG`);
  translate them when localising.
- Optimise for speed (images are usually the heaviest assets); responsive
  images; reference each image by one consistent URL.
- **Preferred preview image** via `og:image` or schema.org
  `primaryImageOfPage` / `image`: relevant and representative, **not a logo**,
  **no text in the image**, no extreme aspect ratio, high resolution.
- Image sitemaps may list images on other domains (CDNs).

## 9. Video

- Embed with `<video>`, `<embed>`, `<iframe>` or `<object>`; don't load by
  fragment or require user interaction to load it; JavaScript-injected players
  must appear in rendered HTML.
- Video features need a **dedicated watch page** (the video is the main
  purpose) with a unique title and description, an **indexed** page, a
  **valid thumbnail at a stable URL**, and a video not hidden behind other
  elements.
- Provide metadata (`VideoObject` structured data, video sitemap, or OGP);
  keep thumbnail and details identical across all sources. Allow Google to
  fetch the video file for previews and key moments.
- Remove an expired video with a `404` or `noindex` on its watch page, or a
  correct `expires` date.

## 10. Structured data

- **JSON-LD recommended** (Microdata and RDFa also valid). Google's docs, not
  schema.org, define what Search uses.
- **MUST** describe **content visible on that page** — never content hidden
  from users, never on empty pages created to hold markup, never misleading
  (fake reviews, impersonation, misrepresenting ownership or purpose).
- Include **all required properties** for a feature; recommended ones only if
  complete and accurate (fewer correct beats many sloppy). Use the most
  specific types. Image URLs must be crawlable and indexable.
- Duplicate pages carry the same markup as their canonical. Link related items
  with `@id`; include the type that represents the page's main purpose.
- Don't block marked-up pages with robots.txt or `noindex`.
- **Markup enables a rich result; it never guarantees one.** Violations can
  bring a manual action that removes rich-result eligibility (ranking itself is
  unaffected).
- Validate with the **Rich Results Test** before release; watch Search
  Console's rich result reports after each deploy and template change.
- **Breadcrumbs** (`BreadcrumbList`): at least two `ListItem`s with `position`,
  `name`, `item` (the last may omit `item`). Represent a typical user path rather
  than mirroring the URL structure; the domain and the page itself may be
  omitted.
- **Organization**: on the home page or one About page, not every page. Use
  the most specific subtype (`OnlineStore`, a `LocalBusiness` type). No
  required properties; add those that apply — `name`, `alternateName`, `url`,
  `logo`, `address`, `telephone`, `sameAs` for official profiles. It helps
  Google tell the organization apart from others and pick the logo shown in
  results and the knowledge panel.

## 11. Spam policies — never do these

Violations can demote a page or remove a whole site. **MUST NOT:**

- **Cloaking** — different content for search engines than for users.
  (Paywalls are fine if Google sees the full gated content, per Flexible
  Sampling guidance.)
- **Doorway pages** — near-duplicate pages or domains made to rank for similar
  queries (e.g. one page per city) that funnel to one destination.
- **Expired domain abuse** — buying old domains to host low-value content.
- **Hidden text or links** — same-colour text, off-screen CSS, zero font size
  or opacity, one-character links. (Accordions, tabs, sliders, tooltips and
  screen-reader-only text are fine.)
- **Keyword stuffing** — unnatural repetition, lists of cities or phone numbers.
- **Link spam** — buying, selling or exchanging links for ranking, automated
  links, links required by contract, paid or advertorial links without
  `rel="sponsored"`/`nofollow`, footer/widget link networks, optimized-anchor
  forum signatures.
- **Machine-generated traffic** — automated queries to Google, including
  scraping results for rank checking. *(Agents: never scrape Google Search;
  use Search Console or its API.)*
- **Malicious practices** — malware, unwanted software, **back-button
  hijacking**.
- **Misleading functionality** — promising a tool or service that doesn't work.
- **Scaled content abuse** — many low-value pages for rankings, by AI,
  scraping, stitching, synonymizing or translating. Exclude such content from
  Search if it exists.
- **Scraping** — republishing others' content without added value or citation.
- **Site reputation abuse** — hosting third-party content mainly to exploit the
  host's ranking signals.
- **Sneaky redirects** — sending users somewhere different from what search
  engines see. (Site moves, consolidation and post-login redirects are fine.)
- **Thin affiliation** — affiliate pages copied from the merchant with nothing
  added.
- **User-generated spam** — moderate public areas; qualify UGC links.
- **Policy circumvention** — new subdomains or sites to keep violating.
- **Scams and fraud** — impersonation, fake support, false business details.
- Also watch for **hacked content**: injected code, pages, links or redirects.

## 12. Page experience

- **Core Web Vitals are used by ranking**; aim for good scores, but chasing a
  perfect score purely for SEO is rarely worth it. There is no single "page
  experience signal".
- Serve over **HTTPS**; migrate from HTTP.
- **Mobile-friendly** — Google crawls with a mobile crawler by default.
- No excessive or distracting ads; no intrusive interstitials; make the main
  content clearly distinguishable.
- Tools: PageSpeed Insights, Lighthouse, Search Console's Core Web Vitals and
  HTTPS reports.

## 13. International and multilingual sites

- **One URL per language version** — not cookies or browser settings.
- Annotate language/region variants with **`hreflang`** (tags, headers or
  sitemap) and keep variants reciprocal. Use `rel="canonical"` + `hreflang` for
  same-language regional duplicates.
- **Don't auto-redirect by guessed language or IP**; link to the other versions
  and let users switch. Don't adapt content by IP — Google crawls mostly from
  the US and won't see the variants.
- Google detects language from **visible content**, not the `lang` attribute or
  URL. One language per page (content *and* navigation); no side-by-side
  translations.
- Geotargeting: ccTLD (`example.de`), subdomain (`de.example.com`) or
  subdirectory (`example.com/de/`); avoid URL parameters (`?loc=de`). Google
  ignores `geo.position` and similar meta tags. Some ccTLDs (`.io`, `.co`,
  `.me`, `.tv`, `.ai`…) and `.eu`/`.asia` are treated as generic.

## 14. Promotion

- Tell people: social media, communities, ads, offline materials (URL on cards
  and signage), opt-in newsletters. **Word of mouth** is among the most lasting
  channels and grows from genuine community engagement.
- Links from other sites happen naturally over time; don't manufacture them
  (§11). Over-promotion can tire users and look like manipulation.

## 15. Things not to spend time on (Skip)

- **Meta keywords** — Google doesn't use them.
- **Keywords in the domain or URL path** — hardly any ranking effect beyond
  breadcrumbs. Name the site for the business.
- **TLD choice** — only matters for country targeting, and weakly.
- **Content length** — no minimum or maximum word count.
- **Subdomains vs subdirectories** — do what suits the business.
- **PageRank obsession** — links are one signal among many.
- **The "duplicate content penalty"** — doesn't exist for your own duplicates
  (copying others' content does).
- **Heading count and order** — semantic order helps screen readers; Google
  doesn't depend on it. No ideal number of headings.
- **E-E-A-T as a ranking factor** — it isn't one.
- **Artificial freshness** — changing dates or churning content doesn't help,
  and changing dates without substantial changes is a warning sign of
  search-engine-first content.
- **AI text files and AI markup** (`llms.txt` and the like) — Google says no
  new machine-readable files, AI text files or special schema.org markup are
  needed to appear in AI Overviews or AI Mode.

## 16. Monitoring

- **Search Console:** verify ownership, check the Page Indexing report, submit
  the sitemap, watch the Performance report (queries, pages, countries). Use
  URL Inspection per page (live test, rendered HTML, request indexing).
  Also: Manual Actions, Removals (temporary, ~6 months), Change of Address,
  Security Issues, rich result reports, Core Web Vitals. Monthly checks plus
  after significant changes are enough; Google emails new issues.
- **Tools:** Rich Results Test, PageSpeed Insights, URL Inspection.
- **Measure changes:** compare before/after on stable pages over months;
  wait weeks before concluding anything.
- **AI traffic:** AI Overviews and AI Mode are counted in the Performance
  report under the "Web" search type, not reported separately. ChatGPT search
  adds `utm_source=chatgpt.com` to the links it sends, so analytics can
  separate that traffic.

## 17. AI search features and AI crawlers

- **Google's AI Overviews and AI Mode have no extra requirements.** A page is
  eligible as a supporting link if it is indexed and can be shown in Search
  with a snippet. Everything above applies; no special optimization is needed.
- **Google controls:** AI features are part of Search, so robots.txt rules for
  Googlebot are the control. To limit what is shown, use `nosnippet`,
  `data-nosnippet`, `max-snippet` or `noindex` (§6, §2).
- **`Google-Extended`** is a robots.txt token only, with no user agent of its
  own. It controls whether crawled content trains Gemini models and grounds
  Gemini Apps and Vertex AI. It does **not** affect inclusion or ranking in
  Google Search.
- **OpenAI uses independent tokens:**
  - `OAI-SearchBot` surfaces pages in ChatGPT search. A site that blocks it
    is not shown in ChatGPT search answers (it can still appear as a bare
    link). Changes take about 24 hours to apply.
  - `GPTBot` crawls for model training. Blocking it opts out of training and
    does not affect ChatGPT search.
  - `ChatGPT-User` fetches pages when a user asks; robots.txt may not apply,
    and it is not used to decide what appears in search.
  - A blocked page can still show as a link and title. To keep it out, use
    `noindex` and allow crawling, as with Google.
- **DON'T** add a blanket "block AI bots" robots.txt group. Blocking search
  crawlers (`Googlebot`, `OAI-SearchBot`) costs visibility. Blocking training
  crawlers (`GPTBot`, `Google-Extended`) is a business decision for the site
  owner, not an SEO fix — ask before adding it.

---

## Agent checklist — before shipping any page or template change

- [ ] Returns `200`; not blocked by robots.txt; no stray `noindex`.
- [ ] robots.txt doesn't block `OAI-SearchBot` or other search crawlers unless
      the owner chose to; any training opt-out was asked for.
- [ ] Main content, links and metadata present in the **server-rendered or
      prerendered HTML**, not only after JavaScript.
- [ ] Unique, descriptive `<title>`; unique meta description; one clear `<h1>`.
- [ ] Self-referencing absolute `rel="canonical"`, consistent with sitemap and
      internal links.
- [ ] Navigation uses `<a href>` with History-API URLs; no fragment routing.
- [ ] Descriptive anchor text; outbound paid/UGC/untrusted links qualified.
- [ ] Every meaningful `<img>` has an `src` and descriptive `alt`.
- [ ] Structured data describes only visible content, has required
      properties, and passes the Rich Results Test.
- [ ] Missing resources return a real `404` (or `noindex` in an SPA).
- [ ] New URL added to the sitemap (if the site has one) with an honest
      `<lastmod>`; removed URL redirected or `404`.
- [ ] Content: factually checked, original, dated honestly, author/process
      clear where readers would expect it; estimates labelled.
- [ ] Nothing from §11.
- [ ] Mobile-friendly, HTTPS, no intrusive interstitial.

## Sources

Google Search Central, `https://developers.google.com/search/docs/`:
`fundamentals/seo-starter-guide` · `essentials` · `essentials/technical` ·
`essentials/spam-policies` · `fundamentals/how-search-works` ·
`fundamentals/creating-helpful-content` · `fundamentals/using-gen-ai-content` ·
`fundamentals/get-started` · `crawling-indexing/sitemaps/overview` ·
`crawling-indexing/sitemaps/build-sitemap` ·
`crawling-indexing/control-what-you-share` · `crawling-indexing/robots/intro` ·
`crawling-indexing/block-indexing` · `crawling-indexing/canonicalization` ·
`crawling-indexing/consolidate-duplicate-urls` ·
`crawling-indexing/301-redirects` · `crawling-indexing/url-structure` ·
`crawling-indexing/links-crawlable` · `crawling-indexing/qualify-outbound-links`
· `crawling-indexing/javascript/javascript-seo-basics` · `appearance/title-link`
· `appearance/snippet` · `appearance/google-images` · `appearance/video` ·
`appearance/avoid-intrusive-interstitials` · `appearance/page-experience` ·
`appearance/ranking-systems-guide` · `appearance/visual-elements-gallery` ·
`appearance/structured-data/intro-structured-data` ·
`appearance/structured-data/sd-policies` ·
`appearance/structured-data/breadcrumb` ·
`appearance/structured-data/organization` · `appearance/ai-features` ·
`specialty/international/managing-multi-regional-sites` ·
`monitor-debug/search-console-start` · `monitor-debug/trends-start` ·
`monitor-debug/google-analytics-search-console`.

Google crawlers:
`https://developers.google.com/crawling/docs/crawlers-fetchers/google-common-crawlers`.

OpenAI: `https://developers.openai.com/api/docs/bots` ·
`https://help.openai.com/en/articles/12627856-publishers-and-developers-faq`.

Google updates these pages; re-read the starter guide when this file is more
than a year old.
