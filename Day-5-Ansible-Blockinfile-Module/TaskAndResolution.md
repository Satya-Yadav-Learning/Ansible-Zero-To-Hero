# TaskAndResolution.md

# Ansible HTTPD Web Server Deployment & Web Page Automation

---

# Project Overview

A DevOps engineering team needed to automate deployment of Apache HTTPD web servers across multiple Linux application servers using Ansible.

The task included:

- Installing HTTPD packages
- Starting and enabling services
- Deploying a sample web page
- Managing Linux ownership and permissions
- Validating web accessibility
- Ensuring idempotent infrastructure automation

The deployment was fully automated using:
- Ansible inventory
- playbooks
- yum module
- service module
- blockinfile module
- file module

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Automation Controller | automation-node |
| Web Server 1 | web-node-alpha |
| Web Server 2 | web-node-beta |
| Web Server 3 | web-node-gamma |
| Automation User | deploy-user |
| Inventory Path | /home/deploy-user/ansible/inventory |
| Playbook Path | /home/deploy-user/ansible/playbook.yml |
| Web Root | /var/www/html |
| Web Server Package | httpd |
| Automation Tool | Ansible |

---

# Objective

## Requirements

1. Use existing inventory file:

```bash
/home/deploy-user/ansible/inventory
```

2. Create playbook:

```bash
/home/deploy-user/ansible/playbook.yml
```

3. Install Apache HTTPD on all servers.

4. Start and enable HTTPD service.

5. Deploy sample web content using:
- `blockinfile`

6. Configure:
- owner = apache
- group = apache
- permission = 0655

7. Ensure playbook runs successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```

without additional arguments.

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

---

# Step 2 — Verify Existing Inventory

## Command

```bash
cat inventory
```

---

# Existing Inventory

```ini
web-node-alpha ansible_host=web-node-alpha ansible_ssh_pass=SecurePass1 ansible_user=deploy-alpha

web-node-beta ansible_host=web-node-beta ansible_ssh_pass=SecurePass2 ansible_user=deploy-beta

web-node-gamma ansible_host=web-node-gamma ansible_ssh_pass=SecurePass3 ansible_user=deploy-gamma
```

---

# Inventory File Explanation

---

# Host Entry Example

```ini
web-node-alpha ansible_host=web-node-alpha
```

---

# Parameter Breakdown

| Parameter | Meaning |
|---|---|
| web-node-alpha | Inventory hostname |
| ansible_host | Actual target hostname/IP |
| ansible_user | SSH login username |
| ansible_ssh_pass | SSH password |

---

# Step 3 — Create Ansible Playbook

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

    - name: Add content to index.html
      blockinfile:
        path: /var/www/html/index.html
        create: yes
        block: |
          Welcome to XfusionCorp!

          This is Nautilus sample file, created using Ansible!

          Please do not modify this file manually!

    - name: Set ownership on index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache

    - name: Set permissions on index.html
      file:
        path: /var/www/html/index.html
        mode: '0655'
EOF
```

---

# Playbook Explanation

---

# hosts

```yaml
hosts: all
```

Targets all hosts from inventory.

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

Required because:
- package installation
- service management
- system file modification

need elevated privileges.

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

# HTTPD Installation

```yaml
name: httpd
state: present
```

Ensures:
- package installed
- idempotent execution

---

# service Module

```yaml
service:
```

Used for:
- starting services
- enabling boot persistence

---

# HTTPD Service Configuration

```yaml
state: started
enabled: yes
```

Meaning:
- service running now
- auto-start enabled on reboot

---

# blockinfile Module

```yaml
blockinfile:
```

Used for safely inserting managed content into files.

---

# Why blockinfile Was Required

Task explicitly required:
- blockinfile module
- default markers
- no custom markers

---

# block Content

```yaml
block: |
```

Adds multiline text.

---

# create: yes

```yaml
create: yes
```

Creates file if missing.

---

# file Module

```yaml
file:
```

Used for:
- ownership
- permissions
- file attributes

---

# Ownership Configuration

```yaml
owner: apache
group: apache
```

Assigns:
- file owner
- file group

---

# Permission Configuration

```yaml
mode: '0655'
```

Equivalent Linux permissions:

| Entity | Permission |
|---|---|
| Owner | rw- |
| Group | r-x |
| Others | r-x |

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

    - name: Add content to index.html
      blockinfile:
        path: /var/www/html/index.html
        create: yes
        block: |
          Welcome to XfusionCorp!

          This is Nautilus sample file, created using Ansible!

          Please do not modify this file manually!

    - name: Set ownership on index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache

    - name: Set permissions on index.html
      file:
        path: /var/www/html/index.html
        mode: '0655'
```

---

# Step 5 — Validate Connectivity

## Command

```bash
ansible -i inventory all -m ping
```

---

# Successful Output

```json
web-node-alpha | SUCCESS => {
    "ping": "pong"
}

web-node-beta | SUCCESS => {
    "ping": "pong"
}

web-node-gamma | SUCCESS => {
    "ping": "pong"
}
```

---

# Important Clarification

Ansible ping:
- uses SSH
- executes Python remotely
- validates automation communication

This is NOT ICMP ping.

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
ok: [web-node-alpha]
ok: [web-node-beta]
ok: [web-node-gamma]

TASK [Install httpd package] ***************************************************
changed: [web-node-alpha]
changed: [web-node-beta]
changed: [web-node-gamma]

TASK [Start and enable httpd service] ******************************************
changed: [web-node-alpha]
changed: [web-node-beta]
changed: [web-node-gamma]

TASK [Add content to index.html] ***********************************************
changed: [web-node-alpha]
changed: [web-node-beta]
changed: [web-node-gamma]

TASK [Set ownership on index.html] *********************************************
changed: [web-node-alpha]
changed: [web-node-beta]
changed: [web-node-gamma]

TASK [Set permissions on index.html] *******************************************
changed: [web-node-alpha]
changed: [web-node-beta]
changed: [web-node-gamma]

PLAY RECAP *********************************************************************
web-node-alpha : ok=6 changed=5 unreachable=0 failed=0
web-node-beta  : ok=6 changed=5 unreachable=0 failed=0
web-node-gamma : ok=6 changed=5 unreachable=0 failed=0
```

---

# Step 7 — Verify HTTPD Package

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

# Step 8 — Verify HTTPD Service Running

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

# Step 9 — Verify Web Content

## Command

```bash
ansible -i inventory all -a "cat /var/www/html/index.html"
```

---

# Output

```text
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!

This is Nautilus sample file, created using Ansible!

Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
```

---

# Important Note About blockinfile

Default markers are expected because:
- task prohibited custom markers
- task prohibited empty markers

So these markers are correct:

```text
BEGIN ANSIBLE MANAGED BLOCK
END ANSIBLE MANAGED BLOCK
```

---

# Step 10 — Verify Ownership

## Command

```bash
ansible -i inventory all -a "ls -l /var/www/html/index.html"
```

---

# Output

```bash
-rw-r-xr-x 1 apache apache
```

---

# Step 11 — Verify Numeric Permissions

## Command

```bash
ansible -i inventory all -a "stat -c '%a %U %G' /var/www/html/index.html"
```

---

# Output

```bash
655 apache apache
```

---

# Step 12 — Verify HTTP Access

## Commands

```bash
curl http://web-node-alpha
```

```bash
curl http://web-node-beta
```

```bash
curl http://web-node-gamma
```

---

# Output

```text
Welcome to XfusionCorp!

This is Nautilus sample file, created using Ansible!

Please do not modify this file manually!
```

---

# Step 13 — Validate Idempotency

## Command

```bash
ansible-playbook -i inventory playbook.yml
```

---

# Final Output

```bash
changed=0
```

---

# Why This Matters

This proves:
- infrastructure already in desired state
- playbook is idempotent
- automation written correctly

Idempotency is a core DevOps principle.

---

# Final Technical Resolution

| Problem | Resolution |
|---|---|
| HTTPD deployment needed | yum module used |
| Service management required | service module used |
| Web page deployment required | blockinfile used |
| Ownership management needed | file module used |
| Permission management required | mode configured |
| Automation validation required | Ansible verification completed |

---

# Important DevOps Concepts Demonstrated

- Infrastructure as Code
- Configuration Management
- Web Server Automation
- Linux Service Management
- Multi-Host Automation
- SSH-Based Orchestration
- Idempotent Playbooks
- File Ownership Management
- Permission Management
- Infrastructure Verification

---

# Enterprise Best Practices

# Use Ansible Roles

Large deployments should use:
- roles
- handlers
- templates
- reusable variables

---

# Use Ansible Vault

Never store plaintext passwords in inventory.

---

# Use Dynamic Inventory

Production cloud environments should use:
- AWS dynamic inventory
- Azure inventory
- GCP inventory

---

# Use Templates

Prefer:
- Jinja2 templates

instead of static content for scalable deployments.

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is Ansible?

### Answer

Ansible is an agentless automation tool used for:
- configuration management
- deployment
- orchestration

---

## Q2. What does the yum module do?

### Answer

Installs/manages packages on RPM-based Linux systems.

---

## Q3. What does become: yes mean?

### Answer

Enables privilege escalation.

Equivalent to:

```bash
sudo
```

---

## Q4. What is blockinfile used for?

### Answer

Safely inserts managed blocks into files.

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. Difference between copy and blockinfile?

### Answer

| copy | blockinfile |
|---|---|
| Replaces file | Inserts managed block |

---

## Q2. What is idempotency?

### Answer

Repeated runs produce same infrastructure state.

---

## Q3. Why verify HTTP using curl?

### Answer

Confirms:
- service operational
- web page accessible

---

## Q4. Why use file module separately?

### Answer

To explicitly manage:
- ownership
- permissions

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you scale this deployment?

### Answer

Using:
- roles
- templates
- dynamic inventory
- AWX/Tower
- CI/CD integration

---

## Q2. How would you secure automation?

### Answer

Using:
- Vault
- SSH keys
- RBAC
- MFA
- secrets management

---

## Q3. How would you implement zero-downtime deployments?

### Answer

Using:
- load balancers
- rolling updates
- canary deployments

---

## Q4. How would you troubleshoot HTTPD failures?

### Answer

Check:
- service status
- firewall
- SELinux
- logs
- permissions
- syntax errors

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

## Q2. How would you secure enterprise web infrastructure?

### Answer

Using:
- WAF
- reverse proxies
- TLS everywhere
- IAM
- RBAC
- immutable infrastructure

---

## Q3. How would you implement compliance automation?

### Answer

Using:
- policy-as-code
- CIS hardening
- automated auditing

---

## Q4. How would you automate hybrid-cloud deployments?

### Answer

Using:
- dynamic inventory
- cloud-native APIs
- centralized orchestration

---

# AWS Scenario-Based Questions

---

# Q1. How would you automate Apache deployment in AWS?

## Answer

Using:
- EC2
- Ansible
- Auto Scaling
- Launch Templates
- ALB

---

# Q2. How would you distribute content across EC2 instances?

## Answer

Using:
- Ansible
- S3
- CodeDeploy
- EFS

---

## Q3. How would you secure EC2 web servers?

### Answer

Using:
- Security Groups
- private subnets
- WAF
- IAM roles

---

## Q4. How would you troubleshoot EC2 HTTP issues?

### Answer

Check:
- Security Groups
- Route Tables
- NACLs
- Apache service
- firewall
- DNS

---

# Real Production Incident RCA

# Incident — Web Service Outage After Deployment

## Symptoms

- Website inaccessible
- HTTP 503 errors

---

## Root Cause

Apache service not enabled after reboot.

---

## Resolution

Implemented:

```yaml
enabled: yes
```

inside Ansible playbook.

---

## Prevention

- infrastructure testing
- health checks
- monitoring alerts

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between chmod 755 and 655?

### Answer

| 755 | 655 |
|---|---|
| Executable | Non-executable |

---

# Networking

## Q2. Difference between HTTP and HTTPS?

### Answer

| HTTP | HTTPS |
|---|---|
| Plaintext | Encrypted |

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

- End-to-end web server automation
- Multi-host orchestration
- Linux service management
- Secure infrastructure deployment
- Web application deployment
- Infrastructure validation
- Idempotent automation
- Enterprise automation patterns

---

# Final Outcome

✅ HTTPD installed successfully  
✅ HTTPD service running successfully  
✅ HTTPD enabled at boot  
✅ Web page deployed successfully  
✅ blockinfile module used correctly  
✅ Ownership configured correctly  
✅ Permissions configured correctly  
✅ HTTP access verified successfully  
✅ Idempotency validated successfully  
✅ No unreachable hosts  
✅ No task failures  
✅ Infrastructure automation completed successfully

