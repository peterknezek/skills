# JavaScript & TypeScript Coding Style Reference

Complete reference for writing clean, maintainable JavaScript and TypeScript.

## Table of Contents

- [Variables](#variables)
- [Objects](#objects)
- [Arrays](#arrays)
- [Destructuring](#destructuring)
- [Strings](#strings)
- [Functions](#functions)
- [Modules](#modules)
- [Properties & Comparisons](#properties--comparisons)
- [Control Flow](#control-flow)
- [Comments](#comments)
- [Semicolons](#semicolons)
- [Types](#types)
- [Naming Conventions](#naming-conventions)
- [Events](#events)
- [React](#react)
- [Testing](#testing)
- [Code Review Checklist](#code-review-checklist)

---

## Variables

### References

Use `const` by default. Only use `let` when reassignment is needed. Never use `var`.

`const` means "constant reference", not "unchangeable value" — object properties can still be modified.

```tsx
// Good
const name = 'Kent'
let count = 0
count += 1

// Bad
var name = 'Kent'
let name = 'Kent' // never reassigned, use const
```

### Naming

Use descriptive, clear names. Avoid single-letter names except in small loops/reducers where context is obvious.

```tsx
// Good
const workshopTitle = 'Web App Fundamentals'
const instructorName = 'Kent C. Dodds'
const isEnabled = true
const sum = numbers.reduce((total, n) => total + n, 0)
const names = people.map((p) => p.name)

// Bad
const t = 'Web App Fundamentals'
const n = 'Kent C. Dodds'
const e = true
```

### Constants

For truly constant values used across files, use uppercase with underscores:

```tsx
const BASE_URL = 'https://example.com'
const DEFAULT_PORT = 3000
```

---

## Objects

### Literal Syntax and Shorthand

Use object literal syntax. Use property shorthand when property name matches variable name.

```tsx
// Good
const name = 'Kent'
const age = 36
const person = { name, age }

// Bad
const person = { name: name, age: age }
```

### Computed Properties

Use computed property names when creating objects with dynamic keys:

```tsx
// Good
const key = 'name'
const obj = { [key]: 'Kent' }

// Bad
const obj = {}
obj[key] = 'Kent'
```

### Method Shorthand

```tsx
// Good
const obj = {
  method() { /* ... */ },
  async asyncMethod() { /* ... */ },
}

// Bad
const obj = {
  method: function () { /* ... */ },
  asyncMethod: async function () { /* ... */ },
}
```

### No Accessors

Do not use getters and setters. They violate the principle of least surprise — `person.name` should just get the name, `person.name = 'Bob'` should just set it.

```tsx
// Good
const person = { name: 'Hannah' }

// Bad
const person = {
  get name() { /* hidden side effects */ },
  set name(value) { /* hidden side effects */ },
}
```

---

## Arrays

### Literal Syntax

```tsx
// Good
const items = [1, 2, 3]

// Bad
const items = new Array(1, 2, 3)
```

### Filtering Falsey Values

```tsx
// Good
const filtered = items.filter(Boolean)

// Bad
const filtered = items.filter((item) => item !== null && item !== undefined)
```

### Array Methods Over Loops

Use Array methods for pure transformations. Use `for...of` for imperative code. Never use `forEach`.

```tsx
// Good - array method for transformation
const doubled = items.map((n) => n * 2)

// Good - for...of for imperative code
for (const item of items) {
  processItem(item)
}

// Good - entries() for index access
for (const [i, item] of items.entries()) {
  console.log(`${item} at index ${i}`)
}

// Bad - forEach (never more readable than for...of)
items.forEach((item) => { processItem(item) })

// Bad - classic for loop
for (let i = 0; i < items.length; i++) { /* ... */ }
```

### Filter + Map Over Reduce

```tsx
// Good
const result = items.filter((n) => n > 2).map((n) => n * 2)

// Bad
const result = items.reduce((acc, n) => {
  if (n > 2) acc.push(n * 2)
  return acc
}, [])
```

### Non-Mutative Methods

Prefer `toReversed()`, `toSorted()`, `toSpliced()`, and `with()`:

```tsx
// Good
const reversed = items.toReversed()
const sorted = items.toSorted((a, b) => a - b)
const replaced = people.with(0, { name: 'John' })

// Bad - mutates original
items.reverse()
items.sort()
```

### Spread to Copy

```tsx
// Good
const copy = [...items]
const combined = [...arr1, ...arr2]

// Bad
const copy = items.slice()
const combined = arr1.concat(arr2)
```

### TypeScript Array Generic

```tsx
// Good
const items: Array<string> = []
function transform(numbers: Array<number>) {}

// Bad
const items: string[] = []
function transform(numbers: number[]) {}
```

---

## Destructuring

Use destructuring for objects and arrays:

```tsx
// Good
const { name, avatar, 𝕏: xHandle } = instructor
const [first, second] = items

// Bad
const name = instructor.name
const avatar = instructor.avatar
```

Nested destructuring is fine when readable, but avoid excessive depth:

```tsx
// Acceptable
const {
  name,
  address: [{ city, state }],
} = instructor

// Too deep - avoid
const [{ address: [{ city: { latitude, longitude } }] }] = data
```

---

## Strings

### Template Literals

Prefer template literals over concatenation:

```tsx
// Good
const greeting = `Hello ${name}`

// Bad
const greeting = 'Hello ' + name
```

### Multi-line Strings

```tsx
// Good
const html = `
<div>
  <h1>Hello</h1>
</div>
`.trim()

// Bad
const html = '<div>' + '\n' + '<h1>Hello</h1>' + '\n' + '</div>'
```

---

## Functions

### Function Declarations Over Expressions

Function declarations are hoisted, freeing you to organize code without worrying about ordering:

```tsx
// Good
function calculateTotal(items: Array<number>) {
  return items.reduce((sum, item) => sum + item, 0)
}

// Bad
const calculateTotal = function (items: Array<number>) {
  return items.reduce((sum, item) => sum + item, 0)
}

// Bad
const calculateTotal = (items: Array<number>) =>
  items.reduce((sum, item) => sum + item, 0)
```

### Limit Single-Use Functions

Don't break large functions into many small single-use functions. Extract only when logic needs reuse or represents a distinct concern:

```tsx
// Good - one function with clear inline logic
function doStuff() {
  // thing 1
  // thing 2
  // thing 3
}

// Bad - unnecessary extraction
function doThing1(param1: string, param2: number) {}
function doThing2(param1: boolean) {}
function doStuff() {
  doThing1(a, b)
  doThing2(c)
}
```

### Default Parameters

```tsx
// Good
function createUser(name: string, role = 'user') {
  return { name, role }
}

// Bad
function createUser(name: string, role: string) {
  role ??= 'user'
  return { name, role }
}
```

### Early Return

Use guard clauses to avoid deep nesting:

```tsx
// Good
function getMinResolution(resolution: number | undefined) {
  if (!resolution) return undefined
  if (resolution <= 480) return 'noLessThan480p'
  if (resolution <= 540) return 'noLessThan540p'
  return 'noLessThan1080p'
}

// Bad - deep nesting
function getMinResolution(resolution: number | undefined) {
  if (resolution) {
    if (resolution <= 480) {
      return 'noLessThan480p'
    } else if (resolution <= 540) {
      return 'noLessThan540p'
    } else {
      return 'noLessThan1080p'
    }
  } else {
    return undefined
  }
}
```

### Async/Await

Prefer async/await over promise chains. Exception: simple `.catch()` is fine to avoid try/catch boilerplate:

```tsx
// Good
async function fetchUserData(userId: string) {
  const user = await getUser(userId)
  const posts = await getUserPosts(user.id)
  return { user, posts }
}

// Good - simple .catch() is acceptable
function sendAnalytics(event: string) {
  return fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify({ event }),
  }).catch(() => null)
}

// Bad - promise chain
function fetchUserData(userId: string) {
  return getUser(userId).then((user) => {
    return getUserPosts(user.id).then((posts) => ({ user, posts }))
  })
}
```

### Inline Callbacks

Anonymous inline callbacks should be arrow functions with parenthesized parameters:

```tsx
// Good
const doubled = items.filter((n) => n > 2).map((n) => n * 2)

// Bad - function expression
const doubled = items.filter(function (n) { return n > 2 })

// Bad - no parens on single param
const doubled = items.filter(n => n > 2)
```

---

## Modules

### ESM Only

Use ECMAScript modules. Set `"type": "module"` in package.json.

### Named Exports

Prefer named exports. Only use default exports when required by framework conventions:

```tsx
// Good
export function add(a: number, b: number) {
  return a + b
}

// Bad
export default function add(a: number, b: number) {
  return a + b
}
```

### No Barrel Files

Do not use barrel files (`index.ts` that re-exports everything). They cause bundle bloat and circular dependency issues.

### Inline Exports

Export inline with the declaration:

```tsx
// Good
export function add(a: number, b: number) {
  return a + b
}

// Bad
function add(a: number, b: number) {
  return a + b
}
export { add }
```

### Single Import Per Module

Combine type and value imports in a single statement:

```tsx
// Good
import { type MatchSorterOptions, matchSorter } from 'match-sorter'

// Bad
import { type MatchSorterOptions } from 'match-sorter'
import { matchSorter } from 'match-sorter'
```

### Import Order

Group imports semantically:

```tsx
import 'node:fs'             // 1. Built-in
import 'match-sorter'        // 2. External packages
import '#app/components'     // 3. Internal absolute imports
import '../other-folder'     // 4. Internal relative imports
import './local-file'        // 5. Local imports
```

### Import Location

All static imports at the top of the file:

```tsx
// Good
import { matchSorter } from 'match-sorter'
function doStuff() { /* ... */ }

// Bad
function doStuff() { /* ... */ }
import { matchSorter } from 'match-sorter'
```

### Include File Extensions

Always include file extensions in imports (ECMAScript spec requires it):

```tsx
// Good
import { redirect } from 'react-router'  // package with exports field
import { add } from './math.ts'           // local file

// Bad
import { add } from './math'
```

### Import Aliases

Use the `imports` field in package.json:

```json
{
  "imports": {
    "#app/*": "./app/*",
    "#tests/*": "./tests/*"
  }
}
```

```tsx
import { add } from '#app/utils/math.ts'
```

### Pure Modules

Keep modules pure — avoid side effects at the module level:

```tsx
// Good
let serverData
export function init() {
  const el = document.getElementById('server-data')
  serverData = JSON.parse(el.textContent)
}

// Bad - side effect on import
const el = document.getElementById('server-data')
export const serverData = JSON.parse(el.textContent)
```

---

## Properties & Comparisons

### Dot Notation

Use dot notation unless the property is dynamic or uses special characters:

```tsx
// Good
const name = user.name
const id = user['data-id']

// Bad
const name = user['name']
```

### Triple Equals

Use `===` and `!==`. Exception: `== null` and `!= null` for null/undefined checks:

```tsx
// Good
if (user.id === '123') { /* ... */ }
if (value != null) { /* ... */ }

// Bad
if (value == null) { /* ... */ }
if (b !== null && b !== undefined) { /* ... */ }
```

### Rely on Truthiness

```tsx
// Good
if (user) { /* ... */ }

// Bad
if (user === true) { /* ... */ }
```

### Avoid Unnecessary Ternaries

```tsx
// Good
const isAdmin = user.role === 'admin'
const value = input ?? defaultValue

// Bad
const isAdmin = user.role === 'admin' ? true : false
const value = input != null ? input : defaultValue
```

---

## Control Flow

### Switch Statement Braces

Always use braces in switch cases:

```tsx
// Good
switch (action.type) {
  case 'add': {
    const { amount } = action
    add(amount)
    break
  }
  case 'remove': {
    const { removal } = action
    remove(removal)
    break
  }
}
```

### Statements Over Expressions

Unless using the value, prefer statements:

```tsx
// Good
if (user) {
  makeUserHappy(user)
}

// Bad
user && makeUserHappy(user)
```

### Braces for Multi-line Blocks

```tsx
// Good
if (!user) return
if (user.role === 'admin') {
  abilities = ['add', 'remove', 'edit', 'create', 'modify']
}

// Bad
if (user.role === 'admin')
  abilities = ['add', 'remove', 'edit', 'create', 'modify']
```

---

## Comments

### Explain "Why" Not "What"

```tsx
// Good
// We sanitize lineNumber to prevent malicious use on win32
// via: https://example.com/link-to-issue
if (lineNumber && !(Number.isInteger(lineNumber) && lineNumber > 0)) {
  return { status: 'error', message: 'lineNumber must be a positive integer' }
}

// Bad
// Check if lineNumber is valid
if (lineNumber && !(Number.isInteger(lineNumber) && lineNumber > 0)) {
  return { status: 'error', message: 'lineNumber must be a positive integer' }
}
```

### TODO and FIXME

- `TODO:` for future improvements
- `FIXME:` for immediate problems (linter should catch these to prevent accidental commits)

### @ts-expect-error Over @ts-ignore

Always include a comment explaining why:

```tsx
// Good
// @ts-expect-error - library types don't account for our custom prop
if (jsxEl.name !== 'EpicVideo') return

// Bad
// @ts-ignore
if (jsxEl.name !== 'EpicVideo') return
```

### JSDoc for Public APIs

```tsx
/**
 * Generates a TOTP code from a configuration.
 *
 * @param {OTPConfig} config - The configuration for the TOTP
 * @returns {string} The TOTP code
 */
export function generateTOTP(config: OTPConfig) { /* ... */ }
```

### No Redundant Comments

Don't add comments that repeat what the code already expresses:

```tsx
// Bad
// This function calculates the total
function calculateTotal(items: Array<number>) {
  return items.reduce((sum, item) => sum + item, 0)
}
```

---

## Semicolons

Don't use semicolons. Use the `no-unexpected-multiline` ESLint rule and a formatter:

```tsx
// Good
const name = 'Kent'
const age = 36
const person = { name, age }

// Bad
const name = 'Kent';
const age = 36;
```

The only time you need a semicolon is when a statement starts with `(`, `[`, or `` ` `` — prefix with `;`:

```tsx
const name = 'Kent'

;(async () => {
  const result = await fetch('/api/user')
  return result.json()
})()
```

---

## Types

### Type Inference

Let TypeScript infer return types:

```tsx
// Good
function add(a: number, b: number) {
  return a + b // inferred as number
}

// Bad
function add(a: number, b: number): number {
  return a + b
}
```

### Descriptive Generics

Treat generic type names like any other variable:

```tsx
// Good
function createArray<Value>(length: number, value: Value): Array<Value> {
  return Array(length).fill(value)
}

// Bad
function createArray<T>(length: number, value: T): Array<T> {
  return Array(length).fill(value)
}
```

### Type Guards Over Assertions

Avoid `as` — use type guards or runtime validation:

```tsx
// Good
function isError(maybeError: unknown): maybeError is Error {
  return (
    maybeError != null &&
    typeof maybeError === 'object' &&
    'message' in maybeError &&
    typeof maybeError.message === 'string'
  )
}

// Good - schema validation for boundary data
const OAuthDataSchema = z.object({
  accessToken: z.string(),
  refreshToken: z.string(),
  expiresAt: z.date(),
})
type OAuthData = z.infer<typeof OAuthDataSchema>
const oauthData = OAuthDataSchema.parse(rawData)

// Bad
const error = caughtError as Error
const oauthData = rawData as OAuthData
```

### Unknown Over Any

```tsx
// Good
function handleError(error: unknown) {
  if (isError(error)) console.error(error.message)
}

// Bad
function handleError(error: any) {
  console.error(error.message)
}
```

### Explicit Type Conversion

Avoid implicit coercion (truthiness is the exception):

```tsx
// Good
const number = Number(stringValue)
const string = String(numberValue)
if (user) { /* ... */ }

// Bad
const number = +stringValue
const string = '' + numberValue
if (Boolean(user)) { /* ... */ }
```

---

## Naming Conventions

Follow the [Naming Cheatsheet](https://github.com/kettanaito/naming-cheatsheet):

### Key Principles

1. **English** for all names
2. **Consistent** casing (camelCase for variables/functions, PascalCase for types/components)
3. **S-I-D**: Short, Intuitive, Descriptive
4. **No contractions** or context duplication

### A/HC/LC Pattern for Functions

Function names follow: **Action** + **High Context** + **Low Context**

| Part | Description | Example |
|------|-------------|---------|
| Action | What it does | `get`, `set`, `handle`, `create`, `remove` |
| High Context | What it operates on | `User`, `Post`, `Cart` |
| Low Context | Optional qualifier | `ById`, `WithDetails`, `FromCache` |

Examples: `getUserMessages`, `handleClickOutside`, `shouldDisplayMessage`

### Variable Naming

```tsx
// Good
const firstName = 'Kent'
const friends = ['Kate', 'John']
const pageCount = 5
const hasPagination = postCount > 10
const shouldPaginate = postCount > 10

// Bad
const primerNombre = 'Kent'  // not English
const amis = ['Kate', 'John']  // not English
const page_count = 5  // snake_case
const isPaginatable = postCount > 10  // unclear
const onItmClk = () => {}  // contraction
```

---

## Events

### Event Constants

```tsx
export const EVENTS = {
  USER_CODE_RECEIVED: 'USER_CODE_RECEIVED',
  AUTH_RESOLVED: 'AUTH_RESOLVED',
  AUTH_REJECTED: 'AUTH_REJECTED',
} as const
```

### Event Types

Derive from the constants:

```tsx
export type EventTypes = keyof typeof EVENTS
```

### Event Schemas

Use Zod for event payloads (events cross codebase boundaries):

```tsx
const CodeReceivedEventSchema = z.object({
  type: z.literal(EVENTS.USER_CODE_RECEIVED),
  code: z.string(),
  url: z.string(),
})
```

### Event Cleanup

Always clean up listeners:

```tsx
authEmitter.on(EVENTS.USER_CODE_RECEIVED, handleCodeReceived)
return () => {
  authEmitter.off(EVENTS.USER_CODE_RECEIVED, handleCodeReceived)
}
```

### Event Error Handling

Emit error events instead of swallowing errors:

```tsx
// Good
try {
  // logic
} catch (error) {
  authEmitter.emit(EVENTS.AUTH_REJECTED, { error: getErrorMessage(error) })
}

// Bad
try {
  // logic
} catch (error) {
  console.error(error)
}
```

---

## React

### Avoid useEffect

Most `useEffect` usage can be replaced with event handlers, ref callbacks, `useSyncExternalStore`, or CSS. See [You Might Not Need useEffect](https://react.dev/learn/you-might-not-need-an-effect).

Appropriate use: subscribing to external events with cleanup:

```tsx
useEffect(() => {
  const controller = new AbortController()
  window.addEventListener('keydown', (event: KeyboardEvent) => {
    if (event.key !== 'Escape') return
    // handle escape
  }, { signal: controller.signal })
  return () => controller.abort()
}, [])
```

### Derive State

```tsx
// Good
const [count, setCount] = useState(0)
const isEven = count % 2 === 0

// Bad
const [count, setCount] = useState(0)
const [isEven, setIsEven] = useState(false)
useEffect(() => { setIsEven(count % 2 === 0) }, [count])
```

### Don't Render Falsiness

```tsx
// Good
{contacts.length ? <div>You have {contacts.length} contacts</div> : null}

// Bad - renders "0" when contacts is empty
{contacts.length && <div>You have {contacts.length} contacts</div>}
```

### Ternaries for Conditionals

```tsx
{user.role === 'admin' ? <Link to="/admin">Admin</Link> : null}
```

---

## Testing

### Use `test()` Not `describe/it`

```tsx
// Good
test('User can add items to cart', async () => {
  render(<ProductList />)
  await userEvent.click(screen.getByRole('button', { name: /add to cart/i }))
  await expect(screen.getByText(/1 item in cart/i)).toBeInTheDocument()
})

// Bad
describe('ProductList', () => {
  it('should allow user to add items to cart', async () => { /* ... */ })
})
```

### No Nesting

Keep tests flat. No nested `describe` blocks.

### No Shared Setup Variables

```tsx
// Good
test('renders greeting', () => {
  render(<Greeting name="Kent" />)
  expect(screen.getByText('Hello Kent')).toBeInTheDocument()
})

// Bad
let utils
beforeEach(() => { utils = render(<Greeting name="Kent" />) })
```

### userEvent Over fireEvent

```tsx
// Good
await userEvent.type(screen.getByRole('textbox'), 'Hello')

// Bad
fireEvent.change(screen.getByRole('textbox'), { target: { value: 'Hello' } })
```

### Query Priority

1. `getByRole` — most accessible
2. `getByLabelText` — form elements
3. `getByPlaceholderText` — inputs
4. `getByText` — display elements
5. `getByTestId` — last resort

### Query Variants

- `getBy` — element exists now
- `queryBy` — assert element does NOT exist
- `findBy` — element will appear asynchronously

```tsx
// findBy for async
const result = await screen.findByText('Success')

// queryBy only for non-existence
expect(screen.queryByRole('alert')).not.toBeInTheDocument()
```

### MSW for External Services

```tsx
// Good
const server = setupServer(
  http.get('/api/user', () => {
    return HttpResponse.json({ name: 'Kent', role: 'admin' })
  }),
)

// Bad
vi.spyOn(global, 'fetch').mockResolvedValue({ /* ... */ })
```

### Testing Trophy Priority

1. Static Analysis (TypeScript, ESLint)
2. Unit Tests (Pure Functions)
3. Integration Tests (Component Integration)
4. E2E Tests (Critical User Flows)

---

## Code Review Checklist

Use this checklist when reviewing JavaScript/TypeScript code in pull requests.

### Variables & Naming

- [ ] Uses `const` by default, `let` only when reassigning, no `var`
- [ ] Descriptive variable names (no single-letter except loops)
- [ ] True constants use `UPPER_CASE`
- [ ] Names follow camelCase (variables/functions) and PascalCase (types/components)
- [ ] Function names follow A/HC/LC pattern

### Functions

- [ ] Uses function declarations, not function expressions
- [ ] Arrow functions used only for inline callbacks
- [ ] Arrow callbacks have parenthesized parameters
- [ ] Default parameters instead of short-circuiting
- [ ] Early returns / guard clauses instead of deep nesting
- [ ] Async/await instead of promise chains
- [ ] No unnecessary single-use function extraction

### Objects & Arrays

- [ ] Object/array literal syntax (no `new Object()` / `new Array()`)
- [ ] Property and method shorthand used
- [ ] No getters/setters
- [ ] `Array<T>` syntax, not `T[]`
- [ ] Array methods for transformations, `for...of` for imperative, no `forEach`
- [ ] `filter` + `map` over `reduce` for simple chains
- [ ] Non-mutative methods (`toReversed`, `toSorted`, `with`)
- [ ] Spread for copying/combining

### Types

- [ ] Return types inferred, not manually annotated
- [ ] Descriptive generic type names (not `T`, `U`)
- [ ] `unknown` instead of `any`
- [ ] Type guards over `as` assertions
- [ ] Schema validation (Zod) at codebase boundaries
- [ ] `@ts-expect-error` with comment, never `@ts-ignore`

### Modules

- [ ] Named exports only (unless framework requires default)
- [ ] No barrel files
- [ ] Inline exports with declarations
- [ ] Single import per module (combined type/value)
- [ ] File extensions included in local imports
- [ ] Imports at top of file in correct group order
- [ ] Pure modules (no side effects on import)
- [ ] ESM only

### Semicolons & Formatting

- [ ] No semicolons (handled by formatter + `no-unexpected-multiline`)
- [ ] Template literals for interpolation and multi-line strings
- [ ] Destructuring used where appropriate

### Comments

- [ ] Comments explain "why", not "what"
- [ ] No redundant comments
- [ ] `TODO:` for future improvements, `FIXME:` for immediate issues
- [ ] JSDoc on public APIs

### Comparisons & Control Flow

- [ ] Triple equals (`===`/`!==`), except `!= null`
- [ ] Truthiness relied on where appropriate
- [ ] No unnecessary ternaries
- [ ] Switch cases have braces
- [ ] Statements over expressions for side effects
- [ ] Braces for multi-line blocks

### React

- [ ] No unnecessary `useEffect` — state derived, not synced
- [ ] Falsy values not rendered (ternary with `null`)
- [ ] Ternaries used for conditionals in JSX

### Testing

- [ ] `test()` not `describe`/`it`
- [ ] Flat test structure, no nesting
- [ ] No shared `beforeEach` setup variables
- [ ] `userEvent` over `fireEvent`
- [ ] Accessible queries (`getByRole` > `getByTestId`)
- [ ] `queryBy` only for non-existence assertions
- [ ] `findBy` for async elements
- [ ] MSW for external service mocking
- [ ] Tests reflect user interactions, not implementation

### Events

- [ ] Event constants use `UPPER_CASE` in `as const` object
- [ ] Event types derived from constants
- [ ] Zod schemas for event payloads
- [ ] Event listeners cleaned up
- [ ] Error events emitted, not swallowed

### Common Review Feedback Templates

**Using `var` or unnecessary `let`:**
> This variable is never reassigned — use `const` instead of `let`.

**Function expression instead of declaration:**
> Prefer function declarations over arrow function expressions for named functions. Function declarations are hoisted and produce clearer stack traces.

**Array bracket syntax:**
> This project uses `Array<string>` syntax instead of `string[]`. See [Array Types in TypeScript](https://tkdodo.eu/blog/array-types-in-type-script) for rationale.

**Using `forEach`:**
> `forEach` is never more readable than `for...of` for imperative code, or `.map()` for transformations. Please replace with the appropriate alternative.

**Type assertion (`as`):**
> Avoid `as` type assertions — use a type guard or schema validation instead to get runtime safety.

**Using `any`:**
> Use `unknown` instead of `any` to force type checking before usage.

**Unnecessary `useEffect`:**
> This state can be derived directly from `count` instead of synced with `useEffect`. See [Don't Sync State, Derive It](https://kentcdodds.com/blog/dont-sync-state-derive-it).

**Using `describe`/`it`:**
> Use `test()` instead of `describe`/`it` for flatter, more readable test files. See [Avoid Nesting When Testing](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing).

**Using `fireEvent`:**
> Use `userEvent` over `fireEvent` for more realistic user interaction simulation.

**Missing semicolon-free style:**
> This project doesn't use semicolons — the formatter and `no-unexpected-multiline` ESLint rule handle edge cases.

---

## Related Resources

- [Naming Cheatsheet](https://github.com/kettanaito/naming-cheatsheet)
- [Array Types in TypeScript](https://tkdodo.eu/blog/array-types-in-type-script)
- [Pure Modules](https://kentcdodds.com/blog/pure-modules)
- [You Might Not Need useEffect](https://react.dev/learn/you-might-not-need-an-effect)
- [Don't Sync State, Derive It](https://kentcdodds.com/blog/dont-sync-state-derive-it)
- [Avoid Nesting When Testing](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing)
- [Semicolons in JavaScript](https://kentcdodds.com/blog/semicolons-in-javascript-a-preference)
