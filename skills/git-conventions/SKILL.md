---
name: git-conventions
user-invocable: false
description: >
  Enforces git commit message and branch naming conventions for the Valhalla
  frontend platform. ALWAYS use this skill before running git commit,
  git checkout -b, or any branch creation command. Use it when the user says
  "commit", "create a branch", "make a commit", "stage and commit", "push this",
  or when you are about to execute git commit or git checkout -b yourself.
  Applies the format: commits as `<type>(<taskId>): <brief description>` and
  branches as `<type>/<taskId>_<brief-description>`. Trigger even if the user
  doesn't mention conventions — applying them is always the right thing to do.
---

# Git Conventions

Follow these rules every time you create a commit or a branch. The goal is a
consistent, readable git history where each commit and branch name immediately
communicates the task it belongs to and what was changed.

---

## Commit Message Format

```
<type>(<taskId>): <brief description>
```

- **One line only** — no body, no footer
- `type`: `fix` | `feat` | `refactor` | `chore`
- `taskId`: the Jira-style ID (e.g. `CDC-123`, `LA-1427`, `EL-106`)
- `brief description`: imperative mood, lowercase, no period at the end

**Examples:**
```
fix(CDC-590): fix login redirect on session expiry
feat(LA-1427): add call feature in TMS
refactor(EL-106): remove eslint warnings in consents lib
chore(CDC-100): update dependencies in shared lib
```

---

## Branch Name Format

```
<type>/<taskId>_<brief-description>
```

- `brief-description`: kebab-case, **must include the affected lib or module name**
- Separator between `<taskId>` and description is an underscore `_`
- Use hyphens `-` within the description

**Examples:**
```
fix/CDC-590_fix-login-redirect-auth-lib
feat/LA-1427_add-call-feature-tms
refactor/EL-106_eslint-warnings-consents-lib
chore/CDC-100_update-deps-shared-lib
```

---

## Resolving the Task ID

When you don't already know the task ID:

1. Run `git branch --show-current` to get the current branch name
2. Extract the task ID using the pattern: `(fix|feat|refactor|chore)/([A-Z]+-\d+)_`
   - Example: `fix/CDC-590_fix-login` → task ID is `CDC-590`
3. If found, use it and briefly note to the user what you extracted
4. If not found (e.g. on `main`, `master`, or an unformatted branch), ask the user:
   > "What's the task ID for this work? (e.g. CDC-123)"

---

## Workflow

### Creating a commit

1. Resolve the task ID (see above)
2. Infer the `type` from the nature of the staged changes:
   - Bug fix → `fix`
   - New capability → `feat`
   - Code restructure without behavior change → `refactor`
   - Tooling, config, deps → `chore`
3. Write a short imperative description of what changed
4. Assemble: `<type>(<taskId>): <brief description>`
5. Run: `git commit -m "<message>"`

### Creating a branch

1. Resolve the task ID (see above)
2. Determine the `type` for the work being done
3. Write a kebab-case description that includes the lib/module name
4. Assemble: `<type>/<taskId>_<brief-description>`
5. Run: `git checkout -b <branch-name>`
