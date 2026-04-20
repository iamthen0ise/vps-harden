<!-- Generated: 2026-04-20 | Files scanned: 3 | Token estimate: ~600 -->

# Architecture

## Project Type

Single-file Bash hardening tool. Zero runtime dependencies. Runs on Ubuntu/Debian as root.

## File Map

```
vps-harden.sh   2748 lines  Main hardening script — 19 modules, wizard, scorecard
server-report    288 lines  Companion health-check CLI (no colors, bot-safe output)
install.sh        79 lines  Downloader — places both tools in /usr/local/bin/
```

## Execution Flow

```
main()
  └── parse_args()           CLI flags → global vars
  └── parse_config_file()    KEY=VALUE file → global vars (optional)
  └── interactive_setup()    7-step wizard (TTY only, triggered if no --username/--ssh-key)
  └── validate_args()        Resolve SSH key content, assert root
  └── check_distro()         Ubuntu/Debian only
  └── log_init()             /var/log/vps-harden-TIMESTAMP.log
  └── module loop            for mod in ALL_MODULES: should_run → mod_$mod
  └── build_rerun_cmd()      Print equivalent CLI for future runs
```

## Module Execution Order

```
OS hardening (always):
  prereqs → user → ssh → firewall → fail2ban → sysctl
  → netbird → firewall_tighten → sops → upgrades
  → monitoring → shell → misc → docker

Agent workspace (gated by --agent-dir):
  → agent_secrets → agent_webhook_auth → agent_logging → agent_data

Always last:
  → verify   (security scorecard)
```

## Key Design Invariants

- **Idempotent**: every module checks current state before acting, returns early if correct
- **Dry-run**: all mutations go through `run_cmd()` and `write_file()` — both no-op when `DRY_RUN=true`
- **SSH lockout protection**: `verify_ssh_access()` validates config + keys + UFW before `sshd` restart; auto-rollback on failure
- **Scorecard**: `mod_verify` is always the final module; prints PASS/WARN/FAIL grouped by section
