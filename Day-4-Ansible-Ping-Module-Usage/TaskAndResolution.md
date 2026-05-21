# TaskAndResolution.md

# Ansible Password-less SSH Configuration & Connectivity Validation

---

# Project Overview

A DevOps engineering team needed to establish secure password-less SSH authentication between an Ansible Controller Node and managed Linux application servers before executing infrastructure automation playbooks.

The task involved:

- Configuring Ansible inventory
- Generating SSH key pairs
- Establishing trust relationships
- Enabling password-less authentication
- Validating Ansible connectivity
- Testing remote automation communication

The automation controller executed playbooks remotely using SSH-based orchestration.

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Automation Controller | control-node |
| Managed Server 1 | app-server-alpha |
| Managed Server 2 | app-server-beta |
| Managed Server 3 | app-server-gamma |
| Automation User | automation-user |
| Inventory Path | /home/automation-user/ansible/inventory |
| SSH Authentication | RSA Public Key |
| Automation Tool | Ansible |

---

# Objective

## Requirements

1. Use:
- control-node
- automation-user

as Ansible controller environment.

2. Fix inventory file:

```bash
/home/automation-user/ansible/inventory
```

3. Configure password-less SSH.

4. Validate Ansible ping to:
- app-server-gamma

using inventory file.

---

# Step 1 — Switch to Automation User

## Command

```bash
sudo su - automation-user
```

---

# Command Breakdown

| Component | Meaning |
|---|---|
| sudo | Execute privileged command |
| su | Switch user |
| - | Load login environment |
| automation-user | Target user |

---

# Why Needed

Ansible automation must run using:
- correct home directory
- correct SSH keys
- correct inventory ownership

---

# Step 2 — Navigate to Ansible Directory

## Command

```bash
cd /home/automation-user/ansible
```

---

# Purpose

Move into working directory containing:
- inventory
- playbooks
- automation assets

---

# Step 3 — Verify Inventory File

## Initial Command

```bash
cat inventory
```

---

# Initial Problem

The inventory file was malformed:

```ini
app-server-gamma ansible_ssh_pass=SecurePass3automation-user@control-node
```

Shell prompt accidentally became part of inventory content.

---

# Root Cause Analysis

Incorrect HEREDOC termination or accidental terminal append caused corruption.

---

# Step 4 — Fix Inventory File

## Correct Command

```bash
cat > /home/automation-user/ansible/inventory << 'EOF'
app-server-alpha ansible_user=deploy-alpha ansible_ssh_pass=SecurePass1 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-beta ansible_user=deploy-beta ansible_ssh_pass=SecurePass2 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-gamma ansible_user=deploy-gamma ansible_ssh_pass=SecurePass3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
EOF
```

---

# Inventory File Explanation

---

# Host Entry Example

```ini
app-server-alpha ansible_user=deploy-alpha ansible_ssh_pass=SecurePass1
```

---

# Parameter Breakdown

| Parameter | Meaning |
|---|---|
| app-server-alpha | Inventory hostname |
| ansible_user | SSH login username |
| ansible_ssh_pass | SSH password |
| ansible_ssh_common_args | Additional SSH options |

---

# SSH Option Explanation

```bash
-o StrictHostKeyChecking=no
```

Disables interactive SSH trust prompt.

Without this:

```text
Are you sure you want to continue connecting (yes/no)?
```

would interrupt automation.

---

# Step 5 — Verify Corrected Inventory

## Command

```bash
cat /home/automation-user/ansible/inventory
```

---

# Output

```ini
app-server-alpha ansible_user=deploy-alpha ansible_ssh_pass=SecurePass1 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-beta ansible_user=deploy-beta ansible_ssh_pass=SecurePass2 ansible_ssh_common_args='-o StrictHostKeyChecking=no'

app-server-gamma ansible_user=deploy-gamma ansible_ssh_pass=SecurePass3 ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

---

# Step 6 — Navigate to SSH Directory

## Command

```bash
cd ~/.ssh
```

---

# Why Needed

SSH keys are stored under:

```bash
~/.ssh
```

---

# Step 7 — Generate SSH Key Pair

## Command

```bash
ssh-keygen -t rsa
```

---

# Command Breakdown

| Component | Meaning |
|---|---|
| ssh-keygen | SSH key generation utility |
| -t rsa | Generate RSA key |

---

# Interactive Prompts

## Save Location

```text
Enter file in which to save the key:
```

Pressed Enter to use default:

```bash
~/.ssh/id_rsa
```

---

## Passphrase

Pressed Enter twice for empty passphrase.

---

# Generated Files

| File | Purpose |
|---|---|
| id_rsa | Private key |
| id_rsa.pub | Public key |

---

# Verify Generated Keys

## Command

```bash
ls -ltr ~/.ssh
```

---

# Output

```bash
-rw------- 1 automation-user automation-user 2602 id_rsa
-rw-r--r-- 1 automation-user automation-user  568 id_rsa.pub
```

---

# File Permission Explanation

| File | Permission | Meaning |
|---|---|
| id_rsa | 600 | Private key protected |
| id_rsa.pub | 644 | Public key readable |

---

# Step 8 — Copy Public Key to Managed Node

## Command

```bash
ssh-copy-id deploy-gamma@app-server-gamma
```

---

# Purpose

Copies public key into remote server:

```bash
~/.ssh/authorized_keys
```

---

# First-Time SSH Prompt

```text
Are you sure you want to continue connecting (yes/no)?
```

Entered:

```text
yes
```

---

# Password Prompt

```text
deploy-gamma@app-server-gamma's password:
```

Entered remote SSH password.

---

# Successful Output

```bash
Number of key(s) added: 1
```

---

# What Happened Internally

The public key:

```bash
id_rsa.pub
```

was appended to remote file:

```bash
~/.ssh/authorized_keys
```

---

# Step 9 — Validate Password-less SSH

## Command

```bash
ssh deploy-gamma@app-server-gamma
```

---

# Expected Behavior

- login succeeds
- no password prompt appears

---

# Why This Matters

Password-less SSH is required for:
- CI/CD pipelines
- automation frameworks
- non-interactive deployments

---

# Step 10 — Validate Ansible Connectivity

## Command

```bash
ansible -i inventory app-server-gamma -m ping
```

---

# Command Breakdown

| Component | Meaning |
|---|---|
| ansible | Ad-hoc Ansible command |
| -i inventory | Use custom inventory |
| app-server-gamma | Target host |
| -m ping | Execute ping module |

---

# Important Clarification

This is NOT ICMP ping.

Ansible ping:
- SSH connects
- executes Python remotely
- validates automation connectivity

---

# Successful Output

```json
app-server-gamma | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# Output Analysis

| Field | Meaning |
|---|---|
| SUCCESS | Remote execution successful |
| changed=false | No system modification |
| ping=pong | Connectivity validated |

---

# Final Technical Resolution

| Problem | Resolution |
|---|---|
| Malformed inventory | Recreated inventory correctly |
| No SSH keys present | Generated RSA key pair |
| Password-based SSH only | Configured password-less SSH |
| Automation validation required | Ansible ping successful |

---

# Important DevOps Concepts Demonstrated

- SSH Key Authentication
- Public Key Infrastructure
- Password-less Automation
- Ansible Connectivity Validation
- Linux Authentication
- Infrastructure Automation
- Remote Execution
- Configuration Management
- SSH Trust Relationships
- Secure Automation

---

# Enterprise Best Practices

# Prefer SSH Keys Over Passwords

Advantages:
- secure
- scalable
- automation friendly

---

# Use ED25519 Instead of RSA

Modern environments prefer:

```bash
ssh-keygen -t ed25519
```

---

# Disable Password Authentication

Production environments should disable:

```bash
PasswordAuthentication yes
```

inside:

```bash
/etc/ssh/sshd_config
```

---

# Protect Private Keys

Never expose:
- id_rsa
- private credentials

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is password-less SSH?

### Answer

Authentication using SSH keys instead of passwords.

---

## Q2. What does ssh-copy-id do?

### Answer

Copies public key to remote server's:

```bash
authorized_keys
```

file.

---

## Q3. Difference between public and private key?

### Answer

| Public Key | Private Key |
|---|---|
| Shared remotely | Kept secret |

---

## Q4. What is Ansible ping?

### Answer

Validates:
- SSH connectivity
- Python execution
- Ansible communication

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. Why use SSH keys instead of passwords?

### Answer

SSH keys:
- more secure
- automation friendly
- resistant to brute force

---

## Q2. What is authorized_keys?

### Answer

Stores trusted public keys allowed for SSH login.

---

## Q3. How would you troubleshoot SSH failures?

### Answer

Check:
- permissions
- SSH daemon
- firewall
- DNS
- authorized_keys
- inventory syntax

---

## Q4. Why are SSH key permissions important?

### Answer

SSH refuses insecure permissions for security reasons.

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you secure enterprise SSH infrastructure?

### Answer

Using:
- bastion hosts
- MFA
- certificate-based SSH
- Vault integration
- restricted sudo

---

## Q2. How would you manage SSH keys at scale?

### Answer

Using:
- centralized identity management
- Vault
- short-lived certificates
- rotation policies

---

## Q3. How would you integrate Ansible with CI/CD?

### Answer

Using:
- Jenkins
- GitHub Actions
- GitLab CI
- dynamic inventory

---

## Q4. How would you audit SSH access?

### Answer

Using:
- auditd
- SIEM
- centralized logging
- session recording

---

# L4 — DevOps Architect

---

## Q1. Design secure enterprise automation architecture.

### Answer

Components:
- Terraform
- Ansible
- Vault
- Kubernetes
- GitOps
- RBAC
- centralized observability

---

## Q2. How would you implement zero-trust SSH?

### Answer

Using:
- ephemeral credentials
- MFA
- short-lived certificates
- identity-aware proxies

---

## Q3. How would you automate hybrid-cloud infrastructure?

### Answer

Using:
- dynamic inventory
- cloud-native APIs
- Terraform
- centralized orchestration

---

## Q4. How would you enforce enterprise compliance?

### Answer

Using:
- policy-as-code
- RBAC
- immutable infrastructure
- audit logging

---

# AWS Scenario-Based Questions

---

# Q1. How would you avoid SSH password usage in AWS?

## Answer

Using:
- EC2 key pairs
- IAM roles
- SSM Session Manager

---

# Q2. How would you secure EC2 SSH access?

## Answer

Using:
- private subnets
- bastion hosts
- MFA
- restricted security groups

---

# Q3. How would you troubleshoot EC2 SSH failures?

## Answer

Check:
- Security Groups
- NACLs
- Route Tables
- SSH daemon
- key permissions

---

# Real Production Incident RCA

# Incident — Automation Failure Due to SSH Key Rotation

## Symptoms

- CI/CD deployment failed
- SSH authentication errors

---

## Root Cause

Expired SSH keys remained in automation inventory.

---

## Resolution

- Rotated keys
- Updated automation credentials
- Implemented centralized key management

---

## Prevention

- automated rotation
- monitoring
- Vault integration

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between chmod 600 and 644?

### Answer

| 600 | 644 |
|---|---|
| Owner only | Public readable |

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

---

# Docker

## Q4. Difference between CMD and ENTRYPOINT?

### Answer

| CMD | Default command |
| ENTRYPOINT | Main executable |

---

# Kubernetes

## Q5. Difference between ConfigMap and Secret?

### Answer

| ConfigMap | Secret |
|---|---|
| Plain config | Sensitive data |

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

- Secure SSH authentication
- Password-less infrastructure automation
- Centralized orchestration
- Ansible connectivity validation
- Linux security best practices
- Enterprise automation patterns
- Remote execution reliability
- Infrastructure scalability

---

# Final Outcome

✅ Inventory fixed successfully  
✅ SSH keys generated successfully  
✅ Public key deployed successfully  
✅ Password-less SSH configured  
✅ Ansible ping successful  
✅ App server reachable  
✅ Automation validation completed  
✅ No connectivity issues detected  
✅ Infrastructure ready for playbook execution

