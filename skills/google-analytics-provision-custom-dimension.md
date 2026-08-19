---
name: google-analytics-provision-custom-dimension
description: Register a custom dimension on a GA4 property and report on it, respecting the Admin API's write quota and its non-idempotent creates.
api: Google Analytics Admin API v1beta + Data API v1beta
generated: '2026-08-13'
method: generated
source: openapi/google-analytics-properties-api-openapi.yml, conventions/google-analytics-conventions.yml, rate-limits/google-analytics-rate-limits.yml, data-model/google-analytics-data-model.yml
operations:
  - analyticsadmin.properties.customDimensions.list
  - analyticsadmin.properties.customDimensions.create
  - analyticsdata.properties.checkCompatibility
  - analyticsdata.properties.runReport
---

# Register a custom dimension and report on it

## Preconditions

- OAuth 2.0 token with `https://www.googleapis.com/auth/analytics.edit`. The read-only
  scope will not do — this flow writes.
- The property ID as `properties/{id}`.

## Steps

1. **Check whether it already exists.** `analyticsadmin.properties.customDimensions.list` —
   `GET https://analyticsadmin.googleapis.com/v1beta/{parent}/customDimensions`.
   Match on `parameterName`, not `displayName`: `parameterName` is the join key to event
   parameters and to the Data API dimension name. Paginate with `pageSize`/`pageToken`.

2. **Create it if absent.** `analyticsadmin.properties.customDimensions.create` —
   `POST https://analyticsadmin.googleapis.com/v1beta/{parent}/customDimensions`.
   Body: `parameterName`, `displayName`, `scope` (`EVENT` | `USER` | `ITEM`), optional
   `description` and `disallowAdsPersonalization`.

3. **Wait for data.** A custom dimension only reports on events collected *after* it was
   registered. Backfill does not happen.

4. **Confirm the pairing.** `analyticsdata.properties.checkCompatibility` with the
   dimension named `customEvent:{parameterName}` (event scope) or
   `customUser:{parameterName}` (user scope) alongside your metrics.

5. **Report.** `analyticsdata.properties.runReport` with that dimension name.

## Rules

- **Creates are not idempotent.** There is no idempotency key. A retried create makes a
  second custom dimension. Step 1 is your only guard — do the list, and do it again before
  any retry rather than resending blind.
- **The write quota is tight and returns 403.** The Admin API allows 600 writes/minute per
  project and 180 writes/minute per user, and signals exhaustion with HTTP 403, not 429 —
  which is easy to misread as a permissions failure. Quota refills every 60 seconds.
  Every create/patch/delete/archive charges both the request and the write quota.
- **Custom dimensions are capped per property** (50 event-scoped on Standard, more on
  Analytics 360). Exceeding the cap is a 400 `INVALID_ARGUMENT`.
- **Archive, do not delete.** The lifecycle operation is
  `analyticsadmin.properties.customDimensions.archive`; archived dimensions still count
  against nothing but cannot be reported on.
- **Branch on `error.status`.** 403 with `PERMISSION_DENIED` is a role problem;
  403 with `RESOURCE_EXHAUSTED` is the write quota and is retryable after 60 seconds.
