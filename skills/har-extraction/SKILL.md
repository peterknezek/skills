---
name: har-extraction
description: Use when the user has a HAR file (browser network recording) and wants JSON API mocks, MSW handlers for tests/Storybook/local dev, Chrome DevTools Local Overrides, or to reproduce a bug from a recorded session.
dependencies:
  - har-to-mocks
  - mocks-to-msw
  - msw
---

# HAR Extraction & Mock Generation

Turn a HAR file into real-response mocks with [har-to-mocks](https://peterknezek.github.io/har-to-mocks/) (v3, Node.js ≥ 22), then serve them with MSW via mocks-to-msw, or with Chrome DevTools Local Overrides.

```
HAR → har-to-mocks → <dir>/<url path>/<METHOD>.json → mocks-to-msw → MSW → app / Storybook / tests
                   └ --format=chrome-overrides → <dir>/<host>/<path> → Chrome Sources → Overrides
```

## Agent workflow (non-interactive)

`--interactive` needs a real terminal. As an agent, use the list → select → dry run → write loop instead:

```bash
# 1. List matching requests. No output folder = nothing is written.
npx har-to-mocks recording.har --url=/api

# 2. Preview: Status column (create / update / skip) and folder tree
npx har-to-mocks recording.har ./src/mocks/api --url=/api --dry-run

# 3. Optional: keep only some rows, using the numbers from the # column
npx har-to-mocks recording.har ./src/mocks/api --url=/api --select=1,3,5-7 --dry-run

# 4. Write: same command without --dry-run
npx har-to-mocks recording.har ./src/mocks/api --url=/api --select=1,3,5-7
```

Rules:
- Always dry-run first and show the user the table. Rows marked `update` overwrite existing files.
- `--select` numbers refer to the list produced by the **same** `--url`, `--method` and `--type` filters. Re-list if you change filters.
- Rows marked `skip` share a path and method with a later row (for example `?page=1` and `?page=2`). Only the last one is written unless you `--select` the one you want. Use the Query column to tell them apart.
- If the user is at a terminal and wants to pick by eye, suggest they run `--interactive` themselves. It prints the equivalent `--select` value for repeat runs.

## Filters and defaults

| Flag | Values | Default |
|------|--------|---------|
| `-u, --url` | case-sensitive **substring** of the full URL, query included (not a regex) | all |
| `-m, --method` | `GET` `POST` `PUT` `DELETE` `PATCH`, repeatable: `-m GET -m POST` | `GET` |
| `-t, --type` | `xhr` `fetch` `document`, **one value only** | `xhr` |
| `-f, --format` | `mocks` `chrome-overrides` | `mocks` |
| `-s, --select` | rows by `#`: `1,3,5-7`, repeatable | all |
| `-i, --interactive` | checkbox picker with JSON preview | off |
| `--dry-run` | show table + tree, write nothing | off |

"I don't see my requests": the defaults are GET + xhr. Add `--type=fetch` and/or `--method=POST`. Chrome records `fetch()` calls as `fetch` and XMLHttpRequest/axios calls as `xhr`. For both types, run once per type into the same folder.

## Pick the recipe by use case

| User says | Command | Then |
|-----------|---------|------|
| "Backend isn't ready / is down / is behind a VPN" | `npx har-to-mocks staging.har ./src/mocks/api --url=/api --dry-run` | MSW in dev (see below) |
| "Mocks for Storybook" | `npx har-to-mocks file.har ./src/mocks/api --url=/api` | Storybook setup in [REFERENCE.md](./REFERENCE.md#storybook-integration) |
| "Fixtures for Vitest / Jest / RTL / Playwright / Cypress" | `npx har-to-mocks file.har ./src/mocks/api --url=/api -m GET -m POST` | `setupServer` in [REFERENCE.md](./REFERENCE.md#node-tests-vitest--jest) |
| "QA attached a HAR to the ticket" | list → find the failing call → `--select=<n>` | Replay it in a story or test |
| "Debug production without a deploy" / "show designers real data" | `npx har-to-mocks prod.har ./overrides --format=chrome-overrides --dry-run` | Chrome **Sources → Overrides → Select folder for overrides**, then reload |
| "Stable data for demos" | `npx har-to-mocks golden-demo.har ./demo/mocks/api --url=/api` | MSW in the demo build |
| "Same endpoint, different query params" | `--select` the one you want, or use `chrome-overrides` (one file per query string) | |

## Output formats

**`mocks` (default)**: `<output>/<url path>/<METHOD>.json`. The file holds only the raw response body (no headers, no status). Example: `GET /api/users/42` → `src/mocks/api/api/users/42/GET.json`. Paths are concrete, so there are no `:id` patterns.

**`chrome-overrides`**: `<output>/<host>/<path>`, with the query string kept in the file name. Chrome matches by URL only, so GET and POST to one URL share a file. Overrides always return status 200 with the real response headers. A path can't be both a file and a folder (`/api/users` and `/api/users/1` conflict). Binary bodies are decoded. Point DevTools at the exact folder that was written.

## MSW via mocks-to-msw

```bash
npm install -D msw mocks-to-msw
npx msw init ./public --save
npx -p mocks-to-msw generate-mocks --folder=src/mocks/api --output=src/mocks/mocks.ts
```

Use the same folder for har-to-mocks output and `--folder`. The generated keys (`'/api/users/GET'`) then match the real URL paths. Re-run `generate-mocks` after every extraction.

```typescript
// src/mocks/handlers.ts
import { createMockHandler } from 'mocks-to-msw';
import mocks from './mocks';

// createMockHandler returns only { mock }. Build the handlers array yourself.
export const { mock } = createMockHandler<keyof typeof mocks>({
  mocks,
  debug: true,
  // origin: 'https://api.example.com', // needed when the app calls an absolute URL
});

export const handlers = [
  mock.get('/api/users'),
  mock.post('/api/users', (data) => ({ ...data, id: 'new-id' })),
];
```

mocks-to-msw supports GET, POST, PUT and DELETE. `PATCH.json` files are skipped by `generate-mocks`, so write a plain MSW `http.patch` handler for those.

## Safety

- HAR files contain cookies, auth headers and tokens. The mock files keep response bodies only, but those can still contain personal data. Review the output before committing it, and don't commit the `.har` itself.
- Without an output folder, nothing is written. With one, prefer `--dry-run` first.
- Errors such as `File not found`, `File is not valid JSON` and `File is not a valid HAR file (missing "log.entries")` mean a wrong path or a file that isn't a HAR export. If the mock files come out empty, the HAR was saved without response bodies. Re-export it from the Network tab with content.

## Known limits

- **GraphQL**: all operations hit one path + method and collide. Use `--select` to keep one. Operation splitting is not implemented.
- **Query variants** in `mocks` format: one file per path + method. Choose with `--select`.

See [REFERENCE.md](./REFERENCE.md) for recording HARs in Chrome/Firefox/Safari, full output samples, MSW browser/Node/Storybook setup, and troubleshooting.
