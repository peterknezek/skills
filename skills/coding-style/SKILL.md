---
name: coding-style
description: Use when writing JavaScript or TypeScript code, reviewing code style in pull requests, or checking adherence to the Epic Programming Style Guide. Covers variables, functions, modules, types, React patterns, and testing conventions.
---

# JavaScript & TypeScript Coding Style

Opinionated style guide based on the [Epic Programming Style Guide](https://www.epicweb.dev/principles) for writing code that is easy to understand, maintain, and scale.

## Quick Rules

| Topic | Rule |
|-------|------|
| Variables | `const` by default, `let` when reassigning, never `var` |
| Semicolons | Don't use them (use `no-unexpected-multiline` ESLint rule) |
| Functions | Function declarations over expressions; arrow functions for inline callbacks |
| Exports | Named exports only (no default exports unless framework requires it) |
| Arrays | `Array<T>` generic syntax, not `T[]` |
| Types | Infer when possible; `unknown` over `any`; avoid `as` assertions |
| Modules | ESM only; no barrel files; include file extensions in imports |
| React | Avoid `useEffect`; derive state instead of syncing; ternaries for conditionals |
| Testing | `test()` over `describe/it`; no nesting; `userEvent` over `fireEvent` |

## Variables

```tsx
// const by default
const workshopTitle = 'Web App Fundamentals'
const isEnabled = true

// let only when reassigning
let count = 0
count += 1

// UPPER_CASE for true constants across files
const BASE_URL = 'https://epicweb.dev'
const DEFAULT_PORT = 3000

// Descriptive names - no single letters (except small loops)
const sum = numbers.reduce((total, n) => total + n, 0)
const names = people.map((p) => p.name)
```

## Functions

```tsx
// Function declarations (hoisted, named in stack traces)
function calculateTotal(items: Array<number>) {
  return items.reduce((sum, item) => sum + item, 0)
}

// Arrow functions for inline callbacks only
const doubled = items.filter((n) => n > 2).map((n) => n * 2)

// Default parameters over short-circuiting
function createUser(name: string, role = 'user') {
  return { name, role }
}

// Early return with guard clauses
function getStatus(score: number | undefined) {
  if (!score) return undefined
  if (score <= 50) return 'low'
  return 'high'
}

// Async/await over promise chains
async function fetchUserData(userId: string) {
  const user = await getUser(userId)
  const posts = await getUserPosts(user.id)
  return { user, posts }
}
```

## Objects & Arrays

```tsx
// Property shorthand
const person = { name, age }

// Method shorthand
const obj = {
  method() { /* ... */ },
  async asyncMethod() { /* ... */ },
}

// No getters/setters (principle of least surprise)

// Array literal syntax
const items = [1, 2, 3]

// Array methods over loops; never forEach
const doubled = items.map((n) => n * 2)
for (const item of items) { /* imperative code */ }

// filter + map over reduce
const result = items.filter((n) => n > 2).map((n) => n * 2)

// Non-mutative methods
const reversed = items.toReversed()
const replaced = items.with(0, newValue)

// Spread to copy
const copy = [...items]
const combined = [...arr1, ...arr2]

// Array<T> generic syntax
const names: Array<string> = []
function transform(numbers: Array<number>) {}
```

## Types

```tsx
// Let TypeScript infer return types
function add(a: number, b: number) {
  return a + b
}

// Descriptive generic names
function createArray<Value>(length: number, value: Value): Array<Value> {
  return Array(length).fill(value)
}

// Type guards over type assertions
function isError(maybeError: unknown): maybeError is Error {
  return (
    maybeError != null &&
    typeof maybeError === 'object' &&
    'message' in maybeError &&
    typeof maybeError.message === 'string'
  )
}

// Schema validation for boundary data
const UserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
})
type User = z.infer<typeof UserSchema>

// unknown over any
function handleError(error: unknown) {
  if (isError(error)) console.error(error.message)
}

// @ts-expect-error over @ts-ignore (with explanation)
// @ts-expect-error - library types don't account for our custom prop
```

## Modules

```tsx
// Named exports only
export function add(a: number, b: number) {
  return a + b
}

// Single import per module, inline type imports
import { type UserType, getUser } from '#app/utils/user.ts'

// Include file extensions
import { add } from './math.ts'

// Import order: built-in → external → internal absolute → relative → local
import 'node:fs'
import 'match-sorter'
import '#app/components'
import '../other-folder'
import './local-file'

// Inline exports (not separate export statements)
export function subtract(a: number, b: number) {
  return a - b
}

// ESM only, no CommonJS
// Use import aliases via package.json "imports" field
```

## React

```tsx
// Derive state, don't sync it
const [count, setCount] = useState(0)
const isEven = count % 2 === 0  // derived, not separate state

// Avoid useEffect - use event handlers, ref callbacks, etc.

// Don't render falsiness - use ternary with null
{contacts.length ? <ContactList contacts={contacts} /> : null}

// Ternaries for simple conditionals
{isAdmin ? <AdminPanel /> : null}
```

## Testing

```tsx
// test() over describe/it, flat structure, no nesting
test('User can add items to cart', async () => {
  render(<ProductList />)
  await userEvent.click(screen.getByRole('button', { name: /add to cart/i }))
  await expect(screen.getByText(/1 item in cart/i)).toBeInTheDocument()
})

// userEvent over fireEvent
await userEvent.type(screen.getByRole('textbox'), 'Hello')

// Accessible queries: getByRole > getByLabelText > getByTestId
screen.getByRole('textbox', { name: /username/i })

// findBy for async elements, queryBy only for non-existence
const result = await screen.findByText('Success')
expect(screen.queryByRole('alert')).not.toBeInTheDocument()

// MSW for external services, avoid mocking own code
```

## Examples

### Example 1: Write a utility function

**User:** "Create a function to filter and transform user data"

**Action:**
```tsx
function getActiveUserNames(users: Array<User>) {
  return users
    .filter((user) => user.isActive)
    .map((user) => user.name)
}
```

**Not:**
```tsx
const getActiveUserNames = (users: User[]) => {
  const result: string[] = [];
  users.forEach((user) => {
    if (user.isActive) {
      result.push(user.name);
    }
  });
  return result;
};
```

---

### Example 2: Write a React component

**User:** "Create a greeting component with conditional rendering"

**Action:**
```tsx
export function Greeting({ user }: { user: User | null }) {
  if (!user) return null

  const displayName = user.nickname ?? user.name

  return (
    <div>
      <h1>Hello {displayName}</h1>
      {user.isAdmin ? <Link to="/admin">Admin Panel</Link> : null}
    </div>
  )
}
```

---

### Example 3: Write a test

**User:** "Write a test for the search component"

**Action:**
```tsx
test('User can search and see filtered results', async () => {
  render(<SearchPage />)

  await userEvent.type(
    screen.getByRole('textbox', { name: /search/i }),
    'react',
  )

  const results = await screen.findAllByRole('listitem')
  expect(results).toHaveLength(3)
  expect(screen.getByText('React Fundamentals')).toBeInTheDocument()
})
```

---

### Example 4: Review code for style compliance

**User:** "Review this code for style issues"

**Action:** Check against these rules:

| Check | What to Look For |
|-------|-----------------|
| Variables | `const`/`let` usage, no `var`, descriptive names |
| Functions | Declarations (not expressions), early returns, default params |
| Arrays | `Array<T>` syntax, methods over loops, no `forEach`, non-mutative |
| Types | Inference used, `unknown` over `any`, no unnecessary `as` |
| Modules | Named exports, file extensions, single import per module |
| Semicolons | None present (handled by formatter) |
| React | No unnecessary `useEffect`, derived state, ternary conditionals |
| Testing | `test()` not `describe/it`, `userEvent`, accessible queries |

## More Information

See [REFERENCE.md](./REFERENCE.md) for detailed documentation including:
- Complete rules with rationale for each convention
- Naming conventions and the A/HC/LC pattern
- Event handling patterns
- Code review checklist with feedback templates
- All comparison and control flow guidelines
