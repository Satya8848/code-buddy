# Sizing Guide (starting points — validate with load testing)

Concurrency = users actively working at the same time (typically 20–40% of named users).

| Tier | Peak concurrent | Typical named users | App server | DB | Notes |
|---|---|---|---|---|---|
| XS | ≤ 10 | ≤ 25 | 2 vCPU / 4 GB, 60 GB SSD (all-in-one) | same host | dev/staging or tiny client |
| S | 10–30 | 25–100 | 4 vCPU / 8 GB, 100 GB SSD (all-in-one) | same host | most SME clients |
| M | 30–75 | 100–250 | 8 vCPU / 16 GB | same host, or 4 vCPU / 16 GB separate | split DB when reports are heavy |
| L | 75–200 | 250–600 | 2 × 8 vCPU / 16 GB behind LB | 8 vCPU / 32–64 GB dedicated (+ read replica) | shared file storage (NFS/S3) required |
| XL | 200+ | 600+ | 3+ app nodes or Kubernetes | 16 vCPU / 64–128 GB, replica, managed DB option | load test before go-live |

## Derived settings
- Gunicorn workers: `min(2 × vCPU + 1, (RAM_for_app_GB × 1024) / 200)`.
- RQ workers: S: short 1, default 1, long 1; M: 1/2/2; L+: scale by queue backlog.
- `innodb_buffer_pool_size`: all-in-one ≈ 25–40% RAM; dedicated DB ≈ 60–70% RAM.
- Disk: DB size × 3 (growth + backups staging) minimum; use SSD/NVMe; separate volume for backups or push offsite.
- Redis cache `maxmemory`: 256 MB (S), 512 MB–1 GB (M+).

## Adjusters
- Manufacturing with deep BOMs / MRP runs, large stock reposting, payroll for 1000+ employees → +1 tier on CPU.
- Many attachments → S3-compatible storage, not local disk.
- Heavy integrations / webhooks → extra `default`/`long` workers.
- POS with offline sync / many stores → watch socketio and API worker capacity.
