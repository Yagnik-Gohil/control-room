# Public repo rules

This repo is (or may be) public on GitHub. Everything committed here is readable by anyone and stays in history forever.

- The `control-room` section of `CLAUDE.md` holds only relative imports and is fine to commit. Nothing else here may carry workspace details.
- No personal or infrastructure details in tracked files: bucket names, distribution IDs, account IDs, internal hostnames, local paths like `E:\...` or `C:\Users\...`, private email addresses.
- Never commit `.claude/settings.local.json`, `.env*`, deploy scripts or build output that the repo ignores. Check `git status` before any commit.
- Code or assets copied from elsewhere need a compatible license and credit. If the origin is unclear, ask before committing it.
- Commit messages and code comments are public writing: clear, professional, no internal notes or TODOs that expose plans or infra.
- Before suggesting a push, scan the diff for the items above and say what you checked.
