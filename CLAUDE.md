# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`vps-harden` is a single-file Bash script that hardens Ubuntu VPS servers. It is idempotent, dry-run-first, and has zero runtime dependencies. The companion `server-report` CLI provides post-installation health checks. Both tools are distributed via `install.sh`.

## Commands

```bash
# Lint and syntax check (CI runs these)
shellcheck vps-harden.sh server-report
bash -n vps-harden.sh
bash -n install.sh

# Run (requires root on a Ubuntu host)
sudo vps-harden --dry-run --username deploy --ssh-key ~/.ssh/id_ed25519.pub
sudo vps-harden --only verify   # scorecard only
```

There is no build step. CI is ShellCheck + `bash -n` only.

## Architecture

Everything lives in `vps-harden.sh` (~2600 lines). Execution flows top-to-bottom:

1. **Constants and setup** (lines 1–180) — version, log file path, `ALL_MODULES` array, color vars, scorecard tracking, logging helpers (`log_info`, `log_warn`, `log_ok`, `die`), dry-run wrappers
2. **CLI parsing** — flags to variables, config file loading
3. **`main()`** — optional wizard, then the module execution loop
4. **Module functions** (`mod_prereqs` through `mod_verify`) — each is self-contained

### The 18 Modules (in execution order)

OS hardening: `prereqs` → `user` → `ssh` → `firewall` → `fail2ban` → `sysctl` → `netbird` → `firewall_tighten` → `sops` → `upgrades` → `monitoring` → `shell` → `misc` → `verify`

Agent workspace (only when `--agent-dir` is set): `agent_secrets` → `agent_webhook_auth` → `agent_logging` → `agent_data`

### Critical Patterns

**Idempotence** — every module checks current state before acting and returns early if already configured.

**Dry-run** — all mutations go through `run_cmd()` and `write_file()` wrappers. When `DRY_RUN=true` these log the intent but do nothing.

**SSH lockout protection** — `mod_ssh` validates `sshd_config` syntax, verifies the target user is in `AllowUsers`, confirms the public key is in `authorized_keys`, and checks UFW has an SSH allow rule before restarting `sshd`. It rolls back on failure.

**Scorecard** — `mod_verify` runs independent checks and prints grouped PASS/WARN/FAIL results. Always the last module.

### Module Template

```bash
mod_example() {
    log_info "What this module does"

    if [[ -f /etc/example.conf ]]; then
        log_ok "Already configured"
        return 0
    fi

    run_cmd "Install example" apt-get install -y example
    write_file "/etc/example.conf" "644" <<'CONF'
# config content
CONF

    log_ok "Example configured"
}
```

New modules must be added to the `ALL_MODULES` array and follow this pattern so `--only` / `--skip` flags work correctly.

## Contributor Requirements

- `shellcheck vps-harden.sh server-report` must pass with no errors
- `bash -n vps-harden.sh` must pass
- Every new module needs a corresponding check in `mod_verify`
- `server-report` output must remain plain text (no color codes, no control sequences) — it is consumed by bots
