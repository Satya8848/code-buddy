# Setup Runbook Templates

## A. Bare-metal / VM (Ubuntu LTS, single node)
1. **Provision** VM, attach SSD, set hostname, timezone (`timedatectl set-timezone Asia/Kolkata`), create `frappe` user with sudo, SSH keys only.
2. **Harden**: `ufw allow 22,80,443/tcp && ufw enable`; `PasswordAuthentication no`, `PermitRootLogin no`; install `fail2ban`, `unattended-upgrades`; configure swap.
3. **Dependencies**: git, python3-dev + venv (version per Frappe target), Node (per target) + yarn, MariaDB server/client (10.6/10.11), Redis, nginx, supervisor, wkhtmltopdf (patched Qt build), cron.
4. **MariaDB**: run `mariadb-secure-installation`; set utf8mb4, buffer pool, log size, slow log in `/etc/mysql/mariadb.conf.d/99-frappe.cnf`; restart; verify with `SHOW VARIABLES`.
5. **Bench**: `pip install frappe-bench` (pipx preferred); `bench init --frappe-branch version-XX frappe-bench`; `bench get-app --branch version-XX erpnext`; get custom apps (pinned branch/tag).
6. **Site**: `bench new-site <domain> --db-root-password ... --admin-password ...`; `bench --site <domain> install-app erpnext <custom apps>`; `bench --site <domain> enable-scheduler`; set `developer_mode 0`.
7. **Production**: `sudo bench setup production frappe`; set `gunicorn_workers` in `common_site_config.json`; adjust worker counts in supervisor config; `bench setup nginx && sudo service nginx reload`.
8. **TLS**: DNS A record → `sudo bench setup lets-encrypt <domain>` (or LB cert).
9. **Backups**: cron / scheduler backups + offsite sync (Frappe S3 Backup Settings or rclone), retention policy; test restore on staging.
10. **Monitoring**: uptime check, node exporter/agent, disk/RAM/CPU alerts, log rotation.
11. **Verify**: login, create/submit sample transactions, run heavy report, check RQ workers (`bench doctor`), email send (staging muted), PDF print.
12. **Go-live checklist**: data migrated & reconciled, users/roles set, backups verified, monitoring alerting, rollback plan, client sign-off.

## B. Docker (frappe_docker)
1. Provision + harden as above; install Docker Engine + compose plugin.
2. Build a custom image with your apps (`apps.json` + the frappe_docker custom image build), tag with version — never `latest` in production.
3. Compose services: backend, frontend (nginx), websocket, queue-short, queue-long (and queue-default if used), scheduler, redis-cache, redis-queue, db (or external managed DB); volumes for sites and logs; healthchecks; restart policies; resource limits.
4. Reverse proxy with TLS (Traefik or nginx + certbot).
5. `create-site` job, install apps, set config; backups via cron container to S3.
6. Deploy updates: build new tag → pull → `bench --site all migrate` in a backend container → recreate services. Keep previous tag for rollback.

## C. Staging
Same versions, smaller size, `mute_emails: 1`, sandbox integration keys, masked data refreshed from production backups on a schedule.
