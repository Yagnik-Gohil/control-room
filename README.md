# control-room

A Claude Code harness for a folder full of repos. Clone it next to your projects. The main session acts as an **engineering manager**: it plans the work and hands each task to a **specialist agent** (UI, backend, SEO, QA). `/init` links every sibling repo back to the control room through its `CLAUDE.md`, so all of them share the same rules and know their own context.

```
workspace/
  control-room/        ← this repo
  my-app/              ← your repos
  my-site/
  another-project/
```

## What you get

| Piece | What it does |
| --- | --- |
| `CLAUDE.md` | The manager's playbook: plan, delegate, verify, report, record |
| `.claude/agents/` | `ui-engineer`, `backend-engineer`, `seo-specialist`, `qa-engineer`, `repo-scout` |
| `/init` | First-time setup and re-sync: maps every sibling repo with `repo-scout`, then links its `CLAUDE.md` (generates one if missing, asks before changing one) |
| `/preflight <repo>` | QA and SEO audit in parallel, then a go/no-go before launch |
| `rules/` | Shared rules imported live by every linked repo (`core.md`; `public-repo.md` for public repos) |

## Setup

Needs only Claude Code and git.

```sh
cd path/to/your/workspace
git clone https://github.com/<you>/control-room.git
cd control-room
claude
> /init
```

That's it. In this folder `/init` is a project skill that replaces the built-in command: instead of writing a CLAUDE.md for control-room (it already has one), it sets up the whole workspace. Run it again any time to pick up new or changed repos. If your Claude Code client runs the built-in `/init` instead (the override applies to local terminal sessions), just say "set up the workspace".

`/init` finds every sibling repo, has a `repo-scout` agent map each one in parallel, and links them:

- **Repo has no `CLAUDE.md`:** the scout writes one (code guidance only) and a control-room section is added at the end.
- **Repo has a `CLAUDE.md`:** you get one question listing the proposed control-room section (and any fixes the scout found) for each repo. Only what you approve is written.
- **Every repo:** deploy scripts get denied in its `.claude/settings.local.json`, and its private context goes in `.local/repos/<repo>.md`.

Then fill in `.local/workspace.md` (copied from `templates/workspace.md`) with your own hosting, deploy and domain facts, or let the manager interview you.

The first time you open Claude Code **inside** a linked repo, it asks once to approve "external imports" (the files in control-room). Approve them, or the link won't load.

## How linking works

Each repo's `CLAUDE.md` ends with a marked section (`<!-- control-room:begin -->` ... `<!-- control-room:end -->`) that imports, by relative path:

- `rules/core.md`, the shared rules;
- `rules/public-repo.md`, for public repos only;
- `.local/repos/<repo>.md`, the repo's private context (hosting, deploy, indexing).

Because the imports point to files instead of copying them, **editing a rule here takes effect in every repo at once**. Run `/init` again only when you add repos or a repo has moved on; it compares each repo's HEAD with the `head` recorded in its context file.

The section is safe to commit, even in public repos: it holds only relative imports, and the private facts stay in this repo's git-ignored `.local/`. Someone who clones one of your app repos alone gets nothing extra, because the imports point at files they don't have.

## Public vs private

Everything tracked here is generic. Your machine's facts live in the git-ignored `.local/`:

| File | Holds |
| --- | --- |
| `.local/workspace.md` | Owner, hosting, deploy setup, extra rules, repos to skip and extra guard rules (imported into the manager's CLAUDE.md) |
| `.local/repos/<repo>.md` | Per-repo context written by `repo-scout`; its header records visibility and the last-scouted commit |
| `.local/log.md` | One dated line per finished task |

To back up `.local/`, make it a separate **private** git repo (`git init` inside it).

When you change a rule, the manager decides where it goes: rules any workspace would want go into `rules/` or an agent file (and can be pushed upstream), and rules about your own setup go into `.local/workspace.md`.

## Safety defaults

- Deploy scripts (`deploy.bat`, `deploy.sh`, `deploy.ps1`) are denied in every linked repo and in the control room. You run deploys yourself, e.g. `! deploy.bat`.
- `git push` and `aws` commands always ask first in the control room.
- Specialists never commit, push or deploy; they report back to the manager.

## Customising

- Add a specialist: drop a new file in `.claude/agents/` and add a row to the team table in `CLAUDE.md`.
- Add a shared rule: edit `rules/core.md`, or add a file to `rules/` and an import line to `templates/claude-md-section.md`, then `/init`.
- Change guards: the list in `.claude/skills/init/SKILL.md` (public defaults) or "Extra guard rules" in `.local/workspace.md` (yours only).
