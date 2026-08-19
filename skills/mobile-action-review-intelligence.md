---
name: Mine app store reviews for product signal
description: Pull an app's reviews on either store with rating/country/text filters, then run MobileAction's word-frequency and sentiment analysis over the same window to turn free text into ranked themes.
api: https://api.mobileaction.co
mcp: https://mcp.mobileaction.co/mcp
operations:
  - get_credit_status
  - dashboard_app_search
  - appstore_app_reviews_v2
  - appstore_app_review_analysis
  - googleplay_app_reviews
  - googleplay_app_review_analysis
  - appstore_app_version_list_detailed
generated: '2026-08-13'
method: generated
source: mcp/mobile-action-mcp-tools-list.json (probed tools/list) + https://docs.mobileaction.co/app-store/app-services
---

# Mine app store reviews for product signal

## Credit budget

`appstore_app_reviews` / `appstore_app_reviews_v2` 5 · `appstore_app_review_analysis` 10 ·
`appstore_app_version_list_detailed` 50. Check `get_credit_status` first.

## Steps

1. `dashboard_app_search` (`query`) → `track_id`, if you do not already have it.
2. `appstore_app_review_analysis` (`track_id`, `start_date`, `end_date`, optional `rating`,
   `countries`, `text_query`) — **run this before pulling raw reviews.** It returns, per word:
   `frequency`, `numReviews`, `averageRating`, a `sentiment` distribution across the 1–5 star
   buckets, and a `trend`. One 10-credit call replaces reading hundreds of reviews.
   **Max 30 days per request.**
3. Filter to the negative tail — re-run step 2 with `rating: "1,2"` — and diff the word list against
   the unfiltered run. Words that only appear in the negative run are the complaint themes.
4. `appstore_app_reviews_v2` (`track_id`, `start_date`, `end_date`, `rating`, `countries`,
   `text_query`, `replied_review`, `page`) — now pull the actual review text for the two or three
   themes worth quoting. Paginate with `page`; the response carries `totalPages`, `pageNumber`,
   `pageSize`. Use `text_query` so you page through the theme, not the whole corpus.
5. Correlate with releases: `appstore_app_version_list_detailed` (`track_id`, `country_code`,
   `start_date`, `end_date`) returns the update timeline **with the old and new metadata values**,
   so you can line a complaint spike up against the release that caused it. 50 credits — call once.
6. Google Play: `googleplay_app_review_analysis` and `googleplay_app_reviews`. Note the filter
   changes name — App Store review tools take `countries`, the Google Play analysis tool takes
   `languages`.

## Caveats worth stating in any report

- `googleplay_app_reviews_v2` is documented on the Google Play page under an
  `/appstore-app-reviews/v2/` path while its own example passes a Play package id. The published
  path and the documented store disagree; treat results from it with care and prefer
  `googleplay_app_reviews` unless you need pagination.
- 30-day cap per analysis request. Chunk a quarter into three calls and merge client-side.

## Failure handling

- Missing/invalid `token` → HTTP 401, empty body, terminal.
- If the app has never been crawled, the review endpoints return nothing rather than an error;
  queue it with `appstore_app_discovery` / `googleplay_app_discovery` (1 credit) and retry later.
