# Analysis Gotchas

When you perform an analysis on any data, there are some mistakes that are easy to make:

- Do you know what question that you hope to answer?
- Is your sample representative of your population?
- Is your result "real"? How precisely can you state your conclusion?

This section is not about those traps. Instead, it is about quirks and pitfalls that are specific to Mozilla's data systems.

## Table of Contents

<!-- toc -->

## Intermittent issues

Despite best efforts, problems may occur from time to time with the ingestion of data or the faithful creation of datasets.

Issues undergoing investigation are marked with the `[data-quality]` whiteboard tag in Bugzilla. See [currently open issues](https://bugzilla.mozilla.org/buglist.cgi?bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&classification=Client%20Software&classification=Developer%20Infrastructure&classification=Components&classification=Server%20Software&classification=Other&priority=P1&priority=P2&priority=P3&priority=--&product=Data%20Platform%20and%20Tools&resolution=---&status_whiteboard=%5Bdata-quality%5D&status_whiteboard_type=allwordssubstr&list_id=15179084).

Especially severe problems with production data are announced on the `fx-data-dev` mailing list (see [getting help](getting_help.md)). Subscribing to this mailing list is recommended if you are a current or aspiring data practitioner.

## Notable historic events

When you start to evaluate trends, be aware of events from the past that may invite comparisons with history. Here are a few to keep in mind:

- **Feb 16 2021 - Feb 25 2021** - [A small number of stub installer pings may have been discarded due to URI deduplication][bug1694764].
- **August 6, 2020** - [Pings with "automation" tag in X-Source-Tags will no longer appear in stable tables][bq1215]
  This is particularly relevant for removing pings related to automated testing of Fenix.
- **July 20, 2020** - [Glean dropping application lifetime metrics from `metrics` pings][bug1653244].
  Glean Android bindings from version `v25.0.0` up to and including `v31.4.0` had a bug that would cause metrics with “lifetime: application” to be cleared before they could be collected for metrics pings sent during startup. This can result in application lifetime metrics like experiment information being randomly missing from the data.
- **April 14, 2020** - [Telemetry edge server rejects pings for an hour][bug1630096].
  Clients generally retry periodically until success is achieved. Therefore, most of these messages were eventually ingested; submission timestamps appear later than normal. A small number of pings are attributed to a later day or were never sent due to the client never being reopened.
- **February 11, 2020** - Firefox 73 was released and began the start of [4-week release cycles][four_week_cycles]. There was a gradual transition from 7/8 week ones to 4 week ones.
- **December 4 2019** - [AWS Ingestion Pipeline decommissioned][bug1598815].
  Specifically, the last ping relayed through the AWS machinery had a
  timestamp of `2019-12-04 22:04:45.912204 UTC`.
- **October 29 2019** - Glean SDK Timing Distribution(s) report buckets
  1 nanosecond apart. This occurred because of a potential rounding bug in Glean SDK
  versions less than `19.0.0`. See [Bug 1591938].
- **October 23 2019** - [Hot-fix shipped through add-ons][endpoint_hotfix] that
  reset the Telemetry endpoint preference back to the default for a large number of users.
- **September 1 - October 18 2019** - BigQuery Ping tables are
  [missing the `X-PingSender-Version` header information][pull1491].
  This data is available before and after this time period.
- **May 4 - May 11 2019** - [Telemetry source data deleted][armagaddon].
  No source data is available for this period and derived tables may have
  missing days or imputed values.
  Derived tables that depend on multiple days may have have affected dates
  beyond the deletion region.
- **January 31 2019** - [Profile-per-install][bug1474285] landed in `mozilla-central`
  and affected how new profiles were created.
  See [discussion in `bigquery-etl#212`][bq212].
- **October 25 2018** - many `client_id`s on Firefox Android were reset to the
  same `client_id`.
  For more information, see the [post-mortem][reset_cid_retro]
  or [Bug 1501329].
- **November 2017** - Quantum Launch. There was a surge in new profiles and usage.
- **June 1 and 5, 2016** - [Main Summary `v4` data is missing][bug1482509]
  for these two days.
- **March 2016** - Unified Telemetry launched.

[bug 1591938]: https://bugzilla.mozilla.org/show_bug.cgi?id=1591938
[endpoint_hotfix]: https://docs.google.com/document/d/1gQF-iU3E21SG985Cl2Ius4LoRXduUrNa5In9hafLIqs/edit
[pull1491]: https://github.com/mozilla-services/cloudops-infra/pull/1491
[armagaddon]: https://blog.mozilla.org/blog/2019/05/09/what-we-do-when-things-go-wrong/
[bug1474285]: https://bugzilla.mozilla.org/show_bug.cgi?id=1474285
[bq212]: https://github.com/mozilla/bigquery-etl/issues/212
[bq1215]: https://github.com/mozilla/bigquery-etl/pull/1215
[reset_cid_retro]: https://docs.google.com/document/d/1r1PDQnqhsrPkft0pB46v9uhXGxR_FzK4laKJLGttXdA
[bug 1501329]: https://bugzilla.mozilla.org/show_bug.cgi?id=1501329
[bug1482509]: https://bugzilla.mozilla.org/show_bug.cgi?id=1482509
[bug1598815]: https://bugzilla.mozilla.org/show_bug.cgi?id=1598815
[four_week_cycles]: https://docs.google.com/document/d/1oJhnvAOx2c8Mp-Xpk-3j-2d45yu_fghYS2yAbn1aeNY/edit#heading=h.iba82gckexg7
[bug1630096]: https://bugzilla.mozilla.org/show_bug.cgi?id=1630096
[bug1653244]: https://bugzilla.mozilla.org/show_bug.cgi?id=1653244
[bug1694764]: https://bugzilla.mozilla.org/show_bug.cgi?id=1694764

## Pseudo-replication

Telemetry data is a collection of pings.
A single main-ping represents a single subsession.
Some clients have more subsessions than others.

When you say ["63% of beta 53 has Firefox set as its default browser"](https://mzl.la/2q75dbF), you need to specify that it is 63% of _pings_ because it represents only around 46% of clients.
(Apparently users with Firefox Beta 53 set as their default browser submitted
more main-pings than users who did not).

## Profiles vs Users

You may have noticed that the term "clients" and not "users" was applied in the above-listed section because of all the things can be counted, users is not one of them:

Users can have multiple Firefox profiles that runs on the same system at
the same time (like developers).

Users can have the same Firefox profile that runs on several systems on
different days of the week (also developers).

The only things we can count are pings and clients.
Clients can be counted because a `client_id` is sent with each ping that uniquely
identifies the profile from which it originated.
This is generally close enough to the idea of a so-called "user" that can be considered for counting profiles and calling them users. However, you may run into some instances
where the distinction matters.

When in doubt, be precise. You count _clients_.

[This article](./profile/index.md) describes the concept of "profiles" in detail.

## Opt-in versus Opt-out

Mozilla does not collect the same information from everyone.

Every profile that does not have Telemetry disabled sends "opt-out" Telemetry, which
includes:

- Nearly all the data in the [Environment]
- Some specific [Histograms], [Scalars], and [Events] that are marked
  `"releaseChannelCollection": "opt-out"`

Most probes are "opt-in": No information is received from the probes unless a user
opts into sending this information by installing any pre-release version of Firefox:
Beta, Nightly or Developer Edition (the latter is similar to Beta).

If you want to encourage users to collect good information for Mozilla, ask them to install a Beta release.

[environment]: https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/data/environment.html
[histograms]: https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/collection/histograms.html
[scalars]: https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/collection/scalars.html
[events]: https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/collection/events.html

## Trusting Dates

Do not assume that the time reported by an instance of Firefox desktop is correct. The situation is somewhat better on mobile devices, but you should still be cautious.

Any timestamp recorded by the user is subject to "clock skew."
The user's clock can be set (purposefully or accidentally) to any time at all.
SSL certificates tend to keep timestamps in a certain relatively-accurate window
because a user whose clock has been set too far in the past or too far in the future
may confuse certain expiration-date-checking code.

Examples of client times from Firefox desktop pings:

- `crashDate`
- `crashTime`
- `meta/Date`
- `sessionStartDate`
- `subsessionStartDate`
- `profile/creationDate`

Examples of client times from Glean pings:

- [`ping_info.end_time`](https://mozilla.github.io/glean/book/user/pings/index.html#the-ping_info-section)

Examples of server times that you can trust:

- `submission_timestamp`
- `submission_date`

_Note_ `submission_date` does not appear in the [ping documentation]
because it is added in post-processing.

[ping documentation]: https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/data/common-ping.html

## Date Formats

Not all dates and times are created equal.
Most of the dates and times in Telemetry pings are [ISO 8601].
Most are full timestamps, though their resolution may differ from per-second to per-day.

Then there's `profile/creationDate` which is just a number of days since epoch (January 1, 1970).
Like `17177` for the date 2017-01-11.

**Tip:** If you want to convert `profile/creationDate` to a usable date in SQL:
`DATE_FROM_UNIX_DATE(SAFE_CAST(environment.profile.creation_date AS INT64))`

In derived datasets ISO dates are sometimes converted to strings using one of
the following formats: `%Y-%m-%d` or `%Y%m%d`.

The date formats for different rows in `main_summary` are described on the
[`main_summary` reference page][msref].

Although build IDs look like dates, they are not. If you take the first eight characters, you can use them as a proxy for the day when the build was released.

`metadata.header.date` represents an HTTP Date header in a [RFC 7231]-compatible format.

**Tip:** If you want to parse `metadata/Date` to become a usable date in SQL:
`SAFE.PARSE_TIMESTAMP('%a, %d %b %Y %T %Z', REPLACE(metadata.header.date, 'GMT+00:00', 'GMT'))`
Alternatively, you can use the already parsed version that is available in user-facing views (`metatdata.header.parsed_date`)

[iso 8601]: https://en.wikipedia.org/wiki/ISO_8601
[msref]: ../datasets/batch_view/main_summary/reference.md#time-formats
[rfc 7231]: http://tools.ietf.org/html/rfc7231#section-7.1.1.1

## Delays

There is an inherent delay between Telemetry data being created on the client and it being received by Mozilla.
Most Telemetry data produced by desktop Firefox is represented in the main ping. It is sent at the beginning of a client's _next_ Firefox session.
If the user shuts down Firefox for the weekend, Firefox does not receive any data that has been generated on Friday data until Monday morning.

Generally speaking, data from two days ago is usually fairly representative.

If you'd like to read more about this subject, there is a series of blog posts [here][delays1], [here][delays2] and [here][delays3].

[bug 1336360]: https://bugzilla.mozilla.org/show_bug.cgi?id=1336360
[delays1]: https://chuttenblog.wordpress.com/2017/02/09/data-science-is-hard-client-delays-for-crash-pings/
[delays2]: https://chuttenblog.wordpress.com/2017/07/12/latency-improvements-or-yet-another-satisfying-graph/
[delays3]: https://chuttenblog.wordpress.com/2017/09/12/two-days-or-how-long-until-the-data-is-in/

### Pingsender

Pingsender greatly reduces any delay before sending pings to Mozilla.
However, only some types of pings are sent by Pingsender.
[Bug 1310703] introduced Pingsender for crash pings and was merged in Firefox 54,
which was included in release on June 13, 2017.
[Bug 1336360] moved shutdown pings to Pingsender and was merged in Firefox 55,
which was included in release on August 8, 2017.
[Bug 1374270] added sending health pings on shutdown using Pingsender and was
merged in Firefox 56, which was included in release on Sept 28, 2017.
Other types of pings are not sent with Pingsender.
This is usually okay because Firefox is expected to continue to run long
enough to send these pings.

Mobile clients do not have Pingsender. Therefore, a delay occurs as described in [`STMO#49867`][delay_q].

[bug 1310703]: https://bugzilla.mozilla.org/show_bug.cgi?id=1310703
[bug 1374270]: https://bugzilla.mozilla.org/show_bug.cgi?id=1374270
[delay_q]: https://sql.telemetry.mozilla.org/queries/49867#134105

### Submission Date

`submission_date` or `submission_timestamp` represents the server time at which a ping is received from a client. It is used as a partitioning column (useful for both query optimization and restricting the range of data under consideration) and should be considered reliable.

In [bug 1422892](https://bugzilla.mozilla.org/show_bug.cgi?id=1422892) it was decided
to standardize on using `submission_date` as opposed to client-generated dates.

Summary of reasons for this decision:

- not subject to client clock skew
- doesn't require normalization
- good for backfill
- good for daily processing
- and usually good enough

## Pings from Robots

In general, data coming from an application instance not run by a human is not wanted in analysis. As of this writing, [GeckoDriver](https://github.com/mozilla/geckodriver) (one of the official mechanisms to launch and control an automated version of Firefox for e.g. web compatibility testing) is [configured _not_ to send Telemetry by default](https://searchfox.org/mozilla-central/rev/baf1cd492406a9ac31d9ccb7a51c924c7fbb151f/testing/geckodriver/src/prefs.rs#154) but we can't control for other things people might do in the field.

On desktop, one field to watch out for is headless mode (`environment.system.gfx.headless` in the main ping): if that field is set, you are for certain not working with a version of Firefox being operated by a real human. You can see an example of some client pings with this field set skewing the nightly numbers in [bug 1643341](https://bugzilla.mozilla.org/show_bug.cgi?id=1643341). An easy solution is to just filter out these types of clients in your analysis. You can see an example of this pattern in [`STMO#71781`](https://sql.telemetry.mozilla.org/queries/71781/source).

## Build Ids

Generally speaking, application versions are monotonically increasing multipart alphanumeric strings like "89.0a1" or "68.0.3".
Build Ids are not this.
A Build Id is a sequence of characters that is unique to a specific build of a product.
Since the application version may not vary across shipped versions (for example, a Firefox nightly version stays the same across its entire cycle), a build id helps identify which code changes were included in a build as well as what features may have been enabled for it.
For example, in Firefox Desktop, the build id is the date and time the build was built in yyyymmddhhmmss format.
A build id might be formatted in any way and contain the time or version control system revision of the code included in the build.

Do not assume build id's are consistent across the products we ship. A build id format may vary between products, between channels of the same product, or over time within the same channel of the same product.
The build id format for Firefox Desktop has been very stable over time thus far, but even it can be different for different platforms in some respin circumstances (if e.g. only one platform's builder failed).
