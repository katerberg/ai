---
name: stack-commit
description: >-
  Commit all working-tree changes onto the current branch with git, writing the
  commit message from the actual diff, then offering to push and open or update
  a pull request with gh. Use when the user asks to commit, add a commit, or
  commit and push/open a PR.
---

# Stack Commit

Create a new commit on the current branch from everything in the working tree,
with a message derived from what is actually being committed, then ask before
pushing or opening a PR.

## Progress

```
Task Progress:
- [ ] Step 1: Inspect what will be committed
- [ ] Step 2: Write the commit message
- [ ] Step 3: Stage and commit with git
- [ ] Step 4: Offer push / open PR
```

## Step 1: Inspect What Will Be Committed

Always stage everything. Do not ask the user which files to include and do not
stage a subset by hand — but do look at the whole working tree first.

1. Run `git status --short` and `git diff HEAD --stat`.
2. Understand exactly what a full stage sweeps in:
   - Modified tracked files, deletions, and untracked files.
   - Files matching `.gitignore` are **not** staged.
   - Anything already staged is included as well.
3. Read the diff itself (`git diff HEAD`, not just `--stat`) plus the contents
   of any untracked files, so the message describes real behavior rather than
   filenames. For a large diff, read the stat first, then the diff of the files
   carrying the actual change.
4. Before committing, scan the untracked list for anything that clearly should
   not land in history — scratch files, `.env` or credential files, editor
   droppings, large binaries, debug output. If you see one, name it and ask
   whether to commit it, ignore it, or delete it. Do not silently sweep it in.
   Otherwise proceed without asking.
5. If `git status` is clean, say so and stop. Do not create an empty commit.

## Step 2: Write the Commit Message

Write the message from the diff, not from the conversation's intent — what
ended up in the working tree and what was discussed can differ.

1. Match the repository's existing style. Check `git log --oneline -20` for
   conventional-commit prefixes, ticket keys, capitalization, and typical
   length, and follow whatever is dominant there.
2. Subject line: imperative mood, roughly 50–72 characters, no trailing period.
   It should say what the change does, not which files moved.
3. Add a body only when the change needs it — non-obvious reasoning, a
   behavioral consequence, or a constraint that explains the approach. Wrap at
   72 characters. Skip the body for small, self-evident changes.
4. Do not list every touched file, do not restate the diff, and do not include
   co-author or tool-attribution trailers unless the repository's own history
   or instructions use them.
5. If the working tree holds several unrelated changes, write a message that
   honestly covers them and say so in the report. Because this skill commits
   everything, splitting the work into separate commits is the user's call —
   offer it, but do not stall waiting for an answer.

## Step 3: Stage and Commit

```bash
git add -A
git commit -m "$(cat <<'EOF'
Subject line here

Optional body here.
EOF
)"
```

1. Always stage the full working tree with `git add -A` unless the user asked
   to exclude specific paths.
2. Pass the message via a HEREDOC as above. Subject only is fine when no body
   is needed.
3. Confirm the result with `git log -1 --stat` and `git status --short`. The
   working tree should be clean afterwards apart from ignored files.
4. If the commit fails a hook, report the failure and the hook output. Fix the
   issue and create a **new** commit. Do not retry with `--no-verify` unless
   the user asks. Do not amend unless the user explicitly requests it and the
   usual amend safety conditions are met.
5. Never update git config. Never use interactive git flags (`-i`).

## Step 4: Offer to Push / Open PR

Ask the user explicitly, in chat:

> Committed as `<subject>` on `<branch>`. Push and open/update a PR with `gh`?

1. Wait for a clear yes. Pushing and creating PRs — never run it on assumption,
   on silence, or because it was approved earlier in the session.
2. On yes:
   - If still on `main`/`master`, create a short kebab-case branch first with
     `git switch -c <name>` (never push to the default branch).
   - `git push -u origin HEAD`
   - Open or update the PR with `gh pr create` / `gh pr view` as appropriate.
     Prefer the repository's PR template when one exists.
3. Report the branch pushed and the PR URL.
4. On no or no answer, stop with the commit in place and say nothing was pushed.

## Report

Summarize:

- The branch and the commit message used
- What was included, calling out any untracked files that were swept in
- Whether the branch was pushed and/or a PR was opened, with links
