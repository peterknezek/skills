---
name: adding-a-util
description: Decide whether a helper belongs in the shared utils folder and add it test-first. Use when writing a helper function (formatting, dates, parsing, a predicate over API data) inside a page or component, when extracting one, or when adding to utils.
---

# Adding a util

The **catalogue** is the README of the repo's utils folder (usually `src/utils/README.md`): it lists every helper and the repo's own placement rules. A util earns its place by being **reused** and **pure**; everything else stays beside the code that uses it.

## 0. Find the catalogue

Read it. If the repo has a utils folder but no catalogue, write one first: a table of module, exports and what each is for, from reading the folder. Done when the catalogue exists and you have read it.

## 1. Look for an existing helper

Search the catalogue, then grep the source for the logic written inline (the `Intl`, date-library or regex call you were about to write).

Done when you either reuse an existing export, extend an existing module, or can name why none fits. Inline copies you find elsewhere go in your report to the user; leave those call sites unchanged.

## 2. Place it

The catalogue's **Where helpers go** section wins when it covers the case. Otherwise:

| The code… | Goes in |
| --- | --- |
| is pure and serves two or more pages, or clearly will | the utils module of the same subject; a new `<subject>.ts` only when none matches |
| serves one page or component | a file beside that page or component |

**Pure** means the output depends only on the arguments: take "now" (`now: Date = new Date()`), the locale or the translations as parameters when the result depends on them. Accept `null` / `undefined` where API data may still be loading, and return the empty value (`''`, `undefined`) for it.

Done when you can name the destination and the rule that sent it there.

## 3. Write it test-first

1. **Red:** a failing test in `<module>.test.ts` beside the module. Run it and see it fail for the expected reason.
2. **Green:** the smallest implementation that passes.
3. One test at a time, until the loading case (`null` / `undefined`) and the edge the util exists for (a day boundary, a zero amount, a prefix that must stay) are covered.

Run only that test file, with the repo's test script from `package.json`.

## 4. Document it

- JSDoc on every export: one sentence on what it is for, plus an `@example` with the real output.
- A row (or new exports in an existing row) in the catalogue.

Done when the tests are green, the type check is clean, the JSDoc carries an `@example`, and the catalogue row names every new export.
