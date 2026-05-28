# Lab Setup Guide

This document explains how the Elastic SIEM Detection Engineering Lab was built and configured.

The environment was designed to simulate real-world attack detection and investigation workflows using the Elastic Stack.

---

# Lab Overview

The lab consists of:
- A Fleet Server running on Ubuntu
- Elastic Agent enrolled on a monitored Linux host
- Elasticsearch and Kibana for log ingestion and analysis
- Auditd for Linux process telemetry
- Detection rules and dashboards for attack monitoring

---

# Architecture

<p align="center">
  <img src="../architecture/diagram.png" width="900">
</p>

---

# Environment Details

| Component | Details |
|---|---|
| Host OS | Ubuntu Linux |
| SIEM Platform | Elastic Stack |
| Management | Fleet Server |
| Endpoint Monitoring | Elastic Agent |
| Process Auditing | Auditd |
| Detection Language | EQL / KQL |

---

# System Requirements

## Recommended Resources

| Resource | Minimum |
|---|---|
| RAM | 8 GB |
| CPU | 4 cores |
| Storage | 50+ GB |
| OS | Ubuntu 22.04 LTS |

---

# Step 1 — Install Elasticsearch

Install Elasticsearch on the Ubuntu SIEM server.

## Add Elastic Repository

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

sudo apt-get install apt-transport-https

echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

## Install Elasticsearch

```bash
sudo apt update
sudo apt install elasticsearch -y
```

---

## Start Elasticsearch

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

---

## Verify Elasticsearch

```bash
curl -k https://localhost:9200
```

Expected output:

```json
{
  "name" : "ubuntu-vm",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "8.x.x"
  }
}
```

---

# Step 2 — Install Kibana

## Install Kibana

```bash
sudo apt install kibana -y
```

---

## Start Kibana

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```

---

## Access Kibana

Open:

```text
http://<SERVER-IP>:5601
```

---

# Step 3 — Install Fleet Server

Fleet Server is used for centralized agent management.

---

## Add Fleet Server Integration

In Kibana:

1. Navigate to:
   - Management → Fleet
2. Add Fleet Server
3. Generate enrollment token
4. Copy installation command

---

## Example Fleet Server Install

```bash
sudo ./elastic-agent install \
  --fleet-server-es=https://<SERVER-IP>:9200 \
  --fleet-server-service-token=<TOKEN> \
  --fleet-server-policy=<POLICY-ID> \
  --fleet-server-port=8220
```

---

# Step 4 — Install Elastic Agent on Linux Host

Install Elastic Agent on the monitored endpoint.

---

## Download Elastic Agent

```bash
wget https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.x-linux-x86_64.tar.gz
```

---

## Extract Files

```bash
tar -xzf elastic-agent-8.x-linux-x86_64.tar.gz
cd elastic-agent-8.x-linux-x86_64
```

---

## Enroll Agent

```bash
sudo ./elastic-agent install \
  --url=https://<FLEET-SERVER-IP>:8220 \
  --enrollment-token=<TOKEN>
```

---

## Verify Agent Status

```bash
sudo systemctl status elastic-agent
```

In Kibana:
- Fleet → Agents
- Confirm agent appears healthy

---

# Step 5 — Configure Integrations

## System Integration

Added integrations:
- Authentication Logs
- Syslog
- Metrics

These provide:
- SSH authentication visibility
- Host telemetry
- System monitoring

---

## Auditd Integration

Auditd integration was added to capture Linux process execution events.

---

# Step 6 — Install & Configure Auditd

Auditd provides process-level telemetry used for detection engineering.

---

## Install Auditd

```bash
sudo apt update
sudo apt install auditd audispd-plugins -y
```

---

## Start Auditd

```bash
sudo systemctl start auditd
sudo systemctl enable auditd
```

---

## Add Process Execution Rule

```bash
sudo auditctl -a always,exit -F arch=b64 -S execve
```

This logs all executed commands.

---

## Verify Auditd Logs

```bash
sudo tail -f /var/log/audit/audit.log
```

---

# Step 7 — Validate Log Ingestion

## Authentication Logs

In Kibana:

```kql
event.dataset: "system.auth"
```

---

## Auditd Logs

```kql
event.dataset: "auditd.log"
```

---

# Step 8 — Create Detection Rules

Detection rules were built using EQL correlation logic.

---

## Brute Force Detection

```eql
sequence by source.ip, user.name with maxspan=10m
  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "success"]
```

---

## Full Attack Chain Detection

```eql
sequence by host.id, source.ip, user.name with maxspan=20m

  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "failure"]
  [authentication where event.outcome == "failure"]

  [authentication where event.outcome == "success"]

  [process where process.name in ("sudo", "su")]

  [process where process.name in ("useradd", "usermod", "crontab")]
```

---

# Step 9 — Build Dashboards

Dashboards were created in Kibana to visualize:
- Failed vs successful logins
- Top source IPs
- Privilege escalation activity
- Persistence attempts
- Attack timelines
- Detection alerts

---

# Security Monitoring Workflow

The lab supports a full SOC-style investigation workflow:

1. Detect brute force attempts
2. Identify attacker IP
3. Correlate successful authentication
4. Investigate privilege escalation
5. Detect persistence mechanisms
6. Review attack timeline
7. Analyze generated alerts

---

# Troubleshooting

## Elastic Agent Not Appearing

Restart agent:

```bash
sudo systemctl restart elastic-agent
```

---

## Auditd Not Running

```bash
sudo systemctl status auditd
```

---

## No Authentication Fields

Ensure:
- System integration enabled
- Authentication logs enabled
- Agent restarted

---
