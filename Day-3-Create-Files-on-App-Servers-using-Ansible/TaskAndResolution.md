# TaskAndResolution.md

# Ansible File Management Automation Across Multiple Application Servers

---

# Project Overview

A DevOps engineering team needed to automate file creation and permission management across multiple Linux application servers using Ansible.

The automation requirements included:

- Managing multiple servers centrally
- Creating files remotely
- Assigning specific permissions
- Configuring server-specific ownership
- Validating remote execution using Ansible

The solution used:
- static inventory
- Ansible file module
- conditional task execution
- privilege escalation

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Automation Controller | central-node |
| Application Server 1 | app-server-alpha |
| Application Server 2 | app-server-beta |
| Application Server 3 | app-server-gamma |
| Automation Directory | /home/automation/playbook |
| File To Create | /usr/src/webdata.txt |
| Automation Tool | Ansible |
| Remote Users | deploy-alpha / deploy-beta / deploy-gamma |

---

# Objective

## Requirements

1. Create inventory file:

```bash
~/playbook/inventory
```

2. Add all application servers.

3. Create playbook:

```bash
~/playbook/playbook.yml
```

4. Create blank file:

```bash
/usr/src/webdata.txt
```

on all servers.

5. Set:
- permission = 0755
- correct owner/group per server

---

# Step 1 — Create Working Directory

## Command

```bash
mkdir -p ~/playbook
```

---

# Command Explanation

| Component | Meaning |
|---|---|
| mkdir | Create directory |
| -p | Create parent directories if missing |
| ~/playbook | Target directory |

---

# Output

```bash
(no output)
```

---

# Why Needed

This directory stores:
- inventory file
- playbook
- automation assets

---

# Step 2 — Create Inventory File

## Command

```bash
cat > ~/playbook/inventory << 'EOF'
[application_servers]
app-server-alpha ansible_user=deploy-alpha ansible_password=SecurePass1 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-beta ansible_user=deploy-beta ansible_password=SecurePass2 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-gamma ansible_user=deploy-gamma ansible_password=SecurePass3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

---

# Inventory File Explanation

## Group Definition

```ini
[application_servers]
```

Defines a logical group of hosts.

---

# Host Entries

Example:

```ini
app-server-alpha ansible_user=deploy-alpha ansible_password=SecurePass1
```

---

# Parameter Breakdown

| Parameter | Purpose |
|---|---|
| app-server-alpha | Inventory hostname |
| ansible_user | SSH username |
| ansible_password | SSH password |
| ansible_ssh_common_args | Additional SSH options |

---

# SSH Option Explanation

```bash
-o StrictHostKeyChecking=no
```

Disables interactive SSH host verification.

Without this automation may stop at:

```text
Are you sure you want to continue connecting (yes/no)?
```

---

# Step 3 — Create Playbook

## Command

```bash
cat > ~/playbook/playbook.yml << 'EOF'
---
- hosts: application_servers
  become: yes

  tasks:

    - name: Create file on Application Server Alpha
      file:
        path: /usr/src/webdata.txt
        state: touch
        mode: '0755'
        owner: deploy-alpha
        group: deploy-alpha
      when: inventory_hostname == "app-server-alpha"

    - name: Create file on Application Server Beta
      file:
        path: /usr/src/webdata.txt
        state: touch
        mode: '0755'
        owner: deploy-beta
        group: deploy-beta
      when: inventory_hostname == "app-server-beta"

    - name: Create file on Application Server Gamma
      file:
        path: /usr/src/webdata.txt
        state: touch
        mode: '0755'
        owner: deploy-gamma
        group: deploy-gamma
      when: inventory_hostname == "app-server-gamma"
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

Required because `/usr/src` typically needs elevated permissions.

---

# file Module

```yaml
file:
```

Used for:
- file creation
- permission management
- ownership management

---

# state: touch

```yaml
state: touch
```

Creates file if missing.

Equivalent to:

```bash
touch /usr/src/webdata.txt
```

---

# mode: '0755'

```yaml
mode: '0755'
```

Sets permissions:

| Permission | Meaning |
|---|---|
| Owner | rwx |
| Group | r-x |
| Others | r-x |

---

# owner / group

```yaml
owner: deploy-alpha
group: deploy-alpha
```

Assigns ownership.

---

# when Condition

```yaml
when: inventory_hostname == "app-server-alpha"
```

Executes task only on matching host.

---

# Step 4 — Verify Inventory

## Command

```bash
cat ~/playbook/inventory
```

---

# Output

```ini
[application_servers]
app-server-alpha ansible_user=deploy-alpha ansible_password=SecurePass1 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-beta ansible_user=deploy-beta ansible_password=SecurePass2 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-gamma ansible_user=deploy-gamma ansible_password=SecurePass3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

# Step 5 — Verify Playbook

## Command

```bash
cat ~/playbook/playbook.yml
```

---

# Output

```yaml
---
- hosts: application_servers
  become: yes

  tasks:

    - name: Create file on Application Server Alpha
      file:
        path: /usr/src/webdata.txt
        state: touch
        mode: '0755'
        owner: deploy-alpha
        group: deploy-alpha
      when: inventory_hostname == "app-server-alpha"

    - name: Create file on Application Server Beta
      file:
        path: /usr/src/webdata.txt
        state: touch
        mode: '0755'
        owner: deploy-beta
        group: deploy-beta
      when: inventory_hostname == "app-server-beta"

    - name: Create file on Application Server Gamma
      file:
        path: /usr/src/webdata.txt
        state: touch
        mode: '0755'
        owner: deploy-gamma
        group: deploy-gamma
      when: inventory_hostname == "app-server-gamma"
```

---

# Step 6 — Change Directory

## Command

```bash
cd ~/playbook
```

---

# Why Needed

Validation command expects:
- inventory
- playbook.yml

inside current working directory.

---

# Step 7 — Validate Connectivity

## Command

```bash
ansible -i inventory all -m ping
```

---

# Successful Output

```json
app-server-alpha | SUCCESS => {
    "ping": "pong"
}

app-server-beta | SUCCESS => {
    "ping": "pong"
}

app-server-gamma | SUCCESS => {
    "ping": "pong"
}
```

---

# Important Clarification

This is NOT ICMP ping.

Ansible ping:
- establishes SSH connection
- executes Python remotely
- validates Ansible communication

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
ok: [app-server-alpha]
ok: [app-server-beta]
ok: [app-server-gamma]

TASK [Create file on Application Server Alpha] *********************************
changed: [app-server-alpha]
skipping: [app-server-beta]
skipping: [app-server-gamma]

TASK [Create file on Application Server Beta] **********************************
changed: [app-server-beta]
skipping: [app-server-alpha]
skipping: [app-server-gamma]

TASK [Create file on Application Server Gamma] *********************************
changed: [app-server-gamma]
skipping: [app-server-alpha]
skipping: [app-server-beta]

PLAY RECAP *********************************************************************
app-server-alpha : ok=2 changed=1 unreachable=0 failed=0 skipped=2
app-server-beta  : ok=2 changed=1 unreachable=0 failed=0 skipped=2
app-server-gamma : ok=2 changed=1 unreachable=0 failed=0 skipped=2
```

---

# Output Analysis

| Metric | Meaning |
|---|---|
| ok=2 | Tasks executed successfully |
| changed=1 | File created/updated |
| unreachable=0 | SSH successful |
| failed=0 | No failures |
| skipped=2 | Conditional tasks skipped intentionally |

---

# Final Result

Created:

```bash
/usr/src/webdata.txt
```

on:
- app-server-alpha
- app-server-beta
- app-server-gamma

with:
- permission `0755`
- correct owner/group
- correct conditional execution

---

# Important DevOps Concepts Demonstrated

- Configuration Management
- Infrastructure as Code
- Multi-Host Automation
- Conditional Task Execution
- SSH Automation
- Remote File Management
- Permission Management
- Linux Ownership Management
- Privilege Escalation
- Idempotent Automation

---

# Enterprise Best Practices

# Use Ansible Vault

Avoid storing passwords in plaintext.

---

# Use SSH Keys

Better than password authentication.

---

# Use Variables

Move credentials into:
- group_vars
- host_vars
- encrypted vaults

---

# Use Roles

Large playbooks should use:
- roles
- handlers
- templates
- reusable modules

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is an Ansible inventory?

### Answer

An inventory defines:
- target servers
- groups
- connection variables

used by Ansible.

---

## Q2. What does state: touch do?

### Answer

Creates file if it does not exist.

Equivalent to Linux:

```bash
touch filename
```

---

## Q3. What is become: yes?

### Answer

Used for privilege escalation.

Equivalent to:

```bash
sudo
```

---

## Q4. Why were skipped tasks shown?

### Answer

Because `when` conditions executed tasks only on matching hosts.

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. Difference between file and copy module?

### Answer

| file | copy |
|---|---|
| Creates/manages files | Transfers files |

---

## Q2. What is idempotency?

### Answer

Repeated runs produce same system state.

---

## Q3. Why use conditional execution?

### Answer

To apply host-specific configuration.

---

## Q4. How would you secure credentials?

### Answer

Using:
- Ansible Vault
- AWS Secrets Manager
- HashiCorp Vault

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you scale Ansible for enterprise usage?

### Answer

Using:
- AWX/Tower
- dynamic inventory
- CI/CD integration
- execution nodes
- reusable roles

---

## Q2. How would you improve this playbook?

### Answer

Using:
- variables
- loops
- templates
- group_vars
- roles

---

## Q3. How would you troubleshoot unreachable hosts?

### Answer

Check:
- SSH service
- DNS
- routing
- firewall
- credentials
- inventory syntax

---

## Q4. How would you integrate Ansible into CI/CD?

### Answer

Using:
- Jenkins
- GitHub Actions
- GitLab CI

---

# L4 — DevOps Architect

---

## Q1. Design enterprise automation architecture.

### Answer

Components:
- Terraform
- Ansible
- Kubernetes
- Vault
- GitOps
- centralized observability

---

## Q2. How would you secure automation pipelines?

### Answer

Using:
- RBAC
- MFA
- audit logging
- secret rotation
- policy enforcement

---

## Q3. How would you implement immutable infrastructure?

### Answer

Using:
- containers
- AMIs
- auto replacement
- GitOps deployment

---

## Q4. How would you manage hybrid-cloud automation?

### Answer

Using:
- dynamic inventory
- cloud APIs
- centralized orchestration

---

# AWS Scenario-Based Questions

---

# Q1. How would you automate EC2 configuration?

## Answer

Using:
- Ansible
- SSM
- Launch Templates
- User Data

---

# Q2. How would you secure EC2 automation?

## Answer

Using:
- IAM roles
- Secrets Manager
- private networking

---

# Q3. How would you troubleshoot EC2 SSH issues?

## Answer

Check:
- Security Groups
- NACLs
- routing
- SSH daemon

---

# Real Production Incident RCA

# Incident — Incorrect File Ownership

## Symptoms

Application failed after deployment.

---

## Root Cause

File owner mismatch caused permission denial.

---

## Resolution

Implemented:
- explicit owner/group
- automated permission management

---

## Prevention

- CI validation
- configuration testing
- policy checks

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between chmod 755 and 644?

### Answer

| 755 | 644 |
|---|---|
| Executable | Non-executable |

---

# Networking

## Q2. Difference between TCP and UDP?

### Answer

| TCP | UDP |
|---|---|
| Reliable | Faster |

---

# AWS

## Q3. Difference between Security Group and NACL?

### Answer

| Security Group | NACL |
|---|---|
| Stateful | Stateless |

---

# Docker

## Q4. Difference between CMD and ENTRYPOINT?

### Answer

| CMD | Default command |
| ENTRYPOINT | Main executable |

---

# Kubernetes

## Q5. Difference between Deployment and StatefulSet?

### Answer

| Deployment | StatefulSet |
|---|---|
| Stateless | Stateful |

---

# Terraform

## Q6. What is Terraform state?

### Answer

Tracks infrastructure resources and mappings.

---

# Jenkins

## Q7. What is Jenkins pipeline?

### Answer

Automates:
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
| Detect issue | Understand root cause |

---

# Final Technical Learnings

This task demonstrated:

- Multi-host orchestration
- Centralized configuration management
- Conditional automation
- Linux permission management
- Secure remote execution
- Infrastructure consistency
- Enterprise automation patterns

---

# Final Outcome

✅ Inventory created successfully  
✅ All servers added successfully  
✅ Playbook executed successfully  
✅ File created on all servers  
✅ Permissions configured correctly  
✅ Ownership configured correctly  
✅ SSH connectivity validated  
✅ Validation command passed successfully  
✅ No unreachable hosts  
✅ No failures detected

