---
name: google-analytics-audit-property-configuration
description: Read a GA4 property's whole configuration surface — streams, custom definitions, links, retention and the change history behind it — using only the read-only scope.
api: Google Analytics Admin API v1beta
generated: '2026-08-13'
method: generated
source: openapi/google-analytics-accounts-api-openapi.yml, openapi/google-analytics-accountsummaries-api-openapi.yml, openapi/google-analytics-properties-api-openapi.yml, data-model/google-analytics-data-model.yml, rate-limits/google-analytics-rate-limits.yml
operations:
  - analyticsadmin.accountSummaries.list
  - analyticsadmin.properties.list
  - analyticsadmin.properties.dataStreams.list
  - analyticsadmin.properties.customDimensions.list
  - analyticsadmin.properties.customMetrics.list
  - analyticsadmin.properties.conversionEvents.list
  - analyticsadmin.properties.googleAdsLinks.list
  - analyticsadmin.properties.firebaseLinks.list
  - analyticsadmin.properties.runAccessReport
  - analyticsadmin.accounts.searchChangeHistoryEvents
---

# Audit a property's configuration

A read-only sweep that produces the full picture of how a GA4 property is set up, and who
changed it. Nothing here writes.

## Preconditions

- OAuth 2.0 token with `https://www.googleapis.com/auth/analytics.readonly`.

## Steps

1. **Enumerate reachable properties.** `analyticsadmin.accountSummaries.list`. One call,
   every account plus its `propertySummaries[]`.

2. **Collection surface.** `analyticsadmin.properties.dataStreams.list` for each property.
   Record stream `type`, `displayName` and the measurement/Firebase identifier. A property
   with no stream collects nothing.

3. **Custom definitions.** `analyticsadmin.properties.customDimensions.list` and
   `analyticsadmin.properties.customMetrics.list`. Record `parameterName` and `scope` —
   these are what determine which `customEvent:` / `customUser:` dimensions the Data API
   will accept.

4. **Key events.** `analyticsadmin.properties.conversionEvents.list`. Note the resource is
   the deprecated name: `KeyEvent` superseded `ConversionEvent` in 2024, but the v1beta
   `conversionEvents` collection is what the captured contract exposes.

5. **Integrations.** `analyticsadmin.properties.googleAdsLinks.list` and
   `analyticsadmin.properties.firebaseLinks.list`.

6. **Retention.** `GET .../v1beta/{property}/dataRetentionSettings` — a singleton carrying
   `eventDataRetention`, `userDataRetention` and `resetUserDataOnNewActivity`.

7. **Who touched what.** `analyticsadmin.accounts.searchChangeHistoryEvents` —
   `POST https://analyticsadmin.googleapis.com/v1beta/{account}:searchChangeHistoryEvents`.
   Scoped to the ACCOUNT, not the property; filter by `property`, `resourceType`,
   `actorEmail` and a date range.

8. **Who read what.** `analyticsadmin.properties.runAccessReport` —
   `POST .../v1beta/{entity}:runAccessReport`. Data-access records, not configuration
   changes. Charges Core token quota on the Data API side, not the Admin request quota.

## Rules

- **Read-only scope is sufficient for all of it.** If any of these calls returns 403, it is
  a property-role problem or the API is not enabled on the project — not a scope problem.
- **Paginate everything.** `pageSize` is advisory: "the service may return fewer than this
  value, even if there are additional pages". Loop until `nextPageToken` is absent, and
  keep every other parameter identical across pages or the token is rejected.
- **Mind the Admin request quota.** 1,200 requests/minute per project, 600 per user, and
  a per-property audit is roughly 8 calls plus pagination. Auditing hundreds of properties
  in a loop will hit it, and it returns HTTP 403 — not 429.
- **`runAccessReport` is a different quota.** It charges the Data API Core token bucket.
  Do not batch it alongside heavy reporting.
- **Change history is account-scoped and time-limited.** Search from the account resource,
  and do not expect unbounded history.
