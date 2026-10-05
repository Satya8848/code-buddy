---
name: frappe-performance-review
description: >
  This skill should be used when the user says "this is slow", "performance review",
  "optimize this report", "why is saving Sales Invoice slow", "speed up the query",
  "too many queries", "site is slow", or asks to tune Frappe/ERPNext code, reports, hooks,
  background jobs or database queries.
metadata:
  version: "0.1.0"
---

# Frappe Performance Review

## Procedure
1. Load `team-standards` → `frappe.md`, `sql-database.md`, `servers.md`.
2. Identify the slow path: a doctype save/submit, a report, a page/API, a scheduler job, or the whole site. Ask for any evidence available: slow query log, `bench --site x doctor`, RQ queue lengths, Error Log, timings, record counts.
3. Trace the path in code: `hooks.py` → `doc_events` / overrides → controller methods → services → queries. List every DB call on the path.
4. Look for:
   - **N+1**: `get_doc`/`get_value`/`db.exists` inside loops → batch with `get_all(filters={"name": ["in", names]})` or one qb join.
   - **Heavy hooks**: external API calls, PDF/email, recalculating many docs in `validate`/`on_update`/`on_submit` → `frappe.enqueue(..., queue="long", job_id=..., deduplicate=True, enqueue_after_commit=True)`.
   - **Full-doc loads** where `get_value(..., [fields])` suffices.
   - **Unbounded queries**: `get_all` without `limit`, `SELECT *`, missing filters on large tables (GL Entry, Stock Ledger Entry, Sales Invoice Item).
   - **Missing indexes** on fields used in filters/joins/order — propose `search_index` on the field or `frappe.db.add_index` in a patch.
   - **Non-sargable WHERE** (`DATE(col)`, `IFNULL(col,'')=`, leading `%` LIKE).
   - **Python-side aggregation** that SQL `GROUP BY`/`SUM` should do.
   - **Report design**: heavy Script Reports not marked `prepared_report`; repeated subqueries; reports not using `posting_date` ranges.
   - **Caching**: repeated reads of settings/single doctypes → `frappe.get_cached_value` / `frappe.get_cached_doc` / `frappe.cache` with explicit invalidation.
   - **Client side**: `frappe.call` in loops, `refresh` triggers firing server calls on every render.
   - **Infra** (if site-wide): gunicorn workers vs CPU/RAM, RQ workers per queue, MariaDB buffer pool, Redis memory — defer details to `server-review`.
5. For each recommendation, estimate impact (High/Med/Low) and effort (S/M/L). Provide `EXPLAIN` statements to run for suspect queries.

## Output
- **Diagnosis** — the likely bottleneck(s) in 2–4 sentences.
- **Recommendations table** — #, issue, location, fix, impact, effort.
- **Code changes** — before/after snippets for top items.
- **Verify with** — commands/queries to measure before & after (`EXPLAIN`, `frappe.utils.now` timing, `bench --site x execute`, Recorder doctype in Frappe for request profiling).
