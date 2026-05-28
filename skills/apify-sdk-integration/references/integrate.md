# Integrate — call an Actor from app code

For when an existing Actor fits (see [`discovery.md`](discovery.md)) and you want
to run it from your application. Uses the **`apify-client`** package — the API
client. (Not `apify`, which is the SDK for *building* Actors.)

Before writing input, fetch the Actor's schema: `apify actors info <id> --input --json`.
Field names vary between Actors (see [`gotchas.md`](gotchas.md)).

## JavaScript / TypeScript

```bash
npm install apify-client
```

```typescript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
```

### Run and wait (short Actors)

`.call()` blocks until the run finishes — use for Actors that complete in a
few minutes.

```typescript
const run = await client.actor('apify/instagram-scraper').call({
    directUrls: ['https://www.instagram.com/apify/'],
    resultsLimit: 50,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
```

### Start and poll (long Actors)

Use `.start()` + `.waitForFinish()` when the Actor is long-running or you need
the run ID immediately.

```typescript
const run = await client.actor('apify/instagram-scraper').start({ /* input */ });
const finished = await client.run(run.id).waitForFinish();
const { items } = await client.dataset(finished.defaultDatasetId).listItems();
```

### Retrieve results

```typescript
// Dataset items (structured data) — paginate large sets
const { items } = await client.dataset(run.defaultDatasetId).listItems({
    limit: 100,
    offset: 0,
});

// Key-value store (files, screenshots, OUTPUT record)
const record = await client.keyValueStore(run.defaultKeyValueStoreId).getRecord('OUTPUT');
```

### Error handling

```typescript
const run = await client.actor('apify/instagram-scraper').call(input);

if (run.status !== 'SUCCEEDED') {
    const { items: log } = await client.log(run.id).get();
    throw new Error(`Actor run ${run.id} ended as ${run.status}`);
}
```

Catch transport errors too: `error.statusCode === 401` means a bad/missing
`APIFY_TOKEN`; a "not found" message means the Actor ID is wrong or deleted.

## Python

```bash
pip install apify-client
```

### Sync

```python
import os
from apify_client import ApifyClient

client = ApifyClient(token=os.environ['APIFY_TOKEN'])

run = client.actor('apify/instagram-scraper').call(run_input={
    'directUrls': ['https://www.instagram.com/apify/'],
    'resultsLimit': 50,
})

items = client.dataset(run['defaultDatasetId']).list_items().items
```

### Start and poll

```python
run = client.actor('apify/instagram-scraper').start(run_input={...})
finished = client.run(run['id']).wait_for_finish()
items = client.dataset(finished['defaultDatasetId']).list_items().items
```

### Async client (asyncio)

```python
from apify_client import ApifyClientAsync

client = ApifyClientAsync(token=os.environ['APIFY_TOKEN'])
run = await client.actor('apify/instagram-scraper').call(run_input={...})
items = (await client.dataset(run['defaultDatasetId']).list_items()).items
```

## Other languages — REST API

No official client? Call the REST API directly with the token as a Bearer header.

```
POST https://api.apify.com/v2/acts/{actorId}/runs        # start a run
GET  https://api.apify.com/v2/actor-runs/{runId}         # poll status
GET  https://api.apify.com/v2/datasets/{datasetId}/items?format=json
Authorization: Bearer <APIFY_TOKEN>
```

Full reference: https://docs.apify.com/api/v2

## Best practices

- **Reuse one client** instance across calls; don't construct per request.
- **Paginate** dataset reads with `limit`/`offset` for large result sets.
- **Set timeouts** — pass the Actor's own timeout input, or `waitSecs` on `.call()`,
  to avoid indefinite blocking.
- **Check pricing before running** pay-per-event Actors (see [`gotchas.md`](gotchas.md)).
