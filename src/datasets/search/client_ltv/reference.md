# Normalized Client Lifetime Value (LTV)

<!-- toc -->

# Introduction

{{#include ./intro.md}}

# Example Queries

_Percent of users we predict will click an ad in the **next** 365 days by Engine._ ([`STMO#74878`](https://sql.telemetry.mozilla.org/queries/74878/source))

```sql
SELECT
  engine,
  AVG(IF(pred_num_days_seeing_ads > 0, 1, 0)) as pct_predicted_ad_viewers_next_year,
  AVG(IF(pred_num_days_clicking_ads > 0, 1, 0)) as pct_predicted_ad_clickers_next_year,
FROM
  `moz-fx-data-shared-prod`.revenue_derived.client_ltv_v1
WHERE
  submission_date = DATE_SUB(CURRENT_DATE, INTERVAL 1 DAY)
GROUP BY
  1
```

_LTV Value of Users Over Lifetime (by `days_since_created_profile`) of Users Active in Past 7 Days_ ([source](https://sql.telemetry.mozilla.org/queries/74867/source#187036))

```sql
SELECT
  days_since_created_profile,
  SUM(normalized_ltv_ad_clicks_current) AS sum_normalized_ltv_ad_clicks_current,
  SUM(normalized_ltv_ad_clicks_future) AS normalized_ltv_ad_clicks_future,
  SUM(normalized_ltv_ad_clicks_future) / COUNT(*) AS avg_normalized_ltv_ad_clicks_future,
FROM
  `moz-fx-data-shared-prod`.revenue_derived.client_ltv_v1
JOIN
  `moz-fx-data-shared-prod`.search.search_clients_last_seen
  USING(submission_date, client_id)
WHERE
  submission_date = '2020-09-16'
  AND days_since_created_profile <= 365
  AND days_since_seen <= 6
GROUP BY
  days_since_created_profile
ORDER BY
  days_since_created_profile DESC


```

# Schema

As of 2020-09-16,
the current version of `client_ltv` is `v1`,
and has a schema as follows.
The dataset is backfilled through 2020-09-14.

```
root
 |-- submission_date: date (nullable = true)
 |-- engine: string (nullable = true)
 |-- country: string (nullable = true)
 |-- client_id: string (nullable = true)
 |-- total_client_searches_past_year: long (nullable = true)
 |-- total_client_tagged_searches_past_year: long (nullable = true)
 |-- total_client_ad_clicks_past_year: long (nullable = true)
 |-- total_client_searches_with_ads_past_year: long (nullable = true)
 |-- ad_click_days: long (nullable = true)
 |-- search_days: long (nullable = true)
 |-- search_with_ads_days: long (nullable = true)
 |-- tagged_search_days: long (nullable = true)
 |-- active_days: long (nullable = true)
 |-- pred_num_days_clicking_ads: double (nullable = true)
 |-- pred_num_days_seeing_ads: double (nullable = true)
 |-- pred_num_days_searching: double (nullable = true)
 |-- pred_num_days_tagged_searching: double (nullable = true)
 |-- ad_clicks_per_day: double (nullable = true)
 |-- searches_with_ads_per_day: double (nullable = true)
 |-- searches_per_day: double (nullable = true)
 |-- tagged_searches_per_day: double (nullable = true)
 |-- ad_clicks_cutoff: double (nullable = true)
 |-- searches_with_ads_cutoff: double (nullable = true)
 |-- searches_cutoff: double (nullable = true)
 |-- tagged_searches_cutoff: double (nullable = true)
 |-- ad_clicks_per_day_capped: double (nullable = true)
 |-- searches_with_ads_per_day_capped: double (nullable = true)
 |-- searches_per_day_capped: double (nullable = true)
 |-- tagged_searches_per_day_capped: double (nullable = true)
 |-- total_ad_clicks: long (nullable = true)
 |-- normalized_ltv_ad_clicks_current: double (nullable = true)
 |-- normalized_ltv_search_with_ads_current: double (nullable = true)
 |-- normalized_ltv_search_current: double (nullable = true)
 |-- normalized_ltv_tagged_search_current: double (nullable = true)
 |-- normalized_ltv_ad_clicks_future: double (nullable = true)
 |-- normalized_ltv_search_with_ads_future: double (nullable = true)
 |-- normalized_ltv_search_future: double (nullable = true)
 |-- normalized_ltv_tagged_search_future: double (nullable = true)
```

# Code References

- [LTV daily model fitting](https://github.com/mozilla/telemetry-airflow/blob/master/jobs/ltv_daily.py)
- Unnormalized [`client_ltv` query](https://github.com/mozilla/bigquery-etl/blob/master/sql/moz-fx-data-shared-prod/revenue_derived/client_ltv_v1/query.sql) (restricted query access)
- [`client_ltv` view](https://github.com/mozilla/bigquery-etl/blob/master/sql/moz-fx-data-shared-prod/revenue_derived/client_ltv_normalized/query.sql) (for broad use)

# Model Performance

There is additionally a dataset, `ltv_daily_model_perf`, that tracks the LTV model's prediction performance each day it is re-trained. For a given day, one could check the performance with the following [query](https://sql.telemetry.mozilla.org/queries/75244/source#187873):

```sql
SELECT
  active_days,
  actual,
  model
FROM
  `moz-fx-data-shared-prod.analysis.ltv_daily_model_perf`
WHERE
  date = '2020-09-29'
AND
  metric = 'days_clicked_ads'
ORDER BY
  active_days
```

This produces a histogram for the observed user frequencies and the model's predicted frequencies, allowing a chart similar to the one shown in the ["assessing model fit" example](https://lifetimes.readthedocs.io/en/latest/Quickstart.html#assessing-model-fit) in the `lifetimes` documentation. This table only checks performance for clients the model expects have, for example, clicked an ad in 0 to 28 days in the past year, since most of the distribution is contained in that interval.
