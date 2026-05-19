# TaskAndResolution.md

# Ansible File Distribution Automation Across Application Servers

---

# Project Overview

A DevOps engineering team needed to automate file distribution from a centralized automation server to multiple application servers inside a private infrastructure environment using Ansible.

The task involved:

- Creating a centralized Ansible inventory
- Managing multiple remote hosts
- Executing automation playbooks
- Copying application data files to all managed nodes
- Ensuring password-based SSH automation worked correctly

The automation was validated using:

```bash
ansible-playbook -i inventory playbook.yml
```

without any additional runtime arguments.

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Automation Controller | control-node |
| Application Server 1 | app-node-a |
| Application Server 2 | app-node-b |
| Application Server 3 | app-node-c |
| Automation Directory | /home/automation/ansible |
| Source File | /usr/src/appdata/index.html |
| Destination Directory | /opt/appdata |
| Automation Tool | Ansible |

---

# Objective

## Requirements

1. Create inventory file:

```bash
/home/automation/ansible/inventory
```

2. Add all application servers as managed nodes.

3. Create playbook:

```bash
/home/automation/ansible/playbook.yml
```

4. Copy:

```bash
/usr/src/appdata/index.html
```

to:

```bash
/opt/appdata/index.html
```

on all application servers.

---

# Step 1 — Create Working Directory

## Command

```bash
mkdir -p /home/automation/ansible
```

---

# Command Breakdown

| Component | Meaning |
|---|---|
| mkdir | Create directory |
| -p | Create parent directories if missing |
| /home/automation/ansible | Target directory |

---

# Output

```bash
(no output)
```

---

# Why This Step Is Needed

This directory stores:
- inventory file
- playbook
- automation assets

---

# Step 2 — Create Inventory File

## Command

```bash
cat > /home/automation/ansible/inventory << 'EOF'
[application_servers]
app-node-a ansible_user=deployuser ansible_password=SecurePass1 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
app-node-b ansible_user=deployuser ansible_password=SecurePass2 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
app-node-c ansible_user=deployuser ansible_password=SecurePass3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

---

# Inventory File Explanation

## Group Definition

```ini
[application_servers]
```

Defines a logical host group.

All servers under this group become playbook targets.

---

# Host Entries

Example:

```ini
app-node-a ansible_user=deployuser ansible_password=SecurePass1
```

---

# Parameter Breakdown

| Parameter | Purpose |
|---|---|
| app-node-a | Inventory hostname |
| ansible_user | SSH username |
| ansible_password | SSH password |
| ansible_ssh_common_args | Additional SSH options |

---

# SSH Option Explanation

```bash
-o StrictHostKeyChecking=no
```

Disables interactive SSH trust confirmation.

Without this automation may stop at:

```text
Are you sure you want to continue connecting (yes/no)?
```

---

# Step 3 — Create Playbook

## Command

```bash
cat > /home/automation/ansible/playbook.yml << 'EOF'
---
- hosts: application_servers
  become: yes

  tasks:
    - name: Copy application file to servers
      copy:
        src: /usr/src/appdata/index.html
        dest: /opt/appdata/index.html
EOF
```

---

# Playbook Explanation

---

# hosts

```yaml
hosts: application_servers
```

Targets all servers inside the inventory group.

---

# become: yes

```yaml
become: yes
```

Enables privilege escalation.

Equivalent to:

```bash
sudo
```

Required because destination path may require elevated permissions.

---

# copy Module

```yaml
copy:
```

Used to transfer files from:
- Ansible control node

to:
- managed hosts

---

# src

```yaml
src: /usr/src/appdata/index.html
```

Source file on control node.

---

# dest

```yaml
dest: /opt/appdata/index.html
```

Destination file path on remote servers.

---

# Step 4 — Verify Inventory

## Command

```bash
cat /home/automation/ansible/inventory
```

---

# Output

```ini
[application_servers]
app-node-a ansible_user=deployuser ansible_password=SecurePass1 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
app-node-b ansible_user=deployuser ansible_password=SecurePass2 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
app-node-c ansible_user=deployuser ansible_password=SecurePass3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

# Step 5 — Verify Playbook

## Command

```bash
cat /home/automation/ansible/playbook.yml
```

---

# Output

```yaml
---
- hosts: application_servers
  become: yes

  tasks:
    - name: Copy application file to servers
      copy:
        src: /usr/src/appdata/index.html
        dest: /opt/appdata/index.html
```

---

# Step 6 — Move to Working Directory

## Command

```bash
cd /home/automation/ansible
```

---

# Why Needed

The validation command expects:
- inventory
- playbook.yml

inside current directory.

---

# Step 7 — Validate Connectivity

## Command

```bash
ansible -i inventory all -m ping
```

---

# Command Breakdown

| Component | Meaning |
|---|---|
| ansible | Ad-hoc Ansible command |
| -i inventory | Use custom inventory |
| all | Target all hosts |
| -m ping | Run ping module |

---

# Important Clarification

This is NOT ICMP ping.

Ansible ping:
- establishes SSH connection
- executes Python remotely
- returns pong

---

# Successful Output

```json
app-node-a | SUCCESS => {
    "ping": "pong"
}

app-node-b | SUCCESS => {
    "ping": "pong"
}

app-node-c | SUCCESS => {
    "ping": "pong"
}
```

---

# Step 8 — Execute Playbook

## Command

```bash
ansible-playbook -i inventory playbook.yml
```

---

# Successful Output

```bash
PLAY [application_servers] *****************************************************

TASK [Gathering Facts] *********************************************************
ok: [app-node-a]
ok: [app-node-b]
ok: [app-node-c]

TASK [Copy application file to servers] ****************************************
changed: [app-node-a]
changed: [app-node-b]
changed: [app-node-c]

PLAY RECAP *********************************************************************
app-node-a : ok=2 changed=1 unreachable=0 failed=0
app-node-b : ok=2 changed=1 unreachable=0 failed=0
app-node-c : ok=2 changed=1 unreachable=0 failed=0
```

---

# Output Analysis

| Metric | Meaning |
|---|---|
| ok=2 | Tasks executed successfully |
| changed=1 | File copied successfully |
| unreachable=0 | SSH successful |
| failed=0 | No failures |

---

# File Transfer Verification

The following file:

```bash
/usr/src/appdata/index.html
```

was copied to:

```bash
/opt/appdata/index.html
```

on:
- app-node-a
- app-node-b
- app-node-c

---

# Final Resolution Summary

| Problem | Resolution |
|---|---|
| Multiple server management needed | Inventory created |
| File distribution required | Ansible copy module used |
| Privileged directory write needed | become enabled |
| Automation validation required | Playbook executed successfully |

---

# Important DevOps Concepts Demonstrated

- Configuration Management
- Infrastructure as Code
- Remote Automation
- SSH Authentication
- Centralized Orchestration
- Multi-Host Automation
- Parallel Execution
- File Distribution Automation
- Privilege Escalation
- Idempotent Infrastructure

---

# Enterprise Best Practices

# Use SSH Keys Instead of Passwords

Preferred for:
- security
- scalability
- compliance

---

# Use Ansible Vault

Avoid plaintext passwords.

Encrypt:
- credentials
- secrets
- tokens

---

# Organize Inventories

Example:

```ini
[web]
web01

[db]
db01
```

---

# Use Roles

Large playbooks should be modularized using:
- roles
- tasks
- handlers
- templates

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is an Ansible inventory?

### Answer

An inventory defines:
- target servers
- groups
- SSH variables

used by Ansible.

---

## Q2. What does the copy module do?

### Answer

The copy module transfers files from:
- control node

to:
- remote managed nodes

---

## Q3. What is become: yes?

### Answer

It enables privilege escalation.

Equivalent to:

```bash
sudo
```

---

## Q4. What is the purpose of ansible ping?

### Answer

It validates:
- SSH connectivity
- Python execution
- Ansible communication

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. Difference between copy and synchronize modules?

### Answer

| copy | synchronize |
|---|---|
| Small/simple transfers | Large efficient transfers |
| Uses Ansible engine | Uses rsync |

---

## Q2. How would you secure inventory passwords?

### Answer

Using:
- Ansible Vault
- AWS Secrets Manager
- HashiCorp Vault

---

## Q3. Why use host groups?

### Answer

Host groups:
- simplify targeting
- improve scalability
- reduce duplication

---

## Q4. What is idempotency?

### Answer

Repeated runs produce same system state.

Example:
- existing file not recopied unnecessarily

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you scale Ansible for 1000+ servers?

### Answer

Using:
- dynamic inventory
- AWX/Tower
- execution nodes
- parallelism tuning
- reusable roles

---

## Q2. How would you integrate Ansible with CI/CD?

### Answer

Using:
- Jenkins
- GitHub Actions
- GitLab CI

Pipeline stages:
- lint
- test
- deploy
- validate

---

## Q3. How would you secure SSH automation?

### Answer

Using:
- SSH keys
- bastion hosts
- MFA
- Vault integration
- restricted sudo access

---

## Q4. How would you troubleshoot unreachable hosts?

### Answer

Check:
- DNS
- SSH service
- firewall
- routing
- credentials
- inventory syntax

---

# L4 — DevOps Architect

---

## Q1. Design enterprise-grade automation architecture.

### Answer

Components:
- Terraform
- Ansible
- Kubernetes
- Vault
- Jenkins
- GitOps

---

## Q2. How would you implement immutable infrastructure?

### Answer

Using:
- AMIs
- containers
- Kubernetes
- auto replacement instead of patching

---

## Q3. How would you secure enterprise automation pipelines?

### Answer

Using:
- RBAC
- MFA
- signed artifacts
- secret rotation
- audit logging
- policy enforcement

---

## Q4. How would you implement hybrid-cloud automation?

### Answer

Using:
- dynamic inventory
- Terraform
- cloud-native APIs
- centralized observability

---

# AWS Scenario-Based Interview Questions

---

# Q1. How would you automate EC2 configuration?

## Answer

Using:
- Ansible
- SSM
- Launch Templates
- User Data

---

# Q2. How would you distribute application files across EC2 instances?

## Answer

Using:
- Ansible copy/synchronize
- S3
- CodeDeploy
- EFS

---

# Q3. How would you avoid hardcoded passwords in AWS?

## Answer

Using:
- IAM roles
- Secrets Manager
- Parameter Store

---

# Q4. How would you troubleshoot EC2 SSH failures?

## Answer

Check:
- Security Groups
- Route Tables
- NACLs
- subnet routing
- SSH daemon
- key permissions

---

# Real Production Incident RCA Examples

# Incident 1 — Deployment Failure Due to SSH Timeout

## Symptoms

- Automation failed
- Hosts unreachable

---

## Root Cause

Firewall blocked port 22.

---

## Resolution

- Opened SSH rules
- Added connectivity validation pipeline

---

## Prevention

- Infrastructure testing
- Security validation
- Monitoring alerts

---

# Incident 2 — Wrong Inventory Target

## Symptoms

- Production modified accidentally

---

## Root Cause

Incorrect inventory group.

---

## Resolution

- Added environment separation
- Implemented approval workflow

---

## Prevention

- GitOps
- RBAC
- inventory validation

---

# Incident 3 — File Permission Failure

## Symptoms

- Copy module failed

---

## Root Cause

Destination directory required elevated permissions.

---

## Resolution

Used:

```yaml
become: yes
```

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between SCP and Rsync?

### Answer

| SCP | Rsync |
|---|---|
| Full transfer | Incremental transfer |
| Slower for large files | Faster efficient sync |

---

# Networking

## Q2. What happens during SSH handshake?

### Answer

- Key exchange
- Encryption negotiation
- Authentication
- Session establishment

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
| Default command | Main executable |

---

# Kubernetes

## Q5. Difference between ConfigMap and Secret?

### Answer

| ConfigMap | Secret |
|---|---|
| Plain configuration | Sensitive data |

---

# Terraform

## Q6. What is Terraform state?

### Answer

Tracks infrastructure resources and mappings.

---

# Jenkins

## Q7. What is Jenkins pipeline?

### Answer

Pipeline automates:
- build
- test
- deploy
- validation

---

# Monitoring

## Q8. Difference between monitoring and observability?

### Answer

| Monitoring | Observability |
|---|---|
| Detect issues | Understand root cause |

---

# CI/CD

## Q9. Blue-Green vs Canary deployment?

### Answer

| Blue-Green | Canary |
|---|---|
| Full switch | Gradual rollout |

---

# Final Technical Learnings

This task demonstrated:

- Centralized automation
- Multi-host orchestration
- SSH automation
- Secure infrastructure operations
- Configuration management
- Parallel task execution
- Infrastructure reliability
- Enterprise automation patterns

---

# Final Outcome

✅ Inventory created successfully  
✅ All application servers added  
✅ Playbook created successfully  
✅ File copied to all managed nodes  
✅ SSH connectivity validated  
✅ Automation executed successfully  
✅ Validation command passed  
✅ No unreachable hosts  
✅ No task failures

