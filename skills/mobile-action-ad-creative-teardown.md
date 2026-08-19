---
name: Tear down a competitor's ad creative strategy
description: Use MobileAction Ad Intelligence to establish whether an app advertises, then break its creatives down by network, country, language, media type and dimension over a rolling 30-day window.
api: https://api.mobileaction.co
mcp: https://mcp.mobileaction.co/mcp
operations:
  - get_credit_status
  - adintel_check_badge
  - adintel_top_advertisers
  - adintel_network_distribution
  - adintel_country_distribution
  - adintel_language_distribution
  - adintel_creative_type_distribution
  - adintel_creative_dimension_distribution
  - adintel_get_creatives
generated: '2026-08-13'
method: generated
source: mcp/mobile-action-mcp-tools-list.json (probed tools/list) + https://docs.mobileaction.co/ad-intelligence/ad-intelligence-services
---

# Tear down a competitor's ad creative strategy

This is the most expensive family in the API. Sequence matters: qualify first, enumerate last.

## Credit budget

`adintel_check_badge` 5 · `country_distribution` / `language_distribution` / `network_distribution` 10 ·
`creative_type_distribution` / `creative_dimension_distribution` 20 ·
`ad_publisher_creative_dimension_distribution` 50 · `adintel_top_advertisers` **150** ·
`adintel_get_creatives` **200**. Call `get_credit_status` before starting.

## Steps

1. `adintel_check_badge` (`track_id`) — 5 credits. Tells you whether the app is an **advertiser**, a
   **publisher**, or neither. If it is not an advertiser, stop here; the 200-credit call would return
   nothing useful.
2. `adintel_network_distribution` (`track_id`, `start_date`, `end_date`, optional `media_type`,
   `country_code`) — where the spend goes. **Max 30 days per request.**
3. `adintel_country_distribution` and `adintel_language_distribution` — the geographic and language
   footprint, same 30-day cap.
4. `adintel_creative_type_distribution` then `adintel_creative_dimension_distribution` — image vs
   video vs HTML vs playable, and the sizes in use. Filter with `network` and `media_type` to narrow
   before spending on step 5.
5. `adintel_get_creatives` (`track_id`, `page`, `start_date`, `end_date`, `country_code`, `network`,
   `media_type`) — 200 credits, the actual creative list with `mediaOriginalUrl`. Paginate with
   `page` (zero-indexed). Always pass the narrowest filters the earlier steps justified.
6. For the category view rather than one app, `adintel_top_advertisers` (`start_date`, `end_date`,
   `country_code`, `platform_id`, `category`, `network`, `first_seen`, `page`) — 150 credits.

## Rules

- 30-day maximum per request across this whole family. Chunk longer studies by date range.
- Every distribution tool accepts the same filter vocabulary; apply `country_code` + `network`
  consistently or the percentages will not reconcile between calls.
- `adintel_ad_publisher_creative_dimension_distribution` answers the inverse question — where a
  given creative is *published* — and costs 50.

## Failure handling

- Missing/invalid `token` → HTTP 401, empty body, terminal.
- No documented status code for credit exhaustion; check `X-Credit-Remaining` before each
  200-credit call rather than relying on an error.
