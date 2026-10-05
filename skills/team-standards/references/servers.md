# Server Standards (Frappe/ERPNext hosting)

## OS & base
- Ubuntu LTS (22.04 / 24.04), kept patched (`unattended-upgrades` for security updates).
- Dedicated non-root `frappe` user owns the bench. No bench as root.
- Timezone set to client's business timezone (e.g. Asia/Kolkata), NTP on.
- Swap configured (= RAM up to 4 GB, 4 GB beyond) as safety net; `vm.swappiness=10`.

## Stack
- Bare-metal/VM: bench with nginx + supervisor (or systemd), Redis (cache, queue), MariaDB 10.6/10.11 LTS.
- Containers: official `frappe_docker` images, docker compose for single host, Kubernetes (Helm chart) for multi-node/HA.
- Production setup via `bench setup production frappe` (bare-metal), never `bench start`.

## Gunicorn / workers
- Gunicorn workers ≈ (2 × CPU cores) + 1, capped by RAM (~150–250 MB per worker). Set in `common_site_config.json` (`gunicorn_workers`).
- Background workers: at least 1 `short`, 1 `default`, 1 `long`; scale `default`/`long` for heavy queue loads. Monitor RQ queue length.
- `http_timeout` increased only for specific long reports; prefer prepared reports / background jobs.

## MariaDB tuning (starting point)
- `innodb_buffer_pool_size` = 50–70% of RAM on a DB-only server, ~25–40% on an all-in-one server.
- `innodb_log_file_size` 512M–1G; `innodb_flush_log_at_trx_commit=1` (durability) for production.
- `max_connections` sized to workers + headroom; slow query log enabled (`long_query_time=2`).
- Character set `utf8mb4`, collation `utf8mb4_unicode_ci`.

## Redis
- Separate redis-cache and redis-queue (bench default). Cache has `maxmemory` + `allkeys-lru`. Bound to localhost/private IP.

## nginx
- gzip/brotli on, static assets cached, `client_max_body_size` set to allowed upload size, HTTP/2, TLS 1.2+.

## Backups & DR
- `bench backup --with-files` scheduled; offsite copy (S3-compatible) with retention (e.g., 7 daily, 4 weekly, 3 monthly).
- Restore tested on staging at least quarterly.

## Monitoring & logs
- Uptime check on login page, disk/RAM/CPU alerts (>80%), RQ queue backlog alert, MariaDB slow log review weekly.
- Log rotation configured for bench logs, nginx, supervisor.

## Access
- See security.md. Every engineer has an individual SSH key; remove on offboarding.
