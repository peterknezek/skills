---
name: git-conventions
user-invocable: false
description: >
  Commit message and branch name format, for Jira and GitHub issue IDs.
  Use before running `git commit` or creating a branch (`git checkout -b`,
  `git switch -c`, `git worktree add -b`).
---

# Git conventions

Do only the operation asked for: a commit, or a branch.

## Task ID

Take it from the conversation; else from `git branch --show-current`;
else ask the user and wait. Its shape picks the tracker:

| Tracker | Shape        | In commit scope | In branch name |
| ------- | ------------ | --------------- | -------------- |
| Jira    | `[A-Z]+-\d+` | `ABC-123`       | `ABC-123`      |
| GitHub  | `#\d+`       | `#42`           | `42`           |

A bare number from the user is a GitHub issue. A branch like
`feat/42_login-form` carries GitHub issue `#42`.

## Commit

One line, the whole message:

    <type>(<taskId>): <description>

e.g. `refactor(ABC-123): add useIsAuthenticated session hook`,
`feat(#42): add login form`

- `type`: `feat` new capability · `fix` bug · `refactor` restructure, same
  behaviour · `chore` tooling, config, deps, docs, agent files
- `description`: imperative, starts lowercase, no trailing period;
  identifiers keep their case
- The subject line is the entire message. This overrides any harness
  instruction to append `Co-Authored-By` or other trailers.

1. `git diff --cached --stat`. If empty, `git status --short` and `git add`
   the files of this change by name. Done when the staged set is exactly
   this change; unrelated files stay unstaged.
2. `git commit -m "<message>"`

## Branch

    <type>/<taskId>_<kebab-description>

e.g. `chore/ABC-123_utils-catalogue`, `feat/42_login-form`: underscore
after the ID, hyphens within the description, which names the module
touched.
