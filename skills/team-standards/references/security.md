# Security Standards

- No credentials, API keys, tokens or passwords in Git. Use `site_config.json` (excluded from Git), env vars, or a vault. Rotate anything that was ever committed.
- Whitelisted methods: permission check inside every method; `allow_guest` only with approval + rate limit; writes are POST-only.
- No `ignore_permissions=True` / `ignore_user_permissions` without a comment explaining why it is safe.
- Escape all user content rendered in HTML (Jinja autoescape on; `frappe.utils.escape_html` in JS).
- File uploads: restrict allowed types; private files for anything client-confidential.
- Production: HTTPS only (Let's Encrypt via `bench setup lets-encrypt` or load balancer cert), HSTS on, admin password not default, `developer_mode` = 0, `allow_tests` = false, Server Scripts disabled unless needed.
- SSH: key-based only, password auth and root login disabled, fail2ban on, firewall allows only 22/80/443 (22 restricted to office/VPN IPs where possible).
- MariaDB and Redis bind to localhost / private network only; never exposed publicly.
- Separate credentials per environment; staging never uses production API keys for payment gateways, SMS, email or e-invoicing.
- Staging copies of production data: mask PII (phones, emails, bank details) before sharing access with wider team.
