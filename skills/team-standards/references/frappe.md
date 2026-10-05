# Frappe / ERPNext Standards

Applies to Frappe/ERPNext v14, v15 and v16. Where a rule differs by version it is marked.

## Never modify core
- Never edit `frappe` or `erpnext` source. All changes live in a custom app.
- Extend with `doc_events`, `override_doctype_class` (v16: the override class MUST inherit the original class), `override_whitelisted_methods`, `doctype_js`, fixtures and patches.
- Prefer `doc_events` over `override_doctype_class` when only adding behavior; override only when you must change existing method logic.

## App layout
```
my_app/
  my_app/
    hooks.py              # wiring only, no logic
    patches.txt
    api/                  # whitelisted endpoints, one file per domain (api/sales.py)
    services/             # business logic, pure functions/classes, no request handling
    overrides/            # overridden doctype classes (overrides/sales_invoice.py)
    events/               # doc_events handlers, one file per doctype (events/sales_invoice.py)
    utils/                # small generic helpers
    tasks/                # scheduler jobs
    patches/vX_Y/         # data patches grouped by release
    <module>/doctype/...  # app-owned doctypes
    public/js/            # bundled client code (doctype_js files under public/js/<doctype>.js)
    fixtures/             # exported Custom Field / Property Setter / etc.
```
- `hooks.py` contains configuration only. Handlers are dotted paths into `events/`, `tasks/`, `overrides/`.
- Controllers (`<doctype>.py`) stay thin: validation + calls into `services/`. Target < 300 lines per file, < 50 lines per method.

## Database access
- Use `frappe.get_all` / `frappe.get_list` / `frappe.db.get_value` / `frappe.qb` before raw SQL.
- `frappe.get_list` applies user permissions; `frappe.get_all` does not. Use `get_list` in anything user-facing.
- Raw `frappe.db.sql` only for complex reporting queries; always parameterised (`%(name)s` with a dict, or `%s` with a tuple). Never f-strings / `.format()` / `%` string formatting into SQL.
- No `frappe.get_doc` inside loops when only a few fields are needed — fetch once with `get_all(fields=[...])`.
- Use `frappe.db.set_value` for single-field updates that should skip validation, `doc.db_set` on a loaded doc. Use `frappe.db.set_single_value` for Single doctypes (required from v15).
- Never call `frappe.db.commit()` in controllers, doc_events, or whitelisted methods. Frappe commits at end of request. (v16: unsupported inside document hooks.) Commits are allowed only in patches, long background jobs processing batches, and bench/console scripts.
- v16: `get_all`/`get_list` default sort is `creation`, not `modified` — always pass `order_by` explicitly when order matters.

## Whitelisted methods
- Every `@frappe.whitelist()` method validates input types and checks permission (`frappe.has_permission` or `doc.check_permission`) before reading/writing.
- `allow_guest=True` only with explicit approval from the tech lead and documented reason; rate-limit with `@frappe.rate_limit`.
- Specify `methods=["POST"]` for anything that writes.
- Do not use `ignore_permissions=True` to "make it work". If used, comment why.
- Return plain dicts/lists, not full docs, unless the caller needs the whole doc.

## Background work
- Anything > 2–3 s (bulk updates, PDF generation, external API calls, emails to many) goes through `frappe.enqueue` with an explicit `queue` ("short", "default", "long") and `job_id` (v15+: `job_name` is deprecated) plus `deduplicate=True` where repeat runs are possible.
- Scheduler jobs live in `tasks/` and must be idempotent.

## Hooks & events
- Do not put heavy work in `on_update`, `validate`, `before_save` — these run on every save. Move to `on_submit` or enqueue.
- Avoid saving the same doc inside its own hooks (recursion). Use `db_set` or `frappe.flags` guards.
- v16: `has_permission` hooks must explicitly `return True` to grant.

## Customizations
- No Custom Field / Property Setter / Client Script / Server Script / Print Format created only in the UI on production. Everything is exported as fixtures (filtered by `module` or a name list) or defined in app code, and committed.
- Server Scripts: allowed only for quick client-specific tweaks on staging; move to app code before production. (v15+: disabled by default.)
- Fixture filters must be specific — never export all Custom Fields of a site.

## Patches
- Every schema/data change that must run on existing sites gets a patch in `patches/vX_Y/` and an entry in `patches.txt` under the correct `[pre_model_sync]` / `[post_model_sync]` section.
- Patches are idempotent and safe to re-run; process large tables in batches with periodic `frappe.db.commit()`.

## Client-side (JS)
- Doctype JS goes in `public/js/` and is wired with `doctype_js` in hooks; no large Client Scripts in DB.
- Use `frappe.call` / `frm.call` with `freeze` for user-triggered actions; handle `r.message` missing.
- Don't rely on removed window globals (v15 removed `user`, `roles`, `get_today`, `show_alert`, etc. — use `frappe.session.user`, `frappe.user_roles`, `frappe.datetime.get_today()`, `frappe.show_alert`).
- Use `__("text")` for every user-visible string; `_("text")` in Python.

## Reports
- Script Reports: filters validated, query parameterised, heavy reports set `prepared_report` in the report doc.
- Never compute aggregates in Python loops when SQL/qb `GROUP BY` can do it.

## Tests
- New business logic in `services/` gets tests. v16: use `IntegrationTestCase` (DB) / `UnitTestCase` (pure logic); `FrappeTestCase` is deprecated.

## Naming
- Doctypes: Title Case singular (`Dispatch Plan`). Fieldnames: snake_case, no doctype prefix.
- Python modules/functions: snake_case. Classes: PascalCase.
- Custom fields on standard doctypes: prefix with app short code (e.g. `acme_delivery_route`) to avoid clashes.
