# Code Review Checklist (quick reference)

**Blocker** — must fix before merge
- SQL built with string formatting from input
- Whitelisted method without permission check / guest method without approval
- Secrets in code
- `frappe.db.commit()` in controller/doc_event/whitelisted method
- Core (frappe/erpnext) files modified
- Production-affecting customization done only in UI (not in fixtures/code)
- Missing patch for data/schema change on existing sites

**Major** — fix before merge unless tech lead waives
- `get_doc` / `get_value` inside loops over many records (N+1)
- Heavy logic in `validate`/`on_update` that should be enqueued
- `ignore_permissions=True` without justification
- No `order_by` where ordering matters (v16 default changed)
- Deprecated API for the target version
- Controller > 300 lines / method > 50 lines with mixed concerns

**Minor** — fix now or log as tech debt
- Naming, missing `_()`/`__()` translations, missing type hints, dead code, missing docstring on public API

**Nit** — optional style comments
