---
name: story-naming
description: Use when creating Storybook stories, naming story exports, organizing story files, or reviewing story naming conventions. Ensures story names describe user scenarios and component states rather than implementation details.
---

# Story Naming Conventions

Name Storybook stories to describe **what the user sees or does**, not how the story works internally.

## Quick Decision Guide

Before naming a story, ask:

1. Does this describe what the user sees or does?
2. Would a non-technical stakeholder understand this name?
3. Is this name still accurate without the `play` function?

If yes to all three, it's a good name.

## Naming Rules

### Describe User Scenarios or Component States

```tsx
// User scenarios (typically with play functions)
export const SelectAndApply: Story = {};
export const SearchByKeyword: Story = {};
export const ClearAllSelections: Story = {};

// Component states (typically static/visual)
export const DisabledState: Story = {};
export const WithPreselectedItems: Story = {};
export const EmptyState: Story = {};
```

### Never Expose Implementation Details

```tsx
// Bad - exposes implementation
export const InteractiveDefault: Story = {};
export const InteractiveWithValidation: Story = {};
export const TestSearchFunctionality: Story = {};
export const PlayFunctionForSearch: Story = {};

// Good - describes user scenario
export const Default: Story = {};
export const WithValidation: Story = {};
export const SearchByKeyword: Story = {};
export const SearchAndFilter: Story = {};
```

**Rationale:** The presence of a `play` function already indicates interactivity. Story names should remain accurate even if the `play` function is removed.

## Story Title Organization

Use hierarchical `title` paths for sidebar organization:

```tsx
const meta = {
  title: 'Components/StructuredFilter/SelectOptions',
  component: SelectOptions,
} satisfies Meta<typeof SelectOptions>;
```

**Patterns:**

| Pattern | Use Case |
|---------|----------|
| `Components/[ComponentName]` | Single component stories |
| `Components/[ComponentName]/States` | Visual state variations |
| `Components/[ComponentName]/Flows` | User interaction flows |
| `Features/[FeatureName]` | Feature-level compositions |

## Examples

### Example 1: Create a new story file

**User:** "Create stories for the UserProfile component"

**Action:**
```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { UserProfile } from './UserProfile';

const meta = {
  title: 'Components/UserProfile',
  component: UserProfile,
} satisfies Meta<typeof UserProfile>;

export default meta;
type Story = StoryObj<typeof UserProfile>;

export const Default: Story = {};

export const WithLongName: Story = {
  args: { name: 'Alexandria Bartholomew Richardson III' },
};

export const EmptyState: Story = {
  args: { user: null },
};

export const LoadingState: Story = {
  args: { isLoading: true },
};
```

---

### Example 2: Name an interactive story

**User:** "Add a story where the user fills out and submits the form"

**Action:**
```tsx
// Good - describes the user action
export const FillAndSubmit: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.type(canvas.getByLabelText('Email'), 'user@example.com');
    await userEvent.click(canvas.getByRole('button', { name: 'Submit' }));
  },
};
```

**Not:**
```tsx
// Bad - exposes implementation detail
export const InteractiveFormSubmission: Story = { ... };
export const PlayFunctionSubmit: Story = { ... };
export const TestFormValidation: Story = { ... };
```

---

### Example 3: Organize complex component stories

**User:** "Set up stories for the DataGrid with states and interactions"

**Action:**
```tsx
// States file: Components/DataGrid/States
export const Empty: Story = {};
export const WithSingleRow: Story = {};
export const WithPagination: Story = {};
export const DisabledState: Story = {};

// Flows file: Components/DataGrid/Flows
export const SortByColumn: Story = {};
export const FilterAndSearch: Story = {};
export const SelectMultipleRows: Story = {};
export const EditInlineCell: Story = {};
```

---

### Example 4: Review story names for convention compliance

**User:** "Review my story names"

**Action:** Check each export name against the three questions:

| Current Name | Issue | Suggested Name |
|-------------|-------|----------------|
| `InteractiveDefault` | Exposes implementation (`Interactive` prefix) | `Default` |
| `TestSearchFlow` | Exposes implementation (`Test` prefix) | `SearchByKeyword` |
| `PlaySelectAll` | Exposes implementation (`Play` prefix) | `SelectAll` |
| `WithError` | Describes state | `WithError` (keep) |
| `EmptyState` | Describes state | `EmptyState` (keep) |

## Common Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|-------------|-------------|-----|
| `Interactive*` prefix | Play function already signals interactivity | Remove prefix |
| `Test*` prefix | Stories aren't tests, they're demonstrations | Describe the scenario |
| `PlayFunction*` prefix | Implementation detail | Describe what happens |
| `Render*` prefix | All stories render | Describe what's rendered |
| Technical jargon in names | Non-technical stakeholders can't understand | Use plain language |

## More Information

See [REFERENCE.md](./REFERENCE.md) for detailed documentation including:
- Complete naming patterns with examples
- Story title hierarchy best practices
- Code review checklist for story naming
- Migration guide for renaming existing stories
