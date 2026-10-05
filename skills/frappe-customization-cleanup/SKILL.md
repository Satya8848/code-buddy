---
name: frappe-customization-cleanup
description: >
  This skill should be used when the user asks to "clean up customizations", "move custom
  fields to fixtures", "export customizations", "convert client scripts to app code",
  "move server scripts into the app", "too many property setters", or wants UI-made
  Frappe/ERPNext customizations brought under version control.
metadata:
  version: "0.1.0"
---

# Customization Cleanup

## Procedure
1. Load `team-standards` → `frappe.md` (Customizations section).
2. **Inventory** what exists on the site (ask the user to run these in `bench --site <site> console` or a Query Report, or run them if a bench is reachable):
   ```python
   frappe.get_all("Custom Field", fields=["name","dt","fieldname","module"], order_by="dt")
   frappe.get_all("Property Setter", fields=["name","doc_type","field_name","property","module"], order_by="doc_type")
   frappe.get_all("Client Script", fields=["name","dt","enabled","module"])
   frappe.get_all("Server Script", fields=["name","script_type","reference_doctype","disabled","module"])
   frappe.get_all("Print Format", filters={"standard": "No"}, fields=["name","doc_type"])
   frappe.get_all("Report", filters={"is_standard": "No"}, fields=["name","report_type","ref_doctype"])
   frappe.get_all("Workflow", fields=["name","document_type","is_active"])
   ```
3. **Classify** each item: keep in app (owned by our app), belongs to another app, obsolete/disabled (candidate for deletion after confirmation), duplicated.
4. **Plan the target**:
   - Custom Fields / Property Setters → set `module` to the app's module, export via `fixtures` in `hooks.py` with specific filters:
     ```python
     fixtures = [
         {"dt": "Custom Field", "filters": [["module", "=", "My App Module"]]},
         {"dt": "Property Setter", "filters": [["module", "=", "My App Module"]]},
     ]
     ```
     For large sets, prefer defining fields in code via `create_custom_fields` in an `after_install`/patch.
   - Client Scripts → `public/js/<doctype>.js` + `doctype_js` in hooks; then disable the Client Script.
   - Server Scripts → DocEvent type → `events/<doctype>.py` + `doc_events`; API type → `api/` whitelisted method (keep the same method name or update callers); Scheduler type → `tasks/` + `scheduler_events`; Permission Query → `permission_query_conditions` hook.
   - Custom Print Formats / non-standard Reports → mark standard in developer mode on dev so they export to the app.
5. Generate the code and hooks entries. Translate Server Script restricted-python into normal Python (`frappe.` APIs are the same; replace `doc` context var with the handler's `doc` argument).
6. **Rollout**: commit → deploy to staging → `bench migrate` → verify behavior → disable (not delete) the old UI records → after a stable period, delete them.

## Output
- Inventory table with classification.
- Generated files and `hooks.py` diff.
- Rollout checklist with which records to disable.
- Warnings: name collisions, fields whose `insert_after` targets don't exist, scripts with side effects that will now run twice if both old and new stay enabled.
