---
name: init
description: Set up or refresh the workspace from the control room. Replaces the built-in /init here. On first run it creates .local/, maps every sibling repo with repo-scout agents, generates a CLAUDE.md for repos without one, and proposes a control-room section for repos that already have one (applied only after the owner approves). Re-run any time to pick up new or changed repos. Use when the owner runs /init or asks to set up, link or sync the workspace.
argument-hint: "[repo ...]"
---
# /init $ARGUMENTS

This replaces the built-in `/init` in the control room. The control room's own CLAUDE.md is already written, so here `/init` means "set up the workspace". Never rewrite this repo's CLAUDE.md.

Link the repos next to this control room. It's safe to run again: each step skips what's already done. The workspace is the control room's parent folder. If the owner named repos in `$ARGUMENTS`, do only those.

## 1. Prepare the desk
- If `.local/workspace.md` is missing, copy `templates/workspace.md` there and offer to interview the owner to fill it in (after linking, not before).
- Make sure `.claude/settings.local.json` here has the workspace folder (absolute path, forward slashes) in `permissions.additionalDirectories` and the guard rules below in `permissions.deny`.

## 2. Find the repos
List every folder in the workspace that isn't hidden, isn't `node_modules`, isn't the control room, and has a `.git` or a manifest (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, ...). Skip any listed under "Skip" in `.local/workspace.md`.

For each, collect: `git rev-parse --short HEAD`, `git remote get-url origin`, whether `CLAUDE.md` exists, and whether it already contains `<!-- control-room:begin -->`.

## 3. Work out what each repo needs
Read `.local/repos/<repo>.md` if it exists and compare its frontmatter `head` with the repo's HEAD.

- **Context missing, a stub, or HEAD moved** → needs a scout.
- **Visibility:** keep the frontmatter value if it's `public` or `private`. Otherwise check GitHub remotes with `curl -s -o /dev/null -w "%{http_code}" https://api.github.com/repos/<owner>/<repo>` (200 = public; 404 = private or missing). No remote → `private`. If still unsure, ask the owner.

## 4. Scout (parallel)
Launch one `repo-scout` agent per repo that needs it, all in parallel. Brief each with the repo's absolute path, its context file path, the visibility, HEAD, remote and today's date for the frontmatter, and whether the repo has a `CLAUDE.md`.

When they finish, read each context file and check it looks right.

## 5. Link each repo's CLAUDE.md
Build the section from `templates/claude-md-section.md`: fill in `{{repo}}`. Replace `{{public-rule}}` with `@../control-room/rules/public-repo.md` for public repos; for private ones, delete that line. Adjust `../control-room` if the control room's folder has another name.

- **No CLAUDE.md:** the scout has drafted one (code guidance only: commands, architecture, conventions; nothing private, since it's a tracked file). Append the section and write it. List it in the report as new.
- **CLAUDE.md with the section already:** if the section differs from what you just built, replace only the text between the markers. Treat that as a proposed change (next bullet). If it's identical, leave the file alone.
- **CLAUDE.md without the section:** don't write anything yet. Collect a proposal: the section to add at the end, plus any fixes the scout found (wrong commands, outdated stack). Then ask the owner **once for all repos**, with AskUserQuestion (multi-select: one option per repo, with a short description of what changes), showing the proposed text. Apply only what they approve. Never touch the rest of the file without approval.

`CLAUDE.md` is a tracked file. Never commit it: tell the owner which repos now have an uncommitted CLAUDE.md change.

## 6. Guards
In each repo, make sure `.claude/settings.local.json` has the guard rules in `permissions.deny`. Merge them; never remove existing entries. If `git check-ignore -q .claude/settings.local.json` says it isn't ignored, add the path to `.git/info/exclude` (not `.gitignore`).

Guard rules: `Bash(*deploy.bat*)`, `PowerShell(*deploy.bat*)`, `Bash(*deploy.sh*)`, `PowerShell(*deploy.ps1*)`, plus any listed under "Extra guard rules" in `.local/workspace.md`.

## 7. Report
Show a table with these columns: repo, visibility, context (new / refreshed / up to date), CLAUDE.md (created / updated / proposal declined / up to date), and notes from the scouts. Append one dated line to `.local/log.md`.

Remind the owner that the first Claude Code session in each linked repo asks once to approve "external imports" (the control-room files). Approve it, or the link won't load.
