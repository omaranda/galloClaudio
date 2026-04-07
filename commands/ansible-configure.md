---
name: ansible-configure
description: Generate Ansible playbooks and roles for server configuration, environment setup, and application deployment in AWS infrastructure.
---

You are acting as the devops-aws agent. Generate Ansible configuration.

**Step 1 — Gather requirements (ask if not provided):**
- What needs to be configured? (OS packages, Python environment, GDAL, Docker, app config, etc.)
- Target: EC2 instances, ECS worker nodes, or local development machines?
- OS: Ubuntu 22.04 / Amazon Linux 2023?
- Is this a new role or extending an existing playbook?
- Any secrets needed? (DB passwords, API keys — use Ansible Vault)

**Step 2 — Read existing Ansible code:**
Check for existing `roles/`, `inventory/`, `group_vars/`, and `ansible.cfg` before writing new code.

**Step 3 — Generate Ansible role structure:**

```
roles/<role-name>/
├── tasks/
│   └── main.yml        # Main task list
├── defaults/
│   └── main.yml        # Default variables (overridable)
├── vars/
│   └── main.yml        # Non-overridable variables
├── handlers/
│   └── main.yml        # Service restart handlers
├── templates/
│   └── app.conf.j2     # Jinja2 config templates
└── files/
    └── ...             # Static files to copy
```

**Step 4 — Idempotency requirements:**
Every task MUST be idempotent — running the playbook twice produces no changes:
- Use `apt` module with `state: present` (not `shell: apt-get install`)
- Use `copy`/`template` modules (not `shell: echo > file`)
- Use `systemd` module for service management
- Use `become: yes` only where root is needed

**Step 5 — Standard patterns:**

**GDAL/rasterio installation:**
```yaml
- name: Install GDAL system dependencies
  apt:
    name:
      - gdal-bin
      - libgdal-dev
      - python3-gdal
    state: present
    update_cache: yes

- name: Install Python GDAL bindings
  pip:
    name: "GDAL=={{ gdal_version }}"
    virtualenv: "{{ app_venv_path }}"
```

**Environment variables from AWS Secrets Manager:**
```yaml
- name: Fetch DB credentials from Secrets Manager
  community.aws.aws_secret:
    name: "{{ project_name }}/{{ environment }}/db"
    region: "{{ aws_region }}"
  register: db_secret
  no_log: true
```

**Step 6 — Provide:**
- All role files
- Example playbook that uses the role
- Vault-encrypted example for secrets
- Run command:
  ```bash
  ansible-playbook -i inventory/hosts.yml playbook.yml --tags configure --diff
  ```
