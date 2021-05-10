The `addons_daily` table is a small and fast dataset with data on specific add-ons, and the users who have them installed. It contains one row per `addon_id` and `submission_date`.

#### Contents

Many questions about add-ons are of the form: "How many users have add-on A installed?" or "Are users with add-on Z more active than users with add-on Y?" This dataset is aimed at answering these type of questions without having to do cumbersome joins or filters.

This dataset also has detailed search aggregates by each add-on, broken out by our major search engines (`google`, `bing`, `ddg`, `amazon`, `yandex`, `other`), along with total aggregates (`total`). This allows us to identify strange search patterns for add-ons who change a user's search settings on their behalf, often siphoning away SAP searches and Mozilla revenue (see the second example query below).

#### Accessing the Data

The `addons_daily` table is accessible through STMO using the
`Telemetry (BigQuery)` data source.

See [`STMO#71007`](https://sql.telemetry.mozilla.org/queries/71007/source) for an example.
