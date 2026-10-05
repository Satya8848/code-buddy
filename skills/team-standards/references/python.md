# Python Standards

- Formatter: `ruff format` (or black, line length 110). Linter: `ruff`. Frappe apps use the repo's `pyproject.toml` ruff config; run `pre-commit` before pushing.
- Imports: stdlib, third-party, frappe, local — separated by blank lines. No wildcard imports.
- Functions do one thing; target < 50 lines. Prefer early returns over nested `if`.
- Type hints on all new public functions (Frappe v15+ uses them for whitelisted-method argument validation).
- No bare `except:`; catch specific exceptions. Never swallow errors silently — log with `frappe.log_error(title=..., message=frappe.get_traceback())` in Frappe code, `logging` elsewhere.
- No mutable default arguments.
- Constants in UPPER_SNAKE_CASE at module top. No magic numbers/strings in logic.
- Secrets never in code. Use `site_config.json` / `frappe.conf`, env vars, or a secrets manager.
- Use f-strings for messages, never for SQL.
- Docstrings for public functions explaining *why*, not restating *what*.
- Dead code and commented-out blocks are deleted, not left in.
- Python version: Frappe v14 → 3.10; v15 → 3.10–3.12 (3.11 recommended); v16 → 3.14+.
