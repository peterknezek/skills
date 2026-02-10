---
name: storybook-interactions
description: Use when creating Storybook play functions, writing interaction tests in stories, or reviewing play function code in pull requests. Ensures consistent structure, proper query priorities, correct async handling, and best practices for Storybook interaction testing.
dependencies:
  - "@storybook/test"
  - "@storybook/addon-interactions"
---

# Storybook Interaction Tests (Play Functions)

Write play functions with consistent structure, accessible queries, and proper async handling.

## Required Structure

Every play function must follow this pattern:

```tsx
export const SearchAndSelect: Story = {
  args: { options: mockData },
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, args, step }) => {
    const canvas = within(canvasElement);

    await step('Search for option', async () => {
      const input = canvas.getByTestId('search-input');
      await userEvent.type(input, 'react');
    });

    await step('Select filtered option', async () => {
      const option = await canvas.findByTestId('option-react');
      await userEvent.click(option);
    });

    await step('Verify selection', async () => {
      await expect(args.onChange).toHaveBeenCalledWith(
        expect.arrayContaining([expect.objectContaining({ value: 'react' })]),
      );
    });
  },
};
```

### Key Rules

1. **Always destructure** `canvasElement`, `args`, and `step` from the play function argument
2. **Always call** `within(canvasElement)` as the first line
3. **Wrap logical groups** of actions in `step()` for test reporting
4. **Always `await`** user interactions and assertions
5. **Add tags** `['test', 'interaction']` to stories with play functions

## Query Priority Order

Use queries in this order of preference:

| Priority | Query | Use When |
|----------|-------|----------|
| 1st | `getByRole` | Element has an accessible role (button, textbox, etc.) |
| 2nd | `getByLabelText` | Form elements with associated labels |
| 3rd | `getByPlaceholderText` | Inputs with placeholder text |
| Last | `getByTestId` | No accessible query available |

```tsx
// Preferred - accessible queries
const button = canvas.getByRole('button', { name: /submit/i });
const input = canvas.getByLabelText('Email address');

// Acceptable - when role/label not available
const dropdown = canvas.getByTestId('multiselect-dropdown');

// Never use - fragile selectors
const element = canvas.getByClassName('my-class');  // Breaks on style changes
```

## Async Rules

```tsx
// After interactions that render new elements, use findBy (auto-waits)
await userEvent.click(openButton);
const dropdown = await canvas.findByTestId('dropdown');

// For assertions on async state changes, use waitFor
await waitFor(() => {
  expect(canvas.getByText('Loading...')).not.toBeInTheDocument();
});

// Always await userEvent calls
await userEvent.click(button);    // Correct
await userEvent.type(input, 'x'); // Correct
userEvent.click(button);          // WRONG - missing await
```

## Examples

### Example 1: Create an interaction test for form submission

**User:** "Add a play function to test the login form"

**Action:**
```tsx
export const SubmitLoginForm: Story = {
  args: { onSubmit: fn() },
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, args, step }) => {
    const canvas = within(canvasElement);

    await step('Fill in credentials', async () => {
      await userEvent.type(canvas.getByLabelText('Email'), 'user@example.com');
      await userEvent.type(canvas.getByLabelText('Password'), 'password123');
    });

    await step('Submit the form', async () => {
      await userEvent.click(canvas.getByRole('button', { name: /sign in/i }));
    });

    await step('Verify submission', async () => {
      await expect(args.onSubmit).toHaveBeenCalledWith({
        email: 'user@example.com',
        password: 'password123',
      });
    });
  },
};
```

---

### Example 2: Create a keyboard navigation test

**User:** "Add a story that tests keyboard navigation on the dropdown"

**Action:**
```tsx
export const KeyboardNavigation: Story = {
  args: { options: mockOptions },
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Open dropdown with keyboard', async () => {
      const combobox = canvas.getByRole('combobox');
      combobox.focus();
      await userEvent.keyboard('{ArrowDown}');
    });

    await step('Navigate and select option', async () => {
      await userEvent.keyboard('{ArrowDown}');
      await userEvent.keyboard('{ArrowDown}');
      await userEvent.keyboard('{Enter}');
    });

    await step('Verify selection is displayed', async () => {
      const selected = canvas.getByRole('combobox');
      await expect(selected).toHaveTextContent('Option 2');
    });
  },
};
```

---

### Example 3: Test error handling

**User:** "Add a story for invalid email validation"

**Action:**
```tsx
export const InvalidEmail: Story = {
  tags: ['test', 'interaction'],
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);

    await step('Enter invalid email', async () => {
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

---

### Example 4: Review a play function for best practices

**User:** "Review this play function"

**Action:** Check against these criteria:

| Check | What to Look For |
|-------|-----------------|
| Structure | Uses `step()` to group logical actions |
| Queries | Prefers `getByRole`/`getByLabelText` over `getByTestId` |
| Async | All `userEvent` calls and assertions are awaited |
| Async elements | Uses `findBy` for elements that appear after interaction |
| Tags | Story has `['test', 'interaction']` tags |
| Assertions | Uses `expect` to verify outcomes, not just interactions |
| Canvas | Uses `within(canvasElement)`, not global queries |

**Common issues found in reviews:**

```tsx
// Missing await
userEvent.click(button);                    // Fix: await userEvent.click(button);

// Missing step() grouping
play: async ({ canvasElement }) => {        // Fix: wrap in step() calls
  const canvas = within(canvasElement);
  await userEvent.click(canvas.getByRole('button'));
  await expect(...).toBe(...);
};

// Using getBy for async-rendered elements
await userEvent.click(openBtn);
const menu = canvas.getByTestId('menu');    // Fix: await canvas.findByTestId('menu');

// Missing tags
export const MyTest: Story = {              // Fix: add tags: ['test', 'interaction']
  play: async ({ canvasElement }) => { ... },
};
```

## More Information

See [REFERENCE.md](./REFERENCE.md) for detailed documentation including:
- Complete query reference with examples
- All common interaction patterns
- Step function best practices
- Code review checklist for play functions
- Troubleshooting guide
