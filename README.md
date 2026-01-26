# Skills
Agent Skills

# What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add these to your project, Claude Code can recognize when you're working on a marketing task and apply the right frameworks and best practices.

## Available Skills

| Skill | Description | Triggers |
|-------|-------------|----------|


## Installation

### Option 1: CLI Install (Recommended)

Use [add-skill](https://github.com/vercel-labs/add-skill) to install skills directly:

```bash
# Install all skills
npx add-skill peterknezek/skills

# Install specific skills
npx add-skill peterknezek/skills --skill one-skill second-skill

# List available skills
npx add-skill peterknezek/skills --list
```

This automatically installs to your `.claude/skills/` directory.

### Option 2: Clone and Copy

Clone the entire repo and copy the skills folder:

```bash
git clone https://github.com/peterknezek/skills.git
cp -r skills/skills/* .claude/skills/
```

## Usage

Once installed, just ask Claude Code to help with marketing tasks:

```
"Something"
→ Uses one-skill skill
```

You can also invoke skills directly:

```
/one-skill
```
