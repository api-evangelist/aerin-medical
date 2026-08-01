---
name: Search Aerin Medical site content
description: Query aerinmedical.com's cross-content search index for clinical studies, press
  releases and product pages, and understand which content is machine-readable and which is not.
api: openapi/aerin-medical-site-openapi.yml
operations:
- searchSite
- getRouteIndex
- getOEmbed
generated: '2026-07-31'
method: generated
source: openapi/aerin-medical-site-openapi.yml (derived from the provider's live route index)
---

# Search Aerin Medical site content

Aerin Medical's site exposes a search index of 2,242 items — press releases, clinical-study
summaries, product pages, locations and physicians. It is the only `wp/v2` collection that
answers anonymously.

## Before you start

Understand the shape of the restriction, because it determines what you can and cannot do:

- `searchSite` (`GET /wp/v2/search`) returns **200** and gives you `id`, `title`, `url`,
  `type`, `subtype` — enough to find and link a page.
- Every other `wp/v2` collection — `posts`, `pages`, `media`, `categories`, `tags`, `users`,
  `comments`, `types`, `taxonomies`, `statuses` — returns **401**
  `itsec_rest_api_access_restricted`. The `_links.self` href on each search result points at
  one of those, so **following it will fail**.
- Therefore: use the API to *locate* content, then fetch the human page at `url` to read it.
  There is no anonymous route to article bodies as JSON.

## Steps

1. **Confirm the surface is still what you expect.** `getRouteIndex` (`GET /wp-json/`)
   returns the site's own route index: 15 namespaces, 321 routes, and the authentication
   block. Re-read it rather than assuming — this is an unversioned, undocumented surface.

2. **Search.** Call `searchSite` with `search=` your term. Set `per_page` up to 100 and read
   `X-WP-Total` / `X-WP-TotalPages` for the match count.

3. **Scope by content type.** `subtype` accepts the site's real custom post types:
   `post`, `page`, `location`, `physician`, `alert`, `category`, `post_tag`,
   `em_designation`, `any`. Use `subtype=location` to search practice listings, or
   `subtype=page` for the clinical and product pages — the news items and study write-ups
   are published as pages here, not posts.

4. **Retrieve the content.** Fetch the human `url`. For a link preview, `getOEmbed`
   (`GET /oembed/1.0/embed?url=…`) returns a title/author/thumbnail object anonymously.

5. **Do not reach for RSS.** Both https://aerinmedical.com/feed/ and
   https://aerinmedical.com/company/news-and-media/feed/ return HTTP 200 with a valid but
   **empty** RSS document — zero `<item>` elements — because press releases are published as
   WordPress *pages*, not posts. Use `searchSite` with `subtype=page` and then read the human
   pages under `/company/news-and-media/`.

## Rules

- **Do not retry a 401.** `itsec_rest_api_access_restricted` is a deliberate site
  configuration, not a transient failure and not a missing-credential problem. No public
  credential exists that would change the outcome.
- **Attribute to the page, not the API.** Cite the `url` on aerinmedical.com. This surface is
  undocumented and unsupported; do not present it to users as an Aerin Medical API.
- **Respect the content.** Clinical claims about VivAer and RhinAer are regulated statements.
  Quote the company's own wording and link to
  https://aerinmedical.com/important-safety-information/ alongside any efficacy claim; do not
  paraphrase indications, outcomes or safety information.
- **Never call the `aerin/*` POST endpoints** while searching — they submit contact requests,
  newsletter sign-ups and symptom self-assessments into live systems.
- **Be gentle.** No rate-limit policy is published, and its absence is not permission. Responses
  are cached for 600 seconds at the edge; honour that rather than polling.
