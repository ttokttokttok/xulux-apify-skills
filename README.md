# Coding-agent skills

A reframing of the Apify [agent-skills](https://github.com/apify/agent-skills) collection for the **coding-agent** use case: an LLM writing code that integrates an SDK into an app, rather than a user driving the Apify platform interactively.

## The reframing

Upstream treats Actor development as a top-level concern — each Actor-related skill sits as a sibling, and the agent picks one by description match.

For a coding agent that has been asked *"add web scraping to my Next.js app"* or *"scrape Instagram comments from this Python service,"* the right first move is almost never "build a new Actor." It's **discover an existing Actor, then call it from the app**. Building only happens when discovery turns up nothing.

This repo restructures the skills around that workflow:

- A single **router skill** (`apify-sdk-integration`) is the entry point. It walks the discover → fit-check → integrate-or-build decision tree.
- The deep "build an Actor" path is a **nested subskill** (`apify-actor-development`), reached when the router lands on the build branch — or matched directly when the user is already inside Actor-authoring work ("debug my Actor," "fix this standby handler").
- Sibling skills cover adjacent SDK workflows that don't fit the discover/build axis: wrapping existing code (`apify-actorization`), generating output schemas (`apify-generate-output-schema`), and a high-level multi-platform scraping helper (`apify-ultimate-scraper`).

The net effect: a coding agent answering an integration request loads the lightweight router first, runs `apify actors search`, and in the common case never has to touch the heavy Actor-authoring material at all.

## Why SDK-specific

The collection is deliberately scoped to **SDK integration concerns** — calling Actors from app code via `apify-client`, building Actors with the `apify` SDK, and the schemas/lifecycle/deployment around that. It does not try to cover broader Apify platform usage (Console workflows, billing, team management, marketplace publishing as a business).

The `apify-client` vs `apify` package distinction — the most common integration mistake — is front-loaded in the router skill rather than buried in a reference.

## Layout

```
skills/
├── apify-sdk-integration/              Router. Discover → integrate or build.
│   ├── SKILL.md
│   ├── references/                     Discovery, integrate recipe, Actor index, gotchas.
│   └── skills/
│       └── apify-actor-development/    Subskill. Deep dive on building Actors:
│           ├── SKILL.md                templates, schemas, standby, logging, README, deploy.
│           └── references/
├── apify-actorization/                 Wrap existing code as an Actor (`apify init` path).
├── apify-generate-output-schema/       Generate dataset/output/KV schemas from Actor source.
└── apify-ultimate-scraper/             Universal scraper across ~100 Actors / 15+ platforms.
```

## Skill conventions used here

- **Skill vs. reference.** A `SKILL.md` with frontmatter (`name:`, `description:`) is independently discoverable by the agent. Files under `references/` are plain markdown that only get loaded when the parent `SKILL.md` links to them. Promote a reference to a subskill only when it has (a) an independent entry intent, (b) its own workflow, and (c) enough depth to justify the overhead.
- **Nested subskills** live at `<parent>/skills/<child>/SKILL.md`. The parent links to them explicitly; they remain discoverable on their own description.
- **Prerequisites get hoisted to the router.** Subskills assume the parent has run CLI install / auth checks, and only re-do them if invoked directly.

## Upstream

Based on https://github.com/apify/agent-skills (Apache-2.0). Skill content is unmodified where workable; the restructuring is the value-add.
