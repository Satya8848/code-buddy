---
name: team-standards
description: >
  This skill should be used when the user asks "what are our standards", "our coding conventions",
  "team coding rules", "how should we do X", "use our standards repo", or whenever another Code Buddy
  skill (code review, security audit, performance review, upgrade check, restructure, cleanup,
  pattern conversion, server review, server architecture) needs the rules to apply. Provides
  best-practice standards for Frappe/ERPNext, Python, JavaScript, SQL, Git, security,
  environments (dev/staging/production) and servers, and loads a team's own standards repo when
  one is configured.
metadata:
  version: "1.0.0"
---

# Team Standards

Code Buddy ships with general best-practice standards in `references/`. A team can replace or extend them with its own standards repository.

## Finding the team's own standards (in order)

1. **Project config** — look in the working project for a `CODE_BUDDY.md` file, or a `code-buddy-standards:` line in `CLAUDE.md`. Either may contain:
   - `standards_repo: <owner>/<repo>` (optional `branch: <name>`, default `main`, and `path: <folder>`, default `standards`)
   - or `standards_path: <local folder>` relative to the project.
2. **User instruction** — the user names a repo or folder in the conversation ("use our standards from acme/eng-standards").
3. **Fallback** — the bundled files in `references/`.

When a team source is found, read the matching file names from it (e.g. `frappe.md`, `security.md`). Files missing from the team source fall back to the bundled version. The team version wins on any conflict. Mention once which source was used ("Using standards from acme/eng-standards, plus bundled defaults for servers").

If a GitHub tool isn't available to read a remote repo, say so and use the bundled defaults.

## Which files to load

| Task | Files |
|---|---|
| Code review / restructure / cleanup / conversion | `frappe.md`, `python.md`, `javascript.md`, `sql-database.md`, `review-checklist.md` |
| Security audit | `security.md`, `frappe.md` |
| Performance review | `frappe.md`, `sql-database.md`, `servers.md` |
| Upgrade check | `frappe.md`, `python.md`, `javascript.md` |
| Server review / architecture | `servers.md`, `environments.md`, `security.md` |
| PRs, branches, release | `git-workflow.md`, `environments.md` |

## Applying the standards

- Cite the rule when flagging an issue (e.g. "frappe.md › Database access").
- When code breaks a rule for a reason documented in a comment, report it as informational, not a defect.
- When the standards don't cover a question, answer from general best practice and say it isn't covered yet.
- Never invent a team rule. Distinguish clearly between "your team's standard" and "general best practice".

## Setting up a team standards repo

When the user asks how to use their own standards, explain: create a repo with a `standards/` folder using the same file names as `references/` (copy the bundled files as a starting point), then add a `CODE_BUDDY.md` to each project:

```
standards_repo: your-org/your-standards
branch: main
path: standards
```
