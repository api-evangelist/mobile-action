---
name: Track and diagnose App Store / Google Play keyword rankings
description: Pull an app's ranked keywords, its rank history for a chosen keyword, and the keyword's market metadata from MobileAction, on either store, while staying inside the credit budget.
api: https://api.mobileaction.co
mcp: https://mcp.mobileaction.co/mcp
operations:
  - get_credit_status
  - dashboard_app_search
  - appstore_top_keywords
  - appstore_organic_keywords
  - appstore_keyword_ranking
  - appstore_keyword_history
  - appstore_keyword_metadata
  - googleplay_top_keywords
  - googleplay_keyword_history
generated: '2026-08-13'
method: generated
source: mcp/mobile-action-mcp-tools-list.json (probed tools/list) + https://docs.mobileaction.co/app-store/keyword-services
---

# Track and diagnose keyword rankings

Every tool name below was read from a live `tools/list` on `https://mcp.mobileaction.co/mcp`.
Over REST the same calls are `GET https://api.mobileaction.co/...?token=YOUR_API_KEY`.

## Before you start

- MobileAction meters in **credits**, not requests. Call `get_credit_status` first and read
  `creditRemaining` / `creditResetTime`. There is no documented 429; plan against the balance.
- Costs in this flow: `appstore_keyword_ranking` 3, `appstore_keyword_metadata` 5,
  `appstore_keyword_history` 10, `appstore_top_keywords` 20, `appstore_organic_keywords` 50.
  Work cheap-to-expensive.
- `track_id` is the join key: a numeric iTunes id on the App Store (`284882215`), a package name on
  Google Play (`com.facebook.katana`). Use `dashboard_app_search` if you only have a name, and
  `ma_app_match` to get the same app's id on the other store.

## Steps

1. `get_credit_status` — confirm the balance covers the plan below.
2. `dashboard_app_search` with `query` = the app name → take `track_id`.
3. `appstore_top_keywords` (`track_id`, `country_code`, `date`, optional `device`, `limit`) — the
   ranked keyword list for one day, with `searchVolume` and `rank`. 20 credits, so pull it once.
4. For each keyword worth investigating, `appstore_keyword_history`
   (`track_id`, `country_code`, `keyword`, `start_date`, `end_date`) — **max 30 days per request**;
   longer windows must be chunked client-side.
5. `appstore_keyword_metadata` (`country_code`, `keyword`) — search volume, chance score and app
   count, i.e. whether the keyword is worth the fight.
6. `appstore_keyword_ranking` (`track_id`, `country_code`, `keywords`, `date`) — up to **100
   comma-separated keywords** in one 3-credit call. This is the cheapest way to re-check a watchlist
   daily; prefer it over repeating step 3.
7. Google Play: swap the `appstore_` prefix for `googleplay_`. Google Play tools take `lang_code`
   where App Store tools take `country_code` on the app-info endpoints, and have no `device`.

## Reading the result

- Rankings come back per `appKind` (`IPHONE` / `IPAD`) on the App Store — one keyword yields two rows.
- `date` is epoch milliseconds, not ISO.
- Rank is per storefront: always carry `country_code` through the whole analysis.

## Failure handling

- A missing or invalid `token` returns **HTTP 401 with an empty body**. It is terminal, not
  transient — do not retry. The 401 still carries `x-credit-cost`.
- Read `X-Credit-Remaining` / `X-Credit-Reset` after every REST call; over MCP the same values are in
  the `credits` object on each result.
- There is no idempotency key. Every one of these operations is a read, so a retry is safe but costs
  credits again.
