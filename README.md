# AI

This repository contains shared AI-agent resources for the development team.
It currently provides reusable Cursor Agent Skills for common workflows such as
reviewing pull requests and resolving merge conflicts.

Keeping these resources in a separate repository lets the team version,
review, and improve them independently of any application repository.

## Setup

Clone this repository at `~/src/ai`, then make its skills available to Cursor:

```bash
ln -s ~/src/ai/skills/ ~/.cursor/skills
ln -s ~/src/ai/skills/ ~/.agents/skills
```

The command expects `~/.cursor/skills` not to exist already. If it does, move or
remove it first after preserving any personal skills it contains.

Restart Cursor after creating the symlink so it discovers the skills.

## Using the Repository

Each directory under `skills/` contains a `SKILL.md` file that describes a
workflow and when Cursor should use it. Ask Cursor for the relevant task in
natural language—for example, ask it to review a PR or resolve merge conflicts.
Cursor uses each skill's description to select the appropriate instructions.

To update your local skills, pull the latest changes:

```bash
cd ~/src/ai
git pull
```

Because Cursor reads the repository through the symlink, pulled changes become
available without copying files.

## Adding or Updating a Skill

1. Create or edit `skills/<skill-name>/SKILL.md`.
2. Include valid YAML frontmatter with a lowercase, hyphenated `name` and a
   specific `description` explaining what the skill does and when to use it.
3. Keep instructions concise, actionable, and independent of a single project
   unless the skill is intentionally project-specific.
4. Open a pull request so the team can review the workflow.
