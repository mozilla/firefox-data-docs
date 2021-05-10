# Apple App Store

<!-- toc -->

# Introduction

The [`apple_app_store`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&folder=&organizationId=&p=moz-fx-data-marketing-prod&d=apple_app_store&page=dataset) dataset is used to understand the acquisition performance for non-desktop products on the Apple App Store along the key metrics and dimensions. Apple’s documentation for all metrics and dimensions can be found [in the app store connect reference](https://help.apple.com/app-store-connect/#/itc21781223f).

# Contents

The [`apple_app_store`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&folder=&organizationId=&p=moz-fx-data-marketing-prod&d=apple_app_store&page=dataset) dataset contains a collection of aggregated tables by a singular dimension that explains the performance of acquisition activity through the Apple App Store.

The dimensions (saved as individual derived tables) include:

- [`moz-fx-data-marketing-prod.apple_app_store.metrics_by_platform`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_platform&page=table) - The device type on which the app was downloaded or used. E.g. iPad, iPod, or iPhone.
- [`moz-fx-data-marketing-prod.apple_app_store.platform version`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_platform_version&page=table) - The OS version on which the app was downloaded or used. App Units, In-App Purchases, and Sales are based on the version on which the app is downloaded. Active in Last 30 Days, Product Page Views, Retention, and Sessions are based on the iOS version on which the app is used.
- [`moz-fx-data-marketing-prod.apple_app_store.app version`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_app_version&page=table) - The version of the app displayed on the app store at the time of activity.
- [`moz-fx-data-marketing-prod.apple_app_store.region`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_region&page=table) - The App Store region in which purchases were made, based on the customer’s billing address. Regions include (USA and Canada, Europe, Latin America and The Caribbean, Asia Pacific, Africa, The Middle East, and India)
- [`moz-fx-data-marketing-prod.apple_app_store.storefront`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_storefront&page=table) - The app store country in which purchases were made, based on the customer’s billing address.
- [`moz-fx-data-marketing-prod.apple_app_store.source`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_source&page=table) - The source from which a customer tapped a link to your App Store product page to view your app or download it for the first time. You can view metrics based on the source from which users are finding your app. Source types include (App Store Browse, App Store Search, App Referrers, Web Referrers)
- [`moz-fx-data-marketing-prod.apple_app_store.app referrer`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_app_referrer&page=table) - People landing on the app store via links from within other apps. This also includes apps using the store kit API and excludes the native Safari.
- [`moz-fx-data-marketing-prod.apple_app_store.web Referrer`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_web_referrer&page=table) - Previously top websites. Shows the referring website for the app download. Web referrals must be from Safari on devices with iOS 8 or tvOS 9 or later. Taps from websites using web browsers like chrome are attributed to that app.
- [`moz-fx-data-marketing-prod.apple_app_store.campaign`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_by_campaign&page=table) - Previously top campaigns. Tracks app and website referrals to measure the impact of an advertising campaign. Tracked by adding two tokens to any app store link to see results in app analytics.
- [`moz-fx-data-marketing-prod.apple_app_store.total`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&t=metrics_total&page=table): Total metric activity without any dimension breakdown.

The metrics included in the aggregated tables are:

- **`app_units`** - The number of first-time app purchases made on the App Store using iOS 8 and tvOS 9 or later. Updates, re-downloads, download onto other devices are not counted. Family sharing downloads are included for free apps, but not for paid apps.
- **`impressions`** - The number of times the app was viewed in the Featured, Categories, Top Charts and Search Sections of the App Store. Also includes views of the product page.
- **`impressions_unique_device`** - The number of times the app was viewed in the Featured, Categories, Top Charts and Search Sections of the App Store by unique device. Also includes views of the product page.
- **`product_page_views`** - Number of times the app's product page has been viewed on devices iOS 8 and tvOS 9 or later. Includes both App Store app and store kit API
- **`product_page_views_unique_device`** - Number of times the app's product page has been viewed on devices iOS 8 and tvOS 9 or later by unique device. Includes both App Store app and store kit API
- **`active_devices_opt_in`** - The number of devices with at least one session during the selected period. Only devices with iOS 8 and tvOS 9 or later are included. Data for this metric is “opt-in” - collected only if users have agreed to share their diagnostics and usage information with app developers.
- **`active_devices_last_30_days_opt_in`** - The number of active devices with at least one session during the previous 30 days. Data for this metric is “opt-in” - collected only if users have agreed to share their diagnostics and usage information with app developers.
- **`deletions_opt_in`** - The number of times your app was deleted on devices running iOS 12.3 or tvOS 9 or later. This data includes deletions of the app from the Home Screen and deletions of the app through Manage Storage. Data from resetting or erasing a device's content and settings is not included. Data for this metric is “opt-in” - collected only if users have agreed to share their diagnostics and usage information with app developers.
- **`installations_opt_in`** - The total number of times your app has been installed on devices with iOS 8 or tvOS 9, or later. Re-downloads on the same device, downloads to multiple devices sharing the same Apple ID, and Family Sharing installations are included. Data for this metric is “opt-in” - collected only if users have agreed to share their diagnostics and usage information with app developers.
- **`sessions_opt_in`** - The number of times the app has been used for at least two seconds. If the app is in the background and is later used again, that counts as another session. Data for this metric is “opt-in” - collected only if users have agreed to share their diagnostics and usage information with app developers.

## Background and Caveats

The data is received from Apple with only one dimension per metric. As a result, we are unable to do multi-dimensional analysis. i.e. we can tell how each storefront is performing but we can’t see how specific platforms or sources are contributing to it.

## Accessing the Data

Access the data at [`moz-fx-data-marketing-prod.apple_app_store`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&folder=&organizationId=&p=moz-fx-data-marketing-prod&d=apple_app_store&page=dataset)

# Data Reference

## Schema

```
metrics_by_[dimension] tables
root
    |- date: date
    |- app_name: string
    |- [app_referrer | app_version | campaign | platform | platform_version | region | source | storefront | web_referrer]: string
    |- app_units: integer
    |- impressions: integer
    |- impressions_unique_device: integer
    |- product_page_views: integer
    |- product_page_views_unique_device: integer
    |- active_devices_opt_in: integer
    |- active_devices_last_30_days_opt_in: integer
    |- deletions_opt_in: integer
    |- installations_opt_in: integer
    |- sessions_opt_in: integer
```

## Example Queries

### Calculate Apple App Store Activity for a given day by app

```sql
SELECT
  date,
  app_name,
  SUM(impressions_unique_device) as unique_device_impressions,
  SUM(product_page_views_unique_device) as unique_device_page_views,
  SUM(app_units) as installs,
  SAFE_DIVIDE(SUM(product_page_views_unique_device), SUM(impressions_unique_device)) as unique_device_page_view_rate,
  SAFE_DIVIDE(SUM(app_units), SUM(product_page_views_unique_device)) as install_rate
FROM
  `moz-fx-data-marketing-prod.apple_app_store.metrics_total`
WHERE
  date = "2020-08-20"
GROUP BY
  date, app_name
```

[`STMO#74291`](https://sql.telemetry.mozilla.org/queries/74291/source)

### Calculate Apple App Store Activity for a given day and app by source

```sql
SELECT
  date,
  app_name,
  source,
  SUM(impressions_unique_device) as unique_device_impressions,
  SUM(product_page_views_unique_device) as unique_device_page_views,
  SUM(app_units) as installs,
  SAFE_DIVIDE(SUM(product_page_views_unique_device), SUM(impressions_unique_device)) as unique_device_page_view_rate,
  SAFE_DIVIDE(SUM(app_units), SUM(product_page_views_unique_device)) as install_rate
FROM
  `moz-fx-data-marketing-prod.apple_app_store.metrics_by_source`
WHERE
  date = "2020-08-20"
  AND app_name = "Firefox"
GROUP BY
  date, app_name, source
```

[`STMO#74290`](https://sql.telemetry.mozilla.org/queries/74290/source)

## Scheduling

The job to retrieve the raw data from the Apple App Store can be found in [the `app-store-analytics-export` repository](https://github.com/mozilla/app-store-analytics-export). The exported results are individual metrics grouped by a single dimension. These exports are initially loaded into the [`apple_app_store_exported`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store_exported&page=dataset) data source. The exports are scheduled in [`airflow`](https://github.com/mozilla/telemetry-airflow/blob/master/dags/app_store_analytics.py). The job to create the derived tables found in [`moz-fx-data-marketing-prod.apple_app_store`](https://console.cloud.google.com/bigquery?project=moz-fx-data-marketing-prod&p=moz-fx-data-marketing-prod&d=apple_app_store&page=dataset) can be found in [`bigquery-etl` under `apple_app_store`](https://github.com/mozilla/bigquery-etl/tree/master/sql/moz-fx-data-marketing-prod/apple_app_store).
