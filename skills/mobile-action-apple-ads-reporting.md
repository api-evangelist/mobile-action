---
name: Pull Apple Ads campaign performance and paid-keyword competition
description: Read your own Apple Ads / MMP integrations, goals and reports out of MobileAction CMP, then set the results against the paid-keyword and Custom Product Page competition in the same storefront.
api: https://api.mobileaction.co
mcp: https://mcp.mobileaction.co/mcp
operations:
  - get_credit_status
  - searchads_get_search_ads_integrations
  - searchads_get_mmp_integrations
  - searchads_get_apps
  - searchads_get_goals
  - searchads_get_goal
  - searchads_get_reports
  - searchads_get_top_apps_report
  - searchads_get_bid_history_logs
  - searchads_paid_keywords
  - searchads_paying_apps
  - cpp_by_keyword
generated: '2026-08-13'
method: generated
source: mcp/mobile-action-mcp-tools-list.json (probed tools/list) + https://docs.mobileaction.co/search-ads/search-ads-services
---

# Pull Apple Ads performance and read the paid competition

This flow spans two different scopes. `searchads_get_*` reads **your own** SearchAds.com / MobileAction
CMP account. `searchads_paid_keywords`, `searchads_paying_apps` and `cpp_*` are **market
intelligence** about anyone. Do not present them as the same data.

## Credit budget

Discovery calls (`get_search_ads_integrations`, `get_mmp_integrations`, `get_goals`, `get_goal`,
`get_apps`) are **1 credit each**. `get_reports`, `get_top_apps_report` and `get_bid_history_logs`
are 20. `paid_keywords` and `paying_apps` are 50. Every `cpp_*` tool is **200**.

## Steps — your account

1. `searchads_get_search_ads_integrations` — the Apple Ads orgs connected to the account. Take
   `org_id`; the reporting calls need it.
2. `searchads_get_mmp_integrations` — which mobile measurement partners are wired in. This decides
   whether MMP re-attribution columns will be populated in step 5.
3. `searchads_get_apps` — the apps campaigns run for.
4. `searchads_get_goals` (`track_ids`) then `searchads_get_goal` (`goal_id`) — conversion goals.
   A goal's `state` (`NO_RECENT_EVENT`, `PENDING_FIRST_EVENT`, …) tells you whether its numbers are
   trustworthy before you report on them.
5. `searchads_get_reports` (`report_type`, `report_level`, `start_date`, `end_date`, `org_ids`,
   `goal_id`, `app_id`, `re_attr_type`) — the ASA + MMP report. `report_type` is one of
   `REPORT`, `DAILY`, `SUMMARY`, `TOP_APPS`. `searchads_get_top_apps_report` is the `TOP_APPS`
   preset with `keyword_list` and pagination.
6. `searchads_get_bid_history_logs` (`org_id`, `start_date`, `end_date`) — every bid change, so a
   performance shift can be attributed to an action rather than to the market.

## Steps — the competition

7. `searchads_paid_keywords` (`track_id`, `country_code`, `start_date`, `end_date`) — the paid
   keywords an app bids on, with impression share.
8. `searchads_paying_apps` (`country_code`, `keyword`, `start_date`, `end_date`) — the inverse: who
   is paying for a keyword you care about, and at what impression share.
9. `cpp_by_keyword` (`keyword`, `start_date`, `end_date`, `page`, `country`) — 200 credits — which
   apps run a Custom Product Page against that keyword, with screenshots. Only spend this once
   step 8 has told you the keyword is genuinely contested.

## Failure handling

- Missing/invalid `token` → HTTP 401, empty body, terminal. The account-scoped `searchads_get_*`
  tools additionally return nothing if the key's account has no SearchAds.com integration — an empty
  result here means "not connected", not "no spend".
- No idempotency contract; all of these are reads, so retries are safe but re-charged.
