# Persistence Simulation

## Overview

This simulation generates Linux persistence activity commonly used by attackers after compromise.

The purpose is to validate persistence detection logic using Auditd process telemetry and Elastic SIEM correlation rules.

---

# MITRE ATT&CK Mapping

| Technique | ID |
|---|---|
| Account Manipulation | T1098 |
| Scheduled Task / Cron | T1053 |

Reference: https://attack.mitre.org

---

# Environment

| Component | Description |
|---|---|
| Host OS | Ubuntu Linux |
| Monitoring | Auditd |
| SIEM Platform | Elastic Stack |

---

# Simulation Techniques

This simulation includes:
- User creation
- Cron job modification
- SSH authorized_keys modification

---

# Step 1 — Create Local User

```bash
sudo useradd attacker
```

---

# Step 2 — Modify Cron Jobs

```bash
crontab -e
```

Add a simple scheduled task:

```text
* * * * * echo "test"
```

Save and exit.

---

# Step 3 — Modify SSH Authorized Keys

Create SSH directory if needed:

```bash
mkdir -p ~/.ssh
```

Add a fake SSH key:

```bash
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD" >> ~/.ssh/authorized_keys
```

---

# Expected Logs

The following process activity should appear:

```text
process.name: useradd
process.name: crontab
```

And potentially:

```text
authorized_keys modification
```

---

# Validation Queries

## User Creation

```kql
process.name: "useradd"
```

---

## Cron Activity

```kql
process.name: "crontab"
```

---

## SSH Key Modification

```kql
process.args: "*authorized_keys*"
```

---

# Detection Validation

This simulation validates:
- Persistence detection rules
- Attack chain correlation logic
- Auditd process telemetry

---

# Expected Outcome

The SIEM should:
- Detect suspicious persistence mechanisms
- Associate persistence activity with prior escalation events
- Generate alerts for investigation

---
