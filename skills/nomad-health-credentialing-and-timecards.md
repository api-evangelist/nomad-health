---
name: Manage Nomad Health credentialing and timecards
description: >-
  Browse a clinician's credentialing requirements and categories, complete outstanding
  requirements, and record and submit timecards against an active placement. Session-only.
api: openapi/nomad-health-platform-openapi.yml
operations:
  - get_credential_browse
  - get_get_category_definition
  - get_get_valid_categories_by_scope
  - get_category_schema_detail
  - put_requirements_complete
  - post_requirement_application_question_complete
  - post_reset_requirement
  - get_timecards_by_placement_browse_current
  - get_timecard_shift_types
  - post_timecard_update
  - put_timecard_complete
  - put_timecard_submit
  - get_placement_list
generated: '2026-08-04'
method: generated
---

# Manage Nomad Health credentialing and timecards

Base URL: `https://nomadhealth.com/api/v1`

## Preconditions and cautions

- **Session-only.** All operations return `401 {"code": "a0002", ...}` without a session
  cookie. No machine credential exists.
- **This surface touches licensure and pay.** Credentialing records carry clinician
  licensure and identity documents; timecards drive compensation. Treat every write here as
  human-confirmed. `put_timecard_submit` in particular should never be fired autonomously.
- **No idempotency and no declared schemas.** Read category and requirement schemas at
  runtime; never retry a write blindly. See
  `conventions/nomad-health-conventions.yml`.

## Credentialing

1. `get_get_valid_categories_by_scope` — `GET /api/v1/credentialing/categories/scope/`:
   which credential categories apply.
2. `get_get_category_definition` — `GET /api/v1/credentialing/categories/definition/` and
   `get_category_schema_detail` — `GET /api/v1/credentialing/categories/schema/`: the field
   contract for a category. Build payloads from these, not from assumption.
3. `get_credential_browse` — `GET /api/v1/credentialing/credentials/`: the clinician's
   current credential records and what is outstanding.
4. Complete a requirement:
   - `put_requirements_complete` — `PUT /api/v1/credentialing/requirements/complete/`
   - `post_requirement_application_question_complete` —
     `POST /api/v1/credentialing/requirements/complete-application-question/`
5. `post_reset_requirement` — `POST /api/v1/credentialing/requirements/reset/` reverses a
   completion. Destructive; confirm with the user.

## Timecards

1. `get_placement_list` — `GET /api/v1/placements/` to find the active placement.
2. `get_timecards_by_placement_browse_current` —
   `GET /api/v1/credentialing/timecards/browse/current/`: the current timecard.
3. `get_timecard_shift_types` — `GET /api/v1/credentialing/timecards/shifts/shift_types/`:
   the valid shift-type vocabulary. Use these values verbatim.
4. `post_timecard_update` — `POST /api/v1/credentialing/timecards/update/`: record hours.
5. `put_timecard_complete` — `PUT /api/v1/credentialing/timecards/complete/`: mark complete.
6. `put_timecard_submit` — `PUT /api/v1/credentialing/timecards/submit/`: **final, pay-affecting
   submission. Require explicit human confirmation. Do not retry on error — re-read with
   `get_timecards_by_placement_browse_current` first.**

## Errors

`errors/nomad-health-problem-types.yml`. Remember the `404`-returns-HTML and
`403`-returns-`null` behaviours before parsing any failure body.
