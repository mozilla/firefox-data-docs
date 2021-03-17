# Accessing Glean Data in BigQuery

This document describes how to access Glean data using BigQuery, such as in [Redash](https://sql.telemetry.mozilla.org).
This is intended for in-depth analysis: many simple questions can be more easily answered with either [GUD](../tools/interfaces.md#mozilla-growth--usage-dashboard-gud) or [GLAM](../tools/interfaces.md##glean-aggregated-metrics-dashboard-glam).

## Using the Glean Dictionary

The data that Glean applications generates maps cleanly to structures we create in
BigQuery: see the section on [Glean Data](../pipeline/glean_data.md) in the data pipeline
reference.

You can use the [Glean Dictionary](https://dictionary.protosaur.dev/) to access these mappings
when writing queries. For example, say you wanted to get a count of top sites as measured in
Firefox for Android. In this case you would:

- Go to the [Glean Dictionary](https://dictionary.protosaur.dev) home page.
- Navigate to the [Firefox for Android application](https://dictionary.protosaur.dev/apps/fenix)
- Under metrics, search for "top", select [`metrics.top_sites_count`](https://dictionary.protosaur.dev/apps/fenix/metrics/metrics_top_sites_count).
- Scroll down to the bottom. Under BigQuery, you should see an entry like: "In `org_mozilla_fenix.metrics` as `metrics.counter.metrics_top_sites_count`".
  The former corresponds to the table name whilst the latter corresponds to the column name
  You can select which channel you want to view information for and the table name will update accordingly.

## Writing a Query

With this information in hand, you can now proceed to writing a query. For example, to get the
average of this metric on the first of January, you could write something like this:

```sql
SELECT AVG(metrics.metrics.counter.metrics_top_sites_count)
FROM org_mozilla_fenix.metrics
WHERE DATE(submission_timestamp) = '2021-01-01'
```

Note that we need to qualify the column name (`metrics.counter.metrics_top_sites_count`) with
the name of the table, otherwise the BigQuery parser gets confused.
This can also happen with the tables and columns corresponding to the events ping.