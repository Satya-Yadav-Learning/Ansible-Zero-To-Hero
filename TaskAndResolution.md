# TaskAndResolution.md

# Ansible Inventory Configuration & Playbook Execution Troubleshooting

---

# Project Overview

A DevOps engineering team needed to validate Ansible playbooks against an Application Server hosted inside a private datacenter environment.  
The automation validation required:

- Creating an INI-based inventory file
- Configuring remote SSH authentication
- Ensuring hostname-based connectivity
- Executing Ansible playbooks successfully without additional runtime arguments

The task initially failed because the inventory used a direct IP address that was unreachable over SSH.  
The issue was resolved by switching to hostname-based communication.

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Jump Server | gateway-node |
| Application Server | app-node-03 |
| Automation User | deployuser |
| Inventory Path | /home/deployuser/automation/inventory |
| Playbook Path | /home/deployuser/automation/playbook.yml |
| Automation Tool | Ansible |
| Authentication Method | SSH Password Authentication |

---

# Objective

Create an Ansible inventory file that:

- Uses INI format
- Includes Application Server 3
- Uses hostname resolution
- Supports password-based SSH authentication
- Executes playbook successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```

---

# Step 1 — Create Working Directory

## Command

```bash
mkdir -p /home/deployuser/automation
```

---

# Command Explanation

| Component | Meaning |
|---|---|
| mkdir | Create directory |
| -p | Create parent directories if missing |
| /home/deployuser/automation | Target directory |

---

# Output

```bash
(no output)
```

---

# Why This Step Was Needed

This directory stores:

- inventory file
- playbook
- automation assets

---

# Step 2 — Create Inventory File

## Command

```bash
cat > /home/deployuser/automation/inventory << 'EOF'
[appservers]
app-node-03 ansible_user=deployuser ansible_password=SecurePass123 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

---

# Command Breakdown

## cat >

Used for writing multi-line content into a file.

---

## inventory

Target inventory file.

---

## << 'EOF'

HEREDOC syntax used to insert multiple lines.

---

# Inventory Content Explanation

## Group Definition

```ini
[appservers]
```

Defines an Ansible host group.

---

## Host Entry

```ini
app-node-03
```

Inventory hostname.

---

## SSH Username

```ini
ansible_user=deployuser
```

SSH user used by Ansible.

Equivalent to:

```bash
ssh deployuser@app-node-03
```

---

## SSH Password

```ini
ansible_password=SecurePass123
```

Password used for authentication.

---

## SSH Arguments

```ini
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Automatically accepts SSH host keys.

Without this:

```text
Are you sure you want to continue connecting (yes/no)?
```

would interrupt automation.

---

# Step 3 — Verify Inventory

## Command

```bash
cat /home/deployuser/automation/inventory
```

---

# Output

```ini
[appservers]
app-node-03 ansible_user=deployuser ansible_password=SecurePass123 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

# Step 4 — Change Directory

## Command

```bash
cd /home/deployuser/automation
```

---

# Why Needed

Inventory file resides in this directory.

---

# Step 5 — Test Ansible Connectivity

## Command

```bash
ansible -i inventory all -m ping
```

---

# Command Explanation

| Component | Meaning |
|---|---|
| ansible | Ad-hoc Ansible command |
| -i inventory | Use custom inventory |
| all | Target all hosts |
| -m ping | Execute ping module |

---

# Important Clarification

This is NOT ICMP ping.

Ansible ping:
- establishes SSH connection
- executes Python remotely
- returns pong

---

# Initial Failure

## Output

```bash
app-node-03 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 10.x.x.x port 22: Connection timed out",
    "unreachable": true
}
```

---

# Root Cause Analysis

The inventory initially used:

```ini
ansible_host=10.x.x.x
```

Problem:
- Direct IP routing failed
- SSH over hostname worked

This indicated:
- DNS resolution functional
- IP path/firewall issue existed

---

# Connectivity Troubleshooting

## Failed Direct IP SSH

```bash
ssh deployuser@10.x.x.x
```

---

# Output

```bash
ssh: connect to host 10.x.x.x port 22: Connection timed out
```

---

# Successful Hostname SSH

## Command

```bash
ssh deployuser@app-node-03
```

---

# Output

```bash
[deployuser@app-node-03 ~]$
```

---

# Final Inventory Fix

## Updated Inventory

```ini
[appservers]
app-node-03 ansible_user=deployuser ansible_password=SecurePass123 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Removed:

```ini
ansible_host=10.x.x.x
```

---

# Step 6 — Retest Ansible Connectivity

## Command

```bash
ansible -i inventory all -m ping
```

---

# Successful Output

```json
app-node-03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Step 7 — Execute Playbook

## Command

```bash
ansible-playbook -i inventory playbook.yml
```

---

# Successful Output

```bash
PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [app-node-03]

TASK [Install httpd package] ***************************************************
changed: [app-node-03]

TASK [Start service httpd] *****************************************************
changed: [app-node-03]

PLAY RECAP *********************************************************************
app-node-03 : ok=3 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

---

# Playbook Execution Analysis

| Metric | Meaning |
|---|---|
| ok=3 | 3 tasks executed successfully |
| changed=2 | 2 tasks modified system state |
| unreachable=0 | SSH connectivity successful |
| failed=0 | No task failure |

---

# Final Resolution Summary

| Problem | Resolution |
|---|---|
| SSH timeout using IP | Switched to hostname |
| Inventory misconfiguration | Removed ansible_host |
| Playbook unreachable | DNS-based SSH fixed connectivity |
| Validation failure | Inventory corrected |

---

# Key DevOps Concepts Covered

- Configuration Management
- Infrastructure Automation
- SSH Authentication
- Ansible Inventory
- Hostname Resolution
- DNS Troubleshooting
- Remote Execution
- Infrastructure as Code (IaC)
- Automation Validation
- Linux Connectivity Troubleshooting

---

# Enterprise Best Practices

## Avoid Hardcoded Passwords

Use:
- Ansible Vault
- IAM Roles
- SSH Keys

instead of plaintext passwords.

---

## Prefer SSH Keys

Better than passwords because:
- secure
- scalable
- automation friendly

---

## Use Groups

Example:

```ini
[webservers]
web01
web02

[dbservers]
db01
```

---

## Use Variables

Move credentials into:
- group_vars
- host_vars
- vaults

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is an Ansible inventory?

### Answer

An inventory is a file containing:
- target servers
- groups
- SSH variables

used by Ansible to know where to execute automation tasks.

---

## Q2. Difference between static and dynamic inventory?

### Answer

| Static | Dynamic |
|---|---|
| Manually maintained | Auto-generated |
| Simple environments | Cloud environments |
| INI/YAML file | AWS/GCP/Azure API |

---

## Q3. What does ansible -m ping do?

### Answer

It:
- SSH connects to remote host
- executes Python code remotely
- verifies Ansible communication

It is not ICMP ping.

---

## Q4. Why did the task initially fail?

### Answer

Because:
- direct IP connectivity failed
- SSH over hostname worked

Inventory used unreachable IP.

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. How does Ansible authenticate to remote hosts?

### Answer

Using:
- SSH keys
- passwords
- Kerberos
- certificates

---

## Q2. Why use StrictHostKeyChecking=no?

### Answer

To avoid interactive SSH prompts during automation.

Without it:
automation pipelines may hang.

---

## Q3. How would you secure passwords in production?

### Answer

Using:
- Ansible Vault
- AWS Secrets Manager
- HashiCorp Vault
- IAM roles

Never plaintext.

---

## Q4. How would you troubleshoot unreachable hosts?

### Answer

1. Test SSH manually
2. Validate DNS
3. Check routing
4. Verify firewall rules
5. Validate SSH service
6. Check inventory syntax

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you design scalable Ansible infrastructure?

### Answer

Using:
- dynamic inventory
- role-based playbooks
- CI/CD integration
- centralized secrets
- reusable roles
- environment separation

---

## Q2. Difference between push and pull configuration management?

### Answer

| Push | Pull |
|---|---|
| Ansible | Puppet/Chef |
| Central execution | Agent-based |
| SSH based | Agent daemon |

---

## Q3. How would you integrate Ansible with CI/CD?

### Answer

Using:
- Jenkins
- GitHub Actions
- GitLab CI
- AWX/Tower

Pipeline stages:
- lint
- test
- deploy
- validate

---

## Q4. How do you make playbooks idempotent?

### Answer

By ensuring repeated runs produce same system state.

Example:
- package already installed → no change

---

# L4 — DevOps Architect

---

## Q1. How would you automate hybrid cloud infrastructure?

### Answer

Architecture:
- Terraform → provisioning
- Ansible → configuration
- Kubernetes → orchestration
- Vault → secrets
- Jenkins/GitHub Actions → CI/CD

---

## Q2. How would you secure enterprise automation?

### Answer

Using:
- RBAC
- MFA
- encrypted secrets
- private networking
- bastion hosts
- immutable infrastructure
- audit logging

---

## Q3. Explain enterprise Ansible scaling strategy.

### Answer

Use:
- AWX/Ansible Tower
- execution nodes
- role repositories
- artifact management
- inventory sync
- GitOps workflow

---

# AWS Scenario-Based Interview Questions

---

# Q1. How would you replace static inventory in AWS?

## Answer

Using:
- EC2 Dynamic Inventory
- AWS Tags
- IAM roles

Example:
- tag instances as webserver
- auto-discover hosts

---

# Q2. How would you deploy playbooks across Auto Scaling Groups?

## Answer

Using:
- Launch Templates
- User Data
- Dynamic Inventory
- SSM Automation

---

# Q3. How would you secure Ansible in AWS?

## Answer

Using:
- IAM roles
- AWS Secrets Manager
- SSM Parameter Store
- private subnets
- bastion hosts

---

# Q4. How would you troubleshoot unreachable EC2 instances?

## Answer

Check:
- Security Groups
- NACLs
- Route Tables
- SSH daemon
- subnet connectivity
- IAM permissions

---

# Real AWS Incident RCA Examples

# Incident 1 — EC2 Unreachable After Deployment

## Symptoms

- Deployment failed
- SSH timeout
- Health checks failed

---

## Root Cause

Security Group removed inbound port 22.

---

## Impact

- Deployment halted
- Autoscaling failed
- Recovery delayed

---

## Resolution

- Re-added SSH rule
- Implemented Terraform validation

---

## Preventive Actions

- Infrastructure testing
- Policy enforcement
- CI validation

---

# Incident 2 — Route Table Misconfiguration

## Symptoms

- EC2 reachable internally
- External SSH failed

---

## Root Cause

Incorrect route table associated with subnet.

---

## Resolution

- Corrected route association
- Added monitoring alerts

---

# Incident 3 — DNS Resolution Failure

## Symptoms

- Hostname unreachable
- IP reachable

---

## Root Cause

Broken Route53 private hosted zone.

---

## Resolution

- Recreated DNS records
- Added DNS monitoring

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between hard link and soft link?

### Answer

| Hard Link | Soft Link |
|---|---|
| Same inode | Different inode |
| Cannot cross filesystem | Can cross filesystem |
| Survives source deletion | Breaks if source removed |

---

# Networking

## Q2. Difference between TCP and UDP?

### Answer

| TCP | UDP |
|---|---|
| Reliable | Fast |
| Connection oriented | Connectionless |
| Ordered delivery | No guarantee |

---

# AWS

## Q3. Difference between Security Group and NACL?

### Answer

| Security Group | NACL |
|---|---|
| Stateful | Stateless |
| Instance level | Subnet level |

---

# Docker

## Q4. Difference between CMD and ENTRYPOINT?

### Answer

| CMD | ENTRYPOINT |
|---|---|
| Default arguments | Main executable |
| Overridable | Less overridable |

---

# Kubernetes

## Q5. Difference between Deployment and StatefulSet?

### Answer

| Deployment | StatefulSet |
|---|---|
| Stateless apps | Stateful apps |
| Random pod identity | Stable identity |

---

# Terraform

## Q6. What is Terraform state?

### Answer

Terraform state tracks infrastructure resources and mappings.

Used for:
- drift detection
- updates
- dependency resolution

---

# Jenkins

## Q7. Difference between scripted and declarative pipeline?

### Answer

| Scripted | Declarative |
|---|---|
| Flexible | Structured |
| Groovy heavy | Simpler syntax |

---

# Monitoring

## Q8. Difference between white-box and black-box monitoring?

### Answer

| White Box | Black Box |
|---|---|
| Internal metrics | External behavior |
| CPU/memory | Endpoint testing |

---

# Observability

## Q9. Three pillars of observability?

### Answer

- Logs
- Metrics
- Traces

---

# CI/CD

## Q10. Blue-Green vs Canary Deployment?

### Answer

| Blue-Green | Canary |
|---|---|
| Full switch | Gradual rollout |
| Faster rollback | Safer testing |

---

# Final Technical Learnings

This task demonstrated:

- Importance of DNS
- Difference between IP vs hostname connectivity
- SSH troubleshooting
- Automation reliability
- Infrastructure debugging
- Configuration management best practices
- Enterprise-grade automation principles

---

# Final Outcome

✅ Inventory created successfully  
✅ SSH connectivity validated  
✅ Hostname resolution fixed  
✅ Ansible ping successful  
✅ Playbook executed successfully  
✅ Services installed and started  
✅ Validation passed successfully

