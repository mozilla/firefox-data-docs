# Search Aggregates

<!-- toc -->

# Introduction

{{#include ./intro.md}}

# Data Reference

## Example Queries

[`STMO#51140`](https://sql.telemetry.mozilla.org/queries/51140/source)
calculates daily US searches.
If you have trouble viewing this query,
it's likely you don't have the proper permissions.
For more details see the [search data documentation].

## Scheduling

This job is
[scheduled on airflow](https://github.com/mozilla/bigquery-etl/blob/ad84a15d580333b41d36cfe8331e51238f3bafa1/dags/bqetl_search.py#L40)
to run daily.

## Schema

As of 2019-11-27,
the current version of `search_aggregates` is `v8`,
and has a schema as follows.
The dataset is backfilled through 2016-03-11

```
root
 |-- submission_date: date (nullable = true)
 |-- submission_date_s3: date (nullable = true)
 |-- country: string (nullable = true)
 |-- engine: string (nullable = true)
 |-- source: string (nullable = true)
 |-- app_version: string (nullable = true)
 |-- distribution_id: string (nullable = true)
 |-- locale: string (nullable = true)
 |-- search_cohort: string (nullable = true)
 |-- addon_version: string (nullable = true)
 |-- tagged-sap: long (nullable = true)
 |-- tagged-follow-on: long (nullable = true)
 |-- sap: long (nullable = true)
 |-- organic: long (nullable = true)
 |-- search_with_ads: long (nullable = true)
 |-- ad_click: long (nullable = true)
 |-- unknown: long (nullable = true)
 |-- client_count: long (nullable = true)
 |-- default_search_engine: string (nullable = true)
 |-- default_private_search_engine: string (nullable = true)
 |-- os: string (nullable = true)
 |-- os_version: string (nullable = true)
 |-- addon_version: string (nullable = true)
```

# Code Reference

The `search_aggregates` job is
[defined in `bigquery-etl`](https://github.com/mozilla/bigquery-etl/blob/master/sql/moz-fx-data-shared-prod/search_derived/search_aggregates_v8/query.sql)

[search data documentation]: ../../search.md
