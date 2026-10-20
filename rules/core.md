# Workspace rules (control room)

These apply in every repo linked to the control room. The repo's own `CLAUDE.md` wins on anything about its code; these rules win on anything that leaves the machine.

- **Ask before anything that leaves this machine:** pushing, deploying, publishing packages, DNS or cloud changes, search-console actions, posting to external services. Say what will happen first.
- **Never run deploy scripts.** `deploy.*` files are the owner's to run (`! deploy.bat` in a session). They're denied in settings; don't work around the deny.
- **No secrets in files.** No keys, tokens, passwords or account IDs in any tracked file. Ignored `.env` files stay unread unless the owner asks.
- **Keep private facts out of this repo.** The only link to the control room is the marked `control-room` section of `CLAUDE.md`; don't edit inside it (`/init` in the control room maintains it). Hosting details, deploy targets and cross-repo facts belong in the control room's `.local/`, not in tracked files here.
- **Done means checked.** Before calling a task done, run the repo's own checks (build, tests, lint, typecheck: whatever its `CLAUDE.md` or `package.json` lists) and report the results, including failures.
- **Stay in scope.** Change only what the task needs. No drive-by refactors, dependency upgrades or new tooling without asking.
- **Commits:** only when asked, on the repo's current branch unless told otherwise. Never force-push, never rewrite published history.
- **Facts you learn that other repos or sessions need** (a new hosting detail, a changed command, an open TODO) go to the owner as a suggested control-room update, not into this repo.
