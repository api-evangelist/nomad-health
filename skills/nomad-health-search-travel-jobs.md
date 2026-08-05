---
name: Search Nomad Health travel healthcare jobs
description: >-
  Search the public Nomad Health marketplace for travel nursing and allied-health
  assignments, read the faceted result envelope, and page through matches. This is the only
  Nomad Health flow an agent can run without a human session.
api: openapi/nomad-health-platform-openapi.yml
operations:
  - get_public_jobpost_search
  - get_discipline_names_list
  - get_sitemap_job_chunk_count
  - get_sitemap_job_chunk
generated: '2026-08-04'
method: generated
---

# Search Nomad Health travel healthcare jobs

Base URL: `https://nomadhealth.com/api/v1`

## Before you start

- **No credential exists.** Nomad Health issues no API key, no bearer token and no OAuth
  client. Everything else on this API requires a browser session cookie from
  `https://nomadhealth.com/sign-in`. The operations in this skill are the only ones that
  answer anonymously — verified 2026-08-04.
- Responses are `application/json`. A `404` on this host returns **HTML** from the marketing
  application, not a JSON error — always check the status code before parsing.
- Every response carries an `x-request-id` header. Log it; it is the only correlation handle
  Nomad Health exposes.
- There are **no rate-limit headers**. A limit may still be enforced. Back off on any
  non-200 rather than retrying tightly.

## 1. Get the discipline vocabulary

Call `get_discipline_names_list` — `GET /api/v1/discipline-names/`.

Returns a `data[]` array where each element is `{"data": {"name", "label"}, "links": {"self"}}`.
Use the `name` value (e.g. `nurse`, `radiology_technologist`, `surgical_technologist`) as the
`discipline` filter in step 2. Do not guess discipline slugs — read them from this call.

## 2. Search the marketplace

Call `get_public_jobpost_search` — `GET /api/v1/jobposts/public_jobpost_search/`.

The published Swagger contract declares **zero query parameters** for this operation. The
filter vocabulary below comes from Nomad Health's own `llms.txt` and is the same set the
human search UI uses:

`discipline`, `specializations`, `jobType` (`travel_and_local` | `per_diem`), `locations`,
`compactState`, `startDate`, `shiftHoursAndDays` (`2x12` | `3x12` | `4x10` | `4x12` | `5x8`),
`shiftTypes` (`days` | `mids` | `evenings` | `nights` | `rotating`), `contractLength`
(`under8` | `8-12` | `13+`), `minPayRateWeekly`, `autoOffer`, `exclusive`, `certifications`,
`allowsNonCertified`.

Because the contract does not declare these, treat any filter as best-effort: compare
`results.count_no_filters` against `results.matches.pagination.total_items` to confirm a
filter actually narrowed the set.

## 3. Read the response envelope

```
results.count_no_filters      total before filters
results.facets                min_pay_rate, max_pay_rate, specializations{}, certifications{}
results.matches.items[]       the job posts
results.matches.pagination    page, pages, per_page, total_items, has_next, has_previous,
                              start_item, end_item
```

Each item carries `id`, `code`, `title`, `discipline`, `specialization_names`,
`facility_city`, `facility_state`, `pay_rate`, `weekly_gross_compensation`,
`contract_length`, `shift_types`, `shift_hours_and_days`, `start_date`, `job_type`,
`applications_count`, `submissions_count` and `source`.

## 4. Page

Pagination is page-number style: pass `page` and `per_page`, and continue while
`results.matches.pagination.has_next` is true. `total_items` caps at 10,000 — do not treat it
as an exact corpus size.

## 5. Enumerate the whole public corpus instead

For a full crawl rather than a query, use the sitemap operations:

1. `get_sitemap_job_chunk_count` — `GET /api/v1/sitemap/jobs/chunk_count/` returns
   `{"chunk_count": n}`.
2. `get_sitemap_job_chunk` — `GET /api/v1/sitemap/jobs/chunk/?chunk=<i>` for each `i` in
   `0..n-1`, returning `{"jobs": [{"last_modified", "url"}]}`.

Respect `https://nomadhealth.com/robots.txt`, which disallows `/jobs/search`.

## Errors

| Status | Body | Meaning |
|---|---|---|
| 401 | `{"code": "a0002", "error": "user is not authenticated"}` | You called a session-bound operation. Nothing in this skill should return this. |
| 405 | `{"message": "The method is not allowed for the requested URL."}` | Wrong HTTP method — all four operations here are `GET`. |
| 404 | HTML | Path does not exist on the API; you have fallen through to the marketing app. |

Full catalog: `errors/nomad-health-problem-types.yml`. Conventions:
`conventions/nomad-health-conventions.yml`.
