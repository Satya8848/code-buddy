# SQL & Database Standards (MariaDB / PostgreSQL)

- All queries parameterised. No string concatenation of user input — ever.
- Select only needed columns; no `SELECT *` in application code.
- Every column used in a frequent `WHERE`, `JOIN` or `ORDER BY` on a large table has an index. Add via doctype field "Index" checkbox or `frappe.db.add_index` in a patch.
- Avoid functions on indexed columns in WHERE (`DATE(posting_date) = ...` → range comparison).
- Paginate large result sets (`limit_start`, `limit_page_length` / `LIMIT ... OFFSET`).
- Long reports: run against a read replica if one exists (`frappe.db.sql(..., ...)` within `@frappe.read_only()` decorated methods).
- Check slow queries with `EXPLAIN` before merging any new report query.
- No schema changes by hand on production. Schema comes from doctype JSON + `bench migrate`.
- MariaDB config must follow the Frappe-recommended settings (utf8mb4, `innodb_file_per_table`, buffer pool sized to RAM — see servers.md).
