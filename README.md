# Skills
Agent Skills

# What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add these to your project, Claude Code can recognize when you're working on a specific task and apply the right frameworks and best practices.

## Available Skills

| Skill | Description | Triggers |
|-------|-------------|----------|
| [har-extraction](./skills/har-extraction/SKILL.md) | Extract JSON mocks from HAR files with har-to-mocks and serve them via MSW (dev, Storybook, tests) or Chrome DevTools Local Overrides | HAR files, API mocking, MSW setup, Storybook integration, Chrome overrides, bug reproduction from a HAR |
| [git-conventions](./skills/git-conventions/SKILL.md) | Commit message (`<type>(<taskId>): <description>`) and branch name (`<type>/<taskId>_<description>`) format for Jira (`ABC-123`) and GitHub (`#42`) issue IDs | Running `git commit`, creating a branch (`git checkout -b`, `git switch -c`, `git worktree add -b`). Applied automatically, not slash-invocable |
| [adding-a-util](./skills/adding-a-util/SKILL.md) | Decide whether a helper belongs in the shared utils folder (reused and pure) or beside its caller, following the repo's utils catalogue (its README, written first if missing), then add it test-first with JSDoc and a catalogue row | Writing or extracting a helper (formatting, dates, parsing, predicates over API data) in a page or component, adding to the utils folder |


## Installation

### Option 1: CLI Install (Recommended)

Use [skills](https://github.com/vercel-labs/skills) to install skills directly:

```bash
# Install all skills
npx skills add peterknezek/skills

# Install specific skills
npx skills add peterknezek/skills --skill har-extraction git-conventions

# List available skills
npx skills add peterknezek/skills --list
```

This automatically installs to your `.claude/skills/` directory.

### Option 2: Clone and Copy

Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/peterknezek/skills.git
cp -r skills/skills/* .claude/skills/
```

### Project setup for adding-a-util

The skill only triggers when the agent is about to write a helper. To make the agent check the utils catalogue earlier, add this pointer to your project's `AGENTS.md` (or `CLAUDE.md`), adjusting the path and helper kinds to your repo:

```markdown
# Agent guide

- **Utils**: before writing a formatting, date, validation, copy or loan-state helper, look it up in the [utils catalogue](src/utils/README.md); most already exist.
```

## Usage

Once installed, just ask Claude Code to help with relevant tasks:

```
"Extract mocks from this HAR file for my API tests"
→ Uses har-extraction skill
```

```
"Set up MSW with mocks from network recordings"
→ Uses har-extraction skill
```

```
"Commit this" / "Create a branch for ABC-123"
→ Uses git-conventions skill
```

```
"Extract this date formatting into a util"
→ Uses adding-a-util skill
```

You can also invoke skills directly:

```
/har-extraction
```
