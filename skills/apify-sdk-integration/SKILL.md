---
name: apify-sdk-integration
description: "Work with the Apify platform in JavaScript/TypeScript or Python: discover existing Actors via the apify CLI, call/integrate them into an app with the apify-client package, or build a new Actor with the apify SDK. Use for web scraping, automation, or data extraction — whether finding and running a ready-made Actor or authoring your own."
license: Apache-2.0
metadata:
  repository: https://github.com/apify/agent-skills
---

# Apify Platform

Add web scraping, automation, or data extraction to an app via Apify. Almost
always there is already an Actor that does the job — so the first move is to
**look**, not to build.

## Prerequisites

Before running any Apify command, check the CLI is installed and authenticated:

```bash
apify --version && apify info
```

- If both succeed: proceed.
- If `apify` is missing: `npm install -g apify-cli`.
- If not authenticated: `apify login` (or set `APIFY_TOKEN`). Get a token at
  https://console.apify.com/settings/integrations.

## Decide the path

Always **discover first**, then branch. Never build an Actor before confirming
none exists.

**Step 1 — Discover.** Search the Apify Store for an existing Actor:

```bash
apify actors search "<keywords>" --json --limit 10
```

Read candidate IDs from `items[].username` + `items[].name` (e.g.
`apify/instagram-scraper`). See [`references/discovery.md`](references/discovery.md).

**Step 2 — Branch on whether an Actor fits.** An Actor "fits" when:
- its input schema covers the task (check with `apify actors info <id> --input --json`),
- it is actively maintained (recent runs, not `isDeprecated`),
- its pricing model is acceptable for the use case (see [`references/gotchas.md`](references/gotchas.md)).

Before writing code that reads results, also confirm the Actor's **output field
names** — never assume them from memory (e.g. it's `totalScore`/`title`, not
`averageRating`/`name`). Check for free, no run needed: `apify actors info <id>
--readme` (read the Output / example section) or `apify actors info <id> --json`
(read the dataset schema, if the Actor defines one). Only if neither documents the
fields, do a tiny capped run (e.g. `maxItems: 3`) and inspect one item. Then code
defensively: treat an empty result as expected, log row counts at each stage
(found → filtered → output), and never crash on `[]`.

```
Actor fits?
├─ YES → Integrate (the common case)
└─ NO  → Build
```

- **Fits → Integrate.** Call it from app code with the `apify-client` package,
  then read its dataset / key-value output. Before writing client code, read
  [`references/integrate.md`](references/integrate.md).
- **Nothing fits → Build.** Author a new Actor with the `apify` SDK. Use the
  nested **[apify-actor-development](skills/apify-actor-development/SKILL.md)**
  subskill — it covers scaffolding, schemas, security, logging, standby mode,
  and deployment.

**Confirm with the user before** building a new Actor or running a
pay-per-event (PPE) Actor — both have real cost/time implications.

## Critical: `apify-client` vs `apify`

Two different packages for two different paths — installing the wrong one is the
most common mistake.

| Path | Package | Purpose |
|------|---------|---------|
| Integrate (call Actors) | **`apify-client`** | API client used *from your app* |
| Build (author an Actor) | **`apify`** | SDK used *inside an Actor you write* |

If you are adding scraping to an existing app, you want `apify-client`. Only
reach for `apify` when the task is to build and deploy a new Actor.

## Resources

| Need | Read |
|------|------|
| Find an Actor / search the Store via CLI | [`references/discovery.md`](references/discovery.md) |
| Call an Actor from app code (`apify-client`) | [`references/integrate.md`](references/integrate.md) |
| Build & deploy a new Actor (`apify` SDK) | [`skills/apify-actor-development/SKILL.md`](skills/apify-actor-development/SKILL.md) (subskill) |
| Pricing, gotchas, error recovery | [`references/gotchas.md`](references/gotchas.md) |

## Documentation

Apify's Actors, input schemas, and pricing change often — verify against current
docs rather than relying on memory:

- Apify docs (LLM-friendly): https://docs.apify.com/llms.txt · full: https://docs.apify.com/llms-full.txt
- Crawlee docs (for building Actors): https://crawlee.dev/llms.txt
- API client refs: https://docs.apify.com/api/client/js · https://docs.apify.com/api/client/python
- REST API: https://docs.apify.com/api/v2

If the Apify MCP server is available, use `search-apify-docs` and
`fetch-apify-docs` for live documentation lookups.

