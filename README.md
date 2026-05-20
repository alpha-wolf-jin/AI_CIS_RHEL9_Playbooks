# AI CIS RHEL9 Playbooks

Ansible playbooks for auditing and remediating RHEL 9 systems against the
**CIS Red Hat Enterprise Linux 9 Benchmark v2.0.0**.

Covers **275 checkpoints** across all 7 CIS sections (1.x through 7.x).
Every playbook is AI-generated and follows a snapshot-execute-restore lifecycle
for safe, repeatable execution.

## Prerequisites

- **Control node**: Ansible 2.14+ with `ansible-navigator` installed
- **Target host**: Red Hat Enterprise Linux 9
- **Privilege**: SSH access with `become` (sudo) capability

## Repository Structure

```
.
├── audit.yml                       # Master audit playbook
├── remediation.yml                 # Master remediation playbook
├── rhel9_cis.yml                   # Checkpoint index (275 entries)
├── actions.yaml                    # Audit action dispatcher
├── execute.yaml                    # Audit task executor
├── remediation_actions.yaml        # Remediation action dispatcher
├── remediation_execute.yaml        # Remediation task executor
├── cis_rhel9_audit_tasks/          # Individual audit task files
│   ├── cis_audit_1_1_1_1.yml
│   ├── cis_audit_var_1_1_1_1.yml   # Variables for the audit task
│   └── ...
└── cis_rhel9_remediation_tasks/    # Individual remediation task files
    ├── cis_remediation_1_1_1_1.yml
    ├── cis_remediation_var_1_1_1_1.yml
    └── ...
```

## Quick Start

### 1. Audit a single checkpoint

```bash
ansible-navigator run audit.yml \
  -e index=5.1.1 \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become
```

### 2. Audit multiple checkpoints

Comma-separated list of checkpoint IDs:

```bash
ansible-navigator run audit.yml \
  -e index=5.1.1,5.4.2.4,5.4.2.5 \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become
```

### 3. Audit an entire section

Use a trailing dot to match all checkpoints under a section:

```bash
# All checkpoints under section 5.1
ansible-navigator run audit.yml \
  -e index=5.1. \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become

# All checkpoints under section 6
ansible-navigator run audit.yml \
  -e index=6. \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become
```

### 4. Remediate a single checkpoint

```bash
ansible-navigator run remediation.yml \
  -e index=5.1.1 \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become
```

### 5. Remediate multiple checkpoints

```bash
ansible-navigator run remediation.yml \
  -e index=5.1.1,5.4.2.4,5.4.2.5 \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become
```

### 6. Remediate without restoring (keep changes)

By default, remediation playbooks follow a snapshot-execute-restore lifecycle
and **roll back changes** after execution (designed for testing). To keep the
remediation applied permanently:

```bash
ansible-navigator run remediation.yml \
  -e index=5.1.1 \
  -e target_host=<your-host> \
  -i <your-host>, \
  --become \
  --skip-tags restore
```

### 7. Full audit-remediate-verify workflow

```bash
# Step 1: Run audit to check current compliance
ansible-navigator run audit.yml \
  -e index=5.1. \
  -e target_host=<your-host> \
  -i <your-host>, --become

# Step 2: Remediate (keep changes)
ansible-navigator run remediation.yml \
  -e index=5.1. \
  -e target_host=<your-host> \
  -i <your-host>, --become \
  --skip-tags restore

# Step 3: Re-audit to verify compliance
ansible-navigator run audit.yml \
  -e index=5.1. \
  -e target_host=<your-host> \
  -i <your-host>, --become
```

## Playbook Lifecycle Tags

Each remediation playbook supports three lifecycle phases via tags:

| Tag | Phase | Purpose |
|-----|-------|---------|
| `snapshot` | Pre-tasks | Backup current system state |
| `execute` | Main tasks | Apply remediation changes |
| `restore` | Always block | Roll back to pre-remediation state |

Run specific phases:

```bash
# Snapshot only (backup without changing anything)
ansible-navigator run remediation.yml -e index=5.1.1 --tags snapshot

# Execute only (apply changes, no backup or restore)
ansible-navigator run remediation.yml -e index=5.1.1 --tags execute

# Restore only (roll back to last snapshot)
ansible-navigator run remediation.yml -e index=5.1.1 --tags restore
```

## Output

Both audit and remediation playbooks set a `cis_result` variable for each
checkpoint with one of these values:

| Value | Meaning |
|-------|---------|
| `PASS` | System is compliant |
| `FAIL` | System is not compliant |
| `SKIP` / `SKIPPED` | Check was skipped (prerequisite not met) |
| `NOT_APPLICABLE` | Check does not apply to this system |
| `ERROR` | Task encountered an error |

The aggregated results are available in the `all_results` dictionary at the
end of the play, keyed by checkpoint ID.

## CIS Benchmark Sections

| Section | Description |
|---------|-------------|
| 1.x | Initial Setup (filesystem, boot, crypto, etc.) |
| 2.x | Services (time sync, special-purpose, clients) |
| 3.x | Network Configuration (firewall, kernel params) |
| 4.x | Access Control (SSH, privilege escalation) |
| 5.x | Authentication and Authorization (PAM, accounts) |
| 6.x | Logging and Auditing (journald, auditd) |
| 7.x | System Maintenance (file permissions, accounts) |

## License

This project is provided as-is for CIS compliance automation.
