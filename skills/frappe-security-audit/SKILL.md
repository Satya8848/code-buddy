---
name: frappe-security-audit
description: >
  This skill should be used when the user asks for a "security audit", "security review",
  "check permissions", "find SQL injection", "audit whitelisted methods", "guest API review",
  or "is this API safe" for a Frappe/ERPNext app or site. Produces a focused security report
  on APIs, permissions, SQL, secrets, file handling and site config.
metadata:
  version: "0.1.0"
---

# Frappe Security Audit

## Procedure
1. Load `team-standards` → `security.md`, `frappe.md`.
2. Inventory the attack surface:
   - Grep for `@frappe.whitelist` (note `allow_guest=True`, `methods=`), `frappe.db.sql`, `ignore_permissions`, `ignore_user_permissions`, `frappe.set_user`, `frappe.flags.ignore_permissions`, `run_as`, `eval(`, `exec(`, `safe_eval`, `subprocess`, `os.system`, `requests.` (SSRF), `frappe.get_doc(` with user-supplied doctype, `frappe.local.form_dict`, `render_template` with user input, `| safe` / `{{ ... | safe }}` in Jinja, `innerHTML`, `$(...).html(` in JS.
   - Read `hooks.py` for `override_whitelisted_methods`, `has_permission`, `permission_query_conditions`, `website_route_rules`, guest-accessible `www/` pages.
   - Search for secrets: API keys, tokens, passwords, private keys in code, fixtures, JSON, `.env` committed.
3. For each whitelisted method build a row: method, guest?, HTTP methods, permission check present?, input validated?, writes data?, risk.
4. Check:
   - SQL built with f-string / `.format` / `%` / concatenation using any input → Blocker.
   - User-controlled `doctype`/`fieldname` passed to `get_doc`, `get_all`, `db.get_value` without allow-list → Blocker.
   - Guest methods without rate limit or that return internal data → Blocker/Major.
   - Missing permission check before read/write of business data → Major (Blocker if it writes).
   - Writes reachable via GET → Major.
   - `ignore_permissions=True` without justification → Major.
   - v16: `has_permission` hooks returning `None` instead of `True` → behavior change, Major.
   - File uploads: public files for confidential data, no type restriction → Major.
5. If site/server config is available (`site_config.json`, `common_site_config.json`), check `developer_mode`, `allow_tests`, `server_script_enabled`, `mute_emails` on staging, exposed DB/Redis host, default admin password usage. Never print secret values — mask them (`sk_live_****`).

## Output
1. **Risk summary** — overall rating (Critical/High/Medium/Low) and top 3 risks.
2. **API inventory table** (from step 3).
3. **Findings** — severity, location, exploit scenario in one sentence, fix with code.
4. **Config findings** (if checked).
5. **Hardening checklist** — remaining items from `security.md` not verified.

Report only issues verified in the code. Mark anything that needs runtime confirmation as "Needs verification".
