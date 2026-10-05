---
name: server-architecture
description: >
  This skill should be used when the user describes a client's requirements and asks to
  "suggest a server architecture", "design the server setup", "what server size do we need",
  "how many users can this handle", "plan hosting for ERPNext", "set up production and staging
  for a new client", "AWS vs on-prem for Frappe", or "best way to set up the server". Produces a
  sized architecture, performance tuning, environment plan and step-by-step setup runbook.
metadata:
  version: "0.1.0"
---

# Server Architecture & Setup Plan

## Step 1 — Gather requirements
Extract what the user already gave; ask (in one AskUserQuestion round, max 4 questions) only for gaps that change the design:
- Users: total named users and **peak concurrent** users; portal/customer users; mobile app/API traffic.
- Workload: modules (Accounting, Stock, Manufacturing, HR/Payroll, POS, CRM), transactions per day, heavy reports, integrations (e-invoice/GST, payment gateways, biometric, WhatsApp/SMS), file/attachment volume.
- Data: current DB size or years of migrated data; expected growth/year.
- Availability: acceptable downtime (business hours only vs 24×7), RPO/RTO, multi-company/multi-site.
- Constraints: budget (monthly), hosting preference (AWS, Azure, GCP, DigitalOcean/Hetzner/E2E, on-prem, Frappe Cloud), data-residency (India), client IT policies, Frappe version.

## Step 2 — Choose a tier
Use `references/sizing.md` to pick a tier from concurrency and workload, then adjust:
- +1 tier for Manufacturing/Stock-heavy with large BOMs, heavy Payroll runs, or many integrations.
- Separate DB server once DB > ~50 GB or concurrency > ~75, or when reports compete with transactions.
- Read replica for heavy reporting/BI.
- HA (multi-app-node + managed/replicated DB + load balancer) only when the client needs 24×7 with RTO < 1 hour — state the cost multiplier honestly.

## Step 3 — Design
Produce:
1. **Architecture diagram** (Mermaid flowchart): users → DNS/CDN → load balancer/nginx → app (gunicorn) + socketio → Redis cache/queue → RQ workers + scheduler → MariaDB (+ replica) → file storage (local / S3) → backups offsite → monitoring. Include staging.
2. **Bill of materials** per environment: instance type/size (give 2 provider options, e.g. AWS + a budget provider), vCPU, RAM, disk type/size, managed vs self-hosted DB, estimated monthly cost range labeled as an estimate to verify on the provider's pricing page.
3. **Tuning values** derived from the chosen size: gunicorn workers, RQ workers per queue, `innodb_buffer_pool_size`, `innodb_log_file_size`, `max_connections`, Redis cache `maxmemory`, nginx `client_max_body_size`, swap — all following `servers.md`.
4. **Environments**: dev / staging / production per `environments.md`; staging size (usually one tier smaller but same versions).
5. **Security**: network layout (private subnet for DB/Redis), firewall rules, SSH policy, TLS, secrets handling, backup encryption.
6. **Backup & DR**: schedule, offsite target, retention, RPO/RTO achieved, restore drill.
7. **Monitoring**: what to monitor and alert thresholds.
8. **Scaling path**: what to change when users double (vertical first, then split DB, then add app nodes).

## Step 4 — Setup runbook
Write numbered, copy-paste-ready steps for the chosen approach (see `references/setup-runbook.md`): bare-metal bench or Docker (`frappe_docker`). Include verification after each phase and the go-live checklist.

## Output format
Present as: Requirements summary (with stated assumptions) → Recommended architecture (diagram + one-paragraph rationale) → BOM & cost table → Tuning table → Environments → Security → Backup/DR → Monitoring → Scaling path → Setup runbook → Open questions. Offer to turn it into a client-facing proposal document.

Rules: be explicit about assumptions; never under-size production to hit a budget without saying the risk; prefer simple single-node setups for small clients.
