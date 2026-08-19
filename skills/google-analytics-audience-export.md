---
name: google-analytics-audience-export
description: Export the users in a GA4 audience through the Data API's only asynchronous resource — create, poll, then query.
api: Google Analytics Data API (GA4) v1beta
generated: '2026-08-13'
method: generated
source: openapi/google-analytics-properties-api-openapi.yml, openapi/_original/google-analytics-data-api.yaml, data-model/google-analytics-data-model.yml, rate-limits/google-analytics-rate-limits.yml
operations:
  - analyticsdata.properties.audienceExports.create
  - analyticsdata.properties.audienceExports.list
  - analyticsdata.properties.audienceExports.query
---

# Export an audience

Every other Data API call is synchronous. Audience exports are not: creation starts a
long-running job and the rows are only readable once it reports `ACTIVE`.

## Preconditions

- OAuth 2.0 token with `https://www.googleapis.com/auth/analytics.readonly`.
- An existing audience on the property. Audiences are created in the Google Analytics UI or
  through the Admin API v1alpha — not by this flow.

## Steps

1. **Find the audience.** Audience resource names are `properties/{p}/audiences/{a}`.
   List existing exports first with `analyticsdata.properties.audienceExports.list` —
   `GET https://analyticsdata.googleapis.com/v1beta/{parent}/audienceExports` — because a
   recent export may already be reusable and creating a second one spends quota for nothing.

2. **Create the export.** `analyticsdata.properties.audienceExports.create` —
   `POST https://analyticsdata.googleapis.com/v1beta/{parent}/audienceExports`
   with `audience` and the `dimensions[]` you want on each row. The response is a
   long-running operation naming the export resource. This charges Core quota.

3. **Poll until ready.** `GET https://analyticsdata.googleapis.com/v1beta/{name}` until
   `state` is `ACTIVE`. `CREATING` means keep waiting. Poll with backoff — every poll is a
   request against the concurrency ceiling (10 concurrent on Standard, 50 on 360).

4. **Query the rows.** `analyticsdata.properties.audienceExports.query` —
   `POST https://analyticsdata.googleapis.com/v1beta/{name}:query` with `offset` and
   `limit`. Read `rowCount` from the response and walk the offsets.

## Rules

- **Creation is not idempotent.** Two identical creates make two export jobs and charge
  quota twice. Always list first (step 1) and reuse an existing `ACTIVE` export when one
  covers your need.
- **The export is a snapshot.** It reflects audience membership at creation time. It does
  not update. Re-export to refresh.
- **Poll politely.** Concurrency is a quota, and a tight poll loop against a slow export
  will exhaust it and start returning 429 `RESOURCE_EXHAUSTED` on unrelated reports.
- **Set `returnPropertyQuota: true`** on the create so you can see what the job cost.
- **Users, not events.** Audience export rows are user-scoped; joining them to event-scoped
  reporting requires a user-scoped dimension on both sides.
