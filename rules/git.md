# Git guidelines

The repo's own `CLAUDE.md` or `CONTRIBUTING.md` wins where it says otherwise. In the control room, `/commit <repo>` applies all of this.

## Branches
- Name: `<type>/<short-kebab-summary>`, using the commit types below. Add an issue number when there is one: `fix/42-touch-volume`. Lowercase, `a-z0-9-` only, at most ~50 characters. Examples: `feat/dark-mode-toggle`, `fix/sitemap-trailing-slash`, `chore/upgrade-next-16`.
- The default branch (`main`, `master` or `develop`) is for finished work. New features and anything risky go on a branch; small docs or chore fixes may go straight to the default branch if the owner agrees.
- One branch per piece of work. Branch from an up-to-date default branch. Delete it once merged.

## Commit messages (Conventional Commits)
```
<type>(<optional scope>): <imperative summary>

<body: what changed and why, wrapped at ~72 columns>

<trailers>
```
- **Types:** `feat` (user-visible feature), `fix` (bug fix), `docs`, `style` (formatting only), `refactor` (no behaviour change), `perf`, `test`, `build` (build system, dependencies), `ci`, `chore` (maintenance), `revert`.
- **Scope:** optional; the area changed, as the repo already uses it (`ui`, `data`, `seo`, a route or package name). Reuse existing scopes from `git log` before inventing new ones.
- **Summary:** imperative mood ("add", not "added"), lowercase start, no trailing period, at most 72 characters including the prefix. Describe the outcome, not the files.
- **Body:** say why and what, in plain sentences or `-` bullets. Leave it out for trivial changes. Mention anything a reader of `git log` must know: migrations, follow-ups, behaviour that changed on purpose.
- **Breaking change:** add `!` after the type or scope (`feat(api)!: ...`) and a `BREAKING CHANGE: <what to do>` trailer.
- **Trailers:** `Refs: #42` / `Fixes: #42` when there's an issue, plus any co-author trailer Claude Code is configured to add.

## What goes in a commit
- One logical change per commit. Split unrelated changes into separate commits, and don't mix refactors with behaviour changes.
- Stage files by name. Never `git add -A` / `git add .` without reading `git status` first.
- Never commit: secrets or `.env*`, `*.local*` files, deploy scripts, build output (`dist/`, `out/`, `.next/`), `node_modules/`, OS or editor junk, or files over a few MB unless the repo tracks such assets on purpose.
- Every commit should build. Run the repo's checks (lint, typecheck, tests) before committing, and say so if any fail.

## Safety
- Never skip hooks (`--no-verify`), and never bypass signing.
- Never amend, rebase or squash commits that are already pushed; add a new commit instead. Never force-push to a shared or default branch. On your own branch, use `--force-with-lease` only, and only when asked.
- Push only when the owner asks: `git push -u origin <branch>` the first time.
- Merge conflicts: stop, show the conflicting files, and resolve them with the owner. Don't pick a side silently.
