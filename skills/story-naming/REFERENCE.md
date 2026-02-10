# Story Naming Conventions Reference

Complete reference for naming Storybook stories and organizing story files.

## Table of Contents

- [Naming Principles](#naming-principles)
- [Story Export Names](#story-export-names)
- [Story Title Organization](#story-title-organization)
- [Code Review Checklist](#code-review-checklist)
- [Migration Guide](#migration-guide)

---

## Naming Principles

### Core Rule

Story names describe **what the user sees or does**, never how the story is implemented.

### The Three-Question Test

Every story name must pass all three:

1. **Does this describe what the user sees or does?** The name should communicate the scenario or visual state being demonstrated.
2. **Would a non-technical stakeholder understand this name?** Product managers, designers, and QA should be able to navigate the Storybook sidebar.
3. **Is this name still accurate without the `play` function?** If removing the play function makes the name misleading, it's exposing implementation details.

### Why This Matters

- **Storybook is documentation.** Story names appear in the sidebar and serve as a living style guide for the entire team.
- **Play functions are an implementation detail.** Whether a story uses `play`, `args`, `loaders`, or `parameters` shouldn't leak into the name.
- **Names should be stable.** Refactoring a story from static to interactive (or vice versa) shouldn't require renaming.

---

## Story Export Names

### Category 1: Component States

Use for stories that demonstrate a visual variation or configuration of the component.

| Pattern | Example | When to Use |
|---------|---------|-------------|
| `Default` | `export const Default: Story = {}` | Base/default rendering |
| `With[Feature]` | `WithAvatar`, `WithBadge` | Optional feature enabled |
| `Without[Feature]` | `WithoutHeader`, `WithoutFooter` | Optional feature disabled |
| `[State]State` | `EmptyState`, `LoadingState`, `ErrorState` | Application states |
| `Disabled` / `DisabledState` | `export const Disabled: Story = {}` | Disabled component |
| `[Size/Variant]` | `Small`, `Large`, `Primary`, `Secondary` | Size or visual variants |

#### Examples

```tsx
// Visual states
export const Default: Story = {};
export const EmptyState: Story = {};
export const LoadingState: Story = {};
export const ErrorState: Story = {};
export const DisabledState: Story = {};

// Feature variations
export const WithAvatar: Story = {};
export const WithBadge: Story = {};
export const WithLongContent: Story = {};
export const WithPreselectedItems: Story = {};
export const WithoutCloseButton: Story = {};

// Size/Variant
export const Small: Story = {};
export const Large: Story = {};
export const Primary: Story = {};
export const Outlined: Story = {};
```

### Category 2: User Scenarios

Use for stories that demonstrate a user interaction or workflow, typically with `play` functions.

| Pattern | Example | When to Use |
|---------|---------|-------------|
| `[Verb][Object]` | `SelectAndApply`, `SearchByKeyword` | User performs an action |
| `[Verb]All[Object]` | `ClearAllSelections`, `SelectAllItems` | Bulk actions |
| `[Verb]And[Verb]` | `SearchAndFilter`, `DragAndDrop` | Multi-step interactions |
| `[Verb]With[Condition]` | `SubmitWithErrors`, `LoginWithMFA` | Conditional flows |

#### Examples

```tsx
// Single actions
export const SelectOption: Story = {};
export const ExpandDetails: Story = {};
export const DismissNotification: Story = {};

// Multi-step flows
export const SearchAndFilter: Story = {};
export const SelectAndApply: Story = {};
export const FillAndSubmit: Story = {};
export const DragAndDrop: Story = {};

// Conditional flows
export const SubmitWithValidationErrors: Story = {};
export const SearchWithNoResults: Story = {};
export const LoginWithExpiredSession: Story = {};
```

### Forbidden Prefixes

These prefixes expose implementation details and must not be used:

| Prefix | Why It's Wrong | Alternative |
|--------|---------------|-------------|
| `Interactive` | Play function already signals interactivity | Drop the prefix |
| `Test` | Stories are demonstrations, not tests | Describe the scenario |
| `PlayFunction` | Leaks implementation mechanism | Describe the outcome |
| `Render` | All stories render something | Describe what's rendered |
| `Snapshot` | Testing strategy, not user scenario | Describe the visual state |
| `E2E` | Testing methodology | Describe the flow |
| `Mock` | Implementation detail | Describe the state |
| `Simulate` | Implementation detail | Describe the scenario |

---

## Story Title Organization

### Hierarchy Patterns

The `title` property in the meta object organizes stories in the Storybook sidebar.

```
Components/
├── Button/                          # Components/Button
│   ├── Default
│   ├── Primary
│   └── Disabled
├── DataGrid/
│   ├── States/                      # Components/DataGrid/States
│   │   ├── Empty
│   │   ├── WithPagination
│   │   └── Loading
│   └── Flows/                       # Components/DataGrid/Flows
│       ├── SortByColumn
│       ├── FilterAndSearch
│       └── SelectMultipleRows
Features/
├── UserDashboard/                   # Features/UserDashboard
│   ├── Default
│   └── WithNotifications
```

### When to Split into Sub-Groups

- **Single file** (`Components/Button`): Component has fewer than ~8 stories
- **States + Flows** (`Components/DataGrid/States`, `Components/DataGrid/Flows`): Component has many stories that fall into distinct categories
- **Feature grouping** (`Features/Checkout`): Stories compose multiple components into a feature view

### Title Naming Conventions

```tsx
// Simple component - flat structure
const meta = {
  title: 'Components/Button',
  component: Button,
} satisfies Meta<typeof Button>;

// Complex component - grouped by type
// File: DataGrid.states.stories.tsx
const meta = {
  title: 'Components/DataGrid/States',
  component: DataGrid,
} satisfies Meta<typeof DataGrid>;

// File: DataGrid.flows.stories.tsx
const meta = {
  title: 'Components/DataGrid/Flows',
  component: DataGrid,
} satisfies Meta<typeof DataGrid>;

// Feature composition
const meta = {
  title: 'Features/UserDashboard',
  component: UserDashboard,
} satisfies Meta<typeof UserDashboard>;
```

---

## Code Review Checklist

Use this checklist when reviewing Storybook stories in pull requests.

### Story Export Names

- [ ] **No implementation prefixes:** Names don't start with `Interactive`, `Test`, `PlayFunction`, `Render`, `Snapshot`, `E2E`, `Mock`, or `Simulate`
- [ ] **Describes user perspective:** Each name describes what the user sees or does
- [ ] **Stakeholder-friendly:** A non-technical team member could understand the name
- [ ] **Play-function independent:** Names remain accurate if the `play` function were removed
- [ ] **PascalCase format:** Export names use PascalCase (e.g., `SelectAndApply`, not `selectAndApply`)

### Story Title Organization

- [ ] **Correct hierarchy:** Title follows `Components/[Name]` or `Features/[Name]` pattern
- [ ] **Logical grouping:** Related stories are grouped under the same title path
- [ ] **No redundant nesting:** Simple components don't have unnecessary sub-groups
- [ ] **Consistent with codebase:** Title structure matches existing patterns in the project

### Story Content Alignment

- [ ] **Name matches behavior:** The story actually demonstrates what the name implies
- [ ] **States vs. Flows distinction:** Static visual stories use state names, interactive stories use action names
- [ ] **Default story exists:** There's a `Default` export showing the base component

### Common Review Feedback Templates

**For implementation-detail names:**
> The story name `InteractiveSearch` exposes an implementation detail. Since the `play` function already signals interactivity, consider renaming to `SearchByKeyword` to describe the user scenario.

**For unclear names:**
> The story name `Variation3` doesn't convey what this demonstrates. Could you rename it to describe the specific visual state, e.g., `WithLongTitle` or `CompactLayout`?

**For missing Default story:**
> Consider adding a `Default` story that shows the base component with standard props. This helps as a reference point for other story variations.

---

## Migration Guide

### Renaming Existing Stories

When renaming stories in an existing codebase, follow these steps to avoid breaking links and bookmarks:

#### Step 1: Audit Current Names

Identify stories with implementation-detail names:

```bash
# Find story exports with common anti-patterns
grep -rn "export const Interactive\|export const Test\|export const PlayFunction\|export const Render" --include="*.stories.tsx" --include="*.stories.ts"
```

#### Step 2: Create a Rename Map

Document current and proposed names:

| File | Current Name | New Name | Reason |
|------|-------------|----------|--------|
| `Button.stories.tsx` | `InteractiveDefault` | `Default` | Remove `Interactive` prefix |
| `Search.stories.tsx` | `TestSearchFlow` | `SearchByKeyword` | Remove `Test` prefix, describe scenario |
| `Form.stories.tsx` | `PlayFunctionSubmit` | `FillAndSubmit` | Remove `PlayFunction` prefix |

#### Step 3: Rename Incrementally

Rename stories in batches per component to keep PRs reviewable. Update any references in documentation or tests that link to specific story IDs.

#### Step 4: Update Story IDs in Tests

If you use Storybook's `composeStories` or reference stories by ID in tests, update those references:

```tsx
// Before
const { InteractiveDefault } = composeStories(stories);

// After
const { Default } = composeStories(stories);
```

---

## Related Resources

- [Storybook Naming Convention Docs](https://storybook.js.org/docs/writing-stories#default-export)
- [Component Story Format (CSF)](https://storybook.js.org/docs/api/csf)
- [Play Functions](https://storybook.js.org/docs/writing-stories/play-function)
