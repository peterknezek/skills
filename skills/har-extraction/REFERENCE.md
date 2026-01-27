# HAR Extraction Reference Documentation

Complete reference for extracting mocks from HAR files and integrating with MSW.

## Table of Contents

- [Recording HAR Files](#recording-har-files)
- [har-to-mocks CLI](#har-to-mocks-cli)
- [mocks-to-msw Integration](#mocks-to-msw-integration)
- [MSW Setup](#msw-setup)
- [Storybook Integration](#storybook-integration)
- [Troubleshooting](#troubleshooting)

---

## Recording HAR Files

HAR (HTTP Archive) files capture network traffic from browser DevTools.

### How to Record

1. Open Chrome DevTools (F12 or Cmd+Option+I)
2. Go to the **Network** tab
3. Ensure "Preserve log" is checked
4. Perform the actions that trigger the API calls you want to mock
5. Right-click on the network list → **Save all as HAR with content**

### Best Practices

- Clear the network tab before recording to avoid irrelevant requests
- Filter by XHR/Fetch to focus on API calls
- Name HAR files descriptively (e.g., `user-dashboard-flow.har`)
- Include response bodies - they contain the mock data

---

## har-to-mocks CLI

### Installation

```bash
npm install -g har-to-mocks
# or use npx without installation
npx har-to-mocks --help
```

### Basic Usage

```bash
# Extract all GET XHR requests
npx har-to-mocks recording.har ./src/mocks

# Filter by URL pattern
npx har-to-mocks recording.har ./src/mocks --url "/api/users"

# Filter by HTTP method
npx har-to-mocks recording.har ./src/mocks --method POST

# Dry run (preview without writing files)
npx har-to-mocks recording.har ./src/mocks --dry-run
```

### Interactive Mode

Use `--interactive` flag for selective extraction:

```bash
npx har-to-mocks recording.har ./src/mocks --interactive
```

Features:
- Checkbox selection for endpoints
- Live JSON preview while navigating
- Folder tree preview before confirmation
- Keyboard shortcuts: Arrow keys, Space to toggle, 'a' for all, Enter to confirm

### CLI Options

| Option | Description | Default |
|--------|-------------|---------|
| `--url` | Filter by URL (case-sensitive) | - |
| `-m, --method` | HTTP method (GET, POST, PUT, DELETE, PATCH) | GET |
| `-t, --type` | Request type | xhr |
| `--interactive` | Enable interactive selection | false |
| `--dry-run` | Preview changes without writing | false |

### Output Structure

har-to-mocks generates a folder structure compatible with mocks-to-msw:

```
src/mocks/
├── api/
│   ├── users/
│   │   ├── GET.json          # GET /api/users
│   │   └── POST.json         # POST /api/users
│   │   └── [id]/
│   │       └── GET.json      # GET /api/users/:id
│   └── products/
│       └── GET.json          # GET /api/products
```

---

## mocks-to-msw Integration

### Installation

```bash
npm install mocks-to-msw msw --save-dev
```

### Generate Type-Safe Imports

Run the CLI to create the mocks registry:

```bash
npx mocks-to-msw generate ./src/mocks
```

This creates `mocks.ts` with dynamic imports:

```typescript
// src/mocks/mocks.ts (auto-generated)
export const mocks = {
  '/api/users/GET': () => import('./api/users/GET.json'),
  '/api/users/POST': () => import('./api/users/POST.json'),
  '/api/users/[id]/GET': () => import('./api/users/[id]/GET.json'),
  '/api/products/GET': () => import('./api/products/GET.json'),
} as const;
```

### Create Mock Handler

```typescript
// src/mocks/handlers.ts
import { createMockHandler } from 'mocks-to-msw';
import { mocks } from './mocks';

const { mock, handlers } = createMockHandler<keyof typeof mocks>({
  mocks,
  debug: true, // Enable debug logging in development
  origin: 'https://api.example.com', // Optional: prepend origin to URLs
});

export { mock, handlers };
```

### Using Mock Handlers

```typescript
// Type-safe mock handlers
mock.get('/api/users')           // GET request handler
mock.post('/api/users')          // POST request handler
mock.put('/api/users/[id]')      // PUT request handler
mock.delete('/api/users/[id]')   // DELETE request handler

// With response modifier
mock.post('/api/users', (data) => ({
  ...data,
  id: 'custom-id',
  createdAt: new Date().toISOString(),
}))
```

---

## MSW Setup

### Browser Setup (for Storybook/Development)

```typescript
// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

export const worker = setupWorker(...handlers);
```

### Node Setup (for Testing)

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### Initialize MSW

Generate the service worker file:

```bash
npx msw init ./public --save
```

### Test Setup (Vitest/Jest)

```typescript
// src/tests/setup.ts
import { beforeAll, afterAll, afterEach } from 'vitest';
import { server } from '../mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterAll(() => server.close());
afterEach(() => server.resetHandlers());
```

---

## Storybook Integration

### Install Addon

```bash
npm install msw-storybook-addon --save-dev
```

### Configure Storybook

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/react-vite';

const config: StorybookConfig = {
  // ... other config
  addons: [
    '@storybook/addon-essentials',
    'msw-storybook-addon',
  ],
  staticDirs: ['../public'], // Serve MSW worker
};

export default config;
```

### Setup Preview

```typescript
// .storybook/preview.tsx
import { initialize, mswLoader } from 'msw-storybook-addon';
import { handlers } from '../src/mocks/handlers';

// Initialize MSW
initialize();

const preview = {
  parameters: {
    msw: {
      handlers: handlers,
    },
  },
  loaders: [mswLoader],
};

export default preview;
```

### Story-Level Mock Customization

```typescript
// UserList.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { http, HttpResponse } from 'msw';
import { mock } from '../mocks/handlers';
import { UserList } from './UserList';

const meta: Meta<typeof UserList> = {
  component: UserList,
};

export default meta;
type Story = StoryObj<typeof UserList>;

// Use default mocks
export const Default: Story = {};

// Override mocks for specific story
export const WithCustomData: Story = {
  parameters: {
    msw: {
      handlers: [
        mock.get('/api/users', () => [
          { id: '1', name: 'Custom User' },
        ]),
      ],
    },
  },
};

// Simulate error state
export const WithError: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get('https://api.example.com/api/users', () => {
          return new HttpResponse(null, { status: 500 });
        }),
      ],
    },
  },
};

// Simulate loading state
export const Loading: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get('https://api.example.com/api/users', async () => {
          await new Promise(resolve => setTimeout(resolve, 10000));
          return HttpResponse.json([]);
        }),
      ],
    },
  },
};

// Empty state
export const Empty: Story = {
  parameters: {
    msw: {
      handlers: [
        mock.get('/api/users', () => []),
      ],
    },
  },
};
```

---

## Troubleshooting

### HAR file not generating mocks

- Ensure the HAR file contains response content (not just headers)
- Check if the requests match your filter criteria (URL, method, type)
- Use `--dry-run` to preview what would be extracted
- Verify the HAR file is valid JSON

### MSW not intercepting requests

- Verify the service worker is initialized before making requests
- Check that the URL origin matches your configuration
- Enable debug mode to see which requests are being handled
- Ensure `mockServiceWorker.js` is in your public directory

### Type errors with mocks

- Regenerate the mocks.ts file after adding/removing mock files
- Ensure the mock file paths match the expected structure
- Check that JSON files are valid

### Storybook mocks not working

- Verify msw-storybook-addon is in the addons array
- Check that mswLoader is included in loaders
- Ensure the public directory with mockServiceWorker.js is served via staticDirs
- Clear browser cache and restart Storybook

### Common Errors

| Error | Solution |
|-------|----------|
| `Failed to register service worker` | Run `npx msw init ./public --save` |
| `No matching handler` | Check URL patterns and origin config |
| `Module not found: mocks.ts` | Run `npx mocks-to-msw generate ./src/mocks` |
| `HAR parsing error` | Verify HAR file is complete and valid |

---

## Related Resources

- [har-to-mocks](https://github.com/peterknezek/har-to-mocks) - CLI for extracting mocks from HAR files
- [mocks-to-msw](https://github.com/peterknezek/mocks-to-msw) - Adapter for MSW integration
- [MSW Documentation](https://mswjs.io/) - Mock Service Worker official docs
- [msw-storybook-addon](https://github.com/mswjs/msw-storybook-addon) - Storybook addon for MSW
