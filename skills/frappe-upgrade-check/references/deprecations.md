# Deprecations & Breaking Changes (snapshot from official Frappe migration wikis)

## v14 → v15
| Change | Grep for | Fix |
|---|---|---|
| Node ≥ 18; `setup.py` removed, use `pyproject.toml` | `setup.py` | migrate to pyproject (flit) |
| `db.sql(as_utf8=, formatted=)` removed | `as_utf8`, `formatted=` | drop args |
| `db.set_value(for_update=)` removed | `for_update=` in set_value | drop arg |
| `db.set`, `db.touch`, `db.clear_table`, `db.update`, `db.set_temp`, `db.get_temp` removed | `db\.set\(`, `db\.touch`, `db\.clear_table`, `db\.update\(`, `set_temp`, `get_temp` | `db.set_value`, `db.delete`/`truncate`, cache |
| Single doctype values | `db.set_value("<Single>", None` / same name | `db.set_single_value` |
| `get_year_ending`, `get_timespan_date_range` return `date` objects | these names | stop treating as strings |
| `convert_utc_to_user_timezone` → `convert_utc_to_system_timezone`; `get_time_zone` → `get_system_timezone` | old names | rename |
| Vue 2 → Vue 3, Vuex 4, Vue Router 4 | `new Vue(`, `Vue.component` | port to Vue 3 |
| Window globals removed: `get_today`, `date`, `dateutil`, `show_alert`, `validated`, `user`, `user_fullname`, `user_email`, `user_defaults`, `roles`, `sys_defaults`, `frappe.query_report_filters_by_name` | bare uses in JS | `frappe.datetime.get_today()`, `frappe.show_alert`, `frappe.validated`, `frappe.session.user`, `frappe.user_roles`, `frappe.boot.sysdefaults` |
| Client Scripts lose local scope (`this`) | `this.` in client scripts | use `frm` |
| SocketIO namespaced by site | custom `io(` usage | `io(\`${url}/${frappe.local.site}\`)` |
| `frappe.enqueue(job_name=)` deprecated | `job_name=` | `job_id=` |
| `frappe.new_doc(parent_doc, parentfield, as_dict)` keyword-only | positional extra args | pass as kwargs |
| `frappe.db.add_before_commit`, `frappe.local.rollback_observers` removed | names | `frappe.db.before_commit.add` / after-commit hooks |
| `override_whitelisted_methods`: last override wins | multiple apps overriding same method | check app order |
| `frappe.compare` moved | `frappe.compare` | `from frappe.utils import compare` |
| `search_link`/`search_widget` response key `results` → `message` | `.results` on these | `r.message` |
| Server Scripts disabled by default | server scripts in use | `bench set-config -g server_script_enabled 1` or move to app |
| "Desk User" role replaces "All" for desk-only perms | perms on "All" | review roles |
| `currentsite.txt` no longer default | scripts relying on it | `bench use` / `FRAPPE_SITE` |
| Error Snapshot removed → Error Log | `Error Snapshot` | `Error Log` |
| Event doctype: "Cancelled" moved to `status` | `event_type == "Cancelled"` | use `status` |
| `get_installed_apps(sort=, frappe_last=)` removed | args | drop |
| Event Streaming is a separate app | `event_streaming` imports | install app |
| Removed deps: `googlemaps`, `urllib3`, `gitdb`, `pyasn1`, `pypng`, `google-auth-httplib2`, `schedule`, `pycryptodome` | imports | add to your app's dependencies |

## v15 → v16
| Change | Grep for | Fix |
|---|---|---|
| Python ≥ 3.14, Node ≥ 24 | server versions | upgrade runtimes |
| `frappe.db.commit()` not supported in document hooks | `db.commit()` in controllers/doc_events | remove; enqueue if needed |
| `get_all`/`get_list` default order `creation` (was `modified`) | `get_all(` / `get_list(` without `order_by` | add explicit `order_by` |
| `db.get_value` on Single doctypes returns typed values | string comparisons on single values | compare typed |
| Translation APIs removed: `get_translated_dict` hook, `frappe.get_lang_dict`, `frappe.translate.get_dict`, `get_lang_js`, `get_dict_from_hooks` | names | use `_()`/`__()` only |
| `has_permission` hook must explicitly return `True` | `has_permission` in hooks | return True/False explicitly |
| `frappe.permission.has_permission(raise_exception=)` removed | `raise_exception=` | `print_logs` |
| `frappe.flags.in_test` deprecated | `flags.in_test` | `frappe.in_test` |
| `override_doctype_class` class must extend the original | override classes | inherit original |
| Transaction Log doctype removed; GeoIP removed | names | remove usage |
| Energy Points, Newsletter, Backup Integrations, Blog → separate apps | imports/doctype links | install apps or remove |
| `/api/method/logout`, `web_logout`, `upload_file` POST-only | GET calls to these | use POST |
| `frappe.sendmail(now=True)` no longer commits | reliance on implicit commit | adjust |
| `/app` → `/desk`; `/apps` deprecated | hardcoded `/app/` URLs | use `frappe.utils.get_url_to_form` / router |
| Time field no default unless "Now" in options | Time fields relying on default | set options |
| Country field requires ISO alpha-2 | country data | clean data |
| `get_doc(doctype, name, field=value)` no longer updates values | that pattern | set attributes then save |
| `meta.get_valid_columns()` excludes virtual fields | usage | adjust |
| Report/Page/Dashboard Chart JS wrapped in IIFE | globals defined in report JS | attach to `frappe.query_reports[...]` explicitly |
| HTML sanitisation bleach → nh3 (strips forbidden tags) | custom HTML fields | test output |
| `FrappeTestCase` deprecated → `IntegrationTestCase`/`UnitTestCase`; `test_dependencies` → `EXTRA_TEST_RECORD_DEPENDENCIES`; `test_ignore` → `IGNORE_TEST_RECORD_DEPENDENCIES`; `setUpClass` overrides must call super | names | rename |
| `@site_cache` no unhashable args; site config cached up to 1 min | usage | adjust |
| List view sidebar removed; Awesome Bar moved | custom list sidebar code | rework UI customizations |
