# TaskAndResolution.md

# Ansible HTTPD Package Installation & Service Management Automation

---

# Project Overview

A DevOps engineering team needed to automate package installation and service management across multiple Linux application servers using Ansible.

The task involved:

- Installing Apache HTTPD web server
- Managing Linux services
- Validating infrastructure connectivity
- Ensuring service persistence after reboot
- Testing automation idempotency
- Managing centralized infrastructure using Ansible

The deployment was fully automated from a centralized automation controller node using SSH-based orchestration.

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Automation Controller | control-node |
| Application Server 1 | app-server-alpha |
| Application Server 2 | app-server-beta |
| Application Server 3 | app-server-gamma |
| Automation User | deploy-user |
| Inventory Path | /home/deploy-user/ansible/inventory |
| Playbook Path | /home/deploy-user/ansible/playbook.yml |
| Web Package | httpd |
| Automation Tool | Ansible |

---

# Objective

## Requirements

1. Use existing inventory:

```bash
/home/deploy-user/ansible/inventory
```

2. Create playbook:

```bash
/home/deploy-user/ansible/playbook.yml
```

3. Install Apache HTTPD on all application servers.

4. Start HTTPD service.

5. Enable HTTPD service during system boot.

6. Ensure automation runs successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```

without extra arguments.

---

# Step 1 — Navigate to Ansible Directory

## Command

```bash
cd /home/deploy-user/ansible
```

---

# Why Needed

The directory contains:
- inventory
- ansible.cfg
- playbooks
- automation assets

---

# Step 2 — Verify Existing Inventory

## Command

```bash
cat inventory
```

---

# Existing Inventory

```ini
app-server-alpha ansible_host=app-server-alpha ansible_ssh_pass=SecurePass1 ansible_user=deploy-alpha

app-server-beta ansible_host=app-server-beta ansible_ssh_pass=SecurePass2 ansible_user=deploy-beta

app-server-gamma ansible_host=app-server-gamma ansible_ssh_pass=SecurePass3 ansible_user=deploy-gamma
```

---

# Inventory File Explanation

---

# Host Entry Example

```ini
app-server-alpha ansible_host=app-server-alpha
```

---

# Parameter Breakdown

| Parameter | Meaning |
|---|---|
| app-server-alpha | Inventory hostname |
| ansible_host | Actual target hostname/IP |
| ansible_user | SSH login username |
| ansible_ssh_pass | SSH password |

---

# Step 3 — Create Playbook

## Command

```bash
cat > /home/deploy-user/ansible/playbook.yml << 'EOF'
---
- hosts: all
  become: yes

  tasks:

    - name: Install httpd package
      yum:
        name: httpd
        state: present

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes
EOF
```

---

# Playbook Explanation

---

# hosts

```yaml
hosts: all
```

Targets all hosts defined in inventory.

---

# become: yes

```yaml
become: yes
```

Enables privilege escalation.

Equivalent Linux command:

```bash
sudo
```

Required because:
- package installation
- service management

need administrative privileges.

---

# yum Module

```yaml
yum:
```

Used for package management on:
- RHEL
- CentOS
- Rocky Linux
- AlmaLinux

---

# HTTPD Package Installation

```yaml
name: httpd
state: present
```

Meaning:
- install package if missing
- do nothing if already installed

This behavior makes playbook idempotent.

---

# service Module

```yaml
service:
```

Used for:
- starting services
- stopping services
- restarting services
- enabling boot persistence

---

# HTTPD Service Configuration

```yaml
state: started
enabled: yes
```

Meaning:

| Parameter | Purpose |
|---|---|
| started | Ensure service running |
| enabled | Start automatically after reboot |

---

# Step 4 — Verify Playbook

## Command

```bash
cat /home/deploy-user/ansible/playbook.yml
```

---

# Output

```yaml
---
- hosts: all
  become: yes

  tasks:

    - name: Install httpd package
      yum:
        name: httpd
        state: present

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes
```

---

# Step 5 — Validate Ansible Connectivity

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
- validates automation communication

---

# Step 6 — Execute Playbook

## Command

```bash
ansible-playbook -i inventory playbook.yml
```

---

# Successful Output

```bash
PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [app-server-alpha]
ok: [app-server-beta]
ok: [app-server-gamma]

TASK [Install httpd package] ***************************************************
changed: [app-server-alpha]
changed: [app-server-beta]
changed: [app-server-gamma]

TASK [Start and enable httpd service] ******************************************
changed: [app-server-alpha]
changed: [app-server-beta]
changed: [app-server-gamma]

PLAY RECAP *********************************************************************
app-server-alpha : ok=3 changed=2 unreachable=0 failed=0
app-server-beta  : ok=3 changed=2 unreachable=0 failed=0
app-server-gamma : ok=3 changed=2 unreachable=0 failed=0
```

---

# Output Analysis

| Metric | Meaning |
|---|---|
| ok=3 | Tasks executed successfully |
| changed=2 | Package installed and service started |
| unreachable=0 | SSH successful |
| failed=0 | No task failures |

---

# Step 7 — Validate HTTPD Service Running

## Command

```bash
ansible -i inventory all -a "systemctl is-active httpd"
```

---

# Output

```bash
active
```

on all servers.

---

# Step 8 — Validate HTTPD Boot Persistence

## Command

```bash
ansible -i inventory all -a "systemctl is-enabled httpd"
```

---

# Output

```bash
enabled
```

on all servers.

---

# Step 9 — Validate HTTPD Package Installation

## Command

```bash
ansible -i inventory all -a "rpm -q httpd"
```

---

# Output

```bash
httpd-2.4.x
```

on all servers.

---

# Step 10 — Validate Idempotency

## Command

```bash
ansible-playbook -i inventory playbook.yml
```

---

# Second Execution Output

```bash
changed=0
```

---

# Why This Is Important

This confirms:
- infrastructure already in desired state
- playbook is idempotent
- automation written correctly

Idempotency is a core DevOps principle.

---

# Inventory Cleanup Recommendation

The inventory file contained malformed last line:

Incorrect:

```ini
app-server-gamma ansible_host=app-server-gamma ansible_ssh_pass=SecurePass3 ansible_user=deploy-gammacontrol-node
```

Correct version:

```ini
app-server-gamma ansible_host=app-server-gamma ansible_ssh_pass=SecurePass3 ansible_user=deploy-gamma
```

Even though automation worked successfully, production inventory files should always remain clean.

---

# Final Technical Resolution

| Problem | Resolution |
|---|---|
| Package installation needed | yum module used |
| Service management required | service module used |
| Boot persistence needed | enabled=yes configured |
| Multi-host deployment required | inventory used |
| Validation required | Ansible verification completed |

---

# Important DevOps Concepts Demonstrated

- Infrastructure as Code
- Configuration Management
- Linux Package Management
- Linux Service Management
- Multi-Host Automation
- SSH-Based Orchestration
- Idempotent Automation
- Infrastructure Validation
- Enterprise Automation Practices
- Centralized Orchestration

---

# Enterprise Best Practices

# Use SSH Keys Instead of Passwords

Preferred because:
- more secure
- automation friendly
- scalable

---

# Use Ansible Vault

Avoid storing plaintext passwords in inventory.

---

# Use Roles

Large playbooks should use:
- roles
- handlers
- variables
- templates

---

# Use Dynamic Inventory

Cloud environments should use:
- AWS inventory plugin
- Azure inventory plugin
- GCP inventory plugin

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is Ansible?

### Answer

Ansible is an agentless automation tool used for:
- configuration management
- application deployment
- orchestration

---

## Q2. What does the yum module do?

### Answer

Installs and manages packages on RPM-based Linux systems.

---

## Q3. What does become: yes mean?

### Answer

Enables privilege escalation.

Equivalent to:

```bash
sudo
```

---

## Q4. What is service module used for?

### Answer

Used to:
- start services
- stop services
- restart services
- enable services

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. What is idempotency?

### Answer

Repeated execution produces same infrastructure state.

---

## Q2. Difference between state=started and enabled=yes?

### Answer

| started | enabled |
|---|---|
| Run service now | Start service after reboot |

---

## Q3. Why validate service using systemctl?

### Answer

To confirm:
- service running
- automation successful

---

## Q4. How would you secure Ansible inventory?

### Answer

Using:
- Ansible Vault
- external secrets managers

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you scale Ansible infrastructure?

### Answer

Using:
- AWX/Tower
- dynamic inventory
- reusable roles
- CI/CD integration

---

## Q2. How would you troubleshoot failed package installation?

### Answer

Check:
- repository availability
- network connectivity
- DNS
- package conflicts
- permissions

---

## Q3. How would you implement zero-downtime deployments?

### Answer

Using:
- rolling deployments
- canary releases
- load balancers

---

## Q4. How would you secure automation pipelines?

### Answer

Using:
- RBAC
- Vault
- MFA
- signed artifacts

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

## Q2. How would you implement immutable infrastructure?

### Answer

Using:
- containers
- AMIs
- auto replacement

---

## Q3. How would you implement hybrid-cloud automation?

### Answer

Using:
- dynamic inventory
- cloud-native APIs
- centralized orchestration

---

## Q4. How would you enforce compliance automation?

### Answer

Using:
- policy-as-code
- CIS hardening
- audit logging

---

# AWS Scenario-Based Questions

---

# Q1. How would you automate Apache deployment on EC2?

## Answer

Using:
- EC2
- Ansible
- Auto Scaling Groups
- Launch Templates

---

## Q2. How would you secure EC2 SSH access?

### Answer

Using:
- Security Groups
- bastion hosts
- MFA
- Session Manager

---

## Q3. How would you troubleshoot EC2 service failures?

### Answer

Check:
- Security Groups
- Route Tables
- service logs
- systemd status

---

## Q4. How would you implement blue-green deployment in AWS?

### Answer

Using:
- ALB
- Auto Scaling
- Route53
- CodeDeploy

---

# Real Production Incident RCA

# Incident — Web Application Downtime After Reboot

## Symptoms

- Website inaccessible after server restart
- Apache service inactive

---

## Root Cause

Service was started manually but not enabled during boot.

---

## Resolution

Implemented:

```yaml
enabled: yes
```

inside automation playbook.

---

## Prevention

- infrastructure validation
- boot persistence checks
- monitoring alerts

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between yum and rpm?

### Answer

| yum | rpm |
|---|---|
| Dependency resolution | Low-level package tool |

---

# Networking

## Q2. Difference between HTTP and HTTPS?

### Answer

| HTTP | HTTPS |
|---|---|
| Unencrypted | Encrypted |

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

- Centralized automation
- Package management automation
- Linux service management
- Infrastructure validation
- Multi-host orchestration
- SSH automation
- Idempotent infrastructure
- Enterprise automation patterns

---

# Final Outcome

✅ HTTPD installed successfully  
✅ HTTPD service started successfully  
✅ HTTPD enabled during boot  
✅ All application servers reachable  
✅ Ansible playbook executed successfully  
✅ Service validation successful  
✅ Package validation successful  
✅ Idempotency verified successfully  
✅ No unreachable hosts  
✅ No task failures  
✅ Infrastructure automation completed successfully

