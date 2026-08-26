---
name: nucleus-security-monitor-platform-status
description: >-
  Check Nucleus Security platform health for a specific region or a specific third-party
  connector before blaming your own integration, using the public Atlassian Statuspage API.
  Unauthenticated.
api: Nucleus Security Public Status Page
base_url: https://status.nucleussec.com/api/v2
auth: none
operations:
  - GET /summary.json
  - GET /status.json
  - GET /components.json
  - GET /incidents/unresolved.json
generated: '2026-08-26'
method: generated
source: >-
  Grounded in a live probe of https://status.nucleussec.com/api/v2/summary.json (HTTP 200,
  2026-08-26), which returned page id 6s7zf54hrbyx and 109 components. Endpoint paths are
  the standard Atlassian Statuspage v2 API.
---

# Check Nucleus Security status before debugging your integration

Nucleus runs an unusually granular public status page: 109 components covering regional
platforms **and** roughly thirty individual scanner and ticketing connectors. That means
you can usually tell within one request whether the problem is yours, Nucleus's, or the
third-party scanner's.

## Steps

1. **Overall state, one call.**

   ```
   GET https://status.nucleussec.com/api/v2/status.json
   ```

   Returns `{page, status:{indicator, description}}`. `indicator` is `none`, `minor`,
   `major`, or `critical`.

2. **Find your region.** Regional platform components are named by region and instance
   number — `US Region`, `UK Platform 1`, `EU Platform 1`, `AU Platform 1`, `AU Platform 2`,
   `AE Platform 1`, plus GovCloud and trial platforms. Pull `summary.json` and match the
   component whose name contains your region:

   ```
   GET https://status.nucleussec.com/api/v2/summary.json
   ```

   Your region is the one your instance hostname resolves to
   (`https://[instance-name].nucleussec.com`). If you do not know it, ask Nucleus support
   rather than guessing from the component list.

3. **Check the specific connector, not just the platform.** If a scan is not ingesting,
   the failing component is often the connector, not Nucleus. Components exist for
   Tenable.io, Tenable.sc, Qualys, Rapid7, Snyk, Checkmarx, Crowdstrike, Orca, Prisma,
   ServiceNow, Jira, GitHub, GitLab, Bitbucket, HackerOne, BugCrowd, Synack, PagerDuty,
   Slack, Sonatype, SonarQube, SonarCloud and others. Match on component name.

4. **Check the docs host separately.** `help.nucleussec.com` and `vip.nucleussec.com` are
   their own components. Documentation being down does not mean the API is down.

5. **Open incidents.**

   ```
   GET https://status.nucleussec.com/api/v2/incidents/unresolved.json
   ```

## Escalation order

1. Status page shows an incident on your region or connector → wait, and subscribe to that
   component.
2. Status page is green but you are getting errors → check your `x-apikey` and the
   RBAC/AGAC role on the key's principal. A permission change takes effect on the next
   call.
3. Still stuck → Nucleus service desk at
   `https://nucleussec.atlassian.net/servicedesk/customer/portal/3`.

## Caveat

There is no API rate-limit signal anywhere in Nucleus, and no `Retry-After` header. If you
poll this status API, pick your own interval — 60 seconds is polite — because the provider
publishes no budget.
