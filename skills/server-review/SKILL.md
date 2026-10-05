---
name: server-review
description: >
  This skill should be used when the user asks to "review this server", "check our production
  setup", "audit staging", "review nginx/supervisor config", "review docker compose for
  ERPNext", "review the CI/CD pipeline", "is our server configured right", "why is the server
  slow", or shares bench configs, site_config, nginx, supervisor, MariaDB, Redis, Docker,
  Kubernetes or CI files for Frappe/ERPNext hosting.
metadata:
  version: "0.1.0"
---

# Server Review (Frappe/ERPNext hosting)

## Inputs
Config files, command outputs, or SSH access through the user's computer if available. When nothing is provided, give the user the collection script below to run (read-only) and paste back.

```bash
# Run as the frappe user from the bench directory. Read-only. Mask secrets before sharing.
lsb_release -d; nproc; free -h; df -h; uptime
python3 --version; node --version; mysql --version; redis-server --version
bench version
cat sites/common_site_config.json | sed -E 's/("(db_password|encryption_key|admin_password|.*secret.*|.*key.*)": *")[^"]+/\1****/I'
ls sites/
cat config/supervisor.conf 2>/dev/null | head -100
sudo nginx -T 2>/dev/null | head -200
sudo mysql -e "SHOW VARIABLES WHERE Variable_name IN ('innodb_buffer_pool_size','innodb_log_file_size','max_connections','character_set_server','collation_server','slow_query_log','long_query_time','innodb_flush_log_at_trx_commit');"
redis-cli -p 13000 INFO memory | grep -E 'used_memory_human|maxmemory_human|maxmemory_policy'
sudo ufw status; sudo ss -tlnp
sudo sshd -T | grep -E 'passwordauthentication|permitrootlogin'
crontab -l; ls -lh sites/*/private/backups | tail -5
```

## Procedure
1. Load `team-standards` → `servers.md`, `environments.md`, `security.md`. Ask which environment this is (production / staging / dev) — rules differ.
2. Check each area and mark ✅ / ⚠️ / ❌:
   - **Versions** vs Frappe version requirements (v14: Py 3.10, Node 16; v15: Py 3.10–3.12, Node 18+; v16: Py 3.14+, Node 24+), and staging = production parity.
   - **Process management**: production via supervisor/systemd, not `bench start`; worker counts (gunicorn ≈ 2×cores+1 limited by RAM; short/default/long RQ workers).
   - **MariaDB**: buffer pool vs RAM, log file size, utf8mb4, slow log, bind address.
   - **Redis**: cache maxmemory/policy, bind address.
   - **nginx**: TLS, HTTP/2, gzip, `client_max_body_size`, static caching, security headers.
   - **Security**: open ports (only 22/80/443 public), SSH key-only + no root login, fail2ban, `developer_mode` 0, `allow_tests` off, server scripts setting, `mute_emails` on staging, separate credentials per env.
   - **Backups**: frequency, offsite copy, retention, last successful backup date, restore test.
   - **Monitoring/logs**: uptime, resource alerts, log rotation, Error Log volume.
   - **Capacity**: CPU/RAM/disk headroom (>80% = ⚠️, >90% = ❌), swap.
   - **Containers** (if Docker/K8s): pinned image tags (no `latest`), volumes for sites/logs, healthchecks, resource limits, secrets via env/secret store, separate worker/scheduler/websocket services.
   - **CI/CD**: lint/tests run, staging deploy automated, production deploy gated by approval, backup + migrate + build + restart steps, rollback.
3. For every ❌/⚠️ provide the exact fix (config snippet or command) and whether it needs downtime.

## Output
- **Scorecard** table: area, status, key finding.
- **Critical fixes** (do this week), **Improvements** (this month), **Nice to have**.
- Commands/config snippets per fix, flagged with "needs restart" / "needs maintenance window".
- Never print secrets; mask them if they appear in input.
