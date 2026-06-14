# Google Analytics GraphQL Schema

This directory contains a conceptual GraphQL schema for the Google Analytics 4 (GA4) APIs. The schema is derived from the [Google Analytics Data API v1](https://developers.google.com/analytics/devguides/reporting/data/v1) and the [Google Analytics Admin API v1](https://developers.google.com/analytics/devguides/config/admin/v1), translating their REST and RPC surface into idiomatic GraphQL types, queries, and mutations.

## Source APIs

| API | Base URL | Purpose |
|-----|----------|---------|
| GA4 Data API v1 | `https://analyticsdata.googleapis.com` | Run standard, realtime, pivot, funnel, and batch reports |
| GA4 Admin API v1 | `https://analyticsadmin.googleapis.com` | Manage accounts, properties, streams, and configuration |
| Measurement Protocol | `https://www.google-analytics.com/mp/collect` | Send server-side events directly to GA4 |

## Schema File

[google-analytics-schema.graphql](google-analytics-schema.graphql)

## Type Coverage

The schema defines **68 named GraphQL types** organized across the following domains:

### Scalars
- `DateTime` — ISO 8601 timestamps
- `JSON` — arbitrary JSON payloads (used for compatibility checks)

### Enums (16)
`MetricAggregation`, `MetricType`, `StringFilterMatchType`, `NumericFilterOperation`, `OrderByDirection`, `PropertyType`, `IndustryCategory`, `DataStreamType`, `CustomDimensionScope`, `CustomMetricMeasurementUnit`, `CustomMetricScope`, `AudienceFilterScope`, `ConversionCountingMethod`, `ReportingAttributionModel`, `SegmentFilterScope`

### Account & Property Management
`Account`, `AccountSummary`, `PropertySummary`, `Property`

### Data Streams
`DataStream`, `WebDataStream`, `AppDataStream`

### Configuration Resources
`MeasurementProtocolSecret`, `FirebaseLink`, `GoogleAdsLink`, `BigQueryLink`, `Attribution`, `ConversionEvent`, `KeyEvent`, `DefaultConversionValue`, `CustomDimension`, `CustomMetric`, `EventCreateRule`, `MatchingCondition`, `ParameterMutation`

### Reporting Primitives
`Dimension`, `Metric`, `DateRange`, `DimensionExpression`, `CaseExpression`, `ConcatenateExpression`, `DimensionValue`, `MetricValue`, `Row`, `DimensionHeader`, `MetricHeader`

### Pivot & Ordering
`Pivot`, `OrderBy`, `MetricOrderBy`, `DimensionOrderBy`, `PivotOrderBy`, `PivotSelection`

### Filters
`FilterExpression`, `FilterExpressionList`, `Filter`, `StringFilter`, `InListFilter`, `NumericFilter`, `BetweenFilter`, `NumericValue`

### Report Requests & Responses
`ReportRequest`, `ReportResponse`, `ResponseMetaData`, `SchemaRestrictionResponse`, `ActiveMetricRestriction`, `SamplingMetadata`, `BatchRunReport`, `BatchRunReportResponse`, `RunRealtimeReport`, `MinuteRange`, `RunRealtimeReportResponse`

### Funnel Reports
`Funnel`, `FunnelStep`, `FunnelFilterExpression`, `FunnelFilterExpressionList`, `FunnelFilter`, `FunnelEventFilter`, `FunnelResponseMetadata`, `FunnelSubFilter`

### Audience Types
`AudienceDefinition`, `Audience`, `AudienceEventTrigger`, `AudienceFilterClause`, `AudienceSimpleFilter`, `AudienceSequenceFilter`, `AudienceSequenceStep`, `AudienceFilterExpression`, `AudienceFilterExpressionList`, `AudienceDimensionOrMetricFilter`, `AudienceEventFilter`, `AudienceFilter`

### Segment Types
`Segment`, `SegmentFilter`

### Cohort Spec
`CohortSpec`, `Cohort`, `CohortsRange`, `CohortReportSettings`

### Quota & Usage
`PropertyQuota`, `QuotaStatus`, `ReportingQuota`, `ActiveUserCount`, `EventCount`

### Users & Access
`UserRow`, `RunAccessReport`, `AccessReportEntity`, `AccessDimension`, `AccessMetric`, `AccessFilterExpression`, `AccessFilterExpressionList`, `AccessFilter`, `AccessOrderBy`, `AccessMetricOrderBy`, `AccessDimensionOrderBy`

### Authentication
`APIKey`, `Token`

## Query Operations

The `Query` type exposes read operations across all GA4 resource domains:

- **Account & Property**: `listAccounts`, `getAccount`, `listAccountSummaries`, `listProperties`, `getProperty`
- **Data Streams**: `listDataStreams`, `getDataStream`
- **Configuration**: `listMeasurementProtocolSecrets`, `getMeasurementProtocolSecret`, `listFirebaseLinks`, `listGoogleAdsLinks`, `listBigQueryLinks`, `getAttribution`
- **Events & Conversions**: `listConversionEvents`, `getConversionEvent`, `listKeyEvents`, `getKeyEvent`
- **Custom Definitions**: `listCustomDimensions`, `getCustomDimension`, `listCustomMetrics`, `getCustomMetric`
- **Audiences**: `listAudiences`, `getAudience`, `listAudienceExports`, `queryAudienceExport`
- **Reporting**: `runReport`, `runPivotReport`, `batchRunReports`, `runRealtimeReport`, `runFunnelReport`, `runAccessReport`, `checkCompatibility`
- **Quota**: `getPropertyQuota`

## Mutation Operations

The `Mutation` type covers all write operations:

- **Property Management**: `createProperty`, `updateProperty`, `deleteProperty`
- **Data Streams**: `createDataStream`, `updateDataStream`, `deleteDataStream`
- **Measurement Protocol Secrets**: `createMeasurementProtocolSecret`, `updateMeasurementProtocolSecret`, `deleteMeasurementProtocolSecret`
- **Links**: `createFirebaseLink`, `deleteFirebaseLink`, `createGoogleAdsLink`, `updateGoogleAdsLink`, `deleteGoogleAdsLink`
- **Conversion Events & Key Events**: `createConversionEvent`, `updateConversionEvent`, `deleteConversionEvent`, `createKeyEvent`, `updateKeyEvent`, `deleteKeyEvent`
- **Custom Dimensions & Metrics**: `createCustomDimension`, `updateCustomDimension`, `archiveCustomDimension`, `createCustomMetric`, `updateCustomMetric`, `archiveCustomMetric`
- **Audiences**: `createAudience`, `updateAudience`, `archiveAudience`, `createAudienceExport`
- **Event Creation Rules**: `createEventCreateRule`, `updateEventCreateRule`, `deleteEventCreateRule`

## Authentication

All Google Analytics API requests require OAuth 2.0 authentication. The schema includes `APIKey` and `Token` types to represent credential artifacts.

Required OAuth 2.0 scopes:

| Scope | Access Level |
|-------|-------------|
| `https://www.googleapis.com/auth/analytics.readonly` | Read-only access to GA4 report data |
| `https://www.googleapis.com/auth/analytics` | Read and write access |
| `https://www.googleapis.com/auth/analytics.edit` | Edit management entities |
| `https://www.googleapis.com/auth/analytics.manage.users` | Manage user permissions |

## References

- [GA4 Data API Reference](https://developers.google.com/analytics/devguides/reporting/data/v1/rest)
- [GA4 Admin API Reference](https://developers.google.com/analytics/devguides/config/admin/v1/rest)
- [Google Analytics GitHub Organization](https://github.com/googleanalytics)
- [googleapis/google-analytics-data (GitHub)](https://github.com/googleapis/google-analytics-data)
- [GA4 Quotas and Limits](https://developers.google.com/analytics/devguides/reporting/data/v1/quotas)
