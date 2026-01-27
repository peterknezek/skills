# HAR Extraction & Mock Generation Skill

## Description

This skill enables the extraction of JSON mock files from HAR (HTTP Archive) files and their integration with Mock Service Worker (MSW) for use in development, testing, and Storybook environments.

## Triggers

- When working with HAR files or network recordings
- When setting up API mocks for testing
- When configuring MSW (Mock Service Worker)
- When integrating mocks with Storybook
- When asked to "extract mocks from HAR"
- When asked to "generate mocks from network traffic"
- When setting up mock data for frontend development

## Workflow Overview

```
HAR File → har-to-mocks → JSON Mocks → mocks-to-msw → MSW Handlers → Storybook/Tests
```

---

## Step 1: Recording HAR Files

HAR files capture network traffic from browser DevTools.

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

---

## Step 2: Extract Mocks with har-to-mocks

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

This enables:
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
│   ├── users/
│   │   └── [id]/
│   │       └── GET.json      # GET /api/users/:id
│   └── products/
│       └── GET.json          # GET /api/products
```

---

## Step 3: Integrate with mocks-to-msw

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

## Step 4: MSW Setup

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

---

## Step 5: Storybook Integration

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
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { mock } from '../mocks/handlers';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  component: Button,
};

export default meta;
type Story = StoryObj<typeof Button>;

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
```

---

## Complete Example Workflow

### 1. Record Network Traffic

```bash
# User records HAR file from browser DevTools
# Save as: user-flow.har
```

### 2. Extract Mocks

```bash
# Extract all API calls interactively
npx har-to-mocks user-flow.har ./src/mocks --interactive

# Or extract specific endpoints
npx har-to-mocks user-flow.har ./src/mocks --url "/api" --method GET
npx har-to-mocks user-flow.har ./src/mocks --url "/api" --method POST
```

### 3. Generate Type-Safe Imports

```bash
npx mocks-to-msw generate ./src/mocks
```

### 4. Create Handlers

```typescript
// src/mocks/handlers.ts
import { createMockHandler } from 'mocks-to-msw';
import { mocks } from './mocks';

export const { mock, handlers } = createMockHandler<keyof typeof mocks>({
  mocks,
  debug: process.env.NODE_ENV === 'development',
  origin: process.env.VITE_API_URL,
});
```

### 5. Use in Tests

```typescript
// src/tests/setup.ts
import { beforeAll, afterAll, afterEach } from 'vitest';
import { server } from '../mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterAll(() => server.close());
afterEach(() => server.resetHandlers());
```

### 6. Use in Storybook

Mocks are automatically available in all stories via the preview configuration.

---

## Troubleshooting

### HAR file not generating mocks

- Ensure the HAR file contains response content (not just headers)
- Check if the requests match your filter criteria (URL, method, type)
- Use `--dry-run` to preview what would be extracted

### MSW not intercepting requests

- Verify the service worker is initialized before making requests
- Check that the URL origin matches your configuration
- Enable debug mode to see which requests are being handled

### Type errors with mocks

- Regenerate the mocks.ts file after adding/removing mock files
- Ensure the mock file paths match the expected structure

### Storybook mocks not working

- Verify msw-storybook-addon is in the addons array
- Check that mswLoader is included in loaders
- Ensure the public directory with mockServiceWorker.js is served

---

## Related Resources

- [har-to-mocks](https://github.com/peterknezek/har-to-mocks) - CLI for extracting mocks from HAR files
- [mocks-to-msw](https://github.com/peterknezek/mocks-to-msw) - Adapter for MSW integration
- [MSW Documentation](https://mswjs.io/) - Mock Service Worker official docs
- [msw-storybook-addon](https://github.com/mswjs/msw-storybook-addon) - Storybook addon for MSW
