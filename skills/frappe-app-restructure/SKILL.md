---
name: frappe-app-restructure
description: >
  This skill should be used when the user asks to "restructure this app", "refactor this
  controller", "clean up hooks.py", "split this file", "organize my Frappe app", "this
  doctype file is too big", or "move logic out of the controller". Reorganizes Frappe/ERPNext
  custom app code into a clean standard layout without changing behavior.
metadata:
  version: "0.1.0"
---

# Frappe App Restructure

Goal: same behavior, cleaner structure. Never mix restructuring with feature changes or bug fixes — report bugs found, fix them in a separate step only if the user agrees.

## Procedure
1. Load `team-standards` → `frappe.md`, `python.md`; load `references/target-layout.md`.
2. **Map the current app**: list modules, doctypes, `hooks.py` entries (doc_events, overrides, scheduler_events, fixtures, doctype_js), whitelisted methods and who calls them (JS `frappe.call` paths, external integrations, mobile apps), patches.
3. **Identify problems**: logic inside `hooks.py`; controllers > 300 lines or methods > 50 lines; mixed concerns (API + business logic + DB in one function); duplicated logic across doctypes; generic `utils.py` dumping ground; doc_events scattered in random files; JS in Client Script records instead of `public/js`.
4. **Propose a move plan** before editing — a table of: current location → new location → reason. Highlight anything that changes a **public path** (whitelisted method dotted path, `doc_events` path, scheduler path, JS `frappe.call` method string). Get the user's confirmation.
5. **Preserve public paths**: when a whitelisted method moves, keep a thin wrapper at the old path that calls the new one (mark `# Deprecated: kept for backward compatibility, remove after <date>`), or update every caller in the same change (JS, mobile apps, integrations) — ask which.
6. **Execute in small steps**, one concern per commit (`refactor: move sales invoice events to events/sales_invoice.py`). After each step:
   - Update `hooks.py` dotted paths.
   - Grep for old import paths and fix them.
   - Run `python -m compileall` / `ruff` on changed files; if a bench is available run `bench --site <site> migrate` and existing tests.
7. **Extract services**: move business logic from controllers into `services/<domain>.py` functions that take plain arguments or a doc, with the controller calling them. Keep controller hook methods (`validate`, `on_submit`, ...) as short orchestrators.
8. Produce a final report.

## Output
- Before/after tree.
- Move table with public-path impact.
- List of commits (or a single diff if the user prefers).
- Bugs/smells found but intentionally not changed.
- Test checklist: forms to open, documents to save/submit/cancel, reports to run, scheduler jobs to trigger, APIs to call.
