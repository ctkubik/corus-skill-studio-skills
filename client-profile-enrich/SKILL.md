---
purpose: utility
name: client-profile-enrich
description: Reads a client website and extracts business name, phones, addresses, GBP URL, coordinates, and a multi-location signal.
user-invokable: true
---

<!-- enrich-template:v3 -->

You are doing the first 5 minutes of agency intake for a new client.
Read their website and extract the data the agency needs to scope
keyword targeting, local SEO, and reporting.

The client's domain is in the `client-context` block above. Treat
that as the homepage to start from. Use the WebFetch tool to read
HTML pages.

## Pages to read

1. The homepage (start here — header / footer often has the basics)
2. /contact, /contact-us, or whatever the homepage's "Contact" link
   points to
3. /locations or /our-locations if a homepage link suggests one
4. /about, /about-us if you still need data after contact pages

If a page returns 404, skip it and move on. Don't fetch more than
about 6 pages — diminishing returns past that.

## What to extract

- **business_name**: the official name, not the domain. Sources, in
  priority order:
  1. Structured data (JSON-LD with @type Organization / LocalBusiness)
  2. The HTML <title> tag (often "Acme Pediatric Dentistry — Phoenix")
  3. Footer copyright line ("© 2025 Acme Pediatric Dentistry")
  4. Header logo alt text or h1 on the homepage
  Don't include taglines or location qualifiers in the name unless
  they're part of the legal entity ("Acme Dental, P.C.").

- **phones**: every distinct phone number on the site. Look for
  tel: links, "Call us" sections, per-location phones on a /locations
  page. Deduplicate by digits-only comparison.

- **contact_emails**: business email addresses. mailto: links, "Email
  us" sections. Skip generic noreply@.

- **locations**: every physical address. For each location:
  - `name` — friendly label if present ("Phoenix Office")
  - `street` — street address
  - `city`
  - `state` — 2-letter abbreviation if US, full name otherwise
  - `postal_code`
  - `country` — 2-letter ISO code if known
  - `phone` — per-location phone if separate from main
  - `gbp_url` — Google Business Profile URL if linked from the
    site. Look for: "Find us on Google", a Google Maps embed iframe,
    a footer "Reviews on Google" link, or any `google.com/maps` /
    `maps.app.goo.gl` / `g.co/kgs/` link near the location's
    address block. Don't synthesize — only emit when actually linked.
  - `latitude`, `longitude` — only if the page exposes them
    explicitly (JSON-LD geo coords, embedded map iframe lat/lon
    parameters, or a meta tag). Skip when not directly available;
    Skill Studio backfills coordinates via a geocoding step.

- **multi_location**: true when the site advertises 2+ distinct
  physical addresses, has a /locations page, or uses language like
  "Our offices" / "Find a location near you". The agency uses this
  flag to decide whether to pursue local-SEO-per-location or a
  single GBP listing.

## Edge cases

- **Single-page sites**: pull from the homepage; don't fail.
- **Sites under construction**: low confidence; emit a warning
  finding and include whatever you found.
- **JS-rendered sites**: WebFetch returns post-render HTML in most
  cases. If the response is genuinely empty, note it as a finding.
- **Aggregator domains** (the "site" is actually a Yelp listing or
  similar): emit a critical finding flagging the agency to confirm
  the real domain.

## Output

Write a brief prose summary first, then end with a fenced
`skill-output` JSON block. Stick to the schema below — the Apply
dialog in Skill Studio parses these fields exactly:

\`\`\`skill-output
{
  "schema": "skill-output@1",
  "score": <0-100, your confidence the data is correct + complete>,
  "tags": ["client-onboarding", "business-discovery"],
  "findings": [
    {
      "id": "stable-id-per-finding",
      "category": "data-quality",
      "severity": "info",
      "title": "Short headline",
      "client_facing_summary": "What the client would understand",
      "internal_notes": "Agency-internal context"
    }
  ],
  "client_profile": {
    "business_name": "Acme Pediatric Dentistry",
    "phones": ["+15551234567"],
    "contact_emails": ["info@acmepedo.com"],
    "locations": [
      {
        "name": "Phoenix Office",
        "street": "123 Main St",
        "city": "Phoenix",
        "state": "AZ",
        "postal_code": "85001",
        "country": "US",
        "gbp_url": "https://maps.app.goo.gl/example",
        "latitude": 33.4484,
        "longitude": -112.0740
      }
    ],
    "multi_location": false
  }
}
\`\`\`

Only emit a single output block. The score should reflect coverage:
100 = found everything cleanly; 70 = some ambiguity; 40 = sparse
data; 10 = site barely usable.
