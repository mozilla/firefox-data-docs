# Google Play Store

<!-- toc -->

# Introduction

The [`google_play_store`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&folder=&organizationId=&p=moz-fx-data-marketing-prod&d=google_play_store&page=dataset) dataset is used to understand the acquisition performance for mobile products on the Google Play Store along key metrics and dimensions. Google’s documentation for all metrics and dimensions can be found [on the Play Store support portal](https://support.google.com/googleplay/android-developer/answer/6263332?hl=en).

# Contents

The [`google_play_store`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&folder=&organizationId=&p=moz-fx-data-marketing-prod&d=google_play_store&page=dataset) dataset contains a collection of tables and views exported from Google. The key tables currently being used for non-desktop reporting are:

- [`Retained_installers_channel_v1`](https://console.cloud.google.com/bigquery?p=moz-fx-data-marketing-prod&d=google_play_store&t=Retained_installers_channel_v1&page=table) - Store activity by acquisition channels. Acquisition channels include:
  - **Play Store (organic)** - Unique users who saw the app’s store listing by browsing or searching on the Play Store app
  - **Third-party referrers** - Unique users who visited the app's store listing on the Play Store app from an untagged deep link to the Play Store.
  - **Tracked Channels (UTM)** - Unique users who visited the app’s store listing on the Play Store app from a UTM-tagged link.
  - **Google Search (organic)** - Unique users who visited the app’s store listing on the Play Store app from a Google Search
  - **Google Ads** - Unique users who visited the app’s store listing on the Play Store app from a Google Ads ad. Data between Google Ads and the Play Console can differ.
  - **Other**
- [`Retained_installers_country_v1`](https://console.cloud.google.com/bigquery?p=moz-fx-data-marketing-prod&d=google_play_store&t=Retained_installers_country_v1&page=table) - Store activity by country.
- [`Retained_installers_play_country_v1`](https://console.cloud.google.com/bigquery?p=moz-fx-data-marketing-prod&d=google_play_store&t=Retained_installers_play_country_v1&page=table) - Store activity for the Play Store (organic) acquisition channel. Includes a country breakdown for the channel.
- [`Retained_installers_utm_tagged_v1`](https://console.cloud.google.com/bigquery?p=moz-fx-data-marketing-prod&d=google_play_store&t=Retained_installers_utm_tagged_v1&page=table) - Store activity for the Tracked Channels (UTM) acquisition channel by campaign UTM.

There are other tables available that are yet to be explored / incorporated into regular reporting. Some that may be of interest include:

- Crashes by app version
- Device and OS
- Installs
- Ratings

The metrics included in the `retained_installers` tables are:

- **`Store_Listing_Visitors`** - Unique users who visited the app’s store listing on the Play Store app but haven’t installed the app
- **`Installers`** - Unique users who installed the app after visiting the app’s store listing on the Play Store app.
- **`Visitors_to_installer_conversion_rate`** - Percentage of `Store_Listing_visitors` that install the app.
- **`Installers_retained_for_1_day`** - Installers who kept the app on at least one of their devices for 1 day. Installation doesn’t mean the app was opened over this period.
- **`Installers_to_1_day_retention_rate`** - Percentage of installers who have the app on one of their devices 1 day after install.
- **`Installers_retained_for_7_days`** - Installers who kept the app on at least one of their devices for 7 days. Installation doesn’t mean the app was opened over this period.
- **`Installers_to_7_days_retention_rate`** - Percentage of installers who have the app on one of their devices 7 days after install.
- **`Installers_retained_for_15_days`** - Installers who kept the app on at least one of their devices for 15 days. Installation doesn’t mean the app was opened over this period.
- **`Installers_to_15_days_retention_rate`** - Percentage of installers who have the app on one of their devices 15 days after install.
- **`Installers_retained_for_30_days`** - Installers who kept the app on at least one of their devices for 30 days. Installation doesn’t mean the app was opened over this period.
- **`Installers_to_30_days_retention_rate`** - Percentage of installers who have the app on one of their devices 30 days after install.

## Background and Caveats

As of Aug 25 2020 not all the tables available in the dataset have been explored and vetted for accuracy by the data science team. Tables that were fully reviewed and being documented here are the `Retained_installers` tables whose primary use case is to explain acquisition.

**Note:** Google does not make the play store data available for export every day. The export job checks for new files every day. However, having monitored the job, it appears the data is made available every 7 - 14 days, and seems to primarily be made available on weekends. Due to this lack of consistency, there will be delays in the data available for this dataset. The data currently in BigQuery is the most current data available from Google.

## Accessing the Data

Access the data at `moz-fx-data-marketing-prod.google_play_store`

# Data Reference

## Schema

```
Retained_installer tables schema
root
    |- Date: date
    |- Package_Name: string
    |- [Acquisition_channel | country | UTM_source_campaign]: string
    |- Store_Listing_Visitors: integer
    |- Installers: integer
    |- Visitor_to_installer_conversion_rate: float
    |- installers_retained_for_1_day: integer
    |- installers_to_1_day_retention_rate: float
    |- installers_retained_for_7_days: integer
    |- installers_to_7_days_retention_rate: float
    |- installers_retained_for_15_days: integer
    |- installers_to_15_days_retention_rate: float
    |- installers_retained_for_30_days: integer
    |- installers_to_30_days_retention_rate: float
```

## Example Queries

### Calculate Google Play Store activity for a given day and app by country

```sql
SELECT
  Date,
  Package_Name,
  Country,
  SUM(Store_Listing_Visitors) as Store_Visits,
  SUM(Installers) as installs,
  SAFE_DIVIDE(SUM(Installers), SUM(Store_Listing_Visitors)) as install_rate
FROM
  `moz-fx-data-marketing-prod.google_play_store.p_Retained_installers_country_v1`
WHERE
  Date = "2020-08-01"
  AND Package_Name = "org.mozilla.firefox"
GROUP BY
  date, Package_name, Country
ORDER BY
  Store_Visits DESC
```

[`STMO#74289`](https://sql.telemetry.mozilla.org/queries/74289/source)

### Calculate Google Play Store activity for a given day by source and app

```sql
SELECT
  Date,
  Package_Name,
  Acquisition_Channel,
  SUM(Store_Listing_Visitors) as Store_Visits,
  SUM(Installers) as installs,
  SAFE_DIVIDE(SUM(Installers), SUM(Store_Listing_Visitors)) as install_rate
FROM
  `moz-fx-data-marketing-prod.google_play_store.p_Retained_installers_channel_v1`
WHERE
  Date = "2020-08-01"
GROUP BY
  date, Package_name, Acquisition_Channel
ORDER BY
  package_name, Store_Visits
```

[`STMO#74288`](https://sql.telemetry.mozilla.org/queries/74288/source)

## Scheduling

The job to retrieve the raw data from the Google Play Store can be found in [the `play-store-export` repository](https://github.com/mozilla/play-store-export) and it is scheduled in [airflow](https://github.com/mozilla/telemetry-airflow/blob/master/dags/play_store_export.py).
