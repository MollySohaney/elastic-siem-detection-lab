# SSH Brute Force Simulation

## Overview

This simulation generates repeated failed SSH authentication attempts followed by a successful login to emulate a brute force attack scenario.

The purpose of this simulation is to validate:
- Authentication log ingestion
- SSH detection rules
- Correlation logic
- Dashboard visualizations

---

# MITRE ATT&CK Mapping

| Technique | ID |
|---|---|
| Brute Force | T1110 |
| Valid Accounts | T1078 |

Reference: https://attack.mitre.org

---

# Environment

| Component | Description |
|---|---|
| Target Host | Ubuntu Linux |
| SIEM Platform | Elastic Stack |
| Monitoring | Elastic Agent + System Integration |

---

# Simulation Steps

## Step 1 — Generate Failed SSH Logins

Run the following command multiple times:

```bash
ssh fakeuser@localhost
```

Enter an invalid password when prompted.

Repeat this step 5–10 times.

---

## Step 2 — Successful Login

Authenticate with a valid user account:

```bash
ssh <valid_user>@localhost
```

---

# Expected Logs

The following events should appear in Kibana:

## Failed Authentication

```text
Failed password for invalid user fakeuser from <IP>
```

## Successful Authentication

```text
Accepted password for <valid_user> from <IP>
```

---

# Validation Queries

## Failed Logins

```kql
event.dataset: "system.auth" and event.outcome: "failure"
```

---

## Successful Logins

```kql
event.dataset: "system.auth" and event.outcome: "success"
```

---

# Detection Validation

This simulation should trigger:

- SSH Brute Force Detection Rule
- Multi-Stage Attack Chain Detection Rule

---

# Expected Outcome

The SIEM should detect:
- Multiple failed authentication attempts
- A successful login following repeated failures
- Correlated attack activity from the same source IP

---
