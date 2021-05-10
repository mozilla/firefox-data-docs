# Exact MAU Data

## Introduction

This article introduces the usage of and methodology behind the "exact MAU"
tables in BigQuery:

- `firefox_desktop_exact_mau28_by_dimensions_v1`,
- `firefox_nondesktop_exact_mau28_by_dimensions_v1`, and
- `firefox_accounts_exact_mau28_by_dimensions_v1`.

The calculation of MAU (monthly active users) has historically been fraught
with troubling details around exact definitions and computational limitations,
leading to disagreements between analyses.
These tables contain pre-computed MAU, WAU, and DAU aggregates for
various usage criteria and dimensions, allowing efficient calculation of
aggregates across arbitrary slices of those dimensions.
The tables follow a consistent methodology which is intended as a standard
across Mozilla for MAU analysis going forward.

Note that this data model is used in the [Growth & Usage Dashboard (GUD)](https://gud.telemetry.mozilla.org/) and that some of this documentation is replicated in the [GUD documentation](https://mozilla.github.io/gud).

## Table of Contents

<!-- toc -->

# Conceptual Model

## Metric

A metric is anything want to (and can) measure. In order for a metric to be
calculated, a _usage criterion_ and a _slice_ must be specified. The metric
will produce a single value per day, summarizing data:

- for one day or more days (i.e. the metric value for a particular day may depend on data from other days as well)
- for all users (whatever notion of user makes sense for the data, generally profiles) in a particular sub-population
- where the sub-population will include users that meet the specified usage criteria and are in the specified slice.

A simple _usage criterion_ is "All Desktop Activity", which includes all Firefox Desktop users that we have any data (telemetry ping) for on the day in question. The simplest slice is "All" which places no restrictions on the sub-population.

For example, the metric "Daily Active Users (DAU)" with usage criteria "All Desktop Activity" and slice "All" involves summarizing data on all users of Firefox Desktop over a single day.

## Usage Criteria

Active user counts must always be calculated in reference to some specific
_usage criterion_, a binary condition we use to determine whether a given
user should be considered "active" in a given product or feature.
It may be something simple like "All Desktop Activity" (as above) or,
similarly, "All Mobile Activity". It may also be something more specific like
"Desktop Visited 5 URI" corresponding to calculation of
[aDAU](../../../cookbooks/active_dau.md).

Distinct usage criteria correspond to distinct `*_mau` columns in the Exact MAU tables.

## Slice

A slice defines the sub-population on which we can calculate a metric and
is specified by setting restrictions in different dimensions.
Examples of dimensions include: "Country", "Attribution Source",
and "Firefox Version".
Thus an example slice may be "Country = US; Firefox Version = 60|61", which
restricts to profiles that report usage in the US on Firefox versions 60 or 61.
There is implicitly no restriction on any other dimensions.
Thus, the empty slice - "All" - is also a valid slice and simply places no
restrictions on any dimension.
Note there are some complexities here:

- Firstly, a dimension may be scalar and need to be suitably bucketed (instead of every possible profile age being a unique slice element, maybe we prefer to group users between 12 and 16 months old into a single slice element); likewise we may need to use normalized versions of string fields
- Secondly, we require that dimensions be non-overlapping, especially for metrics calculated over multiple days of user activity. In a given day, a profile may be active in multiple countries, but we aggregate that to a single value by taking the most frequent value seen in that day, breaking ties by taking the value that occurs last. In a given month, the assigned country may change from day to day; we use the value from the most recent active day up to the day we're calculating usage for.

A slice is expressed as a set of conditions in `WHERE` or `GROUP BY` clauses
when querying Exact MAU tables.

# Using the Tables

The various Exact MAU datasets are computed daily from the `*_last_seen`
tables (see [`clients_last_seen`](/datasets/bigquery/clients_last_seen/reference.md))
and contain pre-computed DAU, WAU, and MAU counts per usage criterion
per each unique combination of dimensions values.
Because of our restriction that dimension values be non-overlapping, we
can recover MAU for a particular slice of the data by summing over all rows
matching the slice definition.

The simple case of retrieving MAU for usage criterion
"All Desktop Activity" and slice "All" looks like:

```sql
SELECT
    submission_date,
    SUM(mau) AS mau
FROM
    `mozdata.telemetry.firefox_desktop_exact_mau28_by_dimensions_v1`
GROUP BY
    submission_date
ORDER BY
    submission_date
```

Now, let's refine our slice to "Country = US; Campaign = `whatsnew`"
via a `WHERE` clause:

```sql
SELECT
    submission_date,
    SUM(mau) AS mau
FROM
    `mozdata.telemetry.firefox_desktop_exact_mau28_by_dimensions_v1`
WHERE
    country = 'US'
    AND campaign = 'whatsnew'
GROUP BY
    submission_date
ORDER BY
    submission_date
```

Perhaps we want to compare MAU as above to [aDAU](../../../cookbooks/active_dau.md)
over the same slice. The column `visited_5_uri_dau` gives DAU as calculated
with the "Desktop Visited 5 URI" usage criterion, corresponding to aDAU:

```sql
SELECT
    submission_date,
    SUM(mau) AS mau,
    SUM(visited_5_uri_dau) AS adau
FROM
    `mozdata.telemetry.firefox_desktop_exact_mau28_by_dimensions_v1`
WHERE
    country = 'US'
    AND campaign = 'whatsnew'
GROUP BY
    submission_date
ORDER BY
    submission_date
```

Additional usage criteria may be added in the future as new columns named
`*_*mau`, etc. where the prefix describes the usage criterion.

For convenience and clarity, we make the exact data presented in the [Key Performance Indicator Dashboard](https://go.corp.mozilla.com/kpi-dash) available as views that do not require any aggregation:

- `firefox_desktop_exact_mau28_v1`,
- `firefox_nondesktop_exact_mau28_v1`, and
- `firefox_accounts_exact_mau28_v1`.

An example query for desktop:

```sql
SELECT
    submission_date,
    mau,
    tier1_mau
FROM
    mozdata.telemetry.firefox_desktop_exact_mau28_v1
```

These views contain no dimensions and abstract away the detail that FxA
data uses the "Last Seen in Tier 1 Country" usage criterion while desktop
and non-desktop data use the "Country" dimension to determine tier 1 membership.

# Additional Details

## Inclusive Tier 1 Calculation for FxA

The 2019 Key Performance Indicator definition for Relationships relies on a MAU calculation restricted
to a specific set of "Tier 1" countries.
In the Exact MAU datasets, country is a dimension that would normally be specified
in a slice definition.
Indeed, for desktop and non-desktop clients, the definition of "Tier 1 MAU" looks like:

```sql
SELECT
    submission_date,
    SUM(mau) AS mau
FROM
    mozdata.telemetry.firefox_desktop_exact_mau28_by_dimensions_v1
WHERE
    country IN ('US', 'UK', 'DE', 'FR', 'CA')
GROUP BY
    submission_date
ORDER BY
    submission_date
```

Remember that our non-overlapping dimensions methodology means that the filter
in the query above considers only the country value from the most recent daily
aggregation, so a user that appeared in one of the specified countries early
in the month but then changed location to a non-tier 1 country would not count
toward MAU.

Due to the methodology used when forecasting goal values for the year, however, we need to
follow a more inclusive definition for "Tier 1 FxA MAU" where a user counts if they register
even a single FxA event originating from a tier 1 country in the 28 day MAU window.
That calculation requires a separate "FxA Seen in Tier 1 Country" criterion
and is represented in the exact MAU table as `seen_in_tier1_country_mau`:

```sql
SELECT
    submission_date,
    SUM(seen_in_tier1_country_mau) AS tier1_mau
FROM
    mozdata.telemetry.firefox_accounts_exact_mau28_by_dimensions_v1
GROUP BY
    submission_date
ORDER BY
    submission_date
```

## Confidence Intervals

The Exact MAU tables enable tracking of MAU for potentially very small
subpopulations of users where statistical variation can often overwhelm
real trends in the data.
In order to support statistical inference (confidence intervals and hypothesis tests),
these tables include a "pseudo-dimension" we call `id_bucket`. We assign
each client (or user, in the case of FxA data) to one of 20 buckets based on a
hash of their `client_id` (or `user_id`), with the effect that each user is
randomly assigned to one and only one bucket. If we sum MAU numbers for each
bucket individually, we can use resampling techniques to determine the magnitude
of variation and assign a confidence interval to our sums.

As an example of calculating confidence intervals, see
[`STMO#61957`](https://sql.telemetry.mozilla.org/queries/61957/source)
which uses a jackknife resampling technique implemented as a BigQuery UDF.
