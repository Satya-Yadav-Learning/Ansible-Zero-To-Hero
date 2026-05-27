# TaskAndResolution.md

# Ansible ACL-Based File Permission Automation Across Multiple Linux Servers

---

# Project Overview

A DevOps engineering team needed to automate secure file creation and ACL-based permission management across multiple Linux application servers using Ansible.

The primary goal was:

- Create application-specific files
- Ensure files remain owned by root
- Grant selective permissions using Linux ACLs
- Perform all operations using centralized Ansible automation
- Validate infrastructure consistency across multiple servers

The automation was implemented using:
- Ansible inventory
- playbooks
- file module
- acl module
- conditional task execution

---

# Environment Details (Sanitized)

| Component | Value |
|---|---|
| Automation Controller | control-node |
| Application Server 1 | app-node-alpha |
| Application Server 2 | app-node-beta |
| Application Server 3 | app-node-gamma |
| Automation User | deploy-user |
| Inventory Path | /home/deploy-user/ansible/inventory |
| Playbook Path | /home/deploy-user/ansible/playbook.yml |
| Target Directory | /opt/data |
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

3. Create:

| Server | File |
|---|---|
| app-node-alpha | blog.txt |
| app-node-beta | story.txt |
| app-node-gamma | media.txt |

4. All files must:
- exist under `/opt/data`
- be owned by `root`
- use ACL permissions

5. ACL Requirements:

| File | ACL Type | Entity | Permission |
|---|---|---|---|
| blog.txt | group | devgroup-alpha | r |
| story.txt | user | deploy-beta | rw |
| media.txt | group | media-team | rw |

---

# Step 1 — Navigate to Ansible Directory

## Command

```bash
cd /home/deploy-user/ansible
```

---

# Why Needed

This directory contains:
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
app-node-alpha ansible_host=app-node-alpha ansible_ssh_pass=SecurePass1 ansible_user=deploy-alpha

app-node-beta ansible_host=app-node-beta ansible_ssh_pass=SecurePass2 ansible_user=deploy-beta

app-node-gamma ansible_host=app-node-gamma ansible_ssh_pass=SecurePass3 ansible_user=deploy-gamma
```

---

# Inventory Explanation

---

# Host Entry Example

```ini
app-node-alpha ansible_host=app-node-alpha
```

---

# Parameter Breakdown

| Parameter | Meaning |
|---|---|
| app-node-alpha | Inventory hostname |
| ansible_host | Actual target hostname/IP |
| ansible_user | SSH login user |
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

    - name: Create blog.txt on Application Server Alpha
      file:
        path: /opt/data/blog.txt
        state: touch
        owner: root
        group: root
      when: inventory_hostname == "app-node-alpha"

    - name: Set ACL on blog.txt for group devgroup-alpha
      acl:
        path: /opt/data/blog.txt
        entity: devgroup-alpha
        etype: group
        permissions: r
        state: present
      when: inventory_hostname == "app-node-alpha"

    - name: Create story.txt on Application Server Beta
      file:
        path: /opt/data/story.txt
        state: touch
        owner: root
        group: root
      when: inventory_hostname == "app-node-beta"

    - name: Set ACL on story.txt for user deploy-beta
      acl:
        path: /opt/data/story.txt
        entity: deploy-beta
        etype: user
        permissions: rw
        state: present
      when: inventory_hostname == "app-node-beta"

    - name: Create media.txt on Application Server Gamma
      file:
        path: /opt/data/media.txt
        state: touch
        owner: root
        group: root
      when: inventory_hostname == "app-node-gamma"

    - name: Set ACL on media.txt for group media-team
      acl:
        path: /opt/data/media.txt
        entity: media-team
        etype: group
        permissions: rw
        state: present
      when: inventory_hostname == "app-node-gamma"
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

Equivalent Linux command:

```bash
sudo
```

Required because:
- file creation
- ACL management
- ownership configuration

need administrative privileges.

---

# file Module

```yaml
file:
```

Used for:
- file creation
- ownership management
- permission management

---

# state: touch

```yaml
state: touch
```

Equivalent Linux command:

```bash
touch filename
```

Creates file if missing.

---

# owner/group

```yaml
owner: root
group: root
```

Ensures:
- root ownership
- secure file control

---

# acl Module

```yaml
acl:
```

Used to manage Linux Access Control Lists.

ACL provides:
- granular permissions
- user/group-specific access

beyond traditional Linux permissions.

---

# ACL Parameter Breakdown

| Parameter | Meaning |
|---|---|
| path | Target file |
| entity | User/group receiving access |
| etype | ACL type |
| permissions | Access level |
| state | ACL presence |

---

# etype Values

| Value | Meaning |
|---|---|
| user | ACL for user |
| group | ACL for group |

---

# Permission Values

| Value | Meaning |
|---|---|
| r | Read |
| rw | Read + Write |

---

# when Condition

```yaml
when: inventory_hostname == "app-node-alpha"
```

Ensures task runs only on matching host.

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

    - name: Create blog.txt on Application Server Alpha
      file:
        path: /opt/data/blog.txt
        state: touch
        owner: root
        group: root
      when: inventory_hostname == "app-node-alpha"

    - name: Set ACL on blog.txt for group devgroup-alpha
      acl:
        path: /opt/data/blog.txt
        entity: devgroup-alpha
        etype: group
        permissions: r
        state: present
      when: inventory_hostname == "app-node-alpha"

    - name: Create story.txt on Application Server Beta
      file:
        path: /opt/data/story.txt
        state: touch
        owner: root
        group: root
      when: inventory_hostname == "app-node-beta"

    - name: Set ACL on story.txt for user deploy-beta
      acl:
        path: /opt/data/story.txt
        entity: deploy-beta
        etype: user
        permissions: rw
        state: present
      when: inventory_hostname == "app-node-beta"

    - name: Create media.txt on Application Server Gamma
      file:
        path: /opt/data/media.txt
        state: touch
        owner: root
        group: root
      when: inventory_hostname == "app-node-gamma"

    - name: Set ACL on media.txt for group media-team
      acl:
        path: /opt/data/media.txt
        entity: media-team
        etype: group
        permissions: rw
        state: present
      when: inventory_hostname == "app-node-gamma"
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
app-node-alpha | SUCCESS => {
    "ping": "pong"
}

app-node-beta | SUCCESS => {
    "ping": "pong"
}

app-node-gamma | SUCCESS => {
    "ping": "pong"
}
```

---

# Important Clarification

Ansible ping:
- establishes SSH connection
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
ok: [app-node-alpha]
ok: [app-node-beta]
ok: [app-node-gamma]

TASK [Create blog.txt on Application Server Alpha] *****************************
changed: [app-node-alpha]

TASK [Set ACL on blog.txt for group devgroup-alpha] ****************************
changed: [app-node-alpha]

TASK [Create story.txt on Application Server Beta] *****************************
changed: [app-node-beta]

TASK [Set ACL on story.txt for user deploy-beta] *******************************
changed: [app-node-beta]

TASK [Create media.txt on Application Server Gamma] ****************************
changed: [app-node-gamma]

TASK [Set ACL on media.txt for group media-team] *******************************
changed: [app-node-gamma]

PLAY RECAP *********************************************************************
app-node-alpha : ok=3 changed=2 unreachable=0 failed=0 skipped=4
app-node-beta  : ok=3 changed=2 unreachable=0 failed=0 skipped=4
app-node-gamma : ok=3 changed=2 unreachable=0 failed=0 skipped=4
```

---

# Step 7 — Validate ACL Configuration

# Validate blog.txt

## Command

```bash
ansible -i inventory app-node-alpha -a "getfacl /opt/data/blog.txt"
```

---

# Output

```text
group:devgroup-alpha:r--
```

---

# Validate story.txt

## Command

```bash
ansible -i inventory app-node-beta -a "getfacl /opt/data/story.txt"
```

---

# Output

```text
user:deploy-beta:rw-
```

---

# Validate media.txt

## Command

```bash
ansible -i inventory app-node-gamma -a "getfacl /opt/data/media.txt"
```

---

# Output

```text
group:media-team:rw-
```

---

# ACL Explanation

ACL provides:
- fine-grained access control
- user/group-specific permissions

without modifying:
- primary owner
- primary group

---

# Important Technical Observation

Second playbook execution still showed:

```bash
changed=1
```

because:

```yaml
state: touch
```

updates timestamps every execution.

---

# Better Idempotent Alternative

Production environments often prefer:

```yaml
state: file
```

instead of:

```yaml
state: touch
```

because:
- preserves timestamps
- avoids unnecessary changes

However current implementation is correct for task requirements.

---

# Final Technical Resolution

| Requirement | Resolution |
|---|---|
| File creation required | file module used |
| ACL management required | acl module used |
| Root ownership needed | owner/group configured |
| Multi-host deployment needed | inventory used |
| Conditional execution required | when conditions used |

---

# Important DevOps Concepts Demonstrated

- Infrastructure as Code
- Linux ACL Management
- Multi-Host Automation
- SSH-Based Orchestration
- File Ownership Management
- Conditional Task Execution
- Configuration Management
- Linux Security Management
- Centralized Automation
- Enterprise Infrastructure Automation

---

# Enterprise Best Practices

# Use Ansible Vault

Never store plaintext credentials inside inventory.

---

# Use SSH Keys

Prefer SSH key authentication over passwords.

---

# Use Roles

Large playbooks should use:
- roles
- handlers
- variables
- reusable structures

---

# Prefer state=file

For fully idempotent file management.

---

# Scenario-Based Interview Questions & Answers

# L1 — Junior DevOps Engineer

---

## Q1. What is ACL in Linux?

### Answer

ACL provides additional permissions beyond standard:
- owner
- group
- others

model.

---

## Q2. What does the acl module do?

### Answer

Manages Linux Access Control Lists using Ansible.

---

## Q3. What does become: yes mean?

### Answer

Enables privilege escalation.

Equivalent to:

```bash
sudo
```

---

## Q4. Difference between user ACL and group ACL?

### Answer

| User ACL | Group ACL |
|---|---|
| Applies to user | Applies to group |

---

# L2 — Mid-Level DevOps Engineer

---

## Q1. Why use ACL instead of chmod?

### Answer

ACL allows:
- granular access control
- multiple users/groups

without changing ownership.

---

## Q2. Why use conditional execution?

### Answer

Ensures tasks execute only on intended hosts.

---

## Q3. Why did second playbook run still show changed=1?

### Answer

Because:
- `state: touch`
- updates file timestamp every run

---

## Q4. How would you validate ACLs?

### Answer

Using:

```bash
getfacl filename
```

---

# L3 — Senior DevOps Engineer

---

## Q1. How would you secure enterprise file access?

### Answer

Using:
- ACLs
- RBAC
- SELinux
- least privilege principles

---

## Q2. How would you scale Ansible automation?

### Answer

Using:
- AWX/Tower
- dynamic inventory
- reusable roles
- CI/CD integration

---

## Q3. How would you audit file permission changes?

### Answer

Using:
- auditd
- SIEM
- centralized logging

---

## Q4. How would you enforce compliance?

### Answer

Using:
- policy-as-code
- CIS benchmarks
- automated validation

---

# L4 — DevOps Architect

---

## Q1. Design enterprise access-control architecture.

### Answer

Components:
- centralized IAM
- RBAC
- Vault
- SELinux
- policy-as-code
- immutable infrastructure

---

## Q2. How would you secure hybrid-cloud automation?

### Answer

Using:
- centralized orchestration
- identity federation
- encrypted secrets

---

## Q3. How would you implement zero-trust infrastructure?

### Answer

Using:
- least privilege
- short-lived credentials
- continuous verification

---

## Q4. How would you automate enterprise compliance validation?

### Answer

Using:
- Ansible
- OpenSCAP
- audit automation
- SIEM integration

---

# AWS Scenario-Based Questions

---

# Q1. How would you secure shared file access in AWS?

## Answer

Using:
- EFS ACLs
- IAM
- POSIX permissions

---

## Q2. How would you automate EC2 file permissions?

### Answer

Using:
- Ansible
- SSM
- launch automation

---

## Q3. How would you audit file access in AWS?

### Answer

Using:
- CloudTrail
- CloudWatch
- GuardDuty
- SIEM integration

---

## Q4. How would you implement least privilege in AWS?

### Answer

Using:
- IAM policies
- SCPs
- role-based access

---

# Real Production Incident RCA

# Incident — Unauthorized Access to Shared Files

## Symptoms

- Sensitive application data modified unexpectedly
- Multiple users had excessive access

---

## Root Cause

Improper Linux permissions using broad chmod access.

---

## Resolution

Implemented:
- ACL-based access control
- least privilege permissions
- centralized automation

---

## Prevention

- automated permission audits
- policy enforcement
- compliance monitoring

---

# Full DevOps Mock Interview

# Linux

## Q1. Difference between chmod and setfacl?

### Answer

| chmod | setfacl |
|---|---|
| Basic permissions | Advanced ACL permissions |

---

# Networking

## Q2. Difference between SSH and SCP?

### Answer

| SSH | SCP |
|---|---|
| Remote login | Secure file transfer |

---

# AWS

## Q3. Difference between IAM Role and IAM Policy?

### Answer

| IAM Role | IAM Policy |
|---|---|
| Identity | Permissions document |

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

- ACL-based security management
- Fine-grained Linux permissions
- Centralized infrastructure automation
- Multi-host orchestration
- Secure file management
- Enterprise automation practices
- Conditional task execution
- Infrastructure consistency

---

# Final Outcome

✅ Files created successfully  
✅ Files owned by root successfully  
✅ ACL permissions configured correctly  
✅ User/group ACLs validated successfully  
✅ Ansible playbook executed successfully  
✅ Multi-host automation completed  
✅ Infrastructure consistency achieved  
✅ No unreachable hosts  
✅ No failures detected  
✅ Enterprise-grade automation implemented successfully

