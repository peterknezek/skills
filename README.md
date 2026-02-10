# Skills
Agent Skills

# What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add these to your project, Claude Code can recognize when you're working on a specific task and apply the right frameworks and best practices.

## Available Skills

| Skill | Description | Triggers |
|-------|-------------|----------|
| [har-extraction](./skills/har-extraction/SKILL.md) | Extract JSON mocks from HAR files and integrate with MSW for testing and Storybook | HAR files, API mocking, MSW setup, Storybook integration, network traffic recording |
| [story-naming](./skills/story-naming/SKILL.md) | Naming conventions for Storybook stories describing user scenarios and component states | Creating stories, naming story exports, reviewing story names in PRs |
| [storybook-interactions](./skills/storybook-interactions/SKILL.md) | Best practices for writing Storybook play functions and interaction tests | Play functions, interaction testing, Storybook test structure, reviewing play functions in PRs |
| [coding-style](./skills/coding-style/SKILL.md) | JavaScript and TypeScript coding style conventions for clean, maintainable code | Writing JS/TS code, code style reviews, variable/function/module conventions |


## Installation

### Option 1: CLI Install (Recommended)

Use [skills](https://github.com/vercel-labs/skills) to install skills directly:

```bash
# Install all skills
npx skills add peterknezek/skills

# Install specific skills
npx skills add peterknezek/skills --skill har-extraction second-skill

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

## Usage

Once installed, just ask Claude Code to help with relevant tasks:

```
"Extract mocks from this HAR file for my API tests"
→ Uses har-extraction skill
```

```
"Create stories for the UserProfile component"
→ Uses story-naming skill
```

```
"Add a play function to test the login form"
→ Uses storybook-interactions skill
```

```
"Review this code for style issues"
→ Uses coding-style skill
```

You can also invoke skills directly:

```
/har-extraction
/story-naming
/storybook-interactions
/coding-style
```
