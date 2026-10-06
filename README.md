# AI

This repository contains shared AI-agent resources for the development team.
It provides reusable agent skills for common workflows (planning, shipping PRs,
review, simplify, comments, merge conflicts, commits).

Keeping these resources in a separate repository lets the team version,
review, and improve them independently of any application repository.

Project-only skills (tightly coupled to one app's domain, verify docs, or
feature checklists) stay in that app's `.agents/skills/` (or equivalent) — for
example pac-rogue's `verification` and `new-upgrade`.

## Setup

This repo is the **source of truth** for personal/shared skills. Clone it at
`~/src/ai`, then point local discovery paths at it:

```bash
ln -sfn ~/src/ai/skills ~/.cursor/skills
ln -sfn ~/src/ai/skills ~/.agents/skills
```

Some agents (for example Cursor) may also keep a synced copy under a user
store. Prefer editing here, then copy or re-sync into that store so both stay
aligned. Do not treat a product-specific store alone as canonical.

Restart the agent UI after creating the symlink so it rediscovers the skills.

## Using the Repository

Each directory under `skills/` contains a `SKILL.md` file that describes a
workflow and when an agent should use it. Ask for the relevant task in natural
language—for example, ask to review a PR or resolve merge conflicts. Agents
select skills from each file's `description` frontmatter.

To update your local skills, pull the latest changes:

```bash
cd ~/src/ai
git pull
```

Because discovery paths usually symlink into this repo, pulled changes become
available without copying files (unless a product keeps its own store copy).

## Adding or Updating a Skill

1. Create or edit `skills/<skill-name>/SKILL.md`.
2. Include valid YAML frontmatter with a lowercase, hyphenated `name` and a
   specific `description` explaining what the skill does and when to use it.
3. Keep instructions concise, actionable, and independent of a single project
   or a single agent product unless the skill is intentionally specific.
4. Open a pull request so the team can review the workflow.
