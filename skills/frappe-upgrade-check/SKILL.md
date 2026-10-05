---
name: frappe-upgrade-check
description: >
  This skill should be used when the user asks "is this app ready for v15", "upgrade to v16",
  "migrate from version 14", "check deprecated APIs", "what breaks on upgrade", or plans a
  Frappe/ERPNext version upgrade for a custom app or site. Scans custom code for removed and
  deprecated APIs and produces an upgrade plan.
metadata:
  version: "0.1.0"
---

# Frappe Upgrade Readiness Check

## Procedure
1. Determine current and target versions (v14 → v15, v15 → v16, or v14 → v16 which must pass through both lists).
2. Load `references/deprecations.md` and `team-standards` → `frappe.md`.
3. Scan every custom app on the bench (Python, JS, Jinja, JSON fixtures, `hooks.py`, `pyproject.toml`/`setup.py`, `package.json`) for each item on the relevant list(s). Use grep patterns from the reference file, then read the hit to confirm it is a real use.
4. Check runtime requirements: Python and Node versions on the server, MariaDB version, removed core modules the app depends on (e.g. Event Streaming in v15; Energy Points, Newsletter, Blog, Backup Integrations in v16 now separate apps).
5. Check behavioral changes that don't show up as grep hits but affect results (v16 default sort by `creation`, `has_permission` must return True, `get_doc(..., field=value)` no longer updates, Time field defaults, Country ISO codes).
6. Remind the user to cross-check the official wiki pages (`github.com/frappe/frappe/wiki/Migrating-to-version-15` / `-16`) and ERPNext release notes — the bundled list is a snapshot.

## Output
1. **Readiness verdict** — Ready / Ready with fixes / Not ready, with counts per severity.
2. **Required changes** — table: file:line, current code, required change, version that breaks it.
3. **Behavioral risks** — things to test on staging.
4. **Environment changes** — Python/Node/DB/bench/app dependencies.
5. **Upgrade runbook** — backup → staging clone → switch branches (`bench switch-to-branch version-XX frappe erpnext --upgrade`) → update custom app branch → `bench setup requirements` → `bench migrate` → `bench build` → test plan → production window → rollback steps.
