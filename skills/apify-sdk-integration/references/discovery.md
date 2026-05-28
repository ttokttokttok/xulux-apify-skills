# Discovery — finding the right Actor

Goal: identify one Actor that fits the task, with as few CLI calls as possible.
Work top to bottom and stop as soon as you have a confirmed fit.

## Step 1 — Check the curated index first

Look in [`actor-index.md`](actor-index.md) for the target platform or use case
(Instagram, TikTok, Google Maps, reviews, SEO, web crawling, etc.). If there's a
match, you have the Actor ID with zero API calls — skip straight to Step 3 to
confirm fit.

The index is curated but not exhaustive and can go stale. If the platform isn't
listed, or the listed Actors don't match the specific task, fall through to live
search.

## Step 2 — Live search (fallback)

```bash
apify actors search "<keywords>" --json --limit 10
```

Read candidates from the JSON `items[]`. The Actor ID is `username/name`:

```
items[].username + "/" + items[].name   →  e.g. "apify/instagram-scraper"
items[].title                            →  human-readable name
items[].stats.totalUsers30Days           →  popularity signal
items[].currentPricingInfo.pricingModel  →  FREE | PAY_PER_EVENT | ...
```

Construct the ID from the JSON. Do **not** guess IDs from memory — e.g. the top
Google Maps Actor is `compass/crawler-google-places`, not `apify/google-maps-scraper`.

Use `--limit 10` and read `items[]` directly — never `--limit 1`, and ignore
`count` (Store search has a paging bug). See [`gotchas.md`](gotchas.md).

## Step 3 — Confirm the Actor fits

Before integrating or running, verify with a single info call:

```bash
apify actors info <username/name> --input --json
```

An Actor fits when all three hold:
- **Input schema covers the task** — the fields you need exist (read the schema
  properties; field names vary, e.g. `maxResults` vs `resultsLimit` vs `maxItems`).
- **Actively maintained** — `isDeprecated` is not `true`; has recent runs.
- **Pricing acceptable** — check `currentPricingInfo.pricingModel`. For
  `PAY_PER_EVENT`, estimate cost and confirm with the user. See [`gotchas.md`](gotchas.md).

For human-readable docs instead of the schema: `apify actors info <id> --readme`.

## Outcome

- **Fits** → proceed to [`integrate.md`](integrate.md) (call it with `apify-client`).
- **Nothing fits** after index + a broadened search → use the
  [`apify-actor-development`](../skills/apify-actor-development/SKILL.md) subskill,
  and confirm with the user before building.

## Efficiency

The whole discovery path should be ~1–3 CLI calls: (index lookup, free) →
optional `actors search` → one `actors info`. Do not re-run `search` with many
keyword variations or call `info` repeatedly on the same Actor.
