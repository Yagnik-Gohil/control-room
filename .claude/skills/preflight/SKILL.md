---
name: preflight
description: Pre-launch go/no-go for one repo. Runs the qa-engineer and seo-specialist in parallel on the current state of the repo, then combines their verdicts. Use before deploying or making a site public.
argument-hint: <repo>
disable-model-invocation: true
---
# /preflight $ARGUMENTS

1. Resolve the repo from `$ARGUMENTS` against the files in `.local/repos/`. If it's missing or ambiguous, ask. Read `.local/repos/<repo>.md` and note the build command, build directory, live URL and indexing decision.
2. Note uncommitted changes (`git -C <repo> status --short`). Preflight checks the working tree as it is; say so in the report if it's dirty.
3. In parallel, launch:
   - `qa-engineer`: full check set plus the built app's critical paths, deep links and mobile width. Audit only: no app code changes.
   - `seo-specialist`: audit of the built output against the repo's indexing decision. Audit only, no fixes.
   Give both the repo path, the context file's facts and "this is a pre-launch check".
4. Combine the results into one report:
   - **Go / no-go** at the top. No-go if any check fails or there's any blocker.
   - Blockers first, then should-fix, then nice-to-have, each with an owner (ui-engineer, backend-engineer, seo-specialist or the owner).
   - Owner actions after launch (e.g. submit the sitemap).
5. Append one dated line to `.local/log.md` with the verdict.
6. Offer to dispatch fixes for the blockers. Never deploy. If it's a go, give the owner the exact deploy command to run themselves.
