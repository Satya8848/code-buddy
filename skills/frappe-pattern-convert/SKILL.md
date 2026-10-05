---
name: frappe-pattern-convert
description: >
  This skill should be used when the user asks to "convert raw SQL to query builder",
  "use frappe.qb", "replace get_doc loop", "remove frappe.db.sql", "modernize this code",
  "convert callbacks to async", or asks to rewrite Frappe/ERPNext code into the team's
  preferred patterns while keeping behavior identical.
metadata:
  version: "0.1.0"
---

# Pattern Conversion

## Procedure
1. Load `team-standards` → `frappe.md`, `sql-database.md`, `python.md`, `javascript.md`; load `references/conversions.md`.
2. Find candidate code (or use what the user points to). Skip conversions that hurt readability — complex reporting SQL with many CTEs/window functions may stay raw SQL if fully parameterised.
3. Convert one unit at a time. For each:
   - Keep the same inputs, outputs (types, ordering, dict vs tuple, `as_dict` shape) and permission semantics (`get_list` vs `get_all`).
   - Keep explicit `order_by` (v16 default sort changed).
   - Show before/after and note any semantic differences.
4. Suggest a quick equivalence check: run old and new on the same filters in `bench console` and compare results.

## Output
For each conversion: location, before, after, equivalence notes. End with a summary count and anything intentionally left unconverted with reason.
