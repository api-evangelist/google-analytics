---
name: google-analytics-discover-and-run-report
description: Find which Google Analytics properties an identity can reach, confirm a dimension/metric pairing is valid, and run a GA4 report without burning quota on rejected requests.
api: Google Analytics Data API (GA4) v1beta + Admin API v1beta
generated: '2026-08-13'
method: generated
source: openapi/google-analytics-accountsummaries-api-openapi.yml, openapi/google-analytics-properties-api-openapi.yml, conventions/google-analytics-conventions.yml, rate-limits/google-analytics-rate-limits.yml, errors/google-analytics-problem-types.yml
operations:
  - analyticsadmin.accountSummaries.list
  - analyticsadmin.properties.list
  - analyticsdata.properties.checkCompatibility
  - analyticsdata.properties.runReport
---

# Discover a property and run a report

The single most common Google Analytics flow. It exists as a skill because the naive
version — guess dimensions, POST `runReport`, retry on error — is the fastest way to get
a project blocked from a property.

## Preconditions

- OAuth 2.0 access token with `https://www.googleapis.com/auth/analytics.readonly`.
- The Google Cloud project has both the Google Analytics Admin API and Data API enabled.
- The calling identity holds a role on the target property. Enabling the API is not enough;
  a 403 `PERMISSION_DENIED` on a valid token almost always means this step was skipped.

## Steps

1. **Discover what you can reach.** `analyticsadmin.accountSummaries.list` —
   `GET https://analyticsadmin.googleapis.com/v1beta/accountSummaries`.
   One call returns every account and its `propertySummaries[]`, each with `property`
   (`properties/{id}`), `displayName` and `propertyType`. Prefer this over
   `analyticsadmin.properties.list`, which requires you to already know an account.
   Paginate with `pageSize` / `pageToken` until `nextPageToken` is absent.

2. **Resolve the property ID.** Match on `displayName`. Every Data API call takes the
   property as `properties/{id}` in the path; the ID alone is accepted on the `property`
   request field only.

3. **Validate the field pairing before you spend tokens.**
   `analyticsdata.properties.checkCompatibility` —
   `POST https://analyticsdata.googleapis.com/v1beta/{property}:checkCompatibility`
   with the same `dimensions` and `metrics` you intend to report on. Incompatible
   combinations are a 400 `INVALID_ARGUMENT` on `runReport`, and every 400 charges the
   client-error quota (10,000 per project per property per 15 minutes). Checking first is
   cheaper than being wrong.

4. **Run the report.** `analyticsdata.properties.runReport` —
   `POST https://analyticsdata.googleapis.com/v1beta/{property}:runReport`.
   Body: `dateRanges[]`, `dimensions[]`, `metrics[]`, optional `dimensionFilter`,
   `metricFilter`, `orderBys[]`, `limit`, `offset`, `currencyCode`.
   Set `"returnPropertyQuota": true` — it is the only way to see what the request cost and
   what is left.

5. **Page through results.** Reporting is offset-based, not cursor-based. Read `rowCount`
   from the response and re-issue with an increased `offset` until you have `rowCount`
   rows. Default `limit` is 10,000; maximum is 250,000.

## Rules

- **No idempotency.** There is no idempotency key on any Google Analytics API. Reads are
  naturally safe to repeat, but do not build a retry loop that assumes the API will
  de-duplicate anything.
- **Quota is token-based and unpredictable.** Cost scales with rows, dimension count,
  filter complexity, date range and dimension cardinality. Standard properties get 200,000
  Core tokens/day and 40,000/hour; Analytics 360 gets 10x. Realtime and Funnel are separate
  buckets and do not draw down Core.
- **There are no rate-limit headers.** Nothing in the response tells you you are close to
  the ceiling unless you asked for `returnPropertyQuota`.
- **Bound your retries.** 500/503 charge a server-error quota of 10 per hour on Standard;
  exhausting it blocks *all* requests from your project to that property. Exponential
  backoff with a hard retry cap, always.
- **Branch on `error.status`, not the HTTP code.** The envelope is
  `{"error": {"code", "message", "status"}}` — Google canonical, not RFC 9457.
  429 `RESOURCE_EXHAUSTED` is retryable after backoff; 400 `INVALID_ARGUMENT` and
  403 `PERMISSION_DENIED` are not.
- **Thresholding is silent.** Reports using `userAgeBracket`, `userGender`,
  `brandingInterest`, `audienceId` or `audienceName` may be withheld to protect individual
  users, and are capped at 120 requests/hour.
