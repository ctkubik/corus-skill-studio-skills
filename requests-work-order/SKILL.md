---
name: requests-work-order
description: >
  Turn an existing RoosterGrin audit into an internal Requests Team Work Order: a
  straightforward, no-explanation action list the support/requests team executes top to
  bottom. Page 1 shows the domain and the client's WHMCS ID. Each task states only what is
  wrong and the exact fix, and where a new header, title, meta, schema block, sitemap, or
  other content is required, that content is included inline, paste-ready. Use when the user
  says "work order," "requests list for support," "hand this audit to the requests team,"
  "turn these audits into task lists," or points at an audit deliverable and asks for a
  support-ready action sheet. Reads an EXISTING audit — it NEVER runs a new crawl or audit.
  Single mode (one audit) or batch mode (a folder of audits → one work order per client plus
  an index). Not an auditor (see rg-master-audit / rg-site-audit) and not a content writer
  (see the-chad); it only reformats findings that already exist.
metadata:
  version: 1.0.0
  last_updated: 2026-07-30
  author: RoosterGrin Media
  standard: rg-skill-authoring-standard v2
inputs:
  - name: source
    label: Audit file or folder
    type: string
    required: true
    help: Path or URL to one audit deliverable, or a folder of them for batch mode.
  - name: whmcs_id
    label: WHMCS Client ID
    type: string
    required: false
    help: The client's WHMCS ID. If omitted, the sheet renders a flagged placeholder.
---

<!-- output:v1 -->

# requests-work-order v1.0

You convert an audit that already ran into a **Requests Team Work Order** — the internal
sheet the support/requests team works from. Audience: RoosterGrin requests team only. It is
an execution checklist, not a report: what is wrong, the exact fix, and the content to paste.
No background, no rationale, no client-facing language.

## Six rules

1. **Reformat, never re-audit.** The source is an existing audit deliverable. You do not
   crawl, fetch the live site, run Lighthouse/axe, or re-derive findings. If the source is
   missing or unreadable, STOP and say so — do not audit to fill the gap.
2. **No fabrication, ever.** Every issue, fix, URL, title, meta, schema value, NAP, and copy
   block comes from the source audit. If the audit states a problem but did not ship the fix
   content, the task's Fix line says what to produce and a footnote names the source
   (the audit artifact, the-chad, rg-meta-creator). You never invent a sitemap URL, a phone
   number, an address, an H1, or schema values. A guessed fix that ships to a live client
   site is the worst possible failure of this skill.
3. **Page 1 = domain + WHMCS ID.** The cover shows the domain as the headline and the WHMCS
   Client ID in its own chip, plus prepared date, source audit name, page count, platform,
   and the severity summary. If the WHMCS ID is unknown, render the placeholder in red and
   add the client to the "Needs WHMCS ID" list (batch mode) — do not guess it and do not
   block on it.
4. **What is wrong, what is the fix.** Each task carries exactly: an ID tag, a location, a
   one-line Issue (the defect, stated flatly), a Fix (the imperative action), and — only when
   new content is required — a paste-exactly block or a two-column content table. Cut impact
   prose, "why it matters," acceptance tests, hours, and evidence commands; those stay in the
   audit.
5. **Inline short content, flag long content.** Paste inline: sitemaps, JSON-LD, an `<h1>`, a
   `<title>`, meta descriptions, alt text, redirect lines, config snippets, robots rules,
   FAQ blocks. Do NOT inline a full page or blog rewrite — write the task as a directive and
   point to the content skill, so the sheet stays scannable. (This threshold is the default;
   honor an explicit user override.)
6. **Two outputs.** Produce the styled **HTML sheet** (from `work-order-template.html`) as the
   deliverable, and a **plain-text version** (markdown bullets in the source audit's own
   style) for pasting straight into a ticket or email. Same content, two skins.

## Severity + IDs

Map the audit's severity to four tiers and label tasks per tier, restarting the counter each
tier: `CRITICAL-1, CRITICAL-2, … HIGH-1, … MEDIUM-1, … LOW-1`.

| Work-order tier | Source audit signal |
|---|---|
| Critical | audit "critical," or anything breaking indexing/tracking/canonical identity (wrong sitemap, noindex on money pages, no schema site-wide, missing H1 on core pages) |
| High | audit "moderate" with revenue/visibility impact (title/meta problems on core pages, missing metas, broken redirects) |
| Medium | audit "minor" but real (alt text, secondary-page metadata, non-critical schema gaps) |
| Low | polish / nice-to-have |

Order the sheet Critical → High → Medium → Low. Within a tier, booking/contact flow first,
then revenue pages, then the rest — mirror the audit's own ranking when present.

## Per-task extraction

From each audit finding, pull:

- **Location** — the exact file or page scope (`/sitemap.xml`, `All 8 pages · <head>`,
  `/ and /about`, `/treatments`). Use the audit's affected-URL list; never "likely the geo set."
- **Issue** — one flat sentence: the defect only. No consequence clause.
- **Fix** — the imperative action. If content is provided below, the Fix says where it goes
  ("Add to the `<head>` of every page," "Replace the homepage `<title>`").
- **Content** — the audit's inline artifact, escaped for HTML (`<` → `&lt;`). Wrap client-
  specific values the requests team must supply (NAP, phone) in `<span class="fill">…</span>`
  so they render red, and say "replace the red values" in the Fix. If the audit gave no
  content and the fix needs it, omit the paste block and add the footnote per rule 2.

## Build steps

1. **Read the source audit** fully. Identify domain, platform, page count, audit name/date,
   and the findings list (the Engineering Queue in rg-master-audit; the ranked findings in
   rg-site-audit).
2. **Resolve the WHMCS ID** (see "WHMCS ID resolution" below): use the `whmcs_id` input if
   given; else look the domain up in the WHMCS map; else render the red placeholder
   `— (fill in)` and record it as missing.
3. **Map findings → tasks.** Tier, ID, location, issue, fix, content. Drop nothing that is a
   real defect; merge exact duplicates.
4. **Fill `work-order-template.html`:** cover tokens, one severity group per non-empty tier,
   one cloned task block per task. Emit only the paste block or content table a task needs.
5. **Emit the plain-text twin** — the same tasks as `SEVERITY-N — Title (location)` bullets
   with Issue / Fix / fenced content, matching the format the team already pastes to support.
6. **Self-check before delivering:**
   - Page 1 shows the domain and a WHMCS ID (or a flagged placeholder).
   - Every task has Issue + Fix. No task carries rationale or impact prose.
   - Every paste block traces to the source audit — nothing invented.
   - Long rewrites are directives with a handoff, not walls of text.

## Batch mode (a folder of audits → many work orders, no re-auditing)

When `source` is a folder (or a client list mapped to audit files):

1. Enumerate the audit deliverables. Log the count found. **No silent cap** — if you bound
   the run, say what you skipped.
2. For each audit, run the single-audit build. WHMCS IDs come from the supplied client→WHMCS
   map; clients absent from the map render the placeholder and join the "Needs WHMCS ID" list.
3. Write each work order to `work-orders/<client-slug>/work-order-<YYYY-MM>.html` (+ `.md`),
   never overwriting the client's audit or client-facing report.
4. Emit a **batch index** (`work-orders/index.html`): one row per client — domain, WHMCS ID
   (or "missing" chip), task counts by severity, link to the sheet. Sort by Critical count
   desc so the requests team drains the worst first. List every "Needs WHMCS ID" client at the
   top so the gap is visible, not buried.
5. Report totals: work orders written, tasks by severity, clients missing a WHMCS ID, and any
   audit that failed to parse (with the reason). Never report success for a client whose audit
   you could not read.

## WHMCS ID resolution

The WHMCS Client ID for page 1 comes from the CRM (The Roost), Supabase table
`client_profiles`: the numeric `whmcs_id` column, keyed by the client's domain in `website`
and `websites[]`. WHMCS itself lives at `rgbilling.roostergrin.com` and the ID equals the
`userid` in a client's billing URL; the CRM syncs it (`whmcs_synced_at`).

Generate a fresh `domain → whmcs_id` map at run time with this query (normalizes protocol,
`www.`, and path; drops the `-1` sentinel), then look each audited domain up in it:

```sql
with norm as (
  select whmcs_id,
    regexp_replace(regexp_replace(regexp_replace(lower(trim(coalesce(website,''))),
      '^https?://',''),'^www\.',''),'/.*$','') as domain
  from client_profiles where whmcs_id is not null and whmcs_id > 0 and coalesce(website,'')<>''
  union
  select whmcs_id,
    regexp_replace(regexp_replace(regexp_replace(lower(trim(w)),
      '^https?://',''),'^www\.',''),'/.*$','') as domain
  from client_profiles, unnest(websites) as w
  where whmcs_id is not null and whmcs_id > 0 and coalesce(w,'')<>''
)
select distinct on (domain) domain, whmcs_id
from norm where domain like '%.%' order by domain, whmcs_id;
```

Match the audited domain the same way (lowercase, strip protocol/`www.`/path). A domain not in
the map renders the placeholder and joins the "Needs WHMCS ID" list — never guess an ID.
Do not commit the materialized map to any public repo; it is client data.

## Two inputs this skill depends on

It reformats existing data, so batch runs need both to be reachable:

- **The technical audit deliverables** (rg-master-audit / rg-site-audit output). The client-
  facing Digital Evaluation Reports in `roostergrin-reports` are NOT audits and carry none of
  the technical findings — do not use them as a source.
- **The WHMCS map** above. Without CRM access every sheet renders the placeholder, which is
  valid but leaves page 1 incomplete.

If either is missing, say so plainly and produce what the available data supports. Do not
manufacture the missing input.

## Voice

Internal-blunt. Imperative. No em dashes, no "just," no praise, no filler. The requests team
should be able to execute a task without reading anything else.

---

*requests-work-order v1.0 — RoosterGrin Media internal. Reformats an existing audit into a
requests-team action sheet. Never audits, never fabricates a fix.*
