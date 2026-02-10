# Storybook Interaction Tests Reference

Complete reference for writing play functions and interaction tests in Storybook.

## Table of Contents

- [Play Function Structure](#play-function-structure)
- [Query Reference](#query-reference)
- [Async Patterns](#async-patterns)
- [Common Interaction Patterns](#common-interaction-patterns)
- [Step Function Best Practices](#step-function-best-practices)
- [Assertions](#assertions)
- [Code Review Checklist](#code-review-checklist)
- [Troubleshooting](#troubleshooting)

---

## Play Function Structure

### Anatomy of a Play Function

Every play function follows this structure:

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { expect, fn, userEvent, within, waitFor } from '@storybook/test';
import { MyComponent } from './MyComponent';

const meta = {
  title: 'Components/MyComponent',
  component: MyComponent,
} satisfies Meta<typeof MyComponent>;

export default meta;
type Story = StoryObj<typeof MyComponent>;

export const UserAction: Story = {
  args: {
    onAction: fn(),       // Mock callback for assertion
    data: mockData,       // Test data
  },
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, args, step }) => {
    // 1. Setup canvas
    const canvas = within(canvasElement);

    // 2. Arrange - prepare the component state
    await step('Setup initial state', async () => {
      // ...interactions to reach starting state
    });

    // 3. Act - perform the user action
    await step('Perform action', async () => {
      // ...user interactions
    });

    // 4. Assert - verify the outcome
    await step('Verify result', async () => {
      // ...expectations
    });
  },
};
```

### Play Function Parameters

The play function receives a context object with these properties:

| Parameter | Type | Description |
|-----------|------|-------------|
| `canvasElement` | `HTMLElement` | The story's root DOM element |
| `args` | `Story['args']` | The story's args (including mocked functions) |
| `step` | `(label: string, fn: () => Promise<void>) => Promise<void>` | Groups actions for test reporting |
| `canvas` | `ReturnType<typeof within>` | Pre-bound `within(canvasElement)` (Storybook 8+) |

### Tags

Stories with play functions should include tags for filtering and CI integration:

```tsx
export const MyInteraction: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement }) => { ... },
};
```

- `test` - Marks the story as a test (used by `storybook test` runner)
- `interaction` - Categorizes the story as having user interactions

---

## Query Reference

### Priority Order

Queries are listed from most to least preferred. Always use the highest-priority query that works for your element.

### 1. `getByRole` (Preferred)

Queries by ARIA role. Most closely mirrors how users and assistive technology interact with elements.

```tsx
// Buttons
canvas.getByRole('button', { name: /submit/i });
canvas.getByRole('button', { name: 'Cancel' });

// Links
canvas.getByRole('link', { name: /learn more/i });

// Form elements
canvas.getByRole('textbox', { name: /email/i });
canvas.getByRole('checkbox', { name: /agree to terms/i });
canvas.getByRole('combobox');
canvas.getByRole('spinbutton');  // number input

// Headings
canvas.getByRole('heading', { level: 2 });

// Dialogs
canvas.getByRole('dialog');

// Navigation
canvas.getByRole('navigation');
canvas.getByRole('tab', { name: /settings/i });
canvas.getByRole('tabpanel');
```

### 2. `getByLabelText` (Form Elements)

Queries by the associated `<label>` text. Ideal for form inputs.

```tsx
canvas.getByLabelText('Email address');
canvas.getByLabelText(/password/i);
canvas.getByLabelText('Remember me');  // checkbox with label
```

### 3. `getByPlaceholderText` (Inputs)

Queries by placeholder attribute. Use when no label is available.

```tsx
canvas.getByPlaceholderText('Search...');
canvas.getByPlaceholderText(/enter your name/i);
```

### 4. `getByText` (Display Elements)

Queries by visible text content. Useful for non-interactive elements.

```tsx
canvas.getByText('No results found');
canvas.getByText(/welcome/i);
```

### 5. `getByTestId` (Last Resort)

Queries by `data-testid` attribute. Use only when no accessible query works.

```tsx
canvas.getByTestId('custom-dropdown');
canvas.getByTestId('chart-container');
```

### Query Variants

Each query has three variants for different timing needs:

| Variant | Behavior | Use When |
|---------|----------|----------|
| `getBy` | Synchronous, throws if not found | Element is already in DOM |
| `queryBy` | Synchronous, returns `null` if not found | Asserting element is NOT present |
| `findBy` | Async, waits up to timeout | Element appears after interaction |

```tsx
// Element exists now
const button = canvas.getByRole('button');

// Element should NOT exist
expect(canvas.queryByText('Error')).not.toBeInTheDocument();

// Element will appear after async operation
const result = await canvas.findByText('Success');
```

### `getAllBy` / `queryAllBy` / `findAllBy`

Returns arrays when multiple matching elements exist:

```tsx
const options = canvas.getAllByRole('option');
expect(options).toHaveLength(5);

const items = await canvas.findAllByTestId(/item-/);
```

---

## Async Patterns

### Rule: Always Await

Every `userEvent` call and every assertion must be awaited:

```tsx
// Correct
await userEvent.click(button);
await userEvent.type(input, 'text');
await expect(element).toHaveTextContent('result');

// Wrong - missing await causes race conditions
userEvent.click(button);
expect(element).toHaveTextContent('result');
```

### After Interactions: Use `findBy`

When an interaction causes new elements to render, use `findBy` queries which automatically wait:

```tsx
await step('Open dropdown and select', async () => {
  await userEvent.click(canvas.getByRole('button', { name: /open/i }));

  // findBy waits for the dropdown to appear
  const dropdown = await canvas.findByRole('listbox');
  const option = await canvas.findByRole('option', { name: /react/i });
  await userEvent.click(option);
});
```

### For State Changes: Use `waitFor`

When asserting on state that changes asynchronously:

```tsx
await step('Wait for loading to complete', async () => {
  await waitFor(() => {
    expect(canvas.queryByText('Loading...')).not.toBeInTheDocument();
  });
});

await step('Verify data loaded', async () => {
  await waitFor(() => {
    expect(canvas.getByRole('table')).toBeInTheDocument();
    expect(canvas.getAllByRole('row')).toHaveLength(5);
  });
});
```

### For Disappearing Elements: Use `waitFor` + `queryBy`

```tsx
await step('Dismiss notification', async () => {
  await userEvent.click(canvas.getByRole('button', { name: /dismiss/i }));

  await waitFor(() => {
    expect(canvas.queryByRole('alert')).not.toBeInTheDocument();
  });
});
```

---

## Common Interaction Patterns

### Form Submission

```tsx
export const FillAndSubmit: Story = {
  args: { onSubmit: fn() },
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, args, step }) => {
    const canvas = within(canvasElement);

    await step('Fill in form fields', async () => {
      await userEvent.type(canvas.getByLabelText('Name'), 'John Doe');
      await userEvent.type(canvas.getByLabelText('Email'), 'john@example.com');
      await userEvent.selectOptions(
        canvas.getByLabelText('Country'),
        'United States',
      );
      await userEvent.click(canvas.getByLabelText('Agree to terms'));
    });

    await step('Submit the form', async () => {
      await userEvent.click(canvas.getByRole('button', { name: /submit/i }));
    });

    await step('Verify submission', async () => {
      await expect(args.onSubmit).toHaveBeenCalledWith({
        name: 'John Doe',
        email: 'john@example.com',
        country: 'United States',
        agreedToTerms: true,
      });
    });
  },
};
```

### Keyboard Navigation

```tsx
export const KeyboardNavigation: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Focus and open with keyboard', async () => {
      const combobox = canvas.getByRole('combobox');
      combobox.focus();
      await userEvent.keyboard('{ArrowDown}');
    });

    await step('Navigate options', async () => {
      await userEvent.keyboard('{ArrowDown}');
      await userEvent.keyboard('{ArrowDown}');
    });

    await step('Select with Enter', async () => {
      await userEvent.keyboard('{Enter}');
    });

    await step('Verify selection', async () => {
      await expect(canvas.getByRole('combobox')).toHaveTextContent('Option 2');
    });
  },
};
```

### Keyboard Shortcuts Reference

```tsx
await userEvent.keyboard('{Enter}');
await userEvent.keyboard('{Escape}');
await userEvent.keyboard('{ArrowDown}');
await userEvent.keyboard('{ArrowUp}');
await userEvent.keyboard('{ArrowLeft}');
await userEvent.keyboard('{ArrowRight}');
await userEvent.keyboard('{Tab}');
await userEvent.keyboard('{Backspace}');
await userEvent.keyboard('{Delete}');
await userEvent.keyboard(' ');           // Space
await userEvent.keyboard('{Home}');
await userEvent.keyboard('{End}');
```

### Multiple Selections

```tsx
export const SelectMultiple: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Select multiple options', async () => {
      await userEvent.click(canvas.getByTestId('option-1'));
      await userEvent.click(canvas.getByTestId('option-2'));
      await userEvent.click(canvas.getByTestId('option-3'));
    });

    await step('Verify count', async () => {
      const count = canvas.getByTestId('selected-count');
      await expect(count).toHaveTextContent('3 selected');
    });
  },
};
```

### Search and Filter

```tsx
export const SearchByKeyword: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Type search query', async () => {
      const searchInput = canvas.getByRole('textbox', { name: /search/i });
      await userEvent.type(searchInput, 'react');
    });

    await step('Verify filtered results', async () => {
      await waitFor(() => {
        const results = canvas.getAllByRole('listitem');
        expect(results).toHaveLength(2);
      });
    });
  },
};
```

### Clear Input and Retype

```tsx
await step('Clear and retype', async () => {
  const input = canvas.getByLabelText('Search');
  await userEvent.clear(input);
  await userEvent.type(input, 'new value');
});
```

### Drag and Drop

```tsx
export const DragAndDrop: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Drag item to target', async () => {
      const dragItem = canvas.getByTestId('drag-item-1');
      const dropTarget = canvas.getByTestId('drop-zone');

      await userEvent.pointer([
        { keys: '[MouseLeft>]', target: dragItem },
        { target: dropTarget },
        { keys: '[/MouseLeft]' },
      ]);
    });

    await step('Verify item moved', async () => {
      await waitFor(() => {
        expect(canvas.getByTestId('drop-zone')).toHaveTextContent('Item 1');
      });
    });
  },
};
```

### Error Validation

```tsx
export const InvalidInput: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Enter invalid data', async () => {
      await userEvent.type(canvas.getByLabelText('Email'), 'not-an-email');
    });

    await step('Submit and verify error', async () => {
      await userEvent.click(canvas.getByRole('button', { name: /submit/i }));
      const errorMsg = await canvas.findByRole('alert');
      await expect(errorMsg).toHaveTextContent(/invalid email/i);
    });
  },
};
```

### Toggle and Verify State

```tsx
export const ToggleSwitch: Story = {
  args: { onToggle: fn() },
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, args, step }) => {
    const canvas = within(canvasElement);

    await step('Toggle on', async () => {
      const toggle = canvas.getByRole('switch');
      await userEvent.click(toggle);
      await expect(toggle).toBeChecked();
    });

    await step('Toggle off', async () => {
      const toggle = canvas.getByRole('switch');
      await userEvent.click(toggle);
      await expect(toggle).not.toBeChecked();
    });

    await step('Verify callback count', async () => {
      await expect(args.onToggle).toHaveBeenCalledTimes(2);
    });
  },
};
```

---

## Step Function Best Practices

### Why Use Steps

Steps group related actions and provide:
- **Better test reporting** - Storybook's interaction panel shows collapsible step groups
- **Clearer intent** - Step labels describe the purpose of each action group
- **Easier debugging** - When a test fails, you can see which step failed

### Step Naming Conventions

Use descriptive, action-oriented labels:

```tsx
// Good step names
await step('Fill in shipping address', async () => { ... });
await step('Apply discount code', async () => { ... });
await step('Verify order total updates', async () => { ... });

// Bad step names - too vague
await step('Step 1', async () => { ... });
await step('Click stuff', async () => { ... });
await step('Check', async () => { ... });
```

### Step Granularity

Group by logical user intent, not by individual DOM interactions:

```tsx
// Good - grouped by intent
await step('Fill in login credentials', async () => {
  await userEvent.type(canvas.getByLabelText('Email'), 'user@test.com');
  await userEvent.type(canvas.getByLabelText('Password'), 'pass123');
});

// Bad - one step per interaction (too granular)
await step('Type email', async () => {
  await userEvent.type(canvas.getByLabelText('Email'), 'user@test.com');
});
await step('Type password', async () => {
  await userEvent.type(canvas.getByLabelText('Password'), 'pass123');
});
```

### Common Step Pattern: Arrange / Act / Assert

```tsx
play: async ({ canvasElement, args, step }) => {
  const canvas = within(canvasElement);

  await step('Set up initial state', async () => {
    // Arrange - prepare the component
  });

  await step('Perform user action', async () => {
    // Act - simulate the interaction
  });

  await step('Verify outcome', async () => {
    // Assert - check the result
  });
};
```

---

## Assertions

### Common Assertions

```tsx
// Visibility
await expect(element).toBeVisible();
await expect(element).not.toBeVisible();

// Presence
await expect(element).toBeInTheDocument();
expect(canvas.queryByText('Gone')).not.toBeInTheDocument();

// Text content
await expect(element).toHaveTextContent('Expected text');
await expect(element).toHaveTextContent(/partial match/i);

// Form state
await expect(input).toHaveValue('expected value');
await expect(checkbox).toBeChecked();
await expect(checkbox).not.toBeChecked();
await expect(input).toBeDisabled();
await expect(input).toBeEnabled();
await expect(input).toBeRequired();
await expect(input).toHaveAttribute('aria-invalid', 'true');

// CSS classes and styles
await expect(element).toHaveClass('active');
await expect(element).toHaveStyle({ color: 'red' });

// Callback assertions
await expect(args.onClick).toHaveBeenCalled();
await expect(args.onClick).toHaveBeenCalledTimes(1);
await expect(args.onClick).toHaveBeenCalledWith('expected-arg');
await expect(args.onChange).toHaveBeenCalledWith(
  expect.objectContaining({ value: 'test' }),
);
```

### Mocking Callbacks with `fn()`

Always use `fn()` from `@storybook/test` for callbacks you want to assert on:

```tsx
import { fn } from '@storybook/test';

export const WithCallback: Story = {
  args: {
    onClick: fn(),
    onChange: fn(),
    onSubmit: fn(),
  },
  play: async ({ args }) => {
    // ... interactions ...
    await expect(args.onClick).toHaveBeenCalled();
  },
};
```

---

## Code Review Checklist

Use this checklist when reviewing Storybook play functions in pull requests.

### Structure

- [ ] **Canvas setup:** `within(canvasElement)` is called at the start
- [ ] **Step grouping:** Logical actions are wrapped in `step()` calls
- [ ] **Step labels:** Step names describe user intent, not DOM operations
- [ ] **Tags present:** Story has `tags: ['test', 'interaction']`
- [ ] **Destructuring:** Play function destructures `canvasElement`, `args`, `step` as needed

### Queries

- [ ] **Query priority:** Uses most accessible query available (`getByRole` > `getByLabelText` > `getByTestId`)
- [ ] **No fragile selectors:** Does not use `getByClassName`, `querySelector`, or other DOM-specific selectors
- [ ] **Correct variant:** Uses `getBy` for present elements, `findBy` for async elements, `queryBy` for absence checks
- [ ] **Regex for text:** Uses case-insensitive regex (`/submit/i`) for text matching where appropriate

### Async Handling

- [ ] **All awaited:** Every `userEvent` call is awaited
- [ ] **All assertions awaited:** Every `expect` is awaited
- [ ] **findBy for async elements:** Uses `findBy` after interactions that render new elements
- [ ] **waitFor for state changes:** Uses `waitFor` for assertions on async state changes
- [ ] **No race conditions:** Doesn't assume elements exist immediately after interaction

### Assertions

- [ ] **Meaningful assertions:** Tests verify outcomes, not just that interactions don't crash
- [ ] **Callbacks mocked:** Uses `fn()` for callback props that are asserted on
- [ ] **Specific matchers:** Uses appropriate matchers (`toHaveTextContent`, `toBeChecked`, etc.)

### Common Review Feedback Templates

**Missing step grouping:**
> The play function has multiple interactions without `step()` wrapping. Please group related actions into steps for better test reporting. For example, wrap the form fill actions in `await step('Fill in form', async () => { ... })`.

**Wrong query priority:**
> `getByTestId('submit-btn')` is used here, but this button likely has an accessible role. Consider using `getByRole('button', { name: /submit/i })` instead, which better reflects how users interact with the element.

**Missing await:**
> `userEvent.click(button)` on line X is missing `await`. All `userEvent` calls must be awaited to prevent race conditions.

**Missing findBy for async elements:**
> After clicking the open button, `getByTestId('dropdown')` is used, but the dropdown renders asynchronously. Use `await canvas.findByTestId('dropdown')` instead to wait for it to appear.

**Missing tags:**
> This story has a play function but is missing `tags: ['test', 'interaction']`. Please add the tags so the test runner can discover it.

---

## Troubleshooting

### Play function doesn't run

- Verify `@storybook/addon-interactions` is installed and listed in `.storybook/main.ts` addons
- Check the Interactions panel in Storybook for errors
- Ensure the story is actually using the `play` property (not a custom name)

### Element not found errors

- Use `findBy` instead of `getBy` if the element appears after an interaction
- Check that the element's role, label, or test ID matches what you're querying
- Use Storybook's accessibility panel to inspect available ARIA roles

### Timing issues / flaky tests

- Replace `getBy` with `findBy` for elements that render asynchronously
- Wrap assertions in `waitFor` when checking state that updates asynchronously
- Avoid fixed `setTimeout` delays - use `waitFor` or `findBy` instead

### Assertions fail but UI looks correct

- Ensure all previous interactions are awaited before asserting
- Check that `fn()` mocks are defined in `args`, not created inline in the play function
- Verify the assertion uses the correct matcher (e.g., `toHaveTextContent` vs `toHaveValue`)

### Common Errors

| Error | Solution |
|-------|----------|
| `Unable to find role "button"` | Check the element's actual role with accessibility panel |
| `Received: null` from `getBy` | Element not in DOM yet - use `findBy` |
| `toHaveBeenCalled` fails | Ensure callback uses `fn()` in `args` |
| `act()` warning | Wrap state-changing operations in `await` |
| `findBy` timeout | Increase timeout or check if element ever renders |

---

## Related Resources

- [Storybook Interaction Testing](https://storybook.js.org/docs/writing-tests/interaction-testing)
- [Testing Library Queries](https://testing-library.com/docs/queries/about)
- [Storybook Play Functions](https://storybook.js.org/docs/writing-stories/play-function)
- [@storybook/test API](https://storybook.js.org/docs/api/test)
