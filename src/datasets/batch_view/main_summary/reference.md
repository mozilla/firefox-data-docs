# Main Summary

> **⚠** Since the introduction of BigQuery, we are able to represent the
> full `main` ping structure in a table, available as `telemetry.main`.
> New analyses should avoid `main_summary`, which exists only for compatibility.

The `main_summary` table contains one row for each ping.
Each column represents one field from the main ping payload,
though only a subset of all main ping fields are included.
This dataset **does not include most histograms**.

This table is massive, and due to its size, it can be difficult to work with.

Instead, we recommend using the `clients_daily` or `clients_last_seen` dataset
where possible.

If you do need to query this table, make use of the `sample_id` field and
limit to a short submission date range.

## Table of Contents

<!-- toc -->

## Accessing the Data

The `main_summary` table is accessible through STMO.
See [`STMO#4201`](https://sql.telemetry.mozilla.org/queries/4201/source) for an example.

## Data Reference

### Example Queries

Compare the search volume for different search source values:

```sql
WITH search_data AS (
  SELECT
    s.source AS search_source,
    s.count AS search_count
  FROM
    telemetry.main_summary
    CROSS JOIN UNNEST(search_counts) AS s
  WHERE
    submission_date_s3 = '2019-11-11'
    AND sample_id = 42
    AND search_counts IS NOT NULL
)

SELECT
  search_source,
  sum(search_count) as total_searches
FROM search_data
GROUP BY search_source
ORDER BY sum(search_count) DESC
```

### Sampling

The `main_summary` dataset contains one record for each `main` ping
as long as the record contains a non-null value for
`documentId`, `submissionDate`, and `Timestamp`.
We do not ever expect nulls for these fields.

### Scheduling

This dataset is updated daily via the [telemetry-airflow](https://github.com/mozilla/telemetry-airflow) infrastructure.
The DAG is defined in
[`dags/bqetl_main_summary.py`](https://github.com/mozilla/bigquery-etl/blob/master/dags/bqetl_main_summary.py)

### Schema

As of 2019-11-28, the current version of the `main_summary` dataset is `v4`.

For more detail on where these fields come from in the
[raw data](https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/data/main-ping.html),
please look [in the `main_summary` ETL code][main_summary_code].

Most of the fields are simple scalar values, with a few notable exceptions:

- The `search_count` field is an array of structs, each item in the array representing
  a 3-tuple of (`engine`, `source`, `count`). The `engine` field represents the name of
  the search engine against which the searches were done. The `source` field represents
  the part of the Firefox UI that was used to perform the search. It contains values
  such as `abouthome`, `urlbar`, and `searchbar`. The `count` field contains the number
  of searches performed against this engine+source combination during that subsession.
  Any of the fields in the struct may be null (for example if the search key did not
  match the expected pattern, or if the count was non-numeric).
- The `loop_activity_counter` field is a simple struct containing inner fields for each
  expected value of the `LOOP_ACTIVITY_COUNTER` Enumerated Histogram. Each inner field
  is a count for that histogram bucket.
- The `popup_notification_stats` field is a map of `String` keys to struct values,
  each field in the struct being a count for the expected values of the
  `POPUP_NOTIFICATION_STATS` Keyed Enumerated Histogram.
- The `places_bookmarks_count` and `places_pages_count` fields contain the **mean**
  value of the corresponding Histogram, which can be interpreted as the average number
  of bookmarks or pages in a given subsession.
- The `active_addons` field contains an array of structs, one for each entry in
  the `environment.addons.activeAddons` section of the payload. More detail in
  [Bug 1290181](https://bugzilla.mozilla.org/show_bug.cgi?id=1290181).
- The `disabled_addons_ids` field contains an array of strings, one for each entry in
  the `payload.addonDetails` which is not already reported in the `environment.addons.activeAddons`
  section of the payload. More detail in
  [Bug 1390814](https://bugzilla.mozilla.org/show_bug.cgi?id=1390814).
  Please note that while using this field is generally OK, this was introduced to support
  the [TAAR](https://github.com/mozilla/taar/pulls) project and you should not count on it
  in the future. The field can stay in the `main_summary`, but we might need to slightly change
  the ping structure to something better than `payload.addonDetails`.
- The `theme` field contains a single struct in the same shape as the items in the
  `active_addons` array. It contains information about the currently active browser
  theme.
- The `user_prefs` field contains a struct with values for preferences of interest.
- The `events` field contains an array of event structs.
- Dynamically-included histogram fields are present as key->value maps,
  or key->(key->value) nested maps for keyed histograms.

### Time formats

Columns in `main_summary` may use one of a handful of time formats with different precisions:

| Column Name              | Origin                                   | Description                                  | Example                         | Spark                                                                   | Presto                                                                       |
| ------------------------ | ---------------------------------------- | -------------------------------------------- | ------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `timestamp`              | stamped at ingestion                     | nanoseconds since epoch                      | `1504689165972861952`           | `from_unixtime(timestamp/1e9)`                                          | `from_unixtime(timestamp/1e9)`                                               |
| `submission_date_s3`     | derived from timestamp                   | `YYYYMMDD` date string of timestamp in UTC   | `20170906`                      | `from_unixtime(unix_timestamp(submission_date, 'yyyyMMdd'))`            | `date_parse(submission_date, '%Y%m%d')`                                      |
| `client_submission_date` | derived from HTTP header: `Fields[Date]` | HTTP date header string sent with the ping   | `Tue, 27 Sep 2016 16:28:23 GMT` | `unix_timestamp(client_submission_date, 'EEE, dd M yyyy HH:mm:ss zzz')` | `date_parse(substr(client_submission_date, 1, 25), '%a, %d %b %Y %H:%i:%s')` |
| `creation_date`          | `creationDate`                           | time of ping creation ISO8601 at UTC+0       | `2017-09-06T08:21:36.002Z`      | `to_timestamp(creation_date, "yyyy-MM-dd'T'HH:mm:ss.SSSXXX")`           | `from_iso8601_timestamp(creation_date) AT TIME ZONE 'GMT'`                   |
| `timezone_offset`        | `info.timezoneOffset`                    | timezone offset in minutes                   | `120`                           |                                                                         |
| `subsession_start_date`  | `info.subsessionStartDate`               | hourly precision, ISO8601 date in local time | `2017-09-06T00:00:00.0+02:00`   |                                                                         | `from_iso8601_timestamp(subsession_start_date) AT TIME ZONE 'GMT'`           |
| `subsession_length`      | `info.subsessionLength`                  | subsession length in seconds                 | `599`                           |                                                                         | `date_add('second', subsession_length, subsession_start_date)`               |
| `profile_creation_date`  | `environment.profile.creationDate`       | days since epoch                             | `15,755`                        |                                                                         | `from_unixtime(profile_creation_date * 86400)`                               |

### User Preferences

These are added in the [Main Summary ETL code][user_pref_code].
They must be available in the [ping environment] to be included here.

Once added, they will show as top-level fields, with the string `user_pref` prepended.
For example, `dom.ipc.processCount` becomes `user_pref_dom_ipc_processcount`.

## Code Reference

This dataset is generated by [bigquery-etl][main_summary_code].
Refer to this repository for information on how to run or augment the dataset.

[main_summary_code]: https://github.com/mozilla/bigquery-etl/tree/ad84a15d580333b41d36cfe8331e51238f3bafa1/sql/moz-fx-data-shared-prod/telemetry_derived/main_summary_v4
[user_pref_code]: https://github.com/mozilla/bigquery-etl/blob/ad84a15d580333b41d36cfe8331e51238f3bafa1/sql/moz-fx-data-shared-prod/telemetry_derived/main_summary_v4/part1.sql#L476-L501
[ping environment]: http://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/data/environment.html
