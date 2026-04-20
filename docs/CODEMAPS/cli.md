<!-- Generated: 2026-04-20 | Files scanned: 1 | Token estimate: ~500 -->

# CLI & Configuration Reference

## Global Variables (defaults)

```bash
USERNAME=""          # --username USER
SSH_KEY=""           # --ssh-key KEY (file path or inline ssh-* string)
SSH_KEY_CONTENT=""   # resolved key text (set by wizard or validate_args)
SSH_SAFETY_IP=""     # --ssh-safety-ip IP
NETBIRD_KEY=""       # --netbird-key KEY  (skips netbird module if empty)
TIMEZONE=""          # --timezone TZ
SET_HOSTNAME=""      # --hostname NAME
USER_PASSWORD=""     # --user-password PASS
AUTO_REBOOT="false"  # --auto-reboot
OPENCLAW_SKILL="false" # --openclaw-skill
AGENT_DIR=""         # --agent-dir DIR   (enables agent modules)
WEBHOOK_PORT="5000"  # --webhook-port PORT
AGENT_DATA_DIR=""    # --agent-data-dir DIR
SKIP_MODULES=""      # --skip MOD[,MOD]
ONLY_MODULES=""      # --only MOD[,MOD]
DRY_RUN="false"      # --dry-run
VERBOSE="false"      # --verbose
INTERACTIVE="false"  # --interactive
CONFIG_FILE=""       # --config FILE
```

## Config File Format (`--config FILE`)

```bash
username=deploy
ssh_key=/root/.ssh/authorized_keys
ssh_safety_ip=203.0.113.10
netbird_key=nbs-XXXX
timezone=UTC
hostname=my-vps
user_password=secret123
auto_reboot=true
agent_dir=/opt/agent
webhook_port=5000
agent_data_dir=/opt/agent/data
skip=netbird,sops
```

## Interactive Wizard Steps (7 steps)

```
1. Username          read -r       default: $SUDO_USER or "deploy"
2. User Password     read -rs      optional, confirm loop
3. SSH Public Key    read -r       auto-detect → GitHub → ssh-copy-id → paste
4. SSH Safety IP     read -r       auto-detect via detect_ssh_client_ip()
5. Timezone          read -r       auto-detect via timedatectl
6. Netbird Key       read -r       optional
7. Dry Run?          read -r       default: Y
```

## Key Helper Functions

| Function | Purpose |
|----------|---------|
| `run_cmd CMD` | Execute or dry-log command |
| `write_file PATH MODE [OWNER]` | Write stdin to file or dry-log |
| `should_run MOD` | Evaluate --only / --skip |
| `require_agent_dir` | Gate for agent modules |
| `verify_ssh_access USER` | Pre-restart lockout check |
| `detect_ssh_service` | Returns "ssh" or "sshd" |
| `detect_arch` | Returns "amd64" or "arm64" |
| `detect_ssh_client_ip` | 3-method client IP detection |
| `sc_add RESULT LABEL [HINT]` | Append scorecard entry |
| `build_rerun_cmd [true]` | Build equivalent CLI string |

## Log File

`/var/log/vps-harden-YYYYMMDD-HHMMSS.log` — created at start, mode 600, contains all `[+]`, `[!]`, `[-]`, `[CMD]`, `[DRY]` lines.
