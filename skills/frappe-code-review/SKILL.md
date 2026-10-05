---
name: frappe-code-review
description: >
  This skill should be used when the user asks to "review this code", "review this PR",
  "review my Frappe app", "check this doctype", "code review for ERPNext", "is this code okay",
  or shares Frappe/ERPNext Python, JS, hooks.py, report or doctype files for feedback. Performs a
  full review against the team standards and Frappe best practices (v14/v15/v16) and returns
  severity-ranked findings with fixes.
metadata:
  version: "0.1.0"
---

# Frappe Code Review

## Inputs
Accept any of: a GitHub PR link or number, a repo + path, pasted code, uploaded files, or a folder on the user's computer. If the Frappe version is unknown, detect it (`frappe` version in `pyproject.toml`/`requirements`, `required_apps`, or the branch name such as `version-15`) or ask once.

## Procedure
1. Load the `team-standards` skill files for code review.
2. Gather scope. For a PR, read the changed files plus `hooks.py`, `patches.txt` and any controller the change touches. For a whole app, start with `hooks.py`, then `api/`, controllers, `patches/`, reports, `public/js`.
3. Review in this order, noting file and line for each finding:
   - **Correctness** — logic bugs, wrong hook event, missing `super()` in overrides, doc saved inside its own hook, wrong `doc_events` path, child-table handling.
   - **Security** — apply the checks in `frappe-security-audit` (summary level only; recommend the full audit when several issues appear).
   - **Data integrity** — `frappe.db.commit()` in request cycle, missing patch for schema/data change, non-idempotent patch, `db_set`/`set_value` skipping needed validation.
   - **Performance** — N+1 queries, heavy `validate`/`on_update`, unbounded `get_all`, missing `order_by`/limits, Python aggregation.
   - **Version compatibility** — deprecated or removed APIs for the target version (see `frappe-upgrade-check/references/deprecations.md`).
   - **Structure** — fat controllers, logic in `hooks.py`, UI-only customizations, Server Scripts in production.
   - **Style** — naming, translations `_()`/`__()`, type hints, dead code.
4. Classify each finding: **Blocker**, **Major**, **Minor**, **Nit** (definitions in `review-checklist.md`).
5. Verify each finding before reporting: re-read the surrounding code and drop anything that is guarded elsewhere or not actually reachable. Prefer fewer, certain findings over many speculative ones.

## Output
```
## Review summary
<1–3 sentences: overall quality, merge recommendation: Approve / Approve with changes / Request changes>

## Findings
### 🔴 Blocker (n)
1. **<title>** — `path/file.py:L42`
   Why: <impact, cite standard>
   Fix:
   ```python
   <corrected code>
   ```
### 🟠 Major (n) ...
### 🟡 Minor (n) ...
### ⚪ Nit (n) ...

## Good practices noticed
- <1–3 items worth keeping>

## Suggested follow-ups
- e.g. run frappe-security-audit / frappe-app-restructure / add patch
```
- If posting to a GitHub PR, post one review with inline comments for Blocker/Major and a summary comment, only when the user asks to post.
- Keep fixes minimal and behavior-preserving. Do not rewrite the whole file in a review.
