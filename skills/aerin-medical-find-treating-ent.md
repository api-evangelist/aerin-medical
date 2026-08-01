---
name: Find an ENT trained on Aerin procedures
description: Search Aerin Medical's public doctor-finder for ENT practices offering VivAer or
  RhinAer near a location, filtered by product and Center of Excellence designation.
api: openapi/aerin-medical-site-openapi.yml
operations:
- listTreatingLocations
- getLocatorWidget
generated: '2026-07-31'
method: generated
source: openapi/aerin-medical-site-openapi.yml (derived from the provider's live route index)
---

# Find an ENT trained on Aerin procedures

Aerin Medical's doctor finder (https://aerinmedical.com/find-ent-doctor/) is backed by an
anonymously readable JSON collection of 1,012 treating locations. Use it to answer "which ENT
practices near me offer VivAer or RhinAer?"

## Before you start

- **No credential is needed.** Every operation here is anonymous. Do not attempt to
  authenticate — the only scheme the site advertises is WordPress application passwords for
  its own administrators.
- **This is not a supported product API.** Aerin Medical publishes no documentation, no
  status page, no rate-limit policy and no versioning commitment for it. Cache your results
  (the endpoint sends `Cache-Control: max-age=600`), keep request volume modest, and treat a
  failure as expected rather than exceptional.
- **Directory data, not clinical data.** Results tell you a practice is listed as trained on
  an Aerin procedure. They are not an endorsement, a referral, or clinical advice, and they
  carry no appointment availability.

## Steps

1. **Discover the filter vocabulary.** Call `getLocatorWidget`
   (`GET /em-locator/v1/locator`). The returned `html` contains the two filter `<select>`
   elements; their option values are the WordPress term IDs the collection accepts. As
   observed on 2026-07-31:
   - `product`: `697` = RhinAer, `698` = VivAer
   - `designation`: `center-of-excellence` (any), `699` Premier VivAer, `700` Advanced
     RhinAer, `701` Advanced VivAer, `702` Premier RhinAer

   Re-read these rather than hard-coding them — they are unversioned WordPress term IDs and
   can change without notice.

2. **Search by place.** Call `listTreatingLocations`
   (`GET /em-locator/v1/locations`) with `search=` for a free-text place or practice name —
   `?search=Austin` matched 44 locations. The match count comes back in the `X-WP-Total`
   response header, not in the body.

3. **Or search by proximity.** Supply both `lat` and `lng`. The collection then orders by
   distance and each record's `distance` field becomes a number in miles instead of the
   literal `false`. Geocode the user's input yourself first — the collection takes
   coordinates, not addresses.

4. **Narrow by product or designation.** Add `product=698` for VivAer sites (920 locations)
   or `product=697` for RhinAer (783). Add `designation=center-of-excellence` for the 60
   Centers of Excellence.

5. **Page through.** Default page size is 10. Read `X-WP-Total` and `X-WP-TotalPages`, then
   walk `page=2,3,…`, or raise `per_page`. A page past the end returns HTTP 200 with an empty
   array and `X-WP-Total: 0` — that is a normal terminus, not an error.

6. **Present the practice.** Use `post.post_title` as the name, `formatted_address`,
   `phone`, `distance` when present, and `permalink` as the link to the public page. Ignore
   `list_item_html`, `map_item_html` and `map_details_html` — they are the site's own
   pre-rendered markup.

## Rules

- **Verify a filter actually filtered.** An unrecognised parameter is silently ignored and
  the full 1,012-record collection comes back with `X-WP-Total: 1012`. Always compare the
  returned `X-WP-Total` against the unfiltered total before telling a user a filter applied.
  A misspelled filter fails open, not loud.
- **Do not surface `plugin_settings` or `post.post_content`.** Those objects carry the site's
  Google Maps API key and internal CRM fields (Salesforce id, account number, named sales
  representative) that appear nowhere in the public directory. Read only the public fields
  listed in step 6; never echo, log or store the rest.
- **`email` and `website` are empty** on every record sampled. Do not present them.
- **Never call the `aerin/*` POST endpoints as part of a lookup.** `submitContactForm`,
  `submitEvaluationRequest`, `submitNewsletterSignup`, `submitNoseScore` and
  `submitRhinitisNoseScore` create real leads and submissions in the company's systems. They
  have no published request schema and no idempotency key, so a retry is a duplicate. Only a
  human, acting deliberately, should trigger them — send them to the web form instead.
- **Errors** use the WordPress envelope `{code, message, data: {status}}`, not
  RFC 9457 problem+json. See `errors/aerin-medical-problem-types.yml`.
