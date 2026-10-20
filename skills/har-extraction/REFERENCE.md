# HAR Extraction Reference

Detailed reference for [har-to-mocks](https://github.com/peterknezek/har-to-mocks) v3 and [mocks-to-msw](https://github.com/peterknezek/mocks-to-msw).

## Table of Contents

- [Recording HAR Files](#recording-har-files)
- [har-to-mocks CLI](#har-to-mocks-cli)
- [Chrome DevTools Local Overrides](#chrome-devtools-local-overrides)
- [mocks-to-msw Integration](#mocks-to-msw-integration)
- [MSW Setup](#msw-setup)
- [Storybook Integration](#storybook-integration)
- [Troubleshooting](#troubleshooting)

---

## Recording HAR Files

1. Open DevTools and go to the **Network** tab.
2. Enable **Preserve log**. Clear the list to drop unrelated requests.
3. Use the app to trigger the API calls you want to mock.
4. Export:
   - **Chrome / Edge**: the download icon (**Export HAR…**) or right-click → **Save all as HAR**.
   - **Firefox**: gear menu or right-click → **Save All As HAR**.
   - **Safari**: Network tab → **Export**.

The export must include response bodies, because they become the mock data. Name files after the flow (`checkout-flow.har`, `ticket-4821.har`). HAR files contain cookies and tokens, so don't commit them.

---

## har-to-mocks CLI

```bash
npm install -g har-to-mocks   # or use npx har-to-mocks
```

```
har-to-mocks <file.har> [output-folder] [flags]

  -u, --url          substring match in the full URL (case-sensitive)
  -m, --method       GET | POST | PUT | DELETE | PATCH   (default: GET, repeatable)
  -t, --type         xhr | fetch | document             (default: xhr, single value)
  -f, --format       mocks | chrome-overrides           (default: mocks)
  -s, --select       rows by # column, e.g. 1,3,5-7     (repeatable)
  -i, --interactive  checkbox picker with response preview
      --dry-run      show tables and folder tree, write nothing
  -v, --version
  -h, --help
```

### Listing (no output folder)

```bash
npx har-to-mocks ./file.har --url=api/service
```

```
Filtered requests:

 # │ Name          │ Method │ Path                        │ Query
 ──┼───────────────┼────────┼─────────────────────────────┼──────
 1 │ userRoles     │ GET    │ /api/service/userRoles      │
 2 │ currentUserId │ GET    │ /api/service/currentUserId  │
 3 │ active        │ GET    │ /api/service/clients/active │
```

Nothing is written without an output folder.

### Dry run with output folder

```bash
npx har-to-mocks ./file.har ./mocks --url=api/service --dry-run
```

```
 # │ Name          │ Method │ Path                        │ Query │ Status
 ──┼───────────────┼────────┼─────────────────────────────┼───────┼───────
 1 │ userRoles     │ GET    │ /api/service/userRoles      │       │ create
 2 │ currentUserId │ GET    │ /api/service/currentUserId  │       │ create
 3 │ active        │ GET    │ /api/service/clients/active │       │ create

Folder tree which will be applied:

└─ mocks
   └─ api
      └─ service
         ├─ userRoles
         │  └─ GET.json
         ├─ currentUserId
         │  └─ GET.json
         └─ clients
            └─ active
               └─ GET.json

No files were written. If you want to write files remove the (--dry-run) flag.
```

Status values:
- **create**: the file doesn't exist yet.
- **update**: the file exists and will be overwritten. The tree also marks it `[UPDATE]`.
- **skip**: a later row writes the same file (same path + method, different query). A hint suggests `--select` or `--interactive`.

### Selecting rows (`--select`)

Numbers come from the `#` column of the list made with the same `--url`, `--method` and `--type`. Use commas, ranges (`5-7`) and repeated flags.

```bash
npx har-to-mocks ./file.har ./mocks --url=api/service --select=1,3
```

```
Selected requests:

 # │ Name      │ Method │ Path                        │ Query │ Status
 ──┼───────────┼────────┼─────────────────────────────┼───────┼───────
 1 │ userRoles │ GET    │ /api/service/userRoles      │       │ create
 3 │ active    │ GET    │ /api/service/clients/active │       │ create
```

Query variants: `npx har-to-mocks ./file.har ./mocks --url=?page= --select=2` keeps only the second `?page=` response.

### Interactive mode (`-i`), for humans at a TTY

```bash
npx har-to-mocks ./file.har ./mocks --interactive
```

The picker shows a checkbox list with the focused endpoint's JSON response below it (up to 12 lines). Rows that would be written by default are pre-selected (or the rows given by `--select`). Keys: `↑/↓` navigate, `Space` toggle, `a` all, `i` invert, `Enter` confirm. After you pick, it prints the equivalent `--select` value and the folder tree, then asks for confirmation. It works with both formats.

Agents can't drive this prompt. Use the list → `--select` → `--dry-run` loop instead.

### `mocks` output layout

`<output>/<url path>/<METHOD>.json`, containing the raw response body only (no headers, no status code). This is the layout that mocks-to-msw and [connect-api-mocker](https://www.npmjs.com/package/connect-api-mocker) consume.

```
src/mocks/api/                 ← output folder
└── api/
    ├── users/
    │   ├── GET.json           # GET  /api/users
    │   ├── POST.json          # POST /api/users
    │   └── 42/
    │       └── GET.json       # GET  /api/users/42  (concrete path, not :id)
    └── products/
        └── GET.json           # GET  /api/products
```

Entries with an unparsable URL are skipped with a warning.

---

## Chrome DevTools Local Overrides

Use this to replace responses on any site, including production, with no proxy and no code change. Good for debugging production, design handoff and prototyping.

```bash
npx har-to-mocks ./prod.har ./overrides --format=chrome-overrides --dry-run
npx har-to-mocks ./prod.har ./overrides --format=chrome-overrides
```

```
└─ overrides
   └─ sample.com
      ├─ api
      │  └─ service
      │     ├─ userRoles
      │     └─ clients
      │        └─ active
      └─ complete
         ├─ search?q=javascript
         └─ search?q=python
```

To load it: DevTools → **Sources → Overrides → Select folder for overrides**, pick the same folder, allow access, then reload the page. The user has to do this step in the browser.

Behaviour:
- File names follow Chrome's scheme: `<host>/<path>`, query string kept in the name, `index.html` for paths ending in `/`, special characters escaped, and very long URLs under `longurls/`. The `longurls` names depend on the absolute folder path, so select exactly the folder that was written.
- Matching is by URL only, not by method. GET and POST to the same URL share one file, and the last one wins (status **skip**). Different query strings get separate files.
- Overridden responses return status 200 with the real request's headers.
- A path can't be both a file and a folder. `/api/users` plus `/api/users/1` fails with `A file and a folder would have the same path`. Use `--select` to drop one.
- Base64 bodies (images, fonts) are written as the original bytes.

---

## mocks-to-msw Integration

```bash
npm install -D msw mocks-to-msw   # msw >= 2
```

### Generate the typed mocks file

The CLI binary is `generate-mocks`. There is no `mocks-to-msw generate` command.

```bash
npx -p mocks-to-msw generate-mocks --folder=src/mocks/api --output=src/mocks/mocks.ts
```

Add it as a script so it's easy to re-run after each extraction:

```json
{
  "scripts": {
    "mocks:generate": "generate-mocks --folder=src/mocks/api --output=src/mocks/mocks.ts"
  }
}
```

Keys are paths relative to `--folder`, plus the method. Values are import promises, and the file has a **default** export:

```typescript
// src/mocks/mocks.ts (generated)
const mocks = {
  '/api/products/GET': import('./api/api/products/GET.json'),
  '/api/users/42/GET': import('./api/api/users/42/GET.json'),
  '/api/users/GET': import('./api/api/users/GET.json'),
  '/api/users/POST': import('./api/api/users/POST.json'),
} as const;

export default mocks;
```

Keep `--folder` equal to the har-to-mocks output folder, so the keys equal the real URL paths. Only `GET`, `POST`, `PUT` and `DELETE` files are picked up. `PATCH.json` is skipped with a warning.

### Create handlers

```typescript
// src/mocks/handlers.ts
import { createMockHandler } from 'mocks-to-msw';
import mocks from './mocks';

export const { mock } = createMockHandler<keyof typeof mocks>({
  mocks,
  debug: true,                         // logs "[mocks-to-msw] Mock file was loaded…"
  origin: 'https://api.example.com',   // optional; resolved with new URL(path, origin)
});

export const handlers = [
  mock.get('/api/users'),
  mock.get<{ id: string; name: string }>('/api/users/42'),
  mock.post('/api/users', (data) => ({ ...data, createdAt: new Date().toISOString() })),
];
```

- `createMockHandler` returns `{ mock }` only, not `handlers`.
- `mock.get|post|put|delete(path, modifier?)` is type-checked. A path must have a mock file for that method.
- The modifier gets the parsed JSON and returns the response body. The response is always `HttpResponse.json(...)` with status 200. For error, empty or delayed responses, use plain MSW `http.*` handlers.
- Without `origin`, MSW resolves the path against the page origin. Set `origin` when the app calls a different API host.

---

## MSW Setup

### Browser (local dev, demos)

```bash
npx msw init ./public --save
```

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

```typescript
// src/main.tsx: start the worker before rendering
if (import.meta.env.DEV && import.meta.env.VITE_MOCKS === 'true') {
  const { worker } = await import('./mocks/browser');
  await worker.start({ onUnhandledRequest: 'bypass' });
}
```

### Node tests (Vitest / Jest)

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// vitest.setup.ts
import { afterAll, afterEach, beforeAll } from 'vitest';
import { server } from './src/mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Override a handler per test with `server.use(http.get('*/api/users', () => HttpResponse.json([])))`.

The JSON imports need `resolveJsonModule` in `tsconfig.json`. Bundlers like Vite and webpack handle JSON imports natively.

### Playwright / Cypress

Run the app with the browser worker enabled (for example `VITE_MOCKS=true`) and point the E2E tests at it. The same recorded responses then back the end-to-end runs.

---

## Storybook Integration

```bash
npm install -D msw-storybook-addon
npx msw init ./public --save
```

```typescript
// .storybook/main.ts
const config: StorybookConfig = {
  // ...
  staticDirs: ['../public'], // serves mockServiceWorker.js
};
```

```typescript
// .storybook/preview.tsx
import { initialize, mswLoader } from 'msw-storybook-addon';
import { handlers } from '../src/mocks/handlers';

initialize();

const preview: Preview = {
  parameters: { msw: { handlers } },
  loaders: [mswLoader],
};

export default preview;
```

### Story-level overrides

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { delay, http, HttpResponse } from 'msw';
import { mock } from '../mocks/handlers';
import { UserList } from './UserList';

const meta = { component: UserList } satisfies Meta<typeof UserList>;
export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {};

export const Empty: Story = {
  parameters: { msw: { handlers: [mock.get('/api/users', () => [])] } },
};

export const ServerError: Story = {
  parameters: {
    msw: { handlers: [http.get('*/api/users', () => new HttpResponse(null, { status: 500 }))] },
  },
};

export const Loading: Story = {
  parameters: {
    msw: { handlers: [http.get('*/api/users', async () => { await delay('infinite'); })] },
  },
};
```

Story-level `msw.handlers` replace the global array for that story. To keep the other defaults, use the named form, for example `msw: { handlers: { users: [...] } }`, and define the globals with matching keys.

---

## Troubleshooting

| Symptom | Cause / fix |
|---------|-------------|
| `File not found: …` | Wrong HAR path. |
| `File is not valid JSON` / `File is not a valid HAR file (missing "log.entries")` | Not a HAR export, or a truncated file. Re-export it. |
| Table is empty | The defaults are GET + xhr. Try `--type=fetch`, `-m POST`, and check that `--url` matches the case and is a substring (not a regex). |
| `Flag --type can only be specified once` | `--type` takes one value. Run once per type. |
| Many rows show `skip` | Same path + method with different queries, or GraphQL. Pick one with `--select`. |
| Mock JSON files are empty | The HAR was exported without response bodies. |
| `Mock file was not found` (debug log) | Regenerate `mocks.ts`. Check that the key path equals the request path, and that `--folder` is the har-to-mocks output folder. |
| MSW doesn't intercept | `mockServiceWorker.js` is missing from `public/`, so run `npx msw init ./public --save`. The worker must start before the first request. Check `origin` against the app's API base URL. |
| Type error on `mock.get('/x')` | No `x/GET.json` in `mocks.ts`. Regenerate it, or the method is PATCH (unsupported). |
| Chrome override not applied | Overrides are disabled or the wrong folder is selected. Check that the host folder matches the page's API host. DevTools must stay open. |

---

## Related Resources

- [har-to-mocks site](https://peterknezek.github.io/har-to-mocks/): use cases, FAQ, CLI reference
- [har-to-mocks on GitHub](https://github.com/peterknezek/har-to-mocks)
- [mocks-to-msw](https://github.com/peterknezek/mocks-to-msw)
- [MSW docs](https://mswjs.io/)
- [msw-storybook-addon](https://github.com/mswjs/msw-storybook-addon)
- [Chrome Local Overrides](https://developer.chrome.com/docs/devtools/overrides)
