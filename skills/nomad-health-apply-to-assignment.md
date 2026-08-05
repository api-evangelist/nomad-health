---
name: Apply to a Nomad Health assignment
description: >-
  Walk a signed-in clinician from a job post through requirements, application data and
  submission, then track application status. Requires a human-established session — read the
  cautions before automating any step.
api: openapi/nomad-health-platform-openapi.yml
operations:
  - get_job_post_requirements_api
  - get_application_form_schemas_list_resource
  - get_application_sections_list_resource
  - post_application_data_list_resource
  - get_application_validation_resource
  - post_application_apply_resource
  - get_quick_apply_applications_resource
  - get_job_post_application
generated: '2026-08-04'
method: generated
---

# Apply to a Nomad Health assignment

Base URL: `https://nomadhealth.com/api/v1`

## Preconditions and cautions

- **Session-only.** Every operation here returns
  `401 {"code": "a0002", "error": "user is not authenticated"}` without a valid Nomad Health
  session cookie. There is no API key, token or OAuth flow to obtain one programmatically.
  An agent can only run this flow inside an authenticated user's session.
- **No idempotency contract.** Nomad Health declares no `Idempotency-Key` header anywhere in
  its 498 published operations. `post_application_apply_resource` is a consequential write
  with no replay protection — **never retry it automatically.** On a timeout or 5xx, call
  `get_job_post_application` for that `jobpost_id` to check whether the application already
  landed before doing anything else.
- **No schemas.** The published Swagger contract declares zero definitions and no request
  bodies, so the payload shape for every write below must be read at runtime from
  `get_application_form_schemas_list_resource` and `get_application_sections_list_resource`.
  Do not hard-code a body.
- This flow submits a real job application on behalf of a real clinician. Treat it as
  human-in-the-loop: confirm the job post with the user before step 5.

## 1. Read the job's requirements

`get_job_post_requirements_api` — `GET /api/v1/jobposts/{jobpost_id}/requirements/`

Tells you what the facility demands (licenses, certifications, experience) before you build
an application.

## 2. Check whether an application already exists

`get_job_post_application` — `GET /api/v1/applications/jobpost/{jobpost_id}`

Run this first and again after any failed submit. It is the duplicate guard the API does not
give you.

## 3. Fetch the form contract

- `get_application_form_schemas_list_resource` — `GET /api/v1/application/form-schemas/`
- `get_application_sections_list_resource` — `GET /api/v1/application/sections/`

These return the live field definitions. Build the payload from them.

## 4. Write application data

`post_application_data_list_resource` — `POST /api/v1/application/data/`

Then verify with `get_application_validation_resource` —
`GET /api/v1/application/validation/`. Do not proceed to submit while validation reports
outstanding items.

## 5. Submit

`post_application_apply_resource` — `POST /api/v1/application/apply/`

Confirm with the user first. On any non-2xx, go back to step 2 rather than retrying.

> A second, older submit path exists on the sibling surface —
> `post_submit_application_api` at `POST /api/apply/{jobpost_id}/submit/`. Pick one. Calling
> both is how you create a duplicate.

## 6. Track

`get_quick_apply_applications_resource` — `GET /api/v1/quick-apply/applications/` lists the
clinician's applications. `get_application_resource` —
`GET /api/v1/applications/{id}/` reads one.

## Errors

See `errors/nomad-health-problem-types.yml`. Note that a `404` here returns HTML, and a `403`
returns a bare JSON `null` with no code — neither is safe to parse as an error object.
