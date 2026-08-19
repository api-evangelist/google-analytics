---
name: google-analytics-send-measurement-protocol-event
description: Send a server-side GA4 event through the Measurement Protocol safely — validate it first, because the production endpoint reports no errors and retries duplicate data.
api: Measurement Protocol (GA4)
generated: '2026-08-13'
method: generated
source: openapi/google-analytics-events-api-openapi.yml, openapi/google-analytics-validation-api-openapi.yml, openapi/google-analytics-properties-api-openapi.yml, sandbox/google-analytics-sandbox.yml, conventions/google-analytics-conventions.yml
operations:
  - analyticsadmin.properties.dataStreams.list
  - analyticsadmin.properties.dataStreams.measurementProtocolSecrets.list
  - analyticsadmin.properties.dataStreams.measurementProtocolSecrets.create
  - validateEvents
  - collectEvents
---

# Send a GA4 event server-side

The Measurement Protocol is the one Google Analytics surface with no safety net: it returns
2xx with an empty body whether the payload was perfect or garbage. Every guard has to be
built by the caller.

## Preconditions

- A GA4 data stream and its identifier: `measurement_id` (web streams) or
  `firebase_app_id` (app streams).
- An `api_secret` for that stream.
- A device identifier: `client_id` for web streams, `app_instance_id` for app streams.

## Steps

1. **Find the data stream.** `analyticsadmin.properties.dataStreams.list` —
   `GET https://analyticsadmin.googleapis.com/v1beta/{parent}/dataStreams`.
   Read `webStreamData.measurementId` or `androidAppStreamData.firebaseAppId` /
   `iosAppStreamData.firebaseAppId` off the stream you want.

2. **Get or mint the secret.**
   `analyticsadmin.properties.dataStreams.measurementProtocolSecrets.list` to see existing
   secrets, `...measurementProtocolSecrets.create` to mint one. The secret is returned as
   `secretValue`. Requires `https://www.googleapis.com/auth/analytics.edit`.

3. **Validate the event.** `validateEvents` —
   `POST https://www.google-analytics.com/debug/mp/collect?measurement_id=...&api_secret=...`
   with the exact body you intend to send, plus `"validation_behavior":
   "ENFORCE_RECOMMENDATIONS"`. The response carries `validationMessages[]` with
   `fieldPath`, `description` and `validationCode`. Events sent here never reach reports.

4. **Send the event.** `collectEvents` —
   `POST https://www.google-analytics.com/mp/collect?measurement_id=...&api_secret=...`.
   Drop `validation_behavior` in production so less data is rejected. The regional
   endpoint `https://region1.google-analytics.com/mp/collect` accepts the same payload.

5. **Verify in DebugView or with a Data API realtime report.** There is no acknowledgement
   in the response, so confirmation has to come from the other side.

## Rules

- **The response is meaningless.** No error codes, no body, ever — even for a malformed
  event with a missing required parameter. Treat a 2xx as "the bytes arrived", nothing more.
  Step 3 is not optional.
- **Retries duplicate.** There is no idempotency key and no de-duplication. A network
  timeout followed by a retry produces two events in the property. If your transport
  retries automatically, turn it off here and handle failure by logging, not resending.
- **Structural limits are hard.** 25 events per request; 25 parameters per event; 25 user
  properties per event; event and parameter names ≤ 40 characters, alphanumeric plus
  underscore, starting with a letter; parameter values ≤ 100 characters; user property
  names ≤ 24 and values ≤ 36 characters. Over the limit, the event is dropped silently.
- **Backdating is capped at 72 hours** via `timestamp_micros`. Older events are rejected.
- **The hourly ceiling is silent too.** 100 million non-conversion requests per property
  per hour; past it, non-conversion requests are ignored for the rest of the hour with no
  signal. Analytics 360 properties are exempt.
- **The secret is a bearer credential in a query string.** It grants write-only ingestion —
  anyone holding it can inject arbitrary events and corrupt the property's reporting.
  Server-side only. Rotate by creating a new secret and deleting the old one; there is no
  expiry and no automatic rotation.
- **Ingestion is not read-your-writes.** An event sent now is not immediately queryable
  through the Data API.
