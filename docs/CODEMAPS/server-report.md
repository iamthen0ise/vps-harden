<!-- Generated: 2026-04-20 | Files scanned: 1 | Token estimate: ~300 -->

# server-report

Companion health-check CLI installed to `/usr/local/bin/server-report` by `mod_monitoring`.

## Commands

| Subcommand | What it shows |
|------------|--------------|
| `summary` | Hostname, uptime, load, memory, disk, failed SSH attempts (24h), service status, pending updates |
| `auth` | Failed/successful logins (48h), active sessions, fail2ban banned IPs |
| `audit` | Audit events grouped by key (ssh_config, user_db, sudoers, su_attempts, privesc, data_access) |
| `full` | Full logwatch report (today, detail low) |
| `version` | Print version string |

## Design Rules

- **No color codes** — plain text output only, safe for bot/pipe consumption
- **Graceful degradation** — tools checked with `cmd_exists()` before use; prints `(unavailable)` if missing
- **No external dependencies** — reads from journalctl, last, ss, df, free, auditctl, logwatch, apt

## Key Helper Functions

```bash
cmd_exists()   # command -v wrapper
safe_run()     # run command or return fallback string
section()      # print "--- label ---" separator
```

## Install Path

`mod_monitoring` in `vps-harden.sh` downloads this script from the repo and places it at `/usr/local/bin/server-report` (mode 755). Source: same GitHub repo as the main script.
