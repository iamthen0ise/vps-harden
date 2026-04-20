<!-- Generated: 2026-04-20 | Files scanned: 1 | Token estimate: ~900 -->

# Module Reference

Each module is a function `mod_<name>()` in `vps-harden.sh`. Gating is done by `should_run()` which respects `--only` / `--skip`.

## OS Modules

| Module | Lines | What it does | Key files written |
|--------|-------|-------------|-------------------|
| `prereqs` | 392–413 | Installs foundation packages | — |
| `user` | 415–469 | Creates user, sudo, SSH keys, optional password | `~/.ssh/authorized_keys` |
| `ssh` | 471–547 | SSH hardening config + banner | `/etc/ssh/sshd_config.d/00-hardening.conf`, `/etc/ssh/banner.txt` |
| `firewall` | 549–576 | UFW default-deny, allow SSH | — |
| `fail2ban` | 578–613 | 3 retries / 3h ban, UFW integration | `/etc/fail2ban/jail.local` |
| `sysctl` | 615–675 | Kernel hardening + IP forwarding | `/etc/sysctl.d/99-zz-hardening.conf` |
| `netbird` | 677–727 | Installs + connects Netbird mesh VPN | — |
| `firewall_tighten` | 729–780 | Allow wt0 traffic, restrict SSH to safety IP | — |
| `sops` | 782–858 | SOPS + age install, keypair generation | `~/.config/sops/age/keys.txt`, `~/.sops.yaml` |
| `upgrades` | 860–892 | Unattended-upgrades, optional auto-reboot | `/etc/apt/apt.conf.d/50unattended-upgrades` |
| `monitoring` | 894–1167 | auditd rules, logwatch config, server-report install, optional openclaw | `/etc/audit/rules.d/vps-harden.rules`, `/etc/logwatch/conf/logwatch.conf` |
| `shell` | 1169–1223 | umask 027, bash history timestamps, plaintext secret scan | `~/.bashrc` |
| `misc` | 1225–1280 | Timezone, hostname, lock root pw, restrict su | `/etc/pam.d/su` |
| `docker` | 1282–1373 | Docker Engine via official apt repo, adds user to docker group | `/etc/apt/keyrings/docker.asc`, `/etc/apt/sources.list.d/docker.list` |

## Agent Modules (require `--agent-dir`)

| Module | Lines | What it checks / enforces |
|--------|-------|--------------------------|
| `agent_secrets` | 1375–1526 | Scans for plaintext secrets in workspace, verifies SOPS encryption |
| `agent_webhook_auth` | 1528–1641 | Webhook listener, UFW rules, TLS proxy, auth headers, rate limiting |
| `agent_logging` | 1643–1829 | Log directory (750), logrotate, append-only (`chattr +a`), auditd rules |
| `agent_data` | 1831–1955 | Data dir permissions (700), .gitignore, git history scan, encryption check |

## Verify Module

`mod_verify` (1957–2340) — always runs last. Calls `sc_add()` to accumulate PASS/WARN/FAIL entries, then prints a grouped scorecard.

Sections checked: SSH Hardening → Firewall → Intrusion Prevention → Kernel Hardening → Docker → Monitoring → Network → Secrets → System → (Agent Security if `--agent-dir`)

## prereqs Package List (v1.5.0)

```
curl wget jq htop tree unzip ufw fail2ban
netcat-openbsd telnet iputils-ping cron
dnsutils net-tools traceroute lsof
```

## Module Template Pattern

```bash
mod_example() {
    log_header "Module: example"

    if [[ <already configured> ]]; then
        log_info "Already done"; return 0
    fi

    run_cmd <command>                  # no-op in dry-run
    write_file "/path" "644" <<'EOF'   # no-op in dry-run
content
EOF
    log_ok "Done"
}
```
