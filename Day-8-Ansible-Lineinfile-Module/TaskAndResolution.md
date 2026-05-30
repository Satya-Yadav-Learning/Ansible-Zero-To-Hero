# TaskAndResolution.md

# Ansible HTTPD Web Server Deployment Using lineinfile Module

---

# Project Overview

A DevOps engineering team was tasked with automating Apache HTTPD installation and sample web page deployment across multiple Linux application servers using Ansible.

The objective was to:

- Install Apache HTTPD on all application servers.
- Start and enable the HTTPD service.
- Create a sample web page.
- Use the Ansible `lineinfile` module to insert a welcome message at the beginning of the web page.
- Configure ownership and permissions correctly.
- Validate service availability and web content.

---

# Environment Details (Sanitized)

| Component | Value |
|------------|---------|
| Automation Controller | Control-Node |
| Application Server A | WebNode-A |
| Application Server B | WebNode-B |
| Application Server C | WebNode-C |
| Automation User | AutomationUser |
| Inventory Location | /home/automation/ansible/inventory |
| Playbook Location | /home/automation/ansible/playbook.yml |
| Web Root | /var/www/html |
| Service | httpd |
| Automation Tool | Ansible |

---

# Business Requirement

Deploy Apache HTTPD on all application servers and host a sample web page with the following content:

```text
Welcome to xFusionCorp Industries!
This is a Nautilus sample file, created using Ansible!
```

Requirements:

- Install HTTPD.
- Start and enable service.
- Create `/var/www/html/index.html`.
- Use `lineinfile` module to add welcome text at the top.
- Set ownership to apache:apache.
- Set permissions to 0755.

---

# Solution Architecture

```text
                +----------------+
                |  Control Node  |
                |    Ansible     |
                +--------+-------+
                         |
          ---------------------------------
          |               |               |
          |               |               |
  +-------+-----+ +-------+-----+ +-------+-----+
  | WebNode-A   | | WebNode-B   | | WebNode-C   |
  | HTTPD       | | HTTPD       | | HTTPD       |
  +-------------+ +-------------+ +-------------+
```

---

# Inventory Verification

## Command

```bash
cd /home/automation/ansible

cat inventory
```

## Sample Inventory

```ini
WebNode-A ansible_user=userA ansible_ssh_pass=PasswordA
WebNode-B ansible_user=userB ansible_ssh_pass=PasswordB
WebNode-C ansible_user=userC ansible_ssh_pass=PasswordC
```

---

# Playbook Creation

## Command

```bash
cat > /home/automation/ansible/playbook.yml << 'EOF'
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

    - name: Create index.html with initial content
      copy:
        dest: /var/www/html/index.html
        content: |
          This is a Nautilus sample file, created using Ansible!
        owner: apache
        group: apache
        mode: '0755'

    - name: Add welcome line at top of file
      lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to xFusionCorp Industries!"
        insertbefore: BOF

    - name: Set ownership and permissions
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0755'
EOF
```

---

# Playbook Explanation

## hosts

```yaml
hosts: all
```

Targets every server in inventory.

---

## become

```yaml
become: yes
```

Allows privilege escalation.

Equivalent Linux command:

```bash
sudo
```

---

## yum Module

```yaml
yum:
  name: httpd
  state: present
```

Installs Apache HTTPD package.

---

## service Module

```yaml
service:
  name: httpd
  state: started
  enabled: yes
```

Ensures:

- Service is running.
- Service starts automatically after reboot.

---

## copy Module

```yaml
copy:
```

Creates initial HTML file.

---

## lineinfile Module

```yaml
lineinfile:
```

Used to insert a single line into a file.

---

### insertbefore: BOF

```yaml
insertbefore: BOF
```

BOF = Beginning Of File

This ensures:

```text
Welcome to xFusionCorp Industries!
```

appears as the first line.

---

## file Module

Used for:

- Ownership management
- Permission management

---

# Connectivity Validation

## Command

```bash
ansible -i inventory all -m ping
```

## Output

```json
WebNode-A | SUCCESS => {
  "ping": "pong"
}

WebNode-B | SUCCESS => {
  "ping": "pong"
}

WebNode-C | SUCCESS => {
  "ping": "pong"
}
```

---

# Playbook Execution

## Command

```bash
ansible-playbook -i inventory playbook.yml
```

## Output

```bash
TASK [Install httpd package]
changed

TASK [Start and enable httpd service]
changed

TASK [Create index.html with initial content]
changed

TASK [Add welcome line at top of file]
changed

TASK [Set ownership and permissions]
ok
```

---

# Service Validation

## Verify Package

```bash
ansible -i inventory all -a "rpm -q httpd"
```

Output:

```bash
httpd-2.4.62-13.el9.x86_64
```

---

## Verify Service Running

```bash
ansible -i inventory all -a "systemctl is-active httpd"
```

Output:

```bash
active
```

---

## Verify Service Enabled

```bash
ansible -i inventory all -a "systemctl is-enabled httpd"
```

Output:

```bash
enabled
```

---

# Web Content Validation

## Command

```bash
ansible -i inventory all -a "cat /var/www/html/index.html"
```

## Output

```text
Welcome to xFusionCorp Industries!
This is a Nautilus sample file, created using Ansible!
```

---

# Ownership Validation

## Command

```bash
ansible -i inventory all -a "stat -c '%U %G' /var/www/html/index.html"
```

## Output

```text
apache apache
```

---

# Permission Validation

## Command

```bash
ansible -i inventory all -a "stat -c '%a' /var/www/html/index.html"
```

## Output

```text
755
```

---

# HTTP Validation

## Command

```bash
ansible -i inventory WebNode-A -a "curl localhost"
```

## Output

```text
Welcome to xFusionCorp Industries!
This is a Nautilus sample file, created using Ansible!
```

---

# Idempotency Analysis

## Observation

Subsequent executions showed:

```bash
changed=2
```

### Why?

The `copy` module overwrites the file:

```yaml
copy:
  content: |
    This is a Nautilus sample file, created using Ansible!
```

Then:

```yaml
lineinfile:
```

adds the welcome line again.

Therefore two tasks change every execution.

---

# Better Production Design

Use:

```yaml
copy:
  content: |
    Welcome to xFusionCorp Industries!
    This is a Nautilus sample file, created using Ansible!
```

or

```yaml
template:
```

module.

This would achieve:

```bash
changed=0
```

on subsequent executions.

---

# DevOps Concepts Demonstrated

- Infrastructure as Code
- Configuration Management
- Service Management
- Multi-Server Automation
- Ansible Modules
- Linux File Permissions
- Linux Ownership
- HTTP Service Validation
- Idempotent Automation
- Agentless Architecture

---

# Scenario-Based Interview Questions & Answers

# L1 – Junior DevOps Engineer

---

## Q1. What is Ansible?

### Answer

Ansible is an open-source automation tool used for:

- Configuration Management
- Application Deployment
- Server Provisioning
- Orchestration

---

## Q2. What is lineinfile module?

### Answer

Used to:

- Add
- Modify
- Remove

a single line inside a file.

---

## Q3. What does BOF mean?

### Answer

Beginning Of File.

Used to insert content at the top.

---

## Q4. Difference between copy and lineinfile?

### Answer

| copy | lineinfile |
|--------|------------|
| Creates/Replaces file | Modifies single line |

---

# L2 – Mid-Level DevOps Engineer

---

## Q1. Why use lineinfile instead of shell commands?

### Answer

Because it is:

- Idempotent
- Safe
- Repeatable
- Structured

---

## Q2. Why use become: yes?

### Answer

Administrative privileges are required for:

- Package installation
- Service management
- System file modifications

---

## Q3. What is idempotency?

### Answer

Running automation multiple times results in the same final state.

---

## Q4. How would you validate service status?

### Answer

```bash
systemctl is-active httpd
```

---

# L3 – Senior DevOps Engineer

---

## Q1. How would you improve this playbook?

### Answer

Use:

- Roles
- Templates
- Variables
- Handlers

---

## Q2. How would you secure credentials?

### Answer

Use:

- Ansible Vault
- Secrets Manager
- SSH Keys

---

## Q3. Why use templates over copy?

### Answer

Templates support dynamic values and environment-specific configuration.

---

## Q4. How would you automate validation?

### Answer

Using:

- Molecule
- CI/CD pipelines
- Ansible lint

---

# L4 – DevOps Architect

---

## Q1. Design enterprise configuration management.

### Answer

Components:

- Git
- Ansible
- Vault
- Jenkins
- Monitoring Stack

---

## Q2. How would you manage 10,000 servers?

### Answer

Using:

- Dynamic Inventory
- AWX/Tower
- RBAC
- CI/CD

---

## Q3. How would you prevent configuration drift?

### Answer

Use:

- Scheduled playbook execution
- Compliance scanning
- Immutable infrastructure

---

## Q4. How would you secure automation?

### Answer

Implement:

- RBAC
- MFA
- Vault
- Audit Logging

---

# AWS Scenario-Based Questions

---

## Q1. How would you deploy Apache on EC2 instances?

### Answer

Using:

- Ansible
- EC2
- Auto Scaling
- Load Balancer

---

## Q2. How would you manage secrets?

### Answer

Use:

- AWS Secrets Manager
- Parameter Store

---

## Q3. How would you verify service health?

### Answer

Use:

- CloudWatch
- ALB Health Checks

---

## Q4. How would you deploy across multiple regions?

### Answer

Use:

- Route53
- Multi-Region Infrastructure
- Automation Pipelines

---

# Real AWS Incident RCA

## Incident

Web application unavailable after maintenance reboot.

### Symptoms

- HTTP 503
- Website inaccessible

### Root Cause

Service was started manually but not enabled.

### Resolution

Implemented:

```yaml
enabled: yes
```

inside automation playbook.

### Prevention

- Service validation
- Monitoring alerts
- Automated compliance checks

---

# Full DevOps Mock Interview

## Linux

Q: Difference between chmod 755 and 775?

Answer:

| 755 | 775 |
|-------|-------|
| Others Read/Execute | Others No Access |

---

## Networking

Q: Difference between HTTP and HTTPS?

Answer:

HTTPS provides encryption using TLS.

---

## AWS

Q: Difference between Security Group and NACL?

Answer:

| Security Group | NACL |
|---------------|------|
| Stateful | Stateless |

---

## Docker

Q: CMD vs ENTRYPOINT?

Answer:

ENTRYPOINT defines executable.

CMD provides default arguments.

---

## Kubernetes

Q: Deployment vs StatefulSet?

Answer:

Deployment → Stateless.

StatefulSet → Stateful workloads.

---

## Terraform

Q: What is Terraform State?

Answer:

Tracks infrastructure resources managed by Terraform.

---

## Jenkins

Q: What is a Jenkins Pipeline?

Answer:

Automates:

- Build
- Test
- Deploy

---

## Monitoring

Q: Monitoring vs Observability?

Answer:

Monitoring detects problems.

Observability explains why they occurred.

---

# Final Outcome

✅ HTTPD installed successfully

✅ HTTPD service running

✅ HTTPD service enabled

✅ Sample web page deployed

✅ lineinfile module used successfully

✅ Ownership configured

✅ Permissions configured

✅ HTTP validation successful

✅ Infrastructure automated successfully

✅ Multi-server deployment completed

✅ No failures encountered

