# Aerin Medical

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Aerin Medical, Inc. — 2565 Leghorn Street, Mountain View, California — develops
temperature-controlled radiofrequency devices that let ENT physicians treat chronic nasal
conditions in the office under local anesthetic, without incisions. Two FDA-cleared products,
delivered through the Aerin Console: the **VivAer Stylus** for nasal airway obstruction and the
**RhinAer Stylus** for chronic rhinitis. More than 200,000 patients treated as of March 2026.

- https://aerinmedical.com/
- https://vivaer.com · https://rhinaer.com

## API posture

**No developer API programme.** No portal, no documentation, no keys, no SDKs, no CLI, no MCP
server, no agent card, no webhooks, no status page, no Postman collection, no sandbox, and no
terms of use for programmatic access. `/.well-known/*`, `/llms.txt`, `/openapi.json`,
`/swagger.json` and `/api-docs` all return 404 on aerinmedical.com, vivaer.com and rhinaer.com.

What does exist is an **anonymously readable WordPress REST API** at
`https://aerinmedical.com/wp-json` — 321 routes across 15 namespaces — with an unusual and
inverted access posture:

- **Open:** the company's own doctor-finder plugin, `em-locator/v1`. `GET
  /em-locator/v1/locations` returns **1,012 treating ENT locations** with practice name,
  formatted address, phone, latitude/longitude and public permalink, and supports free-text
  `search`, `lat`/`lng` proximity ranking, `product` (RhinAer 783 / VivAer 920) and
  `designation` (60 Centers of Excellence) filters, paged with `X-WP-Total` /
  `X-WP-TotalPages`. `Access-Control-Allow-Origin: *`. This is a genuinely useful public
  dataset.
- **Open:** `wp/v2/search` (2,242 items), oEmbed, the route-discovery documents, the footer.
- **Closed:** every standard `wp/v2` content collection — posts, pages, media, users,
  taxonomies — returns `401 itsec_rest_api_access_restricted`.

So the routes holding public marketing copy are locked, while the open locator route
over-returns: each record echoes the site's Google Maps API key and internal CRM fields
(Salesforce id, account number, named sales rep) that appear nowhere in the rendered directory.
Recorded as an observation in `security/aerin-medical-domain-security.yml` — the values
themselves are deliberately not reproduced in this repository.

Aerin Medical does publish a real **Coordinated Vulnerability Disclosure Policy**
(https://aerinmedical.com/cybersecurity/): report to `security@aerinmedical.com`,
acknowledgement within 5 business days, safe harbour offered, scope limited to the Aerin
Console. It is not advertised at `/.well-known/security.txt`, which 404s — publishing an RFC
9116 file is the cheapest discovery win available here.

## Artifacts

| Path | Type | Method |
|---|---|---|
| `openapi/aerin-medical-site-openapi.yml` | OpenAPI 3.1, 15 operations | derived from the live route index + probing |
| `overlays/aerin-medical-site-overlay.yaml` | Overlay 1.0.0 | generated |
| `authentication/aerin-medical-authentication.yml` | Authentication | derived + probed |
| `conventions/aerin-medical-conventions.yml` | Conventions | derived |
| `errors/aerin-medical-problem-types.yml` | ErrorCatalog | derived from observed responses |
| `data-model/aerin-medical-data-model.yml` | DataModel | derived |
| `lifecycle/aerin-medical-lifecycle.yml` | Lifecycle | derived |
| `conformance/aerin-medical-conformance.yml` | Conformance | searched |
| `security/aerin-medical-vulnerability-disclosure.yml` | VulnerabilityDisclosure | searched |
| `security/aerin-medical-domain-security.yml` | DomainSecurity | probed |
| `well-known/aerin-medical-well-known.yml` | WellKnown | probed (all 404) |
| `skills/` | AgentSkill ×2 | generated |
| `llms/aerin-medical-llms.txt` | LLMsTxt | generated |

Deliberately **not** written, because nothing real was found: `a2a/` (agent card — search-only
by contract), `mcp/`, `packages/`, `scopes/`, `asyncapi/`, `grpc/`, `cli/`, `components/`,
`sandbox/`, `changelog/`, `rate-limits/`, `plans/`, `security/…-trust-center.yml`.
