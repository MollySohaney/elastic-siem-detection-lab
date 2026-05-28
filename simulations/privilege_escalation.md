# Privilege Escalation Simulation

## Overview

This simulation generates Linux privilege escalation activity to validate process monitoring and escalation detection logic.

The objective is to simulate an attacker gaining elevated privileges after initial access.

---

# MITRE ATT&CK Mapping

| Technique | ID |
|---|---|
| Abuse Elevation Control Mechanism | T1548 |

Reference: https://attack.mitre.org

---

# Environment

| Component | Description |
|---|---|
| Host OS | Ubuntu Linux |
| Monitoring | Auditd + Elastic Agent |
| Detection Platform | Elastic SIEM |

---

# Simulation Steps

## Step 1 — Verify Auditd

Ensure Auditd is running:

```bash
sudo systemctl status auditd
```

---

## Step 2 — Generate Privilege Escalation Activity

### sudo Usage

```bash
sudo su
```

---

### Verify Root Context

```bash
id
```

Expected output:

```text
uid=0(root) gid=0(root)
```

---

# Expected Logs

The following process activity should appear:

```text
process.name: sudo
process.name: su
```

---

# Validation Queries

## sudo Execution

```kql
process.name: "sudo"
```

---

## su Execution

```kql
process.name: "su"
```

---

# Detection Validation

This simulation validates:
- Privilege escalation dashboards
- Auditd telemetry ingestion
- Attack chain correlation rules

---

# Expected Outcome

The SIEM should:
- Detect elevated command execution
- Associate escalation activity with the authenticated user
- Correlate escalation with earlier SSH activity

---
