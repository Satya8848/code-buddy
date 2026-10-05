# Environments: Development, Staging, Production

| Item | Development | Staging | Production |
|---|---|---|---|
| `developer_mode` | 1 | 0 | 0 |
| Data | sample / masked | recent masked prod copy | live |
| Deploys | anyone | via CI/merge to `staging` | via release PR + tech lead approval |
| Email/SMS/payments | disabled or sandbox | sandbox keys, email to test inbox only | live |
| Scheduler | off unless testing | on | on |
| Backups | none | daily | at least every 6h DB + daily files, off-server copy (S3/compatible) |
| Monitoring | none | basic | uptime + error + resource alerts |

## Rules
- Staging mirrors production versions exactly (Frappe, ERPNext, custom apps, Python, Node, MariaDB).
- Never test on production. Never point staging at production DB, Redis or storage.
- Production deploy checklist: backup taken → maintenance window communicated → `git pull` / image update → `bench migrate` → `bench build` (if assets changed) → `bench restart` → smoke test → monitor error log 30 min.
- Rollback plan written in the PR for anything with patches.
- Use `bench --site <site> set-maintenance-mode on` during risky migrations.
- Email outgoing on staging must be disabled or redirected (`mute_emails: 1` in site config).
