---
name: commit
description: Create well-formed commits in a workspace repo (or the control room) following rules/git.md - picks or creates a correctly named branch, splits unrelated changes, scans for things that must not be committed, writes Conventional Commit messages, and pushes only when asked. Use when the owner asks to commit, save work to git, or create a branch.
argument-hint: "[repo] [push] [on <branch>]"
---
# /commit $ARGUMENTS

Read the control room's `rules/git.md` first and follow it. Work in the repo named in the arguments: a sibling folder of the control room, or `control-room` itself. With no repo named, use the repo the conversation has been working in; if that's unclear, ask.

## 1. Look before touching anything
Run these in the repo, in parallel:
- `git status --short --branch`
- `git diff` and `git diff --staged`
- `git log --oneline -15`, to see the repo's scopes and style
- `git branch --show-current`, and the default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`, else `main`/`master`/`develop` as present)

If there's nothing to commit, say so and stop. If a merge, rebase or cherry-pick is in progress, stop and report it.

## 2. Screen the changes
Flag and leave **unstaged** anything matching the "never commit" list in the guidelines: secrets or tokens in the diff, `.env*`, `*.local*`, deploy scripts, build output, `node_modules`, large binaries. Tell the owner what you left out and why. If a secret appears in a tracked file, stop and say so; don't commit it.

## 3. Group into commits
Sort the changes into logical commits: one purpose each, refactors apart from behaviour changes. For each, pick the type and scope (reuse scopes from the log) and the files. One change → one commit; don't split for the sake of it.

## 4. Branch
- If the arguments say `on <branch>`, use that branch, creating it if needed.
- If you're already on a non-default branch, stay on it unless the changes clearly belong to different work.
- If you're on the default branch, propose a branch name built from the main commit's type and summary (`<type>/<kebab-summary>`). Ask the owner with AskUserQuestion: create that branch (recommended for features, fixes and anything risky), or commit on the default branch (fine for small docs or chore changes). Skip the question if the owner already said which.
- New branch: `git switch -c <name>`. Uncommitted changes carry over.

## 5. Check
Run the repo's quick checks that cover the changed files (lint, typecheck, unit tests: whatever its CLAUDE.md or `package.json` lists). Skip slow suites such as e2e unless the owner asks. If checks fail, show the failures and ask whether to fix first or commit anyway; never hide a failure.

## 6. Commit
For each group, in order:
- `git add <files by name>` (or `git add -p` style partial staging via `git apply --cached` when a file holds changes for two commits).
- Commit with a heredoc so the body keeps its formatting. Add the co-author trailer Claude Code is configured to use, if any:
  ```sh
  git commit -F - <<'EOF'
  feat(ui): add dark mode toggle to the header

  The toggle follows the system setting until the user picks a theme,
  then remembers the choice in localStorage.
  EOF
  ```
- If a hook rejects the commit, fix the cause and make a **new** commit attempt. Never use `--no-verify` and never amend a commit you didn't just create.

## 7. Push (only when asked)
Push only if the arguments include `push` or the owner asks: `git push -u origin <branch>` for a new branch, else `git push`. Never force-push. If the push is rejected because the remote moved, report it and ask before pulling or rebasing.

## 8. Report
In a few lines: the branch, each commit as `<short-sha> <subject>`, anything left uncommitted and why, the check results, and whether it was pushed. If a branch was pushed, add the compare URL for opening a pull request (`https://github.com/<owner>/<repo>/compare/<branch>?expand=1` for GitHub remotes).
