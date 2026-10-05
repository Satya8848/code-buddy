# Git & Release Workflow

## Branches
- `main` / `version-XX` → production. Protected; merge via PR only.
- `staging` → deployed to staging server.
- `develop` → integration branch.
- Feature branches: `feat/<ticket>-short-desc`, fixes: `fix/<ticket>-short-desc`, hotfixes: `hotfix/<desc>` branched from production.

## Commits
- Conventional Commits: `feat:`, `fix:`, `refactor:`, `perf:`, `chore:`, `docs:`, `test:`. Optional scope: `fix(dispatch): ...`.
- One logical change per commit. No "wip", "changes", "final" messages on shared branches.

## Pull requests
- Every PR: description of what & why, linked ticket, screenshots for UI, list of patches added, migration notes.
- At least one reviewer approval. Tech lead approval for: hooks.py changes, new whitelisted guest methods, patches touching > 10k rows, server config changes.
- CI must pass (lint, tests) before merge.
- Run Code Buddy's code review on the PR before requesting human review.

## Promotion path
`feature → develop → staging (UAT with client) → production`. Nothing goes to production that has not run on staging with a production-like data copy.
