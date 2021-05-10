## Legacy Census Metrics

> **⚠** The information in this document is obsolete. This content was originally included in the [Project Smoot existing metrics report][smootv1] (Mozilla internal link).

[smootv1]: https://mozilla-private.report/smoot-existing-metrics/book/05_overview.html

ADI and DAU are oft-discussed censuses. This chapter discusses their history and definition.

### ADI / Active Daily Installs (blocklist fetches)

> **⚠** The Blocklist mechanism described below is no longer used and has been replaced with [remote settings](https://bugzilla.mozilla.org/show_bug.cgi?id=1257565). The content is left verbatim for historical reference.

ADI, one of Firefox’s oldest client censuses, is computed as the number
of conforming requests to the Firefox
[blocklist](https://wiki.mozilla.org/Blocklisting) endpoint. ADI data is
available since July 13, 2008.

It is not possible to opt-out of the blocklist using the Firefox UI, but
users can disable the update mechanism by changing preference values.

A blocklist is shipped in each release and updated when Firefox notices
that more than 24 hours have elapsed since the last update.

The blocklist request does not contain the telemetry `client_id` or any
other persistent identifiers. Some data about the install are provided
as URI parameters:

- App ID
- App version
- Product name
- Build ID
- Build target
- Locale
- Update channel
- OS version
- Distribution
- Distribution version
- Number of pings sent by this client for this version of Firefox
  (stored in the pref `extensions.blocklist.pingCountVersion`)
- Total ping count (stored in the pref
  `extensions.blocklist.pingCountTotal`)
- Number of full days since last ping

so subsets of ADI may be queried along these dimensions.

The blocklist is kept up-to-date locally using the `UpdateTimerManager`
facility; the update is scheduled in a [manifest] and performed by
[`Blocklist#notify`][bl_notify].

Upon browser startup, after a delay (30 seconds by default),
`UpdateTimerManager` checks whether any of its scheduled tasks are
ready. At each wakeup, the single most-overdue task is triggered, if one
exists. `UpdateTimerManager` then sleeps at least two minutes or until
the next task is scheduled.

Failures are ignored.

The raw data is available in BigQuery (see [`STMO#66481`](https://sql.telemetry.mozilla.org/queries/66481)).

Telemetry only reports whether blocklist checking is enabled or disabled
on the client; there is no data in telemetry about blocklist fetches,
age, or update failures.

[manifest]: https://searchfox.org/mozilla-central/rev/b36e97fc776635655e84f2048ff59f38fa8a4626/toolkit/mozapps/extensions/extensions.manifest#1
[bl_notify]: https://searchfox.org/mozilla-central/rev/b36e97fc776635655e84f2048ff59f38fa8a4626/toolkit/mozapps/extensions/Blocklist.jsm#569

### DAU / Daily Active Users

> **⚠** This description of DAU is not authoritative; please see the [DAU definition in metrics](../metrics/metrics.md#daily-active-users-dau) for the canonical definition.

Firefox DAU is currently computed as the number of unique `client_id`s
observed in `main` pings received on a calendar day. The DAU count
excludes users who have [opted out of telemetry][optout].

[optout]: https://support.mozilla.org/en-US/kb/share-data-mozilla-help-improve-firefox

Each `main` ping describes a single subsession of browser activity.

When and how a ping is sent depends on the reason the subsession ends:

<div id="tbl:pingreasons">

<table style="width:99%;">
<caption>Table 1: When <code>main</code> pings are sent, and why.</caption>
<colgroup>
<col style="width: 9%" />
<col style="width: 7%" />
<col style="width: 7%" />
<col style="width: 75%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Reason</th>
<th style="text-align: left;">Trigger</th>
<th style="text-align: left;">Percent of subsessions [1]</th>
<th style="text-align: left;">Mechanism</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><code>shutdown</code></td>
<td style="text-align: left;">Browser is closed</td>
<td style="text-align: left;">77%</td>
<td style="text-align: left;">For Firefox 55 or later, sent by <a href="https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/internals/pingsender.html"><code>Pingsender</code></a> on browser close unless the OS is shutting down. Otherwise, sent by <a href="https://searchfox.org/mozilla-central/rev/532e4b94b9e807d157ba8e55034aef05c1196dc9/toolkit/components/telemetry/app/TelemetrySend.jsm#677">`TelemetrySendImpl.setup`</a> on the following browser launch.</td>
</tr>
<tr class="even">
<td style="text-align: left;"><code>environment-change</code></td>
<td style="text-align: left;">The telemetry environment changed</td>
<td style="text-align: left;">13%</td>
<td style="text-align: left;">Sent when change is detected by <a href="https://searchfox.org/mozilla-central/rev/532e4b94b9e807d157ba8e55034aef05c1196dc9/toolkit/components/telemetry/pings/TelemetrySession.jsm#1510">`TelemetrySession._onEnvironmentChange`</a></td>
</tr>
<tr class="odd">
<td style="text-align: left;"><code>daily</code></td>
<td style="text-align: left;">more than 24 hours have elapsed since the last ping was sent and the time is local midnight</td>
<td style="text-align: left;">8%</td>
<td style="text-align: left;">Sent at local midnight after a random 0-60 min delay</td>
</tr>
<tr class="even">
<td style="text-align: left;"><code>aborted-session</code></td>
<td style="text-align: left;">A session terminates uncleanly (e.g. crash or lost power)</td>
<td style="text-align: left;">3%</td>
<td style="text-align: left;">Sent by the browser on the next launch; the payload to send is <a href="https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/concepts/crashes.html">written to disk every 5 minutes</a> during an active session and removed by a clean shutdown</td>
</tr>
</tbody>
</table>

</div>
